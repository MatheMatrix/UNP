## 网络相关优化

### UnitedStack 知识库相关文章

 - [网络节点优化参数汇总](https://confluence.ustack.com/pages/viewpage.action?pageId=16091446)
 - [检查网卡多队列绑定状态](https://confluence.ustack.com/pages/viewpage.action?pageId=12782706)
 - [网络节点负载高情景一 ，conntrack问题]( https://confluence.ustack.com/pages/viewpage.action?pageId=9641517)

### CPU 亲和性绑定

CPU的亲和性， 就是进程要尽量在指定的 CPU 上尽量长时间地运行而不被调度到其他处理器。
在多核运行的机器上，每个CPU本身自己会有缓存，缓存着进程使用的信息，进程可能会被操作系统调度到其他CPU上，
导致 CPU cache命中率就低了，当进行 CPU 亲和性绑定后，程序就会一直在指定的 CPU 运行，
不会被操作系统调度到其他 CPU 上在一定程度上能提高性能。

网卡队列跟 CPU 绑定是将各个队列通过中断绑定到不同的核上，以满足网卡的需求。同时也可以降低单个CPU的负载，提升系统的吞吐能力。

配置网卡多队列中断绑定的脚本在 UnitedStack GIT 仓库中 maintenance 项目内

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

### UDP flow hash算法

网卡驱动支持的哈希算法，能够把同一条流的数据包哈希到同一个队列中。在上一步的操作中，已经实现了队列
跟 CPU 的亲和性，此时就能做到数据流跟 CPU 的亲和性。在云平台中开启该参数主要是作用于 UDP，
为了提高 VxLAN 的性能。

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

在宿主机上调整链接跟踪库允许的最大值，可以调整为12582912，

```
sysctl -w net.ipv4.netfilter.ip_conntrack_max=12582912
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
options nf_conntrack hashsize=1572864
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

考虑到 ovs-vswitchd 可能被重启等问题，建议在系统层面设置文件描述符数量：

```
vi /etc/security/limits.conf
* hard nofile 102400
* soft nofile 102400
```

### netdev_max_backlog 等内核参数

在 ovs datapath 复制报文的过程中，如果是网络节点，可能会有上千个 port。当在 br-ex 或类似的场景发送 ARP 报文时，ovs 的 normal 动作会对此进行遍历处理，这个处理过程中不会进行 CPU 释放的操作，也即将持续调用 enqueue_to_backlog（openvswitch datapath最初调用的是netif_rx，但最终调到enqueue_to_backlog），因此默认的 netdev_max_backlog 可能会不足够使用，因此需要调整 netdev_max_backlog。

此外，还有其他通用网络调优实践，并列在此：

```
/etc/sysctl.conf
net.core.netdev_max_backlog = 250000
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.core.rmem_default = 600000
net.core.wmem_default = 600000
```

### ASLR

一些优化建议会建议关闭 ASLR 以提升性能，根据 2014 发表在 Proceedings of the International Workshop on Reproducible Research Methodologies 的 *How Much Does Memory Layout Impact Performance? A Wide Study* 显示并没有明显变化，但也有一些实验数据表示 ASLR 会对性能产生不利影响，特别是可能造成性能抖动，我们默认不会关闭 ASLR，但在此提供关闭方法：

```
/etc/sysctl.conf
kernel.randomize_va_space = 0
```

### 网卡其他配置

#### 虚拟网卡 RSS

虚拟网卡可以通过 RSS 显著提高性能，特别在虚拟路由器上的虚拟网卡应当全部开启，参考 UnitedStack 知识库相关文章

#### Ring Buffer

如果出现网卡层面丢包，可以考虑调高网卡的 Ring Buffer，例如：

```
ethtool -G ethX rx 2048
```

这样的设置是临时生效的，如果机器发生重启将会覆盖配置，因此我们建议通过配置文件设置：
 
```
/etc/sysconfig/network-scripts/ifcfg-ethX
...
ETHTOOL_OPTS="-G ethX rx 2048"
```

注意这有可能会在一定程度上增大网络延时。但一般来说，在 latency 本来就比较小的情况下，适当的增大 buffer 并不会带来明显的影响，除非是一些对 latency 极度敏感的交易场所、流媒体等。

#### 中断聚合

如果系统中断过高，可以考虑通过中断聚合优化，对于 Intel 82599、Intel X540 网卡，经实测只有 rx-usecs 是有效的。

```
ethtool -C ethX rx-usecs 50
```

配置的固化同上。

#### Offloading

对于桥接、路由场景，特别是 Overlay 网络，offloading 特性并不建议开启，这是 Intel ixgbe 驱动中有明确建议的。

 > The ixgbe driver compiles by default with the LRO (Large Receive
Offload) feature enabled.  This option offers the lowest CPU utilization for
receives, but is completely incompatible with *routing/ip forwarding* and
*bridging*.  If enabling ip forwarding or bridging is a requirement, it is
necessary to disable LRO using compile time options as noted in the LRO
section later in this document.  The result of not disabling LRO when combined
with ip forwarding or bridging can be low throughput or even a kernel panic.

 > Due to a known general compatibility issue with LRO and routing, do not use
  LRO when routing or bridging packets.
  
 可以使用 `ethtool -k ethX lro=off` 关闭，但是这样的关闭是临时生效的，如果机器发生重启将会覆盖配置，因此我们建议通过配置文件关闭：
 
 ```
/etc/sysconfig/network-scripts/ifcfg-ethX
...
ETHTOOL_OPTS="-K ethX gso off gro off lro off"
 ```
 
 注意这个选项因为是针对 ixgbe 驱动的，所以只在物理网卡设置有效，虚拟网卡是不需要的。

### 虚拟网卡多队列

virtio-net 针对每个虚拟网卡只有一个 tx/rx 队列，对于有较高虚拟网络性能的情况可能希望虚拟机能够充分使用多核与多队列，需要在 Nova 启动虚拟机时对其 XML 文件变化：

```
<interface type='network'>
      <source network='default'/>
      <model type='virtio'/>
      <driver name='vhost' queues='N'/>
</interface>
```

其中 N 为网卡队列数量，此外修改网卡驱动的参数：

```
ethtool -L eth0 combined M
```

M 应当为 1~N 之间，对性能会有较为明显的影响。

### BIOS 推荐配置

特别是在高性能需求下，Intel CPU 的 BIOS 建议做如下配置：

| BIOS 选项 | 推荐值 |
| -------- | ------ |
| Operating Mode /Power profile | Maximum Performance |
| C-States | Disabled |
| Turbo mode | Enabled |
| Hyper-Threading | Enabled |
| IO non posted prefetching | Enabled |
| CPU frequency select | Max performance |
| Memory speed | Max performance |
| Memory channel mode | Independent |
| Node Interleaving | Disabled / NUMA |
| Channel Interleaving | Enabled |
| Thermal Mode | Performance |
￼

## 已知系统问题

 * 内核 network namespace 性能退化，受影响的内核版本 3.8 ～3.19，在 namespace 数量大(1k 的数量级)时，对每个 namespace 的
 创建和配置操作速率会下降数倍，并且在多核系统中影响会更大。详见 https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1328088

实验方法是通过一些对 namespace 的操作，封装成脚本的 transaction 来模拟虚拟路由器的创建和执行，考察不同核心数量和 namespace 数量下的执行速度：

250 个 namespace：

![250][2]

1250 个 namespace：

![1250][3]

2250 个 namespace：

![2250][4]

### 参考文档

 * Exploration of Large Scale Virtual Networks,  http://events.linuxfoundation.org/sites/events/files/slides/Scaling_1.pdf
 * Linux* Base Driver for the Intel(R) Ethernet 10 Gigabit PCI Express Family of
Adapters, https://downloadmirror.intel.com/22919/eng/README.txt 
 * How Much Does Memory Layout Impact Performance? A Wide Study, https://uwaterloo.ca/embedded-software-group/sites/ca.embedded-software-group/files/uploads/files/hpca-datamill.pdf

[1]: ../../images/system/conntrack_hashtable.png
[2]: ../../images/system/QQ20160606-1.png
[3]: ../../images/system/QQ20160606-2.png
[4]: ../../images/system/QQ20160606-3.png
