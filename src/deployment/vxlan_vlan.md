## VXLAN + VLAN 自服务模型

---

### 简介

VXLAN + VLAN 自服务模型是 UnitedStack<sup>®</sup> UNP 默认推荐的部署模型，属于
 Vendor-Free 方案之一。该方案完全是由 UnitedStack<sup>®</sup> 借助开源项目例如
OpenvSwitch、Linux 等基于标准硬件或开放标准的硬件实现的架构，目前主要是OpenvSwitch
 Drvier 实现的纯软件方案。本方案特点是完全开源、成本低、供应商选择丰富等。

#### 设备角色及推荐配置

|设备类型|用途|推荐配置|数量|备注|
|:-:|:-:|:-:|:-:|:-:|
|服务器|计算节点|2路Intel E5 8核 2.4Ghz 以上处理器，支持超线程技术，内存最低 256G|3|3台起配|
|服务器|网络节点|2路Intel E5 8核 2.4Ghz 以上处理器，支持超线程技术，内存最低 128G|3||
|服务器|控制节点|2路Intel E5 8核 2.4Ghz 以上处理器，支持超线程技术，内存最低 128G|3||
|接入层千兆设备|接入交换设备|48口千兆，支持10G上联|1|根据业务需求增加设备数量|
|接入层万兆设备|接入交换设备|48口万兆，支持PBR，40G上联|1|根据业务需求增加设备数量|
|VPN 设备|远程接入|支持 IPSec|1|参考[UnitedStack 远程支持接入](remote_support.md)|
|线缆|设备连接线缆|服务器连接线缆：DAC/SFP+/RJ45(根据设备接口而定) 交换机堆叠线缆：DAC/SFP+/QSFP+(根据设备接口而定)|若干|||

系统 BIOS 的配置参考本书系统一节。

#### 网络适配器相关信息

##### 推荐的网络适配器型号

|类别|品牌|型号|产品链接|备注|
|:-:|:-:|:-:|:-:|:-:|
|万兆网络适配器|Intel|Intel Ethernet Converged Network Adapter X710-DA2 & X710-DA4FH|[链接](http://www.intel.com/content/www/us/en/ethernet-products/converged-network-adapters/ethernet-xl710-brief.html)|- 支持网络虚拟化 Offload - 支持 DPDK - 出色的小包性能（用于NFV场景）- 优化 NAS (SMB, NFS) and SAN (iSCSI)|
|万兆网络适配器|Intel|Intel Ethernet Converged Network Adapter X550T2|[链接](http://www.intel.com/content/www/us/en/ethernet-products/converged-network-adapters/ethernet-x550-brief.html)|- Low cost, low power, 10 GbE performance for the entire datacenter. - Standard CAT 6a cabling with RJ45 connectors. - Supports NBASE-T technology - PCI Express\* (PCIe\*) v 3.0 with up to 8.0 GT/s|
|万兆网络适配器|Mellanox|ConnectX®-4 Lx EN Cards|[链接](http://www.mellanox.com/page/products_dyn?product_family=219&mtag=connectx_4_lx_en_card)| - Highest performing boards for applications requiring high bandwidth, low latency and high message rate - 1/10/25/40/50GbE connectivity for servers and storage - Virtualization acceleration|

注：由于 X550 使用 RJ45 接口，因此需要注意交换机需与之配套。

##### 网卡名称与使用参考

|设备角色|千兆网络适配器|万兆网络适配器|Bonding|
|:-:|:-:|:-:|:-:|
|计算节点|eth0|eth2|eth0 + eth1 => Bond0|
|计算节点|eth1|eth3|eth2 + eth3 => Bond1|
|网络节点|eth0|eth2|eth0 + eth1 => Bond0|
|网络节点|eth1|eth3|eth2 + eth3 => Bond1|

##### 网络适配器兼容列表

|类别|品牌|型号|是否兼容|
|:-:|:-:|:-:|:-:|
|万兆网络适配器|Intel|Intel Corporation Ethernet 10G 2P X520 Adapter|兼容|
|万兆网络适配器|Intel|Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection|兼容|
|万兆网络适配器|Intel|Intel Corporation Ethernet Controller 10-Gigabit X540-AT2 (rev 01)|兼容|
|万兆网络适配器|Intel|Intel(R) 10G 2P X540-t Adapter|兼容|
|万兆网络适配器|Mellanox|ConnectX®-3 Pro EN Dual-Port 10GBASE-T |兼容|
|千兆网络适配器|Intel|Intel Corporation I350 Gigabit Network Connection|兼容|
|千兆网络适配器|Broadcom|Broadcom Corporation NetXtreme BCM5720 Gigabit Ethernet PCIe|兼容|

##### VLAN 规划示例

||VLAN 1100|VLAN 1124|VLAN 1002|VLAN 1116|备注|
|:-:|:-:|:-:|:-:|:-:|:-:|
|计算节点|管理网 + Ceph Mon|SDN|外部网络（FloatingIP）|存储|非 DVR 场景（L3 HA）下无需配置外部网络 VLAN 1002|
|网络节点|管理网 + API|SDN|外部网络（SNAT）|-|DVR 场景和 非 DVR 场景（L3 HA）配置相同|

```
说明：
1. 两个千兆网口 eth0 和 eth1 做Bond0，Bond0 在千兆接入上配置 Access 1100；
2. 两个万兆网口 eth2 和 eth3 做Bond1，Bond1 在万兆接入上配置 Trunk 1124 1002 1116，
其中，VLAN 1124 1116 由 Linux VLAN 子接口实现，VLAN 1002 在 Neutron 中定义
VLAN 为 1002 的外部网络。
```

##### 验证各个 VLAN 配置的正确性

对于 VLAN 1100 1124 1002 1116 可以分为两类，一类是在网络适配器接入交换机上做 Access VLAN，比如 VLAN 1100；
另一类是通过计算节点 / 网络节点上通过 OpenFlow 流表的形式将流量打上相应 VLAN tag，比如 VLAN 1124 1002 1116。

对于验证第一类 VLAN 配置的正确性，可以直接通过在相应网络适配器上配置相应 IP 地址，验证是否通信
正常即可。对于第二类，可以通过 Linux 子接口形式验证相关 VLAN 配置的正确性。

接下来以 VLAN 1002 为例，验证上述第二类 VLAN 配置的正确性。

首先，需要说明的是 VLAN 1002 作为 Neutron 的外部网络，并且创建该外部网络时，需指定网络
的 Gateway IP，本例中的外部网络网段是 2.2.0.0/16，网关地址是 2.2.0.1
其次，通过在 Linux 机器上执行`lsmod | grep 8021q`确定加载了相应内核模块。最后，通过 `ip` 命令
创建 VLAN 子接口并验证连通性：

```
 ip link add link eth1 name eth1.1002 type vlan id 1002
```

或者通过 `vconfig` 命令创建 VLAN 子接口

```
vconfig add eth1 1002 
```
最后通过在相应的 VLAN 子接口上配置相应的 IP 地址来检查联通性是否正常。

```
ifconfig eth1.1002 2.2.2.10/24
```

检查连通性：
```
[root@server-233 ~(keystone_admin)]# ping 2.2.0.1
PING 2.2.2.10 (2.2.2.10) 56(84) bytes of data.
64 bytes from 2.2.2.10: icmp_seq=1 ttl=64 time=0.969 ms
^C
--- 2.2.2.10 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
```

Ping 正常则表示 VLAN 1002 配置正确。

在测试完毕后，一定要删除上述创建的 VLAN 接口：

```
ip link delete eth1.1002
```

##### 配置 Neutron 相关参数

本节中，讲解 VXLAN + VLAN 自服务模型下 Neutron 的详细配置。
`注：本节中使用到的 VLAN 范围是 10 ~ 20`

首先，Neutron 默认创建 VXLAN 网络。对 plugins/ml2/ml2_conf.ini 文件（以下若无特殊说明，文件的根目录均已 /etc/neutron 开头）进行如下修改：

```
tenant_network_types = vxlan,vlan
```

其次，指定 VLAN 池化范围。对 plugins/ml2/ml2_conf.ini 进行如下修改：

```
[ml2_type_vlan]
# (ListOpt) List of <physical_network>[:<vlan_min>:<vlan_max>] tuples
# specifying physical_network names usable for VLAN provider and
# tenant networks, as well as ranges of VLAN tags on each
# physical_network available for allocation as tenant networks.
#
# network_vlan_ranges =
network_vlan_ranges = physnet3:10:20
```

`注：上述配置支持定义多种类型的 physical_network，和多段连续或不连续的 VLAN 范围。比如：physnet3:10:20, physnet3:15:30 或者 physnet3:10:20, physnet3:25:30 等。`

其次，指定创建的外部网络为 VLAN 网络。对 plugins/ml2/ml2_conf.ini 进行如下修改：
```
external_network_type = vlan
```

最后，在计算节点和网络节点上配置网桥对应关系。对 plugins/ml2/openvswitch_agent.ini 进行如下修改：
```
bridge_mappings = physnet3:ovsbr3
```

有关 VXLAN / VLAN 架构细节可参看本书的[VXLAN](../architecture/vxlan.md)一节和 [VLAN](../architecture/vlan.md)一节。

有关 Neutron 的详细配置文件可参看本书的[Neutron 中的配置文件](neutron_conf.md)一节。
