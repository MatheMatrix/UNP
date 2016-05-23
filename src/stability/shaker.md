## 数据平面测试

---

### UnitedStack知识库相关文章

 - [QA提升之Shaker](https://confluence.ustack.com/display/SDN/Shaker)

### 简介

  Shaker是一个由 Mirantis 提交，基于 HEAT 的开源分布式数据平面测试工具，
主要用来测试 OpenStack 数据平面的性能。它集成了各种 Linux 下常用的测试工具，
比如 iPerf，iPerf3，Netperf 等。

它的测试逻辑如图：

 ![shaker][1]

  由上图可以看出，Shaker 大体由两部分组成—— Shaker Server 和 Shaker Agent。
Shaker Agent 运行在预先配置好的镜像中。 Shaker Server 运行在控制节点上，
接收来自各个 Shaker Agent 的数据，Shaker Server 不会主动的连接各个 Shaker Agent，
这样做的好处是省去了给各个运行着 Shaker Agent 的虚拟机绑定 FloatingIP 的过程，
所有虚拟机连接的路由器只需开启公网网关，使得虚拟机可以和控制节点通信即可。

### 关于部署过程

  社区的标准做法，是通过执行 `shaker-image-builder` 下载使用到的基础镜像，并下载相应
的软件包，比如 iPerf，iPerf3，Netperf等，最后将镜像上传到 Glance 中，供创建虚拟机使用。
当通过 Shaker 进行性能测试时，通过 Heat 模板创建的虚拟机在虚拟机中启动 Shaker Agent 进程，
并执行相关测试命令，最后通过 ZeroMQ 统一汇报给 Shaker Server。

  在根据 Shaker 官方文档进行部署的时候，遇到了执行 `shaker-image-builder` 卡死的情况，通过观察
日志发现是通过 Nova 创建虚拟机时使用的 Glance 镜像为空导致陷入了创建虚拟机死循环的状况。
后续我们会将此情况和社区讨论，目前我们手动创建运行 Shaker 所用的镜像。

  我们以 Ubuntu 15.05 为模板，安装 Shaker 所需的所有软件包，指定以下命令运行 shaker-agent：
`python shaker-agent --agent-id [vm-hostname] --server-endpoint 2.2.44.233:9999`

关于指定的 server-endpoint， 我们在 Neutron 里创建外网IP段 2.2.0.0/16，
从而使得当虚拟机绑定了 FloatingIP 后，可以从管理网 ssh 进入虚拟机。我们将 IP 地址2.2.244.233
配置在运行 Shaker 命令节点的 br-ex（外部网桥）上，并且指定系统上一个空闲的端口（这里用的9999），运行
`shaker --server-endpoint 2.2.44.233:9999 --scenario [template-name.yaml] --report report.html -d --agent-loss-timeout 480 --agent-join-timeout 480`
启动 Shaker 测试。在测试的结果中，我们发现了 agent lost 的现象，我们会和社区沟通，询问是否
是我们的使用方法有问题。


  下面给出 Shaker 的一个测试模板和测试结果：

```
title: OpenStack L2

description:
  In this scenario Shaker launches pairs of instances in the same tenant
  network. Every instance is hosted on a separate compute node, all available
  compute nodes are utilized. The traffic goes within the tenant network
  (L2 domain).

deployment:
  template: l2.hot
  accommodation: [pair, single_room]

execution:
  progression: quadratic
  tests:
  -
    title: Download
    class: flent
    method: tcp_download
  -
    title: Upload
    class: flent
    method: tcp_upload
  -
    title: Bi-directional
    class: flent
    method: tcp_bidirectional
```

 ![shaker_result][2]

 完整的 Shaker 测试结果可以[参考这里](../../attachment/shaker.html)

 [1]: ../../images/stability/shaker.png
 [2]: ../../images/stability/shaker_result.png
