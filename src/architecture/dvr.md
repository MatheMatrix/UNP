## DVR

---

### 简介

#### UnitedStack 知识库相关文章

 - [网络软件优化方案之DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR —Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)
 - [通过Rally测试DVR场景下的Neutron控制平面](https://confluence.ustack.com/pages/viewpage.action?pageId=16097482)

#### 传统实现的问题

 传统的 Open vSwitch Driver 实现的网关是集中式的，也就是说虚拟机的跨子网东西流量和外网的南北向流量都是回过网络节点这个中央节点的，参考下面两幅图分别说明了东西流量和南北流量：
 
 ![ew][1]
 
 不同子网之间由于必须要走网关完成路由（参考图中数字即为流量方向），所以即使是同一个计算节点的两个虚拟机依然要经过网络节点才能联通。本图中同时画出了 Vlan 和 Vxlan 的实现。
 
 ![ns][2]
 
 走公网的流量也需要默认路由到网关上再出公网（参考图中数字即为流量方向）。本图中同时画出了 Vlan 和 Vxlan 的实现。
 经过我们的长时期生产观察、验证和调优，可以确定一些地方是不改变架构几乎无法解决的：
  - 网络节点承担巨大压力，东西流量和南北流量都要过网络节点，这样在控制平面上对这个节点上会有数千甚至更多的的 namespace 和 port，这些对象都需要消息队列传递，常常出现单个消息过大引起消息队列性能恶性下滑、长时间无法成功收发消息导致消息队列吞吐下跌；
  - 网络节点带来严重的性能瓶颈，由于 namespace 内的路由和 NAT 性能有限，尤其表现在某一个虚拟路由器绑定了大量的 FloatingIP，而 iptables 性能和 conntrack 性能以及内核相关实现的问题导致这个虚拟路由器承担很高负载无法分散，进而引起性能问题；
  - 网络节点成为一个巨大的故障域，由于目前的软件的高可用切换最短也在数秒内，所以会出现用户感知到网络中断的问题，这样一旦一台网络节点故障可能会影响成百上千个用户的直接通信。

#### DVR 的解决方案

DVR 的基本思路就是将网关分散到各个计算节点，计算节点上的虚拟机就近路由和做地址转换（NAT），目前除了将 FloatingIP 直接绑到虚拟路由器上来让虚拟机通信外（一下简称为 SNAT 方式），所有流量已经实现了分布式，数据路径如下：

 ![ew_dvr][3]
 
 东西流量无需网络节点，可以直接在计算节点间完成传输（参考图中数字即为流量方向）。

 ![ns_dvr][4]
 
 具有 FloatingIP 的虚拟机南北流量同样无需网络节点，可以直接从计算节点通到外网。
 
#### DVR 的限制

工程领域是没有银弹的，所以 DVR 目前确实存在一些问题：
 - 对于 VxLan 拓扑需要开启 L2 Population 从而强制影响了数据平面，具体见 [VxLan](./vxlan.md) 上的描述；
 - 对消息队列的使用加重，增加了大量对 l3_agent 相关 topic 的消费
 - 存在与计算节点争抢资源的问题
 - 在 OpenStack Liberty 的实现上无法做到 DVR 与 L3 HA 共存，即实现 snat 的 snat_router 无法实现基于 VRRP 的原生高可用，对 L3 HA 的更多介绍参考 [L3 HA](./l3_ha.md)
 - snat router 无法很灵活的迁移到一般的计算节点上。因为其 l3 agent 类型是不一样的。

目前来看，我们的应对方案主要是同时提供 L3 HA 的虚拟路由器保证特殊业务可以正确运行


 

 
 [1]: ../../images/architecture/scenario-classic-ovs-flowew1.png
 [2]: ../../images/architecture/scenario-classic-ovs-flowns2.png
 [3]: ../../images/architecture/scenario-dvr-flowew1.png
 [4]: ../../images/architecture/scenario-dvr-flowns2.png