## 部署 Provider Network

----

### 简介

在 OpenStack 的网络模型中，有两种网络类型：Tenant Network 和 Provider Network。

上述这两种网络类型分别对应 UnitedStack<sup>®</sup> United Networking Platform(UNP) 中的自服务网络和与配置网络，可参考 [VXLAN + VLAN 一节](./vxlan_vlan.md)。

 - Tenant Network 

   Tenant Network 支持的网络类型包括：Flat， VLAN 和 Tunnel Network(Gre, VXLAN 等)。Tenant Network
由租户本身创建和管理，并且 Tenant Network 之间相互隔离。

 - Provider Network

   Provider Network 支持的网络类型包括：Flat 和 VLAN (Tunnel Network 由于配置较复杂，不建议使用)。
Provider Network 有管理员创建和管理，因此隔离性体现在不同的 Provider Network 之间。

### Provider Network 拓扑结构

  Provider Network 类似于部署中的 VLAN 网络预配置模型，不同的是，部署 Provider Network 时无需网络节点，
VLAN 网络网关设置在计算节点上联的物理设备上。其网络拓扑可以如下图表示：

![Provider Network][1]

[1]: ../../images/deployment/provider_network.png

需要注意的是，在 Neutron 中预配置的 VLAN 范围一定要在计算节点物理网卡上联的物理设备进行配置(Trunk)。
如有三层网络的需求（通公网、通其他内网），需在上联网络设备上设置相应的 VLAN 网关。

### 创建 Provider Network 示例
 
 举例说明在 Neutron 中创建 VLAN ID 为 101 的网络：

```
neutron net-create vlan101 \
--provider:physical_network physnet3  \
--provider:network_type vlan \
--provider:segmentation_id=101
```

注：上述命令中的 `physnet0` 和 `segmentation_id` 请参考 [Neutron Server 配置](./neutron_conf/neutron_server.md) 一节。
