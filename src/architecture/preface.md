## 架构


---



整体架构是 UnitedStack<sup>®</sup> United Networking Platform(UNP) 的核心，整个 UNP 的架构可以从三方面阐述：
 - 虚拟路由器的分布式与高可用即 DVR 与 L3 HA
 - VxLan 的控制平面和广播抑制即 L2 Population 与 ARP Responder
 - 网络隔离方式 VxLan 与 Vlan

三个方面相互是比较独立的，只有 VxLan 的控制平面这里与分布式虚拟路由器有依赖关系，举个例子：
 网络隔离选择 VxLan 时，可以选择分布式虚拟路由器或高可用路由器，如果选择分布式路由器的话，则必须打开 L2 Population。
 Vlan 也是一样的。
 
关于架构方面，UnitedStack<sup>®</sup> UNP 默认推荐的设置是：
 - 默认打开 DVR，增强扩展性和性能；
 - 默认使用 Pacemaker 解决 Liberty 版本 DVR 的 SNAT 路由器高可用问题；
 - 同时允许创建高可用路由器，解决 DVR 下部分 VIP 类应用受影响的问题；
 - 默认打开 L2 Population + ARP Responder，配合 DVR 使用和提升网络有效连接数；
 - 默认使用 VxLan 租户网络，提升网络扩展性、解决 Vlan 网络 4096 限制。

对应的配置方法参考