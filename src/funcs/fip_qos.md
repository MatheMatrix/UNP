### FloatingIP QoS

---

#### 简介

目前 OpenStack Neutron 所实现的 QoS 功能只针对挂载在 OpenvSwitch Bridge 上的虚拟网卡
进行的限速，最终通过调用如下的 OpenvSwitch 命令实现 QoS 的功能：

```
ovs-vsctl set interface tap0 ingress_policing_rate=1000
ovs-vsctl set interface tap0 ingress_policing_burst=100
```

本文主要说明针对 FloatingIP 进行的 QoS。

默认的，对于新创建的 FloatingIP 限速是 1024 Kb (1Mbit)，对于虚拟路由器的 SNAT 来说，
默认限速是 5Mb，即：虚拟路由器 SNAT 流量的共享带宽是 5Mb。

#### 实现原理

通过 Linux TC (Traffic Control) 实现针对 FloatingIP 的 QoS。

通常，要对网卡进行流量控制的配置，需要进行如下的步骤：

 - 为网卡配置一个队列；
 - 在该队列上建立分类；
 - 根据需要建立子队列和子分类；
 - 为每个分类建立过滤器。

针对 FloatingIP 的 QoS，使用 TC 中的 HTB(Hierarchical Token Bucket) 队列，与其他复杂的队列
类型相比，HTB 具有功能强大，配置简单等优点。在 TC 中，使用 `major:minor` 这样的句柄来表示队列
和类别，其中 `major` 和 `minor` 都是数字。

在 TC 中使用下列的缩写表示相应的带宽:

 - Kbps : kilobytes per second，千字节每秒 ;
 - Mbps : megabytes per second，兆字节每秒 ;
 - Kbit : kilobits per second，千比特每秒 ;
 - Mbit : megabits per second， 兆比特每秒 。

流量的处理由三种对象控制，分别是：qdisc，class，filter

##### QDISC (队列)
QDisc 是理解流量控制的基础。无论何时，内核如果需要通过某个网络接口发送数据包，

它都需要按照为这个接口配置的 qdisc 把数据包加入队列。然后内核会尽可能多得从 qdisc 里面取出数据包，
把它们交给网络适配器驱动模块。
##### CLASS (策略)
class 用来表示控制策略，很多时候，可能要对不通的 IP 实行不同的流量控制策略，
这时候我们就要用不同的 class 来表示不同的控制策略了

##### FILTER (过滤器)
filter 用来将用户划入到具体的控制策略中（即不同的class中），比如我们要对 a 和 b 两个 IP 实行不同的控制策略 A 和 B ，
这时我们可以用 filter 将 a 划入控制策略 A，将 b 划入控制策略 B， filter 划分的标志位可用 u32 打标功能或者 iptables 的 set-mark 功能来实现。

#### 实现流程图

![process][1]

流程图说明：
 1. qg 设备代表桥接在外部网桥上的虚拟网路设备，qr 设备代表虚拟机的默认网关；
 2. 红线代表数据流进入虚拟机时的流量走向；
 3. 蓝线代表数据流离开虚拟机时的流量走向；
 4. ifb 设备用于 qg 设备流量的重定向；
 5. qg 设备提供 ingress redirect 和 egress ratelimit 的功能，ifb 设备只提供 ratelimit 的功能。


[1]: ../../images/funcs/tc.png
