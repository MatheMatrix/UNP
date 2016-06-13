## ARP Responder

---

###简介

#### UnitedStack 知识库相关文章

 - [Neutron中各个OVS网桥的作用以及流表规则](https://confluence.ustack.com/pages/viewpage.action?pageId=12063326)
 - [网络软件优化方案之DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR —Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)

#### ARP

地址解析协议（Address Resolution Protocol），其基本功能为透过目标设备的 IP 地址，查询目标设备的 MAC 地址，以保证通信的顺利进行。所谓地址解析（address resolution）就是主机在发送帧前将目标 IP 地址转换成目标 MAC 地址的过程。

 举个例子，如图的两个虚拟机进行通信

 ![two_machines][1]

 当在 Machine A 上执行ping 192.168.10.20时，如果 Machine A 从来没有和 Machine B 通信过，也就不会知道 Machine B 的 MAC 地址是什么。如何知道 Machine B 的 MAC 地址？答案是洪泛。当 Machine B 上收到了泛洪包后，发现 ARP 请求的地址是自己，进行 ARP 的回复，*注意：此时回复的是单播*。此时，Machine A 知道了 Machine B 的 MAC 地址后，最后在发送 ICMP 的请求报文。

### ARP Proxy

Proxy ARP 就是通过一个主机（通常是Router）来作为指定的设备对另一个设备作出 ARP 的请求进行应答。

 简单的说，当机器发送 ARP 请求时，ARP Proxy 捕捉到此请求，并进行回复。

### ARP Responder
Neutron 通过 OpenvSwitch Bridge Br-tun 实现了 ARP Responder 的功能。由于 Neutron 数据库中存有所有 Port 的信息（IP，MAC信息），
因此可以将这些消息分发到各个 OVS agent 上，实现 ARP Responder 的功能。
ARP Responder 的一条典型流表规则如下：
```
 cookie=0x89509acbf7d4624a, duration=62261.842s, table=21, n_packets=0,
 n_bytes=0, idle_age=62261,priority=1,arp,dl_vlan=21,arp_tpa=10.10.10.18
 actions=move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[],
 mod_dl_src:fa:16:3e:9d:22:99,
 load:0x2->NXM_OF_ARP_OP[],
 move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[],
 move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[],
 load:0xfa163e9d2299->NXM_NX_ARP_SHA[],
 load:0xa0a0a12->NXM_OF_ARP_SPA[],IN_PORT
```
其中，各个字段的意义如下：
 - table=21,priority=1,arp,dl_vlan=1,arp_tpa=10.10.10.18
      当收到了 10.10.10.18 的 ARP 请求
 - move:NXM_OF_ETH_SRC[]->NXM_OF_ETH_DST[]
      将源MAC地址放到目的MAC地址里
 - mod_dl_src:fa:16:3e:9d:22:99
      更改源 MAC 为 10.10.10.18 的 MAC 地址
 - load:0x2->NXM_OF_ARP_OP[]：
      ARP 回复
 - move:NXM_NX_ARP_SHA[]->NXM_NX_ARP_THA[]
      将 ARP 源 MAC 地址放到 ARP 的目的 MAC 地址
 - move:NXM_OF_ARP_SPA[]->NXM_OF_ARP_TPA[]
      将 ARP 源 IP 地址放到 ARP 的目的 IP 地址
 - load:0xfa163e9d2299->NXM_NX_ARP_SHA[]
      ARP 源 MAC 填充成 10.10.10.18 的MAC地址（0xfa163e9d2299）
 - load:0xa0a0a12->NXM_OF_ARP_SPA[]
     ARP 源 IP 填充成 10.10.10.18 的IP地址（0xa0a0a12）
 - IN_PORT
      从哪里来，回哪里去
 

###ARP Responder 的使用场景和限制
通过 ARP Responder，Neutron 将虚拟机的 ARP 请求抑制在本计算节点，从而减少 ARP 广播。

 目前来说，ARP Responder 目前仍然有一些问题：
 - 对消息队列的使用加重，增加了相关 Topic 的消费
 - 开启 ARP Responder 后计算节点上的流表增多，一定程度上影响了数据平面
 - 增加了网络运维的难度

### L2 Population

早期的 Neutron OVS agent 需要和每个 VXLAN 之间都建立 VXLAN 隧道，当遇到未知广播，多播或者单播时，
如果本节点的 VTEP 上没有 cache，这条流就会广播到所有 VTEP，
这样造成严重的网络效率的下降，因此，Neutron 提供了 L2 population 提升网络效率。
一条典型的 L2 population 流表如下：
```
table=20, n_packets=0, n_bytes=0, idle_age=514, priority=2,dl_vlan=2,
dl_dst=fa:16:3e:09:f7:f6,actions=strip_vlan,set_tunnel:0x1d,output:5
```
但是 L2 Population 问题也很明显：
 - 云资源管理平面强制影响了数据平面；
 - 将网络的广播一定程度上转成了消息队列的广播，加重了消息队列的负载。

###开启ARP Responder 和 L2 population

 在 Neutron Server 节点修改 plugin.ini：
```
[ml2]
mechanism_drivers = ...,l2population,...
```

在计算/网络节点修改 plugins/ml2/openvswitch_agent.ini

```
[agent]

l2_population = True
arp_responder = True
```

 [1]: ../../images/architecture/two-node-machine-communicate.png
 [2]: ../../images/architecture/three-node-machine-communicate.png
