## HillStone 云界对接

### 上传镜像
```
glance image-create --name HillStone --disk-format qcow2 --container-format bare
--file SG6000-CloudEdge-VM02-5.5R1P7.1-kvm.qcow2
```
### 逻辑拓扑方案1

如图：

![topology][1]

说明：
图中的 HillStone 表示山石网科的安全网关设备
HillStone 有三个网卡。带内管理网（虚线），通过绑定公网IP，从而可以从外部访问；带外数据网（实线1），
也和外网相连，通过公网IP的形式和外网打通，再在 HillStone 内部做第二次 NAT，NAT 成 Nework1 里的虚拟机IP地址；
内联数据网（实线2），Network1 中的虚拟机的默认网关指向这个口
### 创建HillStone虚拟机
```
nova boot --flavor [flavor-id] --image [Hillstone-image-id]
--nic net-id=[net-id] HillStone-VM-01
```
### 开启 Web 服务
对于创建完成后的山石虚拟机，首先给管理口绑定一个公网IP，然后进入虚拟机内部，开启HTTP服务
命令如下：
```
SG-6000# con
SG-6000# configure
SG-6000(config)# show int
SG-6000(config)# show interface
H:physical state;A:admin state;L:link state;P:protocol state;U:up;D:down;K:ha keep up
==============================================================================================
Interface name      IP address/mask     Zone name       H A L P MAC address     Description
----------------------------------------------------------------------------------------------
ethernet0/0         192.168.10.6/24     trust           U U U U fa16.3e9e.9cb4  ------
vswitchif1          0.0.0.0/0           NULL            D U D D 001c.2ca1.9d12  ------
==============================================================================================
SG-6000(config)# int
SG-6000(config)# interface e
SG-6000(config)# interface ethernet0/0
SG-6000(config-if-eth0/0)# man
SG-6000(config-if-eth0/0)# manage http
```
开启HTTP服务后，可以通过WEB页面访问HillStone系统。

![web][2]

### 将 HillStone 虚拟机当做一个 network 的网关
指定IP地址创建Port
```
neutron port-create [net-id] --fixed-ip ip_address=10.10.10.1
```
### Attach Port 到 HillStone 虚拟机
```
nova interface-attach --port-id=[port-id] [vm-id]
```
### 重启HillStone虚拟机
```
SG-6000# reboot
```
### 重启完成后看到两块网卡
```
SG-6000# show interface
H:physical state;A:admin state;L:link state;P:protocol state;U:up;D:down;K:ha keep up
============================================================================================
Interface name      IP address/mask     Zone name       H A L P MAC address     Description
--------------------------------------------------------------------------------------------
ethernet0/0         192.168.10.6/24     trust           U U U U fa16.3e9e.9cb4  ------
ethernet0/1         0.0.0.0/0           NULL            U U U U fa16.3e2c.614e  ------
vswitchif1          0.0.0.0/0           NULL            D U D D 001c.cc15.d012  ------
============================================================================================
SG-6000#
```
### 手动配置网卡地址并完成相应配置
```
SG-6000# configure
SG-6000(config)# interface ethernet0/1
SG-6000(config-if-eth0/1)# zone trust
SG-6000(config-if-eth0/1)# ip address 10.10.10.1/24
SG-6000(config-if-eth0/1)# manage ping
```
### 创建虚拟机验证网关是否正常通信

![conn][4]

### 逻辑拓扑方案2
上面的方案1中，有个缺点，对于 HillStone 所连的 network1 里的虚拟机，如果想通过公网 IP 访问公网，
比较复杂，需要两次NAT，所以建议如果只用其 VPN 功能的话可以采用下面这种方案。

![topology][3]

在 VM 要和 IPSec 隧道和其他网络通信时，流量通过上图中的红线进行。具体的做法是：
如果虚拟机访问 10.10.10.0/24 网段时，通过 192.168.10.6（HillStone 虚拟机 IP 地址）更新 Subnet Host Routes，并验证
```
[root@server-39.0.bwkj.ustack.in ~ ]$ neutron subnet-update dbf1f6a0-3d0a-46bb-966b-ad5f4e8c5c35 \
--host_routes type=dict list=true destination=10.10.10.0/24,nexthop=192.168.10.6
Updated subnet: dbf1f6a0-3d0a-46bb-966b-ad5f4e8c5c35
[root@server-39.0.bwkj.ustack.in ~ ]$ neutron subnet-show dbf1f6a0-3d0a-46bb-966b-ad5f4e8c5c35
+----------------------+-------------------------------------------------------------+
| Field                | Value                                                       |
+----------------------+-------------------------------------------------------------+
| allocation_pools     | {"start": "192.168.10.2", "end": "192.168.10.254"}          |
| cidr                 | 192.168.10.0/24                                             |
| created_at           | 2016-04-22T02:47:26.000000                                  |
| dns_nameservers      | 219.141.136.10                                              |
|                      | 219.141.140.10                                              |
| enable_dhcp          | True                                                        |
| gateway_ip           | 192.168.10.1                                                |
| host_routes          | {"destination": "10.10.10.0/24", "nexthop": "192.168.10.6"} |
| id                   | dbf1f6a0-3d0a-46bb-966b-ad5f4e8c5c35                        |
| ip_version           | 4                                                           |
| ipv6_address_mode    |                                                             |
| ipv6_ra_mode         |                                                             |
| name                 | subnet-dbf1f6a0                                             |
| network_id           | 5eb77011-a4cc-4829-9a38-d18bfa94a0e7                        |
| shared               | False                                                       |
| tenant_id            | a98ece1cfac745fa8672075f47c0c28d                            |
| uos:service_provider |                                                             |
+----------------------+-------------------------------------------------------------+
```
虚拟机中重新DHCP一次，查看推送来的路由：

![subnet][5]

### 使用HillStone的IPSec功能
#### 申请License
查看系统的 SN 号，并通知 HillStone 官方
```
SG-6000(config-if-eth0/1)# show version
Hillstone StoneOS software, Version 5.5
Copyright (c) 2006-2015 by Hillstone Networks, Inc.
Product name: SG-6000-VM01 S/N: 0010193294545324 Assembly number: 0000
Boot file is SG6000-CloudEdge-5.5R1P7.1-kvm
Storage UUID is 3c83205e-4ce3-4fde-a443-ecd35b641b09
Built by buildmaster8 2016/04/14 10:45:44
Uptime is 0 day 0 hour 29 minutes 48 seconds
System language is "en"
VRouter feature: disabled
APP feature: enabled
APP magic: 66334a351afd05838baf59455e468cabf5be
```
#### 安装License并重启
```
SG-6000# exec license install xxxx
SG-6000# reboot
```

### 创建 IPSec VPN 

1. 新建 VPN 对端列表

  ![ipsec1][6]

2. vpn对端配置（基本）

  ![ipsec2][7]
  
   - 接口：填写防火墙的外网口
   - 对端 IP 地址：填写对端公网地址
   - 本地 ID ：由于防火墙是以虚拟机形式存在，因此外网口地址一般为内网地址，所有跟硬件防火墙建立 VPN 时要启用 NAT-T 特性。
     本地 ID 不能填无，可以选填域名或 IP 等，一般我们选择 IP 。本地 ID 填写本地防火墙的外网口地址（内网地址）。否则 VPN 不能建立。
   - 对端 ID ：同本地 ID。但由于对端地址没有 NAT，因此对端 ID 是公网地址。
   - 提议：提议至少填一个，一般也是填一个。
   - 预共享密钥：必填，需与对端的密钥一致。

3. VPN 对端配置（高级）

  ![ipsec3][8]

  NAT 穿越：即 NAT-T，由于本地防火墙外网口地址被转换，因此要启用 NAT-T
  接受对端任意 ID：可选可不选。


4. IKE VPN 配置

  ![ipsec4][9]
  
   - 对端选项：下拉箭头选择
   - 模式：一般用 tunnel 模式
   - P2 提议：必填，需与对端一致。
   - 本地 IP/掩码：本地需要与对端建立 VPN 的网段，写本地的网段。
   - 远程 IP/掩码：本地需要与对端建立 VPN 的网段，写对端的网段。

5. 创建 tunnel 接口

  ![ipsec5][10]
  
  ![ipsec6][11]
  
  建立 tunnel 接口，指向对端的路由要扔到 tunnel 接口。

6. 创建策略

  策略ID2：此处 tunnel1 放到了 trust 区域，tunnel1 要访问内部网络要开放 trust 到 trust 的访问权限，
  此处以大段形式开放，有明确需求者慎用。

7. 创建路由

  ![ipsec7][13]
  
  添加指向对端内网地址的路由，下一跳为 tunnel1 口。

8. IPSec 的稳定性
经过测试，在虚拟机里通过 IPSec 隧道和另一端的虚拟机进行通信，共发了 893785 个 ping 包（大概是 4 天的时间），丢包率 0.04%

### 虚拟机通过 SNAT 上公网
如图，虚拟机 192.168.10.9 可以像下图这样访问公网

![ipsec2][3]

思路：
为了配置方便，给 HillStone 虚拟机的管理网网卡配置公网 IP ，在 WEB 页面操作，在绑定公网 IP 之前，最好修改 hillstone 用户的密码，具体修改方法如下：
```
SG-6000(config)# admin user hillstone
SG-6000(config-admin)# password xxxx
```
修改后，申请公网IP，并绑定到该网卡上。
开启Hillstone的HTTP服务：
```
SG-6000(config-if-eth0/0)# manage http
```

在 VFW 里配置 SNAT 规则，如图

![ipsec9][14]

配置完成后，虚拟机即可和外网通信。

通过DNAT访问虚拟机
我们需要创建一个NAT的对应关系，：
```
[root@server-39.0.bwkj.ustack.in ~ ]$ neutron floatingip-create 45fd7b72-6ba7-411f-b585-3b19bd9d5ba1
Created a new floatingip:
+----------------------+--------------------------------------+
| Field                | Value                                |
+----------------------+--------------------------------------+
| created_at           | 2016-05-03T07:50:18.456267           |
| fixed_ip_address     |                                      |
| floating_ip_address  | 116.70.3.176                         |
| floating_network_id  | 45fd7b72-6ba7-411f-b585-3b19bd9d5ba1 |
| floating_subnet_id   | 1404aee0-4ca5-4a8c-a4fd-dceafc071fa3 |
| floatingipset_id     |                                      |
| id                   | 14251ccc-96b6-4f4a-8980-85236ebe79bc |
| port_id              |                                      |
| rate_limit           | 1024                                 |
| router_id            |                                      |
| status               | DOWN                                 |
| tenant_id            | a98ece1cfac745fa8672075f47c0c28d     |
| uos:name             | fip-14251ccc                         |
| uos:registerno       |                                      |
| uos:service_provider |                                      |
+----------------------+--------------------------------------+
[root@server-39.0.bwkj.ustack.in ~ ]$ neutron port-create 5eb77011-a4cc-4829-9a38-d18bfa94a0e7
Created a new port:
+-------------------------------+--------------------------------------------------------------------------------------+
| Field                         | Value                                                                                |
+-------------------------------+--------------------------------------------------------------------------------------+
| admin_state_up                | True                                                                                 |
| allowed_address_pairs         |                                                                                      |
| binding:disable_anti_spoofing | False                                                                                |
| binding:host_id               |                                                                                      |
| binding:profile               | {"uos_pps_limits": ["tcp:syn::5000", "udp::53:5000"]}                                |
| binding:vif_details           | {}                                                                                   |
| binding:vif_type              | unbound                                                                              |
| binding:vnic_type             | normal                                                                               |
| device_id                     |                                                                                      |
| device_owner                  |                                                                                      |
| fixed_ips                     | {"subnet_id": "dbf1f6a0-3d0a-46bb-966b-ad5f4e8c5c35", "ip_address": "192.168.10.10"} |
| id                            | a58d49c1-d0ac-4561-8f72-6de6d05ba01a                                                 |
| mac_address                   | fa:16:3e:22:4d:62                                                                    |
| name                          | nic-a58d49c1                                                                         |
| network_id                    | 5eb77011-a4cc-4829-9a38-d18bfa94a0e7                                                 |
| security_groups               | 3ae497a9-8e00-4f92-b0fc-8ab031c15669                                                 |
| status                        | DOWN                                                                                 |
| tenant_id                     | a98ece1cfac745fa8672075f47c0c28d                                                     |
+-------------------------------+--------------------------------------------------------------------------------------+
[root@server-39.0.bwkj.ustack.in ~ ]$ neutron floatingip-associate 14251ccc-96b6-4f4a-8980-85236ebe79bc a58d49c1-d0ac-4561-8f72-6de6d05ba01a
```

在 VFW 里创建 DNAT 规则：

![ipsec10][15]

配置完成后，即可通过 FloatingIP 访问虚拟机。

[1]: ../../images/ecosystem/hillstone.png
[2]: ../../images/ecosystem/web.png
[3]: ../../images/ecosystem/hillstone2.png
[4]: ../../images/ecosystem/conn.png
[5]: ../../images/ecosystem/sub.png
[6]: ../../images/ecosystem/ipsec1.png
[7]: ../../images/ecosystem/ipsec2.png
[8]: ../../images/ecosystem/ipsec3.png
[9]: ../../images/ecosystem/ipsec4.png
[10]: ../../images/ecosystem/ipsec5.png
[11]: ../../images/ecosystem/ipesc6.png
[12]: ../../images/ecosystem/ipsec7.png
[13]: ../../images/ecosystem/ipsec8.png
[14]: ../../images/ecosystem/ipsec9.png
[15]: ../../images/ecosystem/ipsec10.png
