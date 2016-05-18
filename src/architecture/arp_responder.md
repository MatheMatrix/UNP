## ARP Responder 

---

###简介

#### UnitedStack 知识库相关文章

 - [Neutron中各个OVS网桥的作用以及流表规则](https://confluence.ustack.com/pages/viewpage.action?pageId=12063326)
 - [网络软件优化方案之DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR —Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)

#### 什么是ARP

 > 地址解析协议（Address Resolution Protocol），其基本功能为透过目标设备的IP地址，查询目标设备的MAC地址，以保证通信的顺利进行。所谓地址解析（address resolution）就是主机在发送帧前将目标IP地址转换成目标MAC地址的过程。

 举个例子，如图的两个虚拟机进行通信

 ![two_machines][1]

 当在Machine A上执行ping 192.168.10.20时，如果Machine A从来没有和Machine B通信过，也就不会知道Machine B的MAC地址是什么。如何知道Machine B的MAC地址？答案是洪泛。当Machine B上收到了泛洪包后，发现ARP请求的地址是自己，进行ARP的回复，*注意：此时回复的是单播*。此时，Machine A知道了Machine B的MAC地址后，最后在发送ICMP的请求报文。

### 什么是ARP Proxy

 > Proxy ARP 就是通过一个主机（通常是Router）来作为指定的设备对另一个设备作出 ARP 的请求进行应答。

 简单的说，当机器发送ARP请求时，ARP Proxy捕捉到此请求，并进行回复。

### 什么是ARP Responder

 > Neutron通过OpenvSwitch Bridge br-tun提供ARP Responder的功能。
 
 在详细介绍ARP Responder之前，我们需要了解Neutron中的另一个概念——L2 Population。L2 population的工作原理并不复杂。Neutron的数据库中保存了网络中的所有数据，在虚拟机启动过程中，其端口会从Down到Build再到Active，因此，每当端口状态发生改变时，都会触发L2 population的相关逻辑。
  
 在没有ARP Responder之前L2 population如何工作？

 如图，分别在两台宿主机上的两台虚拟机进行通信：

 ![two_node][2]

 根据刚才对于ARP的介绍，如果VM B从来没有和VM A通信过，当VM A要和VM B通信时，Node A并不知道VM B的MAC地址是什么，
此时会将广播请求泛洪到所有Node A相连的Tunnel Ports上，当VM B回复了泛洪请求后，Node A会学习到一条关于VM B的流表，
流表的内容是VM B的Node Tunnel信息。通过这种方式，避免了以后的广播请求（因为以后的广播请求没有了洪泛过程，直接发到了相应的Node上）。
通过这种方式虽然我们优化了广播流量，但是ARP呢？。还是举上面的例子，Node A学习到了关于VM B的流表，
当VM A和VM B通信时，Node A将VM A的ARP请求发到了Node B上，但是Node B上可能不仅仅有VM B一个机器，Node B上依旧收到了多余的ARP请求。
因此ARP的问题依旧没有解决。


 在有了ARP Responder后，由于Neutron数据库中保存了网络中的所有数据，此时在Node C上创建VM C，那么，Node A和Node B上的OpenvSwitch Bridge br-tun
上会添加一条流表：VM C在Node C上，IP地址是x.x.x.x，MAC地址是y.y.y.y.y.y。这样一来，VM A，VM B和VM C通信时，br-tun作为ARP Responder回复各自的ARP请求。

###ARP Responder的使用场景和限制

 目前来说，DVR用到了ARP Responder和L2 population。但是目前仍然有一些问题：
 - 对消息队列的使用加重，增加了相关Topic的消费
 - 开启L2 population后计算/网络节点上的流表增多，一定程度上影响了数据平面
 - 增加了网络运维的难度

###如何开启ARP Responder和L2 population

 在Neutron Server节点修改plugin.ini：
 > [ml2]
 >
 > mechanism_drivers = ...,l2population,...

 在计算/网络节点修改plugins/ml2/openvswitch_agent.ini

 > [agent]
 >
 > l2_population = True
 > arp_responder = True


 [1]: ../../images/architecture/two-node-machine-communicate.png
 [2]: ../../images/architecture/three-node-machine-communicate.png
