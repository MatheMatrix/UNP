## VxLan

---

### 简介

#### UnitedStack 知识库相关链接

 - [如何使用tcpdump(pcap)匹配vni](https://confluence.ustack.com/pages/viewpage.action?pageId=10294007)
 - [Neutron中各个OVS网桥的作用以及流表规则](https://confluence.ustack.com/pages/viewpage.action?pageId=9642224)

#### Overlay 网络

 Overlay 在网络技术领域，指的是一种网络架构上叠加的虚拟化技术模式，
其大体框架是对基础网路不进行大规模修改的条件下，实现应用在网络上的承载，
并能与其他网络业务隔离，并且以基于 IP 的基础网络技术为主。Overlay 技术
是在现有的物理网络之上构建一个虚拟网络，上层应用只与虚拟网络有关。
一个 Overlay 网络主要由三部分组成：

 - 边缘设备：指与虚拟机直接相连的设备
 - 控制平面：主要负责虚拟隧道的建立维护以及主机可达信息的通告
 - 转发平面：指承载 Overlay 报文的物理网络

#### Overlay 主要技术标准和比较

 当前主流的 Overlay 技术主要有 VXLAN，GRE/NVGRE 和 STT。这三种二层 Overlay 
技术，大体思路都是将以太网报文承载到某种隧道层面，差异性在于选择和构造隧道
的不同，而底层均是 IP 转发。

 - VXLAN。 VXLAN 是将以太网报文封装在 UDP 传输层上的一种隧道转发模式，目的 UDP 端口号为4798；
为了使 VXLAN 充分利用承载网络路由的均衡性， VXLAN 通过将原始以太网数据头(MAC、IP、四层端口号等)的
 HASH 值作为 UDP 的号；采用24比特标识二层网络分段，称为 VNI(VXLAN Network Identifier)，
类似于 VLAN ID 作用；未知目的、广播、组播等网络流量均被封装为组播转发，
物理网络要求支持任意源组播(ASM)。
 - NVGRE。 NVGRE 是将以太网报文封装在 GRE 内的一种隧道转发模式；采用24比特标识二层网络分段，
称为 VSI(Virtual Subnet Identifier)，类似于 VLAN ID 作用；为了使 NVGRE 利用承载网络路由的均衡性，
 NVGRE 在GRE扩展字段 flow ID，这就要求物理网络能够识别到 GRE 隧道的扩展信息，
并以 flow ID 进行流量分担；未知目的、广播、组播等网络流量均被封装为组播转发。
 - STT。STT 利用了 TCP 的数据封装形式，但改造了 TCP 的传输机制，数据传输不遵循 TCP 状态机，
而是全新定义的无状态机制，将 TCP 各字段意义重新定义，无需三次握手建立 TCP 连接，因此称为无状态 TCP；
以太网数据封装在无状态 TCP；采用64比特 Context ID 标识二层网络分段；为了使 STT 充分利用承载
网络路由的均衡性，通过将原始以太网数据头(MAC、IP、四层端口号等)的 HASH 值作为无状态 TCP 的源端口号；
未知目的、广播、组播等网络流量均被封装为组播转发。

上述三种 Overlay 技术的比较:

|技术名称|支持者|支持方式|网络虚拟化方式|数据新增报文长度|链路HASH能力|
|:--:|:--:|:--:|:--:|:--:|:--:|
|VXLAN|Cisco/VMWARE/Citrix/Red Hat/Broadcom|L2 over UDP|VXLAN报头 24 bit VNI|50Byte(+原数据)|现有网络可进行L2 ~ L4 HASH|
|NVGRE|HP/Microsoft/Broadcom/Dell/Intel|L2 over GRE|NVGRE 报头 24 bit VSI|42Byte(+原数据)|GRE头的HASH 需要网络升级|
|STT|VMWare|无状态TCP，即L2在类似TCP的传输层|STT报头 64 bit Context ID|58 ~ 76Byte(+原数据)|现有网络可进行 L2 ~ L4 HASH|


#### VXLAN 转发平面的实现

VXLAN 将二层数据帧封装成 UDP 包

 ![vxlan][1]

- 含义：

 - Outer MAC destination address (MAC address of the tunnel endpoint VTEP)
 - Outer MAC source address (MAC address of the tunnel source VTEP)
 - Outer IP destination address (IP address of the tunnel endpoint VTEP)
 - Outer IP source address (IP address of the tunnel source VTEP)
 - Outer UDP header：Src port 往往用于 load balancing，下文有提到；Dst port 即 VXLAN Port，默认值为 4789
 - VNID：表示该帧的来源虚机所在的 VXLAN 网段的 ID

- 特点：
 
 - VNID： 24-bits，最大 16777216。每个不同的 24-bits VNI 代表一个 VXLAN 隔离
 - VXLAN Port：目的 UDP 端口，默认使用 4789 端口。用户可以自己配置
 - 两个 VTEP 之间的 VXLAN tunnels 是无状态的
 - VTEP 可以在虚拟交换机上，物理交换机或者物理服务器上通过软件或者硬件实现
 - 使用多播来传送未知目的的、广播或者多播帧
 - VTEP 不可以对 VXLAN 包分段


### VXLAN 的控制平面

 早期的 VXLAN 实现并没有控制平面，通过 flood-and-learn 来实现，而且没有设计路由如何实现，这样导致 VXLAN 的通信效率低、充斥广播和无效的 ARP，为了解决这个问题，厂商首先实现了 HER、OVSDB 等方法，后来 IETF 撰写了 BGP MPLS-Based Ethernet VPN (RFC7432) 标准，目前网络厂商基本都在实现或向这个标准实现。
 
#### Multicast Based VXLAN

 基于组播的 VXLAN 网络其实是没有控制平面的，依赖于数据平面的 flood-and-learn，如果交换机不支持组播的话，将会退化到广播，目前这类的应用已经很少了。
 
#### Head-End Replication

 为了解决组播的依赖，一种方法是通过 HER 的方法复制报文成单播，这样组播报文或者广播报文可以通过单播复制的形式发送，这种方式被称为 Head-End Replication（简称 HER）。
 
 ![HER][4]
 
 Open vSwitch Driver 实现的 VXLAN 即使用类似这种方式避免组播的依赖。
 
 HER 在即使有控制平面的情况下依然具备价值，因为有可能有静默主机、MAC 表项老化、虚拟机需要使用组播或广播达成业务的需求。在部分厂商的文档可能描述为 Ingress Replication。
 
 
#### OVSDB
 
 HER 是无法真正解决 VXLAN 缺乏控制平面的问题的，因为 MAC 的学习依然是由报文驱动的。那么需要一个控制平面来管理 MAC 地址表项与其 VTEP 的对应关系。VMware NSX 为了解决这一问题通过 OVSDB 来控制 ToR，注入了 MAC 地址与 VTEP 的对应，即每个交换机会作为 OVSDB 的 Client，连接到 OpenFlow Controller。

 OVSDB 定义了一些数据模型：
 
 - Global table
 - Manager table
 - Physical switch table
 - Physical port table
 - Logical switch table
 - Logical binding statistics table
 - Physical locator table
 - Physical locator set table
 - Unicast MACs remote table
 - Unicast MACs local table
 - Multicast MACs remote table
 - Multicast MACs local table

其表关系如下图所示：

![ovsdb][5]

其中 Unicast MACs remote table 保存了虚拟网络内的可达信息表，Unicast MACs local table 保存了物理网络的可达信息表，Multicast MACs remote table 用于将组播映射为单播到软件 VTEP，Multicast MACs local table 用于将组播映射到硬件 VTEP。
 
#### MPBGP EVPN VXLAN

OVSDB 的手段类似于通过外部手段影响 VXLAN 转发，非协议原生的实现，IETF 撰写了 RFC 7432，描述了使用 BGP 作为 VXLAN 的控制平面的参考设计。通过 EVPN，MAC 的学习将类似于三层网络陆游的学习，将有助于有效减少泛洪。

![evpn][6]

如上图，每一个 VTEP 将作为一个 BGP Speaker，向其他 VTEP 通过 EVPN 发送本地的 MAC、IP 信息，BGP RR 可以避免 BGP 的 Full-Mesh，提高通信效率。得益于控制平面，每个 VTEP 将可作为分布式网关、可以抑制 ARP 广播、可以将广播或组播通过单播复制来提升效率、可以对 VTEP 进行认证。

具体到 BGP 租网上，有几种选择，包括 iBGP、eBGP 的选择和外部网络通信。

![iBGP][7]

这种模型下 VTEP 只在 Leaf 上，Spine 中选取两个作为 iBGP RR，Spine 不需要作为 VTEP。此外 RR 也可以有多种放置方法，例如在 Leaf 上，这样 Spine 不需要运行 MPBGP EVPN，或者在额外的专门网络设备。

如果是 eBGP，典型的部署方法如下图，好处是 Spine 作为 eBGP Peer，而不是 iBGP RR，Spine，Spine上 的 BGP 需要有对 address-family l2vpn evpn 的转发能力，但不需要支持 VXLAN。所有 Leaf 可以设置各自的 AS，也可以设置为同一 AS，eBGP 运维难度较高，参考设计见 draft-ietf-rtgwg-bgp-routing-large-dc，目前一般较少采用。

![eBGP][8]

#### Distributed Anycast Gateway

 IETF 在 draft-ietf-bess-evpn-inter-subnet-forwarding中对在 EVPN 中属于不同的 VxLan 下如何通过 Integrated Routing and Bridging（以下简称 IRB）处理跨子网通信做了说明，换句话说，EVPN VxLan 提供了原生的基于 IRB 的分布式三层网关参考。
 
 然而 EVPN VxLan 的实际路由过程可以分成两步来谈，第一部分是虚拟机的 First-hop 的地址，即网关地址，第二部分是如何在不同 VxLan 间路由（IRB），本节会先谈网关地址的问题。
 
 目前一种实践是使用 Anycast Gateway 技术，每个 VTEP 上均配置相同的 vIP 和 vMAC，如图：
 
 ![Anycast_gateway][9]
 
 这样首先每个虚拟机的网关都在最近的 VTEP 上，可以优化网络路径，其次当虚拟机发生迁移时，不会需要重新获取默认网关的 ARP。在一些厂商中，这项技术被称为 Static Anycast Gateway。
 
#### Integrated Routing and Bridging

 IRB 即 VTEP 提供三层和二层功能，但是对于具体如何路由，目前存在两种方法，分别为 Asymmetric IRB mode 和 Symmetric IRB mode。前者是非对称模式，后者是对称模式，对于 Asymmetric，结合 Anycast Gateway 后路径是这样的：
 
 ![SIRB][10]

 报文由虚拟机发出时，目的 MAC 是网关的虚拟 MAC，VTEP-1 收到报文后查询路由找到 IP-2 对应的虚拟机，查询到对应的 VTEP 为 VTEP-2 后，封上 VxLan 的头部发到 VTEP-2，并将 VNI 设置为对方的 VNI-B，VTEP-2 收到报文后，将 VxLan 头部剥掉换成 Vlan 并发往 VM-2。
 
 当虚拟机需要回复时，路径完全反过来，即在 VTEP-2 上完成 VXLAN 封包和设置 VNI 为 VNI-A。所以这个过程是非对称的。
 
 这种实现存在一些显而易见的问题：
 
 - 所有的 VTEP 必须配置上所有的 VXALN VNI，否则不同 VNI 通信会存在问题；
 - 所有的 VTEP 必须获整个 Fabric 完整的 Host tables 信息，否则无法完成完路由。

另一种实现方法是 Symmetric IRB，其实现与 Asymmetric IRB 最显著的不同是源 VTEP 和目标 VTEP 都会承担三层和二层功能，而不像 Symmetric IRB 只在源 VTEP 做路由。这样最终实现是对称的，但前提是必须引入一个新的概念即 L3 VNI。
 
 在 Symmetric IRB 中，每个租户的 VRF 会分配一个 L3 VNI，可达信息（NLDR）会在同一个 L3 VNI 下同步，这样每次路由需要将外层 VXLAN 目的地址设置为目的 VTEP 的地址，将 VNI 设置为 L3 VNI。
 
 ![L3 VNI][11]
 
从上面的分析可以得知，Symmetric IRB 最大的好处一是不需要所有 VTEP 均配置所有 VNI，二是不需要所有的 VTEP 知道整个 Fabric 的完整 Host tables 信息。但是，这是建立在不是最差情况的前提，如果说恰好每个租户都在每个 VTEP 下具有虚拟机，或着整个网络只有一个租户，那么网络可能产生退化。Symmetric IRB 的主要优化场景是针对多租户的。

 ![L3 MultiTenancy][12]
 
#### VLAN Scoping

为了解决 VXLAN 的扩展性，同一个 VXLAN 在不同的 VTEP 下可以对应不同的 VLAN，目前 VXLAN 到 VLAN 映射有 Leaf 层面和 Port 层面两种实现，主流是 Leaf 层面的。

![VlanScoping][13]

### Neutron Open vSwitch Driver 下的 VXLAN

 Neutron 通过 OpenvSwitch （以下简称 OVS）支持 VXLAN。OVS 在计算节点/网路节点的 br-tun 上建立多个 tunnel port，和其他节点的 tunnel port 之间建立点对点的 VXLAN Tunnel。 Tunnel Port 在 OVS 上的形式如下：

```
Bridge br-tun
    fail_mode: secure
    Port patch-int
        Interface patch-int
            type: patch
            options: {peer=patch-tun}
    Port "vxlan-c0a86444"
        Interface "vxlan-c0a86444"
            type: vxlan
            options: {df_default="true", in_key=flow,
                local_ip="192.168.100.233", out_key=flow,
                remote_ip="192.168.100.68"}
```

Neutron VXLAN network 的 segmentation_id 属性即为 VXLAN 的 VNI。

#### L2 Population

 早器的 Neutron 实现的 VXLAN 是 full-mesh 的，即每个 VTEP 之间都建立 VXLAN 隧道，当遇到未知单播、广播时，如果 VTEP 没有 cache 过这条流则会广播到所有 VTEP，造成严重的网络效率下降，为此，我们建议用户打开 L2 Population 来提升网络效率。
 
 在没打开 L2 Population 之前，通信路径如下：
 
 ![without_l2_pop][2]
 
打开 L2 Population 后，如果配合 ARP Repsonder（参考 [ARP Responder](../arp_responder.md)），则可以达到 ARP 本地回复，流量走单播，即下图所示

 ![with_l2_pop_and_arp][3]
 
 但是 L2 Population 问题也很明显：
 
 - 云资源管理平面强制影响了数据平面；
 - 将网络的广播一定程度上转成了消息队列的广播，加重了消息队列的负载。

目前我们通过优化消息队列的性能和高可用来解决 L2 Population 加重了消息队列的使用。

#### DVR

为了解决 IRB 的问题，Neutron 通过软件的方法在每个计算节点实现了分布式的路由和 NAT，可以有效提高整个系统的吞吐和性能，详见 [DVR](../DVR.md)。

#### Conversational Learning

当然来自未知的位置报文时，Neutron 通过 OVS 的 learn 动作可以实现对流的会话学习（Conversational Learning）。在 Open vSwitch 的 br-tun 网桥上，存在一条学习规则如下：

`cookie=0xbf36e76cf0946b9e, duration=16123.771s, table=10, n_packets=419, n_bytes=40096, idle_age=10442, priority=1 actions=learn(table=20,hard_timeout=300,priority=1,cookie=0xbf36e76cf0946b9e,NXM_OF_VLAN_TCI[0..11],NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[],load:0->NXM_OF_VLAN_TCI[],load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[],output:NXM_OF_IN_PORT[]),output:1`

往表 20 中添加对返程包的正常转发规则，并且从 patch-int 送到 br-int。

其中 learn 内的 table=20 说明是修改表 20 中的规则，后面是添加的规则内容；
NXM_OF_VLAN_TCI[0..11]，匹配和当前流同样的 VLan 头，其中 NXM 是 Nicira Extensible Match 的缩写；
NXM_OF_ETH_DST[]=NXM_OF_ETH_SRC[]，包的目的 MAC 跟当前流的源 MAC 匹配；
load:0->NXM_OF_VLAN_TCI[]，将 VLAN 号改为 0；
load:NXM_NX_TUN_ID[]->NXM_NX_TUN_ID[]，将 Tunnel 号修改为当前的 Tunnel 号；
output:NXM_OF_IN_PORT[]，从当前入口发出。

#### Flooding VTEP

社区目前还在推进外部 VTEP（Flooding VTEP）的支持，即将 OpenStack 管理的 VTEP 与外部的非 OpenStack 管理的 VTEP 混合组网，目前的设计比较弱，需要在创建网络时指定每一个外部 VTEP，这里很大一部分原因是目前 Open vSwitch Driver 实现的 VXLAN 并不能支持控制平面协议。

#### Edge Replication

类似于网络设备实现的 Head-End Replication，对于未知的流量，可以通过本地复制避免对组播的依赖：

`cookie=0x0, duration=1322342.725s, table=22, n_packets=4, n_bytes=280, idle_age=65534, hard_age=65534, dl_vlan=17 actions=strip_vlan,set_tunnel:0x10d,output:2,output:9,output:8`

#### ARP Responder

在有了ARP Responder后，由于Neutron数据库中保存了网络中的所有数据，此时可以通信的虚拟机的 IP、MAC 信息可以分发到个计算节点，计算节点上本地做 ARP 代理，直接返回 ARP 请求，详见 [ARP Responder](../arp_responder.md)。

#### 总结

对于早期 VXLAN 协议的种种问题，厂商和 OpenStack 社区都提出了各自的解决方案，厂商的设计偏向于通过设备 + 协议，而社区的参考设计偏向于软件控制，二者各有其优点与缺点，在这里不妨将 EVPN 与 Open vSwitch Driver VXLAN 做一个对比：

| EVPN 实现 | Open vSwitch Driver VXLAN 实现 |
| -------- | ----------------- |
| Head-End Replication | Edge Replication |
| ARP Supression | ARP Responder |
| MAC advertise | L2 Population |
| Conversational Learning | Conversational Learning |
| Asymmetric IRB mode & Anycast Gateway | DVR |

目前 Open vSwitch Driver VXLAN 的短板一是消息的效率目前还没有很好的优化，二是控制平面比较集中在 Neutron Server，不能实现分布式的网络信息分发。

### 性能

得益于 DVR，UnitedStack<sup>®</sup> UNP 的整体吞吐可以得到很高的提高，整个系统的路由吞吐不会再限制到某几个网络节点上，而是会分散到所有计算节点，完整的测试数据、测试方法和测试指标说明参考 UnitedStack 知识库相关文章或本文档其他章节 [性能](../performance/preface.md)。这里简单介绍 DVR 与 Classic 对比的 VxLan 性能数据。
 
 ![Performance][14]
 
 测试项说明如下：
 
 | 编号 | 测试项 | 编号 | 测试项 |
 | :------ | :-------- | :------ | :------ |
 | 1 | 大包同子网跨主机 | 2 | 大包跨子网跨主机 |
 | 3 | 大包挎网络同主机 | 4 | 大包同子网同主机 |
 | 5 | 小包同子网跨主机 | 6 | 小包跨子网跨主机 |
 | 7 | 小包同子网同主机 | 8 | 小包跨子网同主机 |
 | 9 | 大包浮动 IP 跨主机 | 10 | 大包浮动 IP 同主机 |
 | 11 | 小包浮动 IP 同主机 |
 
此外，如果用户希望能获得更高的性能，也可以通过本文档的专业性能章节 [性能](../performance/preface.md) 获取更完整的性能优化建议，简单的说，推荐使用 Intel X710、Mellanox Connect-X3、Mellanox Connect-X4 等支持 VXLAN 卸载的网卡，以及使用 6Wind 加速器等产品。


### 参考链接

1. [draft-ietf-rtgwg-bgp-routing-large-dc](https://tools.ietf.org/html/draft-ietf-rtgwg-bgp-routing-large-dc)
2. [draft-ietf-bess-evpn-inter-subnet-forwarding](https://tools.ietf.org/html/draft-ietf-bess-evpn-inter-subnet-forwarding-01)
- [BGP MPLS-Based Ethernet VPN (RFC7432)](https://tools.ietf.org/html/rfc7432)
- [Data Center Overlay Technologies](http://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/white-paper-c11-730116.html)
- [VXLAN Network with MP-BGP EVPN Control Plane Design Guide](http://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/guide-c07-734107.html)
- [Brocade IP Fabric and Network Virtualization with BGP EVPN](https://www.brocade.com/content/dam/common/documents/content-types/brocade-validated-design/brocade-ip-fabric-bvd.pdf)
- [Learn About VXLAN in Virtualized Data Center Networks](http://www.juniper.net/techpubs/en_US/learn-about/LA_VXLANinDCs.pdf)
- [华为CloudEngine系列交换机VXLAN技术白皮书](http://e.huawei.com/cn/marketing-material/onLineView?MaterialID=%7BAE4F8D00-5D74-414E-91F7-9E3E39E16BF1%7D)
- [云化数据中心互联网络技术浅析](http://www.h3c.com.cn/Solution/Operational/DataCenter/Solutions/201510/895551_30004_0.htm)
- [基于多租户的云计算Overlay网络](http://www.h3c.com.cn/About_H3C/Company_Publication/IP_Lh/2013/04/Home/Catalog/201309/796466_30008_0.htm)

[1]: ../../images/architecture/vxlan.jpg
[2]: ../../images/architecture/L2_population_1.svg
[3]: ../../images/architecture/L2_population_3.svg
[4]: ../../images/architecture/enhanced-vxlan-12.png
[5]: ../../images/architecture/QQ20160526-1.png
[6]: ../../images/architecture/QQ20160526-2.png
[7]: ../../images/architecture/QQ20160526-3.png
[8]: ../../images/architecture/QQ20160526-4.png
[9]: ../../../images/ecosystem/QQ20160525-1.png
[10]: ../../../images/ecosystem/AsymmetricIRB.png
[11]: ../../../images/ecosystem/QQ20160525-5.png
[12]: ../../../images/ecosystem/QQ20160525-4.png
[13]: ../../images/architecture/QQ20160526-5.png
[14]: ../../images/architecture/DVR_CLA_Performance.png
