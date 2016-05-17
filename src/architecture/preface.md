## 架构

---

整体架构是 UnitedStack<sup>®</sup> United Networking Platform(UNP) 的核心，然而通常意义上的架构可能会涉及网络的方方面面：
 - 接入层高可用
 - Underlay 网络规划
 - 服务器角色和数量规划
 - 高级服务的开启和使用
 - 第三方集成的使用
 - ……

然而以上内容会在 [部署](../deploy/preface.md)、[高级服务](../advanced_services/preface)、[生态](../3rd_party/preface.md) 等节讨论，本节讨论的核心问题是 Open vSwitch Driver 实现下的虚拟网络架构。所以以下内容中指代的 UNP 架构均指 Open vSwitch Driver 架构下的虚拟网络架构。

UnitedStack<sup>®</sup> UNP 的架构可以从四方面阐述：
 - 虚拟路由器的分布式与高可用即 Distributed Virtual Router（以下简称 DVR）与 Layer 3 High Availability（以下简称 L3 HA）
 - 网络的广播抑制 ARP Responder；
 - 网络隔离方式 VxLan 与 Vlan
 - 高性能的解决方案 SRIOV

四个方面相互是比较独立的，举个例子：
 网络隔离选择 VxLan 时，可以选择分布式虚拟路由器或高可用路由器，无论哪种选择，均建议打开 ARP Responder;
 选择 Vlan 时，无论选择分布式虚拟路由器或高可用路由器，同样建议打开 ARP Responder;
 SRIOV 比较特别，和其他架构有较大出入，目前生产经验较少，所以在 UnitedStack<sup>®</sup> UNP 1.0 版本可能无法使用，计划在后面的版本中支持。
 
关于架构方面，UnitedStack<sup>®</sup> UNP 默认推荐的设置是：
 - 默认打开 DVR，增强扩展性和性能；
 - 默认使用 Pacemaker 解决 Liberty 版本 DVR 的 SNAT 路由器高可用问题；
 - 同时允许创建高可用路由器，解决 DVR 下部分 VIP 类应用受影响的问题；
 - 默认打开 L2 Population + ARP Responder，配合 DVR 使用和提升网络有效连接数；
 - 默认使用 VxLan 租户网络，提升网络扩展性、解决 Vlan 网络 4096 限制。

但是针对

对应的配置方法参考 [部署-配置文件-默认推荐配置](../deploy/config/dvr.md)，具体的架构原理和实现意义参考本节内的文档。