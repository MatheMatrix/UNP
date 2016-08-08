### 多个外部网络的使用

---

#### 简介

 Neutron 中的外部网络提供了虚拟网络设备通过 NAT / SNAT 访问外网的能力，这里说的`外网`既可以是公网，
也可以是企业内网，外部网络的配置完全依靠客户的需要。

 Neutron 中的外部网络大概有两点功能：
1. FloatingIP 池。在该网络中创建 FloatingIP，供虚拟网卡绑定使用；
2. 外部网关池。供虚拟路由器开启外网网关使用。


 Neutron 中的 network 通过 `--router:external` 属性表示该 network 是否
为`外部网络`，使用如下命令创建一个外部网络：`neutron net-create external_network --router:external=True`
通过如下命令查看所有的外部网络：`neutron net-list --router:external=True`。
在建议的部署外部网络模型中，创建的外部网络是 VLAN 网络，创建方法和创建普通 VLAN 网络相同，
只需添加参数 `--router:external=True` 即可即类似于 `neutron net-create external_network --router:external=True --provider:network_type vlan --provider:physical_network physnet3 --provider:segmentation_id 1002`。
 
 对于部署多个外部网络时，需要将所有运行 L3 Agent 节点上的 l3_agent.ini 文件中的 `gateway_external_network_id` 
和 `external_network_bridge` 两个配置都配置成空，即：
```
external_network_bridge=
gateway_external_network_id=
```
 同时需要注意的是，在使用了多个外部网络的环境里绑定 FloatingIP 时，一定要注意路由器开启了外网网关的外部网络
和要绑定的 FloatingIP 是在同一个外部网络中，否则 Neutron 网络不可达的异常，
异常信息如下：`External network [network-id] is not reachable from subnet [subnet-id].
Therefore, cannot associate Port [port-id] with a Floating IP.`

#### 拓扑结构
 
 外部网络在 Neutron 中的实现可以分为两种模式：

1. 单独的外部网络网卡（一个外部网络）
2. 外部网络网卡和业务网卡共用（多个外部网络）

对于第一种模式，拓扑结构如下：

![external_network][1]


对于第二种模式，拓扑结构如下：

![external_network][2]


上述两种外部网络的部署模式对于硬件交换机上的端口配置不同的是：第一种配置模式中，eth1 配置的 Access
模式；第二种配置模式中，eth3 需要 Trunk 相关外部网络的 VLAN ID ，因为转换成外部 VLAN 是通过 OVS 流表
实现的，而不是通过创建物理网卡子接口的形式实现。

需要特别注意的是，如果部署了多个外部网络，网络节点或者计算节点上一定不能有 VLAN 子接口。

[1]: ../../images/deployment/external.png
[2]: ../../images/deployment/external2.png
