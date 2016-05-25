# Vlan

----

## 数据链路层 ( L2 ) 基础知识

 数据链路层 ( Data Link Layer ) 是 [OSI 参考模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B)第二层，
位于物理层和网络层之间。主要功能是提供在两个网络实体之间提供数据链路连接的创建、维持和释放管理。 

### VLAN 基础知识

 LAN 表示 Local Area Network，本地局域网，通常使用 Hub 和 Switch 来连接 LAN 中的计算机。
一个 LAN 表示一个广播域，它的意思是 LAN 中的所有成员都会收到 LAN 中一个成员发出的广播包。
因此，LAN 的边界在路由器或者类似的三层设备。

 VLAN 表示 Virtual LAN。一个带有 VLAN 功能的 Switch 能够同时处于多个 LAN 中。简单的说，
VLAN 是一种将一个交换机分成多个交换机的一种方法。

 IEEE 802.1Q 标准定义了 VLAN Header 的格式。它在普通以太网帧结构 SA （src address）之后
加入了 4bytes 的 VLAN Tag/Header 数据，其中包括 12bits 的 VLAN ID。VLAN ID的最大值是 4096，
但是有效值范围是 1- 4094。

 如图是 VLAN 数据格式：

 ![vlan][1]

### VLAN 类型

 - 基于端口的 VLAN （untagged VLAN）
 这种模式中，在交换机上创建若干个 VLAN，在将若干端口放在每个 VLAN 中。每个端口在某一时刻
只能属于一个 VLAN。一个 VLAN 可以包含所有端口，或者部分端口。

 - Tagged VLANs （数据帧带有 VLAN tag）
这种模式下， 数据帧 的 VLAN 关系是它自己携带的信息中保存的。当交换机收到一个带有 VLAN tag
的 数据帧，它只将数据帧转发给具有同样 VID 的端口。一个能够接收或者转发 tagged frame 的端口
被称为 a tagged port。所有连接到这种端口的网络设备必须是 802.1Q 协议兼容的。这种设备必须能
处理 tagged frame，以及添加 tag 到其转发的数据帧。

 交换机的所有端口，部分是 tagged port，部分被添加到 VLAN 中。一个 untagged port ，一个时刻
只能在一个 VLAN 中，一个 tagged port 可以是多个 VLAN 的成员。

### 交换机端口类型

 以太网端口有三种链路类型：

- Access：只能属于一个 VLAN，一般用于连接计算机的端口
- Trunk：可以属于多个 VLAN，可以接收和发送多个 VLAN 的报文，一般用于交换机之间连接的接口
- Hybrid：属于多个 VLAN，可以接收和发送多个 VLAN 报文，既可以用于交换机之间的连接，也可以
用户连接用户的计算机。 Hybrid 端口和 Trunk 端口的不同之处在于 Hybrid 端口可以允许多个 VLAN 
的报文发送时不打标签，而 Trunk 端口只允许缺省 VLAN 的报文发送时不打标签。


### VLAN 的不足

- VLAN 使用 12-bit 的 VLAN ID，因此第一个不足之处就是最多只支持 4096 个 VLAN 网络
- VLAN 是基于 L2 的，因此很难跨越 L2 的边界，限制了网络的灵活性
- VLAN 的配置需手动介入较多

## Neutron 对 VLAN 网络的支持

 Neutron 支持预先定义一段或者多段 VLAN 范围，从而将 VLAN 池化，最终直接 VLAN 网络。
创建本节通过具体示例讲解 Neutron 对 VLAN 网络如何支持，以及 Neutron 的详细配置。
`说明：本示例中我们配置 VLAN 的范围是5 ~ 6 。`

### 交换机配置相应 VLAN 并验证

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

### Neutron 配置

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

### 创建 VLAN 网络

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

### 拓扑结构

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

## 用户使用场景

### 对性能网络有要求

 - 对网络性能有高要求的用户，建议选用 VLAN 网络。经过测试，当 VLAN 网络的网关地址
在物理设备上时（此时没有网络节点），通过 iPerf 进行流量测试，可以打满物理网卡的限速。

 - 对网络性能没有较高要求，并且想拥有网络节点的各种功能（比如L3 HA，LBaaS，VPN等）的用户，
也可以使用 VLAN 网络，只需将 VLAN 网络的网关设置在网络节点上即可。


### 打通企业内网

 打通 Neutron 网络和物理网络。通过 VLAN 网络的形式，无论 VLAN 的网关地址设置在物理设备还是网络节点上，
都可以比较容易的将 Neutron 网络和企业内网打通。


References:

1. https://zh.wikipedia.org/wiki/数据链路层
2. http://www.cnblogs.com/sammyliu/p/4626419.html
3. http://www.microhowto.info/tutorials/802.1q.html



[1]: ../../images/architecture/vlan.png
[2]: ../../images/architecture/vlan_topology.png
