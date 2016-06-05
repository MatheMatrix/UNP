# 网络调优

## 简介

### CPU 亲和性绑定

CPU的亲和性， 就是进程要尽量在指定的 CPU 上尽量长时间地运行而不被调度到其他处理器。
在多核运行的机器上，每个CPU本身自己会有缓存，缓存着进程使用的信息，进程可能会被操作系统调度到其他CPU上，
导致 CPU cache命中率就低了，当进行 CPU 亲和性绑定后，程序就会一直在指定的 CPU 运行，
不会被操作系统调度到其他 CPU 上在一定程度上能提高性能。

网卡队列跟 CPU 绑定是将各个队列通过中断绑定到不同的核上，以满足网卡的需求。同时也可以降低单个CPU的负载，提升系统的吞吐能力。

#### 配置方法

配置网卡多队列中断绑定的脚本在 maintenance 中

```
maintenance/maintain/neutron/irqbalance
```

该目录下有多个 shell 脚本，分别有不同的作用。

检查某个网卡的队列绑定情况，以 eth3 为例

```
./check_cpu_affinity.sh eth3
```

设置某个网卡的多队列绑定，以 eth3 为例

```
./set_irq_affinity.sh eth3
```

### flow hash算法

网卡驱动支持的哈希算法，能够把同一条流的数据包哈希到同一个队列中。在上一步的操作中，已经实现了队列
跟 CPU 的亲和性，此时就能做到数据流跟 CPU 的亲和性。在云平台中开启该参数主要是作用于 UDP，
为了提高 VxLAN 的性能。

#### 配置方法

设置某个网卡的 flow 哈希，以 eth3 为例

```
ethtool -N eth3 rx-flow-hash udp4 sdnf
```

### conntrack 内核参数优化

内核中的 netfilter 模块为会纪录经过内核的每一个连接的状态，用来做带状态的防火墙。在 Neutron 中，
conntrack 在两个场景其着关键性作用：

 * security group，带状态的访问控制规则。跟 ACL 不同，Neutron 中的 security group 不需要双向放行,
 conntrack 会纪录该连接的状态，匹配该连接的每个数据包，自动允许该连接双向通信

 * SNAT，路由器开启公网网关之后，与该路由器关联的子网中的虚拟机都能够通过该网关访问公网，在这个过程中
conntrack 会纪录每个连接，在外部回包时就能够 DNAT 到正确的虚拟机上

内核中 conntrack table 实现模型：

![conntrack_hashtable][1]

由 conntrack 表的实现可以得知，conntrack 表是根据哈希算法来查找 tuple 表项，时间复杂度为 O(1),
到前面固定大小的hash table 满了，就会用链地址法在冲突的 conntrack 放在hastable 邻接的链表中，
后续的查找会退化成线性查找，时间复杂度退化为 O(n)，造成包处理流程很长。

#### 配置方法

针对上述问题，由两个优化方向：

 *  增加 hash table 的长度
 
 冲突的概率减小，尽量保持哈希算法直接查找。调整hashsize，一般调整为 conntrack_max/4

```
/sys/module/nf_conntrack/parameters/hashsize
```
 * 减小有效 conntrack count 的数量，需要在 router 的 namespace  中调整

TCP链接established状态的超时时间，同理，还有wait/close/last_ack等的超时时间设定，可以调整为600

```
sysctl -w net.ipv4.netfilter.ip_conntrack_tcp_timeout_established=600
```

generic_timeout可以调整为120

```
sysctl -w net.ipv4.netfilter.ip_conntrack_generic_timeout=120
```

在宿主机上调整链接跟踪库允许的最大值，可以调整为393216，

```
sysctl -w net.ipv4.netfilter.ip_conntrack_max=393216
```

在极端情况下，router 的 namespace 中发生丢包时，查看可能查看系统 message 日志没有类似 conntrack 表满的警告信息，
可以把以下参数的值稍微减小 1/3

```
net.ipv4.netfilter.ip_conntrack_icmp_timeout
net.ipv4.netfilter.ip_conntrack_udp_timeout_stream
net.ipv4.netfilter.ip_conntrack_udp_timeout
net.ipv4.netfilter.ip_conntrack_tcp_timeout_close
net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait
net.ipv4.netfilter.ip_conntrack_tcp_timeout_last_ack
net.ipv4.netfilter.ip_conntrack_tcp_timeout_close_wait
net.ipv4.netfilter.ip_conntrack_tcp_timeout_fin_wait
net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_recv
net.ipv4.netfilter.ip_conntrack_tcp_timeout_syn_sent
```
在新部署网络节点是，需要在系统中调整 hashsize，可以写在 /etc/modprobe.conf 文件中，防止系统重启是配置参数丢失，例如：

```
options nf_conntrack hashsize=98304
```

### ovs-vswitchd 文件描述符数目优化

ovs-vswitchd 会创建很多 netlink socket 跟内核通信，每一个 netlink socket 需要一个文件描述符。这个文件描述符的数量，初始是 112 K，
至少要是

```
Number of cpus * Number of switch ports
```

可以预先在 openvswitch 启动之后给其设置一个较大的值，例如设置为 200K：

```
prlimit -p `cat /var/run/openvswitch/ovs-vswitchd.pid` --nofile=200000
```


## 已知系统问题

 * 内核 network namespace 性能退化，受影响的内核版本 3.8 ～3.19，在 namespace 数量大(1k 的数量级)时，对每个 namespace 的
 创建和配置操作速率会下降数倍，并且在多核系统中影响会更大。见 https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1328088


## 参考文档
 * 网络节点优化参数汇总 https://confluence.ustack.com/pages/viewpage.action?pageId=16091446
 * 检查网卡多队列绑定状态 https://confluence.ustack.com/pages/viewpage.action?pageId=12782706
 * 网络节点负载高情景一 ，conntrack问题 https://confluence.ustack.com/pages/viewpage.action?pageId=9641517
 * Exploration of Large Scale Virtual Networks,  http://events.linuxfoundation.org/sites/events/files/slides/Scaling_1.pdf

[1]: ../../images/system/conntrack_hashtable.png
