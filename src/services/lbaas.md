## LBaaS

### 简介

LBaaS 是 Neutron 中的高级服务，使用户通过 Neutron 的 API 完成云平台的负载均衡需求。
在设计之初就考虑到了数据模型与Driver 实现的解耦， OpenStack Operator 能够指定不同的后端
(HAProxy, Octavia)，UnitedStack(R) UNP 通过和合作伙伴努力，还添加了
F5 BIG-IP (TM)的后端，此外 BIG-IP VE (TM) 版和 Citrix NetScaler(TM) 也正在开发中，
这样能够用统一的接口操作和控制不同的负载均衡设备。

### 架构

LBaaS v2 之后重构了数据模型，从以资源池（pool）为中心转变到以负载均衡器（loadbalancer）
为中心，数据模型如下图：

![lbaas-architecture][1]

### 负载均衡服务

#### 功能列表

| Feature | Description |
|:------- |:----------- |
| Monitors | LBaaS 提供 ping, TCP, HTTP 三种方式的健康检查机制 |
| Management | LBaaS 提供一组 REST API 让用户能够动态的管理负载均衡资源 |
| Connection limits | LBaaS 通过控制接受连接的数量，保护后端的 Server |
| L7 Policy | LBaaS 提供针对七层服务的字段执行一组 action |
| Session persistence | LBaaS 通过Source IP 或 Cookie 的方式来提供 L7 的会话保持 |


#### 负载算法

Loadbalancer 需要确保负载能够被正确的分配到一组后端 Server 上，更好的利用系统资源。
Loadbalancer 接收到的请求会以以下三种方式中的一种进行请求分发：

 * Round robin: 以轮询的方式在一组后端 Server 间分发负载

 * Source IP: 来自同一个源 IP 的请求总是被分发到同一个后端 Server 

 * Least connections: 将请求分发到当前保持的连接数最少的后端 Server


### 服务部署

详细的部署架构参考部署环节和 unp-config 相关配置

#### Neutron API server

需要在 API 节点的 neutron-server 中加载 lbaasv2 的服务

```
# neutron.conf

service_plugins = exist plugin, neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2

# neutron_lbaas.conf

service_provider=LOADBALANCERV2:Haproxy:neutron_lbaas.drivers.haproxy.plugin_driver.HaproxyOnHostPluginDriver:default
```

默认的 Driver 是 HAProxy，基于社区传统的 namespace，是 Liberty Release 最可用的
开源方案。这种方案的优点是完全开源软件实现，不依赖外部特定的硬件或 Service VM，缺点是隔离性不好，
只隔离了单独的网络协议栈，没有对其在网络节点上的资源占用进行隔离；其次是性能一般，跟
硬件方案或 Service VM 方案有一定差距。关于高性能负载均衡设备，请参考[生态](../ecosystem)
相关章节

如果原来没有 neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2 这个 service_plugin 的话，修改后需要升级数据库：

```
neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head
```

#### Neutron LBaaSv2 Agent

如果选用 HAProxy 作为 back-end，则需要在网络节点上部署和启动 neutron-lbaasv2-agent；
如果选用的是其他的 back-end(F5, Octavia 等)，则可能不需要部署 neutron-lbaasv2-agent，
根据 back-end 的文档做具体调整

```
# lbaas_agent.ini

device_driver = neutron_lbaas.drivers.haproxy.namespace_driver.HaproxyNSDriver
```

注意需要启动的是 v2 的 agent，如果出现了 lbaas-agent 需要关掉。


[1]: ../../images/services/lbaas-architecture.png
