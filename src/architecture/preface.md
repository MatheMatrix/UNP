## 架构

整体架构是 UnitedStack<sup>®</sup> UnitedNetworkingPlatform(UNP) 的核心，整个 UNP 的架构可以从三方面阐述：
 - 虚拟路由器的分布式与高可用即 DVR 与 L3 HA
 - VxLan 的控制平面和广播抑制即 L2 Population 与 ARP Responder
 - 网络隔离方式 VxLan 与 Vlan

三个方面相互是比较独立的，只有 VxLan 的控制平面这里与分布式虚拟路由器有依赖关系，举个例子：
 网络隔离选择 VxLan 时，可以选择分布式虚拟路由器或高可用路由器，如果选择分布式路由器的话，则必须打开 L2 Population。
 网络隔离