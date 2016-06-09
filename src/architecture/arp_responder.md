## ARP Responder

---

###简介

#### UnitedStack 知识库相关文章

 - [Neutron中各个OVS网桥的作用以及流表规则](https://confluence.ustack.com/pages/viewpage.action?pageId=12063326)
 - [网络软件优化方案之DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR —Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)

#### 什么是ARP

地址解析协议（Address Resolution Protocol），其基本功能为透过目标设备的 IP 地址，查询目标设备的 MAC 地址，以保证通信的顺利进行。所谓地址解析（address resolution）就是主机在发送帧前将目标 IP 地址转换成目标 MAC 地址的过程。

 举个例子，如图的两个虚拟机进行通信

 ![two_machines][1]

 当在 Machine A 上执行ping 192.168.10.20时，如果 Machine A 从来没有和 Machine B 通信过，也就不会知道 Machine B 的 MAC 地址是什么。如何知道 Machine B 的 MAC 地址？答案是洪泛。当 Machine B 上收到了泛洪包后，发现 ARP 请求的地址是自己，进行 ARP 的回复，*注意：此时回复的是单播*。此时，Machine A 知道了 Machine B 的 MAC 地址后，最后在发送 ICMP 的请求报文。

### 什么是ARP Proxy

Proxy ARP 就是通过一个主机（通常是Router）来作为指定的设备对另一个设备作出 ARP 的请求进行应答。

 简单的说，当机器发送 ARP 请求时，ARP Proxy 捕捉到此请求，并进行回复。

### 什么是ARP Responder

Neutron通过OpenvSwitch Bridge br-tun 提供 ARP Responder 的功能。

 在详细介绍 ARP Responder 之前，我们需要了解 Neutron 中的另一个概念—— L2 Population。L2 population 的工作原理并不复杂。Neutron 的数据库中保存了网络中的所有数据，在虚拟机启动过程中，其端口会从 Down 到 Build 再到 Active，因此，每当端口状态发生改变时，都会触发 L2 population 的相关逻辑。

 在没有ARP Responder之前L2 population如何工作？

 如图，分别在两台宿主机上的两台虚拟机进行通信：

 ![two_node][2]

 根据刚才对于 ARP 的介绍，如果 VM B 从来没有和 VM A 通信过，当 VM A 要和 VM B 通信时，Node A 并不知道 VM B 的 MAC 地址是什么，
此时会将广播请求泛洪到所有 Node A 相连的 Tunnel Ports 上，当 VM B 回复了泛洪请求后，Node A 会学习到一条关于 VM B 的流表，
流表的内容是 VM B 的 Node Tunnel 信息（会话学习）。通过这种方式，避免了以后的广播请求（因为以后的广播请求没有了洪泛过程，直接发到了相应的 Node 上）。
通过这种方式虽然我们优化了 VXLAN 广播流量，但是 ARP 呢？。还是举上面的例子，Node A 学习到了关于 VM B 的流表，
当 VM A 和 VM B 通信时，Node A 将 VM A 的 ARP 请求发到了 Node B 上，但是 Node B 上可能不仅仅有 VM B 一个机器，Node B 上依旧收到了多余的 ARP 请求。
因此 ARP 的问题依旧没有解决。

 在有了 ARP Responder 后，由于 Neutron 数据库中保存了网络中的所有数据，此时在 Node C 上创建 VM C，那么，Node A 和 Node B 上的 OpenvSwitch Bridge br-tun
上会添加一条流表：VM C 在 Node C 上，IP 地址是 x.x.x.x，MAC 地址是 y.y.y.y.y.y。这样一来，VM A，VM B 和 VM C 通信时，br-tun 作为 ARP Responder 回复各自的 ARP 请求。

###ARP Responder 的使用场景和限制

 目前来说，DVR 用到了 ARP Responder 和 L2 population。但是目前仍然有一些问题：
 - 对消息队列的使用加重，增加了相关 Topic 的消费
 - 开启 L2 population 后计算/网络节点上的流表增多，一定程度上影响了数据平面
 - 增加了网络运维的难度

###如何开启ARP Responder 和 L2 population

 在 Neutron Server 节点修改 plugin.ini：
```
[ml2]
mechanism_drivers = ...,l2population,...
```

 在计算/网络节点修改plugins/ml2/openvswitch_agent.ini

```
[agent]

l2_population = True
arp_responder = True
```

 [1]: ../../images/architecture/two-node-machine-communicate.png
 [2]: ../../images/architecture/three-node-machine-communicate.png
