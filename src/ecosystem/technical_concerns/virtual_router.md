## 虚拟路由器实现

### 简介

#### UnitedStack 知识库相关文章

 - [Neutron Router 迁移理论分析、优化思路与实践](https://confluence.ustack.com/download/attachments/8752227/Neutron%20Router%20%E8%BF%81%E7%A7%BB%E7%90%86%E8%AE%BA%E5%88%86%E6%9E%90%E3%80%81%E4%BC%98%E5%8C%96%E6%80%9D%E8%B7%AF%E4%B8%8E%E5%AE%9E%E8%B7%B5.docx?version=1&modificationDate=1448294489623&api=v2)

#### 技术说明

 当虚拟机进行跨子网（东西向）和外网通信（南北向）通信时，需要走虚拟路由器或其他类似服务完成相关的路由和地址转换功能，本节不会讨论虚拟路由器上可能实现的高级功能。对于路由与地址转换，基本的实现思路有几种：
 
 - 集中式软件方案。例如 Neutron Classic 方案，特点是路由集中式，效率偏低，但代码比较成熟，目前外部 SDN 系统一般都会解决这个问题；
 - 集中式硬件方案。例如通过一个硬件路由器或其他设备实现路由和地址转换，得益于网络设备的高性能效果会比前者好；
 - 分布式软件方案。例如 Neutron DVR 方案、DragonFlow、OVN 等，特点是分布式，软件实现，故障域和性能都比较不错，具体参考本文档的 [DVR](src/architecture/dvr.md)；
 - 分布式硬件方案。例如通过 Distributed Anycast Gateway（EVPN）、OpenFlow 等在 ToR 上实现路由功能，特点是分布式、硬件实现，故障域和性能都比较理想；
 - 分布式软硬件融合方案。例如通过 OpFlex 对 Open vSwitch 做控制，对于能在本机路由、地址转换的在服务器端实现，需要跨 ToR 的在 ToR 上实现，性能最好，但需要对服务器端做多改动。

我们可以将涉及硬件的方案简单归纳如下：
 
 ![vxlan_l3_gw][1]
 
 当然其中是否使用 Border Leaf 来提供对外的连接，是否在计算节点内部尽量完成可完成的路由或 NAT，或者是否使用 OpenFlow 作为南向协议需要考虑具体的方案，这里只是基本的原理参考说明。

#### Distributed Anycast Gateway（EVPN）

 关于 EVPN VxLan 的详细说明参考 [架构－VxLan](../../architecture/vxlan.md)，这里简单说明 Anycast Gateway 的工作原理。
 
 EVPN 的 RFC draft（见参考链接）中对在 EVPN 中属于不同的 VxLan 下如何通过 Integrated Routing and Bridging（以下简称 IRB）处理跨子网通信做了说明，换句话说，EVPN VxLan 提供了原生的基于 IRB 的分布式三层网关参考。
 
 然而 EVPN VxLan 的实际路由过程可以分成两步来谈，第一部分是虚拟机的 First-hop 的地址，即网关地址，第二部分是如何在不同 VxLan 间路由（IRB），本节会先谈网关地址的问题。
 
 目前一种实践是使用 Anycast Gateway 技术，每个 VTEP 上均配置相同的 vIP 和 vMAC，如图：
 
 ![Anycast_gateway][2]
 
 这样首先每个虚拟机的网关都在最近的 VTEP 上，可以优化网络路径，其次当虚拟机发生迁移时，不会需要重新获取默认网关的 ARP。
 
#### Integrated Routing and Bridging

 IRB 即 VTEP 提供三层和二层功能，但是对于具体如何路由，目前存在两种方法，分别为 Asymmetric IRB mode 和 Symmetric IRB mode。前者是非对称模式，后者是对称模式，对于 Asymmetric，结合 Anycast Gateway 后路径是这样的：
 
 ![SIRB][3]

 报文由虚拟机发出时，目的 MAC 是网关的虚拟 MAC，VTEP-1 收到报文后查询路由找到 IP-2 对应的虚拟机，查询到对应的 VTEP 为 VTEP-2 后，封上 VxLan 的头部发到 VTEP-2，并将 VNI 设置为对方的 VNI-B，VTEP-2 收到报文后，将 VxLan 头部剥掉换成 Vlan 并发往 VM-2。
 
 当虚拟机需要回复时，路径完全反过来，即在 VTEP-2 上完成 VxLan 封包和设置 VNI 为 VNI-A。所以这个过程是非对称的。
 
 这种实现存在一些显而易见的问题：
 
 - 所有的 VTEP 必须配置上所有的 VxLan VNI，否则不同 VNI 通信会存在问题；
 - 所有的 VTEP 必须获整个 Fabric 完整的 Host tables 信息，否则无法完成完路由。

另一种实现方法是 Symmetric IRB，其实现与 Asymmetric IRB 最显著的不同是源 VTEP 和目标 VTEP 都会承担三层和二层功能，而不像 Symmetric IRB 只在源 VTEP 做路由。这样最终实现是对称的，但前提是必须引入一个新的概念即 L3 VNI。
 
 在 Symmetric IRB 中，每个租户的 VRF 会分配一个 L3 VNI，可达信息（NLDR）会在同一个 L3 VNI 下同步，这样每次路由需要将外层 VxLan 目的地址设置为目的 VTEP 的地址，将 VNI 设置为 L3 VNI。
 
 ![L3 VNI][4]
 
从上面的分析可以得知，Symmetric IRB 最大的好处一是不需要所有 VTEP 均配置所有 VNI，二是不需要所有的 VTEP 知道整个 Fabric 的完整 Host tables 信息。但是，这是建立在不是最差情况的前提，如果说恰好每个租户都在每个 VTEP 下具有虚拟机，或着整个网络只有一个租户，那么网络可能产生退化。Symmetric IRB 的主要优化场景是针对多租户的。

 ![L3 MultiTenancy][5]
 

### 最佳实践

 一般认为分布式的路由实现会比集中式的实现架构更优异，但需要考虑具体场景和需求，例如集中式实现可以比较容易的管理、排错、人工修改配置，但分布式能够带来更小的故障域和更好的吞吐。而软硬融合的方案固然能带来最佳的架构和性能，但是需要看实现，目前的第三方实现可能需要在计算节点带来较多的修改，不利于排错和架构兼容性。
 
 使用 EVPN 实现三层的好处是协议化和标准化，但是其实不如 OpenFlow 灵活。关于 IRB 目前一般认为 Symmetric IRB 是优于 Asymmetric IRB 的，但是一些厂商是同时支持两种模式的。
 
 
### 参考链接
 
 - [draft-ietf-bess-evpn-inter-subnet-forwarding](https://tools.ietf.org/html/draft-ietf-bess-evpn-inter-subnet-forwarding-01)
 - [BGP MPLS-Based Ethernet VPN (RFC7432)](https://tools.ietf.org/html/rfc7432)
 - [VXLAN Network with MP-BGP EVPN Control Plane Design Guide](http://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/guide-c07-734107.html)
 - [Brocade IP Fabric and Network Virtualization with BGP EVPN](https://www.brocade.com/content/dam/common/documents/content-types/brocade-validated-design/brocade-ip-fabric-bvd.pdf)
 - [Learn About VXLAN in Virtualized Data Center Networks](http://www.juniper.net/techpubs/en_US/learn-about/LA_VXLANinDCs.pdf)
 - [华为CloudEngine系列交换机VXLAN技术白皮书](http://e.huawei.com/cn/marketing-material/onLineView?MaterialID=%7BAE4F8D00-5D74-414E-91F7-9E3E39E16BF1%7D)
 - [云化数据中心互联网络技术浅析](http://www.h3c.com.cn/Solution/Operational/DataCenter/Solutions/201510/895551_30004_0.htm)
 
 
 
  [1]: ../../../images/ecosystem/l3_gw.png
  [2]: ../../../images/ecosystem/QQ20160525-1.png
  [3]: ../../../images/ecosystem/AsymmetricIRB.png
  [4]: ../../../images/ecosystem/QQ20160525-5.png
  [5]: ../../../images/ecosystem/QQ20160525-4.png