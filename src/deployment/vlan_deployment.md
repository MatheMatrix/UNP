## VLAN 预配置网络部署

---

### 简介

 在[架构- VLAN](../architecture/vlan.md)一节中已经详细介绍了UnitedStack<sup>®</sup> United Networking Platform(UNP) 中
VLAN 的相关背景和使用场景。本节详细介绍 VLAN 网络的部署步骤。

#### VLAN 预配置网络部署步骤

`注：本节中我们配置 VLAN 的范围是5 ~ 6 。`

#### 交换机配置相应 VLAN 并验证

 Linux 支持通过创建 VLAN 子接口的形式来支持发送或接收相应 VLAN 的数据包。
首先，通过在 Linux 机器上执行`lsmod | grep 8021q`确定加载了相应内核模块。其次，通过 `ip` 命令
创建 VLAN 子接口

```
ip link add link p3p1 name p3p1.5 type vlan id 5
ip link add link p3p1 name p3p1.6 type vlan id 6
```

或者通过 `vconfig` 命令创建 VLAN 子接口

```
vconfig add p3p1 5
vconfig add p3p1 6
```
最后通过在相应的 VLAN 子接口上配置相应的 IP 地址来检查联通性是否正常。

```
ifconfig p3p1.5 192.168.5.10/24
ifconfig p3p1.6 192.168.6.10/24
```

检查连通性：
```
[root@server-233 ~(keystone_admin)]# ping 192.168.6.1
PING 192.168.6.1 (192.168.6.1) 56(84) bytes of data.
64 bytes from 192.168.6.1: icmp_seq=1 ttl=64 time=0.969 ms
^C
--- 192.168.6.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.969/0.969/0.969/0.000 ms
[root@server-233 ~(keystone_admin)]# ping 192.168.5.1
PING 192.168.5.1 (192.168.5.1) 56(84) bytes of data.
64 bytes from 192.168.5.1: icmp_seq=1 ttl=64 time=1.05 ms
^C
--- 192.168.5.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.056/1.056/1.056/0.000 ms
```
联通性测试正常，删除上述创建的 VLAN 接口。

```
ip link delete p3p1.5
ip link delete p3p1.6
```

#### Neutron 配置

 在 Neutron Server 节点上对配置文件 `plugins/ml2/ml2_conf.ini`，进行如下修改：

```
[ml2_type_vlan]
network_vlan_ranges = physnet3:5:5,physnet4:6:6
```

`说明：
network_vlan_ranges 的配置格式是：<physical_network>[:<vlan_min>:<vlan_max>]
上述配置说明在 Neutron 中定义两个 VLAN 网络，VLAN ID 分别是 5 和 6 。
也可以通过 network_vlan_ranges = physnet3:10:20 的形式定义 VLAN 池。`

 在计算节点 / 网络节点上对配置文件 `plugins/ml2/openvswitch_agent.ini`，进行如下修改：

```
bridge_mappings = physnet3:ovsbr3,physnet4:ovsbr4
```

手动创建 OpenvSwitch 网桥：

```
ovs-vsctl add-br ovsbr3
ovs-vsctl add-br ovsbr4
```

#### 创建 VLAN 网络

 - 指定 VLAN ID 创建 VLAN 网络

```

[root@server-233 ~(keystone_admin)]# neutron net-create \
--provider:network_type=vlan \
--provider:physical_network=physnet3 \
--provider:segmentation_id=6 vlan_network
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | 10d0e00f-2e84-4f00-a74e-fb7783ca55c8 |
| mtu                       | 0                                    |
| name                      | vlan_network                         |
| provider:network_type     | vlan                                 |
| provider:physical_network | physnet3                             |
| provider:segmentation_id  | 6                                    |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | ef979882f1954a0fa4ce7daf244aa557     |
+---------------------------+--------------------------------------+
```

 - 从 VLAN 池中创建 VLAN 网络

```
[root@server-233 ~(keystone_admin)]# neutron net-create \
--provider:network_type=vlan vlan_net
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | cc6d7263-3b1a-419e-8bdf-266ab406bf5e |
| mtu                       | 0                                    |
| name                      | vlan_net                             |
| provider:network_type     | vlan                                 |
| provider:physical_network | physnet3                             |
| provider:segmentation_id  | 5                                    |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | ef979882f1954a0fa4ce7daf244aa557     |
+---------------------------+--------------------------------------+
```

#### 拓扑结构

根据上述的配置， VLAN 网络在 Neutron 中的拓扑结构如下：


 ![vlan_topology][2]

 - tap：虚拟机的虚拟网卡
 - qbr：Linux Bridge
 - qvb、qvo：Linux Veth device
 - br-int、ovsbr3、ovsbr4：OpenvSwitch Bridge
 - patch：OpenvSwitch Patch device
 - eth3、eth4：物理网卡

`
说明：在同一计算节点内的所有虚拟机都会首先上联到各自的 Linux Bridge 上，
在通过 Linux Veth 设备连接到本地虚拟交换机（br-int）上，br-int 通过 OpenvSwitch Patch
设备连接若干本地虚拟机物理交换机（ovsbr3, ovsbr4），最后，计算节点上的物理网卡桥接到 ovsbr3 或者ovsbr4 上。
`

[1]: ../../images/architecture/vlan.png
[2]: ../../images/architecture/vlan_topology.png
