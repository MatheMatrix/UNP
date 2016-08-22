## 部署 Provider Network

----

### 简介

在 OpenStack 的网络模型中，有两种网络类型：Tenant Network 和 Provider Network。

上述这两种网络类型分别对应 United Networking Platform(UNP) 中的自服务网络和与预配置网络，
关于自服务网络可参考 [VXLAN + VLAN 自服务模型](./vxlan_vlan.md)。

 - Tenant Network 

   Tenant Network 支持的网络类型包括：Flat， VLAN 和 Tunnel Network(Gre, VXLAN 等)。Tenant Network
由租户本身创建和管理，并且 Tenant Network 之间相互隔离。

 - Provider Network

   Provider Network 支持的网络类型包括：Flat 和 VLAN (Tunnel Network 由于配置较复杂，不建议使用)。
Provider Network 有管理员创建和管理，因此隔离性体现在不同的 Provider Network 之间。

### 配置 Provider Network

  配置 Provider Network 和配置自服务 VLAN 网络的大体相似，不同的是，自服务 VLAN 网络 需要网络节点（VLAN 网关在网络节点上），
配置 Provider Network 时不需要网络节点，各个 VLAN 网络的网关地址配置在物理设备上。
 
  在 Neutron 中配置 Provider Network 和自服务 VLAN 网络的配置相同，配置 `/etc/neutron/plugins/ml2/ml2_conf.ini` 文件，
比如，配置 Provider Network 的 VLAN 范围分为两段，分别是是 5~10 和 12 ~ 120，可以如下配置：

```
[ml2_type_vlan]
# (ListOpt) List of <physical_network>[:<vlan_min>:<vlan_max>] tuples
# specifying physical_network names usable for VLAN provider and
# tenant networks, as well as ranges of VLAN tags on each
# physical_network available for allocation as tenant networks.
#
# network_vlan_ranges =
network_vlan_ranges = physnet3:5:10,physnet4:12:120
# Example: network_vlan_ranges = physnet1:1000:2999,physnet2
```

创建 Provider Network 示例
 
 举例说明在 Neutron 中创建 VLAN ID 为 101 的网络：

```
neutron net-create vlan101 \
--provider:physical_network physnet3  \
--provider:network_type vlan \
--provider:segmentation_id=101
```

### Provider Network 拓扑结构

  Provider Network 类似于部署中的 VLAN 网络预配置模型，不同的是，部署 Provider Network 时无需网络节点，
VLAN 网络网关设置在计算节点上联的物理设备上。其网络拓扑可以如下图表示：

![Provider Network][1]

[1]: ../../images/deployment/provider_network.png

需要注意的是，在 Neutron 中预配置的 VLAN 范围一定要在计算节点物理网卡上联的物理设备进行配置(Trunk)。
如有三层网络的需求（通公网、通其他内网），需在上联网络设备上设置相应的 VLAN 网关。


