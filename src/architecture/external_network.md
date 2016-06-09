## External Network

---

### External Network 的定义
 
 External Network 是 UnitedStack<sup>®</sup> UNP 提供的用于企业内网互联或外部网络连接用的网络。租户可以通过在 External Network 中创建
Floating IP，将其绑定到虚拟设备（如虚拟机、负载均衡器等）上，从而只需访问 FloatingIP 即可访问租户的虚拟设备。通过在虚拟路由器上开启网关并开启 SNAT（默认开启）即可多个虚拟机共用 FloatingIP 访问外部网络。
Neutron 允许创建多个 External Networks。

### External Network 的工作原理

 External Network 中的 IP 地址通过 [NAT](https://en.wikipedia.org/wiki/Network_address_translation)
的形式和虚拟 IP 建立一对一（FloatingIP）或者一对多（SNAT）的映射关系，从而对外提供服务。

### 不同类型的 External Networks

 External Network 的典型创建方式分为以下两种：

#### Local 类型的 External Network

```
[root@server-233 ~(keystone_admin)]# neutron net-create --provider:network_type=local \
--router:external=True public
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | d9d2fc93-de86-47fa-86d4-7374c1488263 |
| mtu                       | 0                                    |
| name                      | public                               |
| provider:network_type     | local                                |
| provider:physical_network |                                      |
| provider:segmentation_id  |                                      |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | ef979882f1954a0fa4ce7daf244aa557     |
+---------------------------+--------------------------------------+
```

#### VLAN 类型的 External Network
 创建 VLAN 类型的 External Network 和 Neutron 创建 VLAN 网络类似，只需指定某些特定的参数即可，比如：

```
[root@server-233 ~(keystone_admin)]# neutron net-create --provider:network_type=vlan \
--router:external=True \
--provider:segmentation_id=5 \
--provider:physical_network=physnet3 \
public

Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | f3e296b8-073c-4a49-9242-2dfaf9d889b5 |
| mtu                       | 0                                    |
| name                      | public                               |
| provider:network_type     | vlan                                 |
| provider:physical_network | physnet3                             |
| provider:segmentation_id  | 5                                    |
| router:external           | True                                 |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | ef979882f1954a0fa4ce7daf244aa557     |
+---------------------------+--------------------------------------+
```

### External Network 的工作原理

#### Local 类型的 External Network 工作原理

拓扑结构如下：

 ![local_external_network][1]

对于 Local 类型的 External Network，在计算节点和网络节点上有专门的外网网桥。
所有的外部网关设备（qg设备，fg设备）都会挂载到 OpenvSwitch 网桥 br-ex 上。
通过将外部网卡（ethx）桥接在 br-ex 上，使得虚拟设备可对外访问，或外部设备访问虚拟设备。


#### VLAN 类型的 External Network 工作原理

拓扑结构如下：

 ![vlan_external_network][2]

和 Local 类型的 External Network 相比，VLAN 类型的 External Network 最大的区别就是
没有专门的外网网桥（上例中的 br-ex）。所有的外部网关设备（qg设备，fg设备）都挂载到 OpenvSwitch 网桥 br-vlan 上。
本拓扑中的 OpenvSiwtch 网桥 br-vlan 既可作为SDN 网络流量的网桥，也可作为外部网桥。
将 SDN 网络流量和外部网络流量公用一个 OpenvSwitch
网桥最大的好处就是节约了网卡和相关物理资源。


#### 比较两种类型的 External Network

|| OVS 网桥 |外部网络网卡| 交换机端口配置 |物理网卡速率|虚拟网关所在网桥|
|:-:|:-:|:-:|:-:|:-:|:-:|
|Local 类型|单独的 OVS 网桥 br-ex|单独的外网网卡|Access|默认千兆|br-ex|
|VLAN 类型|和 VLAN 网桥共用 OVS 网桥 ovsbr3|共用的 VLAN 网络网卡|Trunk|万兆|ovsbr3|

因此，我们推荐的是基于 br-vlan 这种模式的外部网络。这种模式的优点是和 SDN 网络共用万兆网卡
从而节约网络资源、方便网卡多队列性能的提升、并且节省硬件网络成本。

### 分布式路由

UnitedStack<sup>®</sup> UNP 默认提供分布式的路由，因此通过 FloatingIP 访问外部网络是分布式的，但对于 1:N NAT，即多个虚拟机通过开启网关的路由器共享一个地址访问仍然是集中式的。


 有关 External Network 的具体配置和部署过程请参考[部署 - External Network](../deployment/external_network.md)。

[1]: ../../images/architecture/local_external_network.png
[2]: ../../images/architecture/vlan_external_network.png
