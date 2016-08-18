### Neutron 性能优化

------

#### 简介

  本文主要介绍 Neutron 方面的性能优化，为[优化](./optimization.md) 一节的补充，在阅读
本文之前，请先参考[优化](./optimization.md) 一节。


#### 测试环境

##### 测试工具

  本文进行性能测试的工具为 pktgen，关于 pktgen 工具的说明，可参考[pktgen](./pktgen.md)一节。

##### 服务器配置

```
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                32
On-line CPU(s) list:   0-31
Thread(s) per core:    2
Core(s) per socket:    8
座：                   2
NUMA 节点：            2
厂商 ID：              GenuineIntel
CPU 系列：             6
型号：                 63
型号名称：             Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz
步进：                 2
CPU MHz：              1204.031
BogoMIPS：             4804.28
虚拟化：               VT-x
L1d 缓存：             32K
L1i 缓存：             32K
L2 缓存：              256K
L3 缓存：              20480K
NUMA 节点0 CPU：       0,2,4,6,8,10,12,14,16,18,20,22,24,26,28,30
NUMA 节点1 CPU：       1,3,5,7,9,11,13,15,17,19,21,23,25,27,29,31
```

##### 网卡型号

Intel Corporation 82599ES 10-Gigabit SFI/SFP+ Network Connection

##### Neutron L3 Agent 工作模式

本次测试使用的虚拟路由器为集中式路由器，关于集中式路由器的数据包流程，
请参考[Neutron 网络拓扑](https://confluence.ustack.com/download/attachments/3047471/neutron%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91.png?version=1&modificationDate=1435761215873&api=v2) 一节。
  
##### 测试虚拟机配置

```
+----------------------------+----------+
| Property                   | Value    |
+----------------------------+----------+
| OS-FLV-DISABLED:disabled   | False    |
| OS-FLV-EXT-DATA:ephemeral  | 0        |
| disk                       | 80       |
| extra_specs                | {}       |
| id                         | 4        |
| name                       | m1.large |
| os-flavor-access:is_public | True     |
| ram                        | 8192     |
| rxtx_factor                | 1.0      |
| swap                       |          |
| vcpus                      | 4        |
+----------------------------+----------+
```

#### 调优过程

##### 关闭物理网卡智能限速

命令：

`ethtool -A em1 autoneg off rx off tx off`

References:
- [Autonegotiation](https://en.wikipedia.org/wiki/Autonegotiation)
- [Linux ethtool Examples to Manipulate Ethernet Card](http://www.thegeekstuff.com/2010/10/ethtool-command/)
- [Ethernet flow control](https://en.wikipedia.org/wiki/Ethernet_flow_control)

##### 设备网卡队列长度为 4096

命令：

`ethtool  -G em1 rx 4096 tx 4096`

References:
- [HOWTO for the linux packet generator](https://www.kernel.org/doc/Documentation/networking/pktgen.txt)


##### 开启虚拟网卡的 RPS(Receive Packet Steering)

修改虚拟路由器中的 qr、qg、和 虚拟机 tap 设备的 RPS，例如可进行如下修改:

```echo "ffff,ffffffff,ffffffff,ffffffff,ffffffff"> /sys/class/net/qg-0631b6eb-03/queues/rx-0/rps_cpus```

References:
- [Receive Packet Steering (RPS)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Performance_Tuning_Guide/network-rps.html)

##### 物理网卡中断绑定以及 XPS

有关物理网卡中断绑定以及 XPS 可参考[优化](./optimization.md) 一节。

##### 关闭irqbalance

rqbalance 用于优化中断分配，它会自动收集系统数据以分析使用模式，
并依据系统负载状况将工作状态置于 Performance mode 或 Power-save mode。

 - Performance mode :
   irqbalance 会将中断尽可能均匀地分发给各个 CPU core，以充分利用 CPU 多核，提升性能。

 - Power-save mode :
   irqbalance 会将中断集中分配给第一个 CPU，以保证其它空闲 CPU 的睡眠时间，降低能耗。
但实际中往往影响 CPU 的使用均衡，建议服务器环境中关闭。

通过命令 `systemctl stop irqbalance` 关闭 irqbalance。


##### 修改 nf_conntrack_max 为 1000W，hashsize为400w

命令：

`echo '10000000' > /proc/sys/net/netfilter/nf_conntrack_max`

`echo '4000000' > /sys/module/nf_conntrack/parameters/hashsize`


References:

- [Conntrack tuning](https://confluence.ustack.com/pages/viewpage.action?pageId=4372185)


##### 修改 UDP RSS 算法

UDP 采用 sport&dport 进行哈希。

命令：

`ethtool -N eth3 rx-flow-hash udp4 sdnf`

References：

 - [Scaling in the Linux Networking Stack](https://www.kernel.org/doc/Documentation/networking/scaling.txt)


##### 增加虚拟网卡 txqueuelen

 增加发送时最多缓存的数据包。

命令：

`echo 1000000 > /sys/devices/virtual/net/qvo33d6c75c-b8/tx_queue_len`
