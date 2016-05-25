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
 
### DVR 的限制

工程领域是没有银弹的，所以 DVR 目前确实存在一些问题：
 - 对于 VxLan 拓扑需要开启 L2 Population 从而强制影响了数据平面，具体见 [VxLan](./vxlan.md) 上的描述；
 - 对消息队列的使用加重，增加了大量对 l3_agent 相关 topic 的消费
 - 存在与计算节点争抢资源的问题
 - 在 OpenStack Liberty 的实现上无法做到 DVR 与 L3 HA 共存，即实现 SNAT 的 snat_router 无法实现基于 VRRP 的原生高可用，对 L3 HA 的更多介绍参考 [L3 HA](./l3_ha.md)，关于 Bug 的详细描述见 [neutron-1365473](https://bugs.launchpad.net/neutron/+bug/1365473)
 - SNAT router 无法很灵活的迁移到一般的计算节点上。因为其 l3 agent 类型是不一样的
 - 目前 DVR 场景无法使用 VIP 类应用，也就是说，如果需要使用 keepalived、虚拟路由器等应用（参考[生态-Hillstone-云界]）时，是不能创建 DVR 虚拟路由器的。原因是当创建虚拟网卡并绑定浮动 IP 时，Neutron 无法确定如何绑定浮动 IP。

目前来看，我们的应对方案主要是以下几点：
 - 同时提供 L3 HA 的虚拟路由器保证特殊业务可以正确运行
 - 提高消息队列的性能和做大量的保证测试（参考 [系统－消息队列](../system/mq.md)）
 - 通过 cgroup 将计算节点上的重要业务（如 ceph）与其相隔离，将网卡中断绑定到与 qemu 所运行的 CPU 相同位置上
 - 基于 Pacemaker 实现对 snat 路由器的故障感知和自动迁移，详见下面的 Pacemaker 相关内容
 - 提供创建 HA 虚拟路由器的选项，用户可以自主选择虚拟路由器规格。

 SNAT 的灵活迁移目前没有很好的解决方案，可能需要大版本的升级。
 
### 稳定性验证
 
我们针对 DVR 环境进行了严格稳定性测试，具体包括控制平面性能并发验证，测试了不同次数和并发下的创建删除虚拟路由器、浮动 IP、创建更新虚拟路由器等。这里节选了创建、删除浮动 IP 的测试数据，完整的测试数据、测试方法和测试指标说明参考 UnitedStack 知识库相关文章或本文档其他章节 [性能](../performance/preface.md)。
 
 ![rally-1][5]
 
 此外为了达到模拟实际生产的效果，我们还做了压力测试，测试计算节点在很高的负载（比如整体 CPU 负载处在 50%、80%、100% 的状态）时的效果。详见 [稳定性]。

  
### 性能测试
 
 得益于架构的提升，DVR 环境下的性能同样有所改善，对于单流环境（单个虚拟机角度）主要提升效果在同主机上，但是由于架构提升，实际上整个系统的完整吞吐可以得到很高的提高，整个系统的路由吞吐不会再限制到某几个网络节点上，而是会分散到所有计算节点，完整的测试数据、测试方法和测试指标说明参考 UnitedStack 知识库相关文章或本文档其他章节 [性能](../performance/preface.md)。这里简单介绍与 Classic 对比的 VxLan 性能数据。
 
 ![Performance][7]
 
 测试项说明如下：
 
 | 编号 | 测试项 | 编号 | 测试项 |
 |:------|:--------|:------|:------|
 | 1 | 大包同子网跨主机 | 2 | 大包跨子网跨主机 |
 | 3 | 大包挎网络同主机 | 4 | 大包同子网同主机 |
 | 5 | 小包同子网跨主机 | 6 | 小包跨子网跨主机 |
 | 7 | 小包同子网同主机 | 8 | 小包跨子网同主机 |
 | 9 | 大包浮动 IP 跨主机 | 10 | 大包浮动 IP 同主机 |
 | 11 | 小包浮动 IP 同主机 |
 
此外，如果用户希望能获得更高的性能，也可以通过本文档的专业性能章节 [性能](../performance/preface.md) 获取更完整的性能优化建议

### 基于 Pacemaker 的高可用设计

 由于前文所述局限，目前分布式虚拟路由器的 1:N SNAT 模块是单点且无法支持原生基于 VRRP 的高可用，对此我们开发了基于 Pacemaker 的高可用方案：
 
 ![pacemaker][8]
 
 基本原理是不断检查 L3 Agent（如果部署了 VPN 服务的话也可以是 VPN Agent，下文简称为 L3 Agent）的状态。当探测到 L3 Agent 状态被 Neutron 置为 Down 时，通过 ICMP 检查网络节点状态，如果确定网络节点用户虚拟机业务网络的 VxLan/Vlan 相应网卡上的 IP 不可达时，则将虚拟路由器全部迁出，为了避免出现两个虚拟路由器同时服务，会通过 IPMI 将原节点关闭。

### 其他

#### 部署方式

核心配置是起相应 agent 和确保 Underlay 网络符合预期，具体可以查阅本文档的 [部署](../deployment) 了解具体细节。这里只简单描述一下服务部署模型：

![deploy_dvr][6]

值得注意的我们的 HA 方案对 Underlay 网络的要求，最佳实践是 Pacemaker 需要和管理网、业务网和 IPMI 网均通。

#### FAQ

 - Q：DVR 和 HA 路由器之间能否相互转换？
   A：不能。
 - Q：如果计算节点宕机了怎么办，是否需要额外操作？
   A：不需要，只要虚拟机执行了迁移操作，会自动反映到 Neutron 从而保证网络正常。

 
 [1]: ../../images/architecture/scenario-classic-ovs-flowew1.png
 [2]: ../../images/architecture/scenario-classic-ovs-flowns2.png
 [3]: ../../images/architecture/scenario-dvr-flowew1.png
 [4]: ../../images/architecture/scenario-dvr-flowns2.png
 [5]: ../../images/architecture/2016-05-18_00-56-11.png
 [6]: ../../images/architecture/scenario-dvr-services.png
 [7]: ../../images/architecture/DVR_CLA_Performance.png
 [8]: ../../images/architecture/ha_cluster_components_arch.png