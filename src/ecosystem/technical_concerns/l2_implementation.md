## 二层隔离网络实现

### 简介

#### UnitedStack 知识库相关文章

 - [ML2 插件](https://confluence.ustack.com/pages/viewpage.action?pageId=16108282)

#### 技术说明

二层隔离网络是 SDN 系统的核心部分，该考察点主要针对核心网络实现类集成。对于二层隔离网络的实现，我们希望能在集成时确定一下内容：

 - 集成插件。目前对 OpenStack Neutron 进行核心网络实现的替换往往是通过自行撰写插件（Plugin）或为 ML2 插件（社区提供的通用化插件）撰写驱动（Mechanism Driver）；
 - 支持的类型驱动。如果使用 ML2 插件，需要提供支持的类型驱动（Type Driver），目前常见的有 local,flat,vlan,gre,vxlan,geneve；
 - 推荐的 OpenStack Neutron 租户网络默认类型。例如在 ML2 插件下 Neutron 的配置 ml2_conf.ini 中 tenant_network_types 的配置项推荐；
 - 调用过程。根据 OpenStack 目前的实现模型，创建虚拟机会先从 Nova 调用 Neutron API 创建 Port，然后当虚拟网卡（VIF）在计算节点创建成功后，调用 Neutron API 更新 Port。以上过程都是 OpenStack 的，关键是调用 Neutron API 之后对 Plugin 或 Mechanism Driver 内的调用过程；
 - 多层端口绑定支持。为了帮助厂商实现更合理的集成，从 OpenStack K 版开始提供完整的多层端口绑定（Hirachical Port Binding，以下简称多层端口绑定）的功能，有利于保证 Neutron 与外部系统的数据库一致和增强维护便利。

#### ML2 说明

 ML2 是由 Neutron 社区开发的，针对多个厂商的网络实现的统一层。 其架构如下图所示：
 
 ![ml2_arch][1]
 
 早期的 Neutron 实现是每个核心网络实现都需要自己撰写 Plugin，这样带来几个问题：
 
 - 很多 Plugin 需要处理很多类似的工作。例如数据库的操作等；
 - 会带来 API 不一致的风险。由于每个 Plugin 将直接对接核心资源（Network、Subnet、Port）的操作，而核心资源的属性有可能变化，此时需要更新所有 Plugin，很难推动；
 - 无法保证数据库的一致性。因为 Plugin 可以直接操作数据库；
 - 无法获得通用的提升。例如 Networking OVO 将解决数据库升级的 Downtime，但前提是所有 Plugin 均使用这一技术，如果是各自的 Plugin 很难保证一致。

 ML2 中的核心概念是 ML2 Type Drivers 和 Mechanism Drivers：
 
 - ML2 Type Driver 会维护与网络拓扑类型相关的状态、提供租户网络的分配等。目前支持的有：
  - local,flat,vlan,gre,vxlan,geneve；
 - ML2 Mechanism Driver 会对 Type Driver 提供的信息作出处理，确保被开启的具体网络实现所执行，具体原理是通过 MechanismManager 按照配置文件的注册顺序，依次调用 Mechanism Driver 撰写的相应 hook 方法。

#### 多层端口绑定

 在实现多层端口绑定之前，Neutron 的数据库只会记录一个 Port 的宿主机位置、绑定类型和对应的网络的 Segment（例如 Vlan ID、VxLan VNI）。然而这些 Segment 是一层的，这样如果是多层的架构模型的话无法提供良好的支持：
 
 ![Hirachical Port Binding][2]
 
 从 Kilo 版本开始，随着多层端口绑定的支持，Neutron 可以在绑定过程和数据库记录中对多层的绑定关系保存记录，从而提供灵活的绑定关系，而且这样即使外部的 SDN 系统不修改计算节点上的 Open vSwitch Agent 也能保证其最终动态 Vlan tag 可以被正确使用。
 
 其实现的核心是`PortContext`下新提供了`continue_binding`方法。如果一个 Mechanism Driver 可以完成完整的端口绑定过程，则调用`set_binding`，如果需要后面的 Mechanism Driver 继续绑定，则调用`continue_binding`来继续下一层的绑定过程，绑定的结果后保存在`ml2_port_binding_levels`数据库表里。
 
#### 网络拓扑

 目前 OpenStack 比较主流的是 VxLan 网络类型，所以我们也推荐用户使用 VxLan 网络解决 Vlan 网络 Scale 的限制。同时为了实现最佳性能，目前大部分厂商的参考实现都是基于 Vlan－VxLan 双层架构的，即“多层端口绑定”上所描述的架构。鉴于早期版本的 VxLan 缺乏控制平面和三层路由的相关参考，我们建议通过 EVPN 或其他手段作为 VxLan 的控制平面解决 VxLan 的一些 Scale 问题，关于 VxLan 的详细说明参考 [架构－VxLan](../../architecture/vxlan.md) 和 [生态系统-技术考察点-虚拟路由器](../virtual_router.md)。

### 最佳实践

 由于社区的灵活性，实际上是否使用 ML2、是否支持多层绑定都是可选的，但根据我们现在的使用、运维经验，推荐的集成方案是：
 
 - 针对 ML2 框架撰写 Mechanism Driver；
 - 支持 VxLan 类型驱动，并默认使用 VxLan，通过控制平面抑制 VxLan 的广播和提升三层路由效率；
 - 支持多层端口绑定，并保证 Neutron 数据库与外部 SDN 系统的数据库一致。

 这样做有以下好处：
 
 - 解决前述的 ML2 所需要解决的问题；
 - 多层端口绑定可以提升维护便利，和提供潜在的网络核心实现迁移可能性；
 - 提升网络 Scale 能力。

### 已完成测试的软件



### 参考链接

 [ml2-hierarchical-port-binding](https://specs.openstack.org/openstack/neutron-specs/specs/kilo/ml2-hierarchical-port-binding.html)
 


 [1]: ../../../images/architecture/ml2-arch1.png
 [2]: ../../../images/architecture/HPE.png