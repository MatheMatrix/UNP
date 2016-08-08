# 系统

---

UnitedStack<sup>®</sup> UNP 在系统和运行环境方面做了大量优化和调研，以保证软件栈能够在一个稳定、高性能的环境下运行，具体的优化方向包括：

 - 消息队列 RabbitMQ 3.3.5
 - 系统内核 Linux Kernel 3.10.0-327.13.1.el7.x86_64
 - 网卡驱动 ixgbe 4.0.1
 - 一系列内核模块 nf_conntrack、nf_nat
 - CPU 软中断
 - vhost-net
 - BIOS

此外还有一些其他的优化内容，我们把不仅影响网络，还影响其他软件运行的部分放置在系统一节，基本针对网络的内容（例如 DPDK、Open vSwitch 等）会放在性能一节。

将来关于性能还会继续优化，特别是关于 vHost-user、ivshmem 等和虚拟化相关的技术。

稳定性方面在调研 RPC 与 Notification 消息队列的分离与向分布式消息队列如 ZeroMQ 等。