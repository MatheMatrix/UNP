# Vlan

----

## UnitedStack 知识库相关文章

 - [UnitedStack Vlan 网络方案说明](https://confluence.ustack.com/download/attachments/3642944/UnitedStack%20Vlan%20%E7%BD%91%E7%BB%9C%E6%96%B9%E6%A1%88%E8%AF%B4%E6%98%8E.pdf?version=1&modificationDate=1448375766440&api=v2)
 - [关于基础网络（与 Vlan 网络）、策略路由](https://confluence.ustack.com/pages/viewpage.action?pageId=9642682)

## 数据链路层 ( L2 ) 基础知识

 数据链路层 ( Data Link Layer ) 是 [OSI 参考模型](https://zh.wikipedia.org/wiki/OSI%E6%A8%A1%E5%9E%8B)第二层，
位于物理层和网络层之间。主要功能是提供在两个网络实体之间提供数据链路连接的创建、维持和释放管理。 

### VLAN 基础知识

 LAN 表示 Local Area Network，本地局域网，通常使用 Hub 和 Switch 来连接 LAN 中的计算机。
一个 LAN 表示一个广播域，它的意思是 LAN 中的所有成员都会收到 LAN 中一个成员发出的广播包。
因此，LAN 的边界在路由器或者类似的三层设备。

 VLAN 表示 Virtual LAN。一个带有 VLAN 功能的 Switch 能够同时处于多个 LAN 中。简单的说，
VLAN 是一种将一个交换机分成多个交换机的一种方法。

 IEEE 802.1Q 标准定义了 VLAN Header 的格式。它在普通以太网帧结构 SA （src address）之后
加入了 4bytes 的 VLAN Tag/Header 数据，其中包括 12bits 的 VLAN ID。VLAN ID的最大值是 4096，
但是有效值范围是 1- 4094。

 如图是 VLAN 数据格式：

 ![vlan][1]

### VLAN 类型

 - 基于端口的 VLAN （untagged VLAN）
 这种模式中，在交换机上创建若干个 VLAN，在将若干端口放在每个 VLAN 中。每个端口在某一时刻
只能属于一个 VLAN。一个 VLAN 可以包含所有端口，或者部分端口。

 - Tagged VLANs （数据帧带有 VLAN tag）
这种模式下， 数据帧 的 VLAN 关系是它自己携带的信息中保存的。当交换机收到一个带有 VLAN tag
的 数据帧，它只将数据帧转发给具有同样 VID 的端口。一个能够接收或者转发 tagged frame 的端口
被称为 a tagged port。所有连接到这种端口的网络设备必须是 802.1Q 协议兼容的。这种设备必须能
处理 tagged frame，以及添加 tag 到其转发的数据帧。

 交换机的所有端口，部分是 tagged port，部分被添加到 VLAN 中。一个 untagged port ，一个时刻
只能在一个 VLAN 中，一个 tagged port 可以是多个 VLAN 的成员。

### 交换机端口类型

 以太网端口有三种链路类型：

- Access：只能属于一个 VLAN，一般用于连接计算机的端口
- Trunk：可以属于多个 VLAN，可以接收和发送多个 VLAN 的报文，一般用于交换机之间连接的接口
- Hybrid：属于多个 VLAN，可以接收和发送多个 VLAN 报文，既可以用于交换机之间的连接，也可以
用户连接用户的计算机。 Hybrid 端口和 Trunk 端口的不同之处在于 Hybrid 端口可以允许多个 VLAN 
的报文发送时不打标签，而 Trunk 端口只允许缺省 VLAN 的报文发送时不打标签。


### VLAN 的不足

- VLAN 使用 12-bit 的 VLAN ID，因此第一个不足之处就是最多只支持 4096 个 VLAN 网络
- VLAN 是基于 L2 的，因此很难跨越 L2 的边界，限制了网络的灵活性
- VLAN 的配置需手动介入较多


## Neutron 中的 VLAN 拓扑

### 无网络节点的 VLAN 逻辑拓扑

![vlan_no_network_node][3]


### 有网络节点的 VLAN 逻辑拓扑

![vlan_network_node][4]

## Neutron 对 VLAN 网络的支持

### VLAN 池化

  Neutron 支持预先定义一段或者多段 VLAN 范围，从而将 VLAN *池化*，最终直接创建 VLAN 网络。
具体的做法是，预先在 OpenStack 物理环境中配置相应的 VLAN 范围，在验证相应 VLAN 创建成功
(详情验证过程请参考部署[检查 VLAN ](../performance/preface.md)一节)后对 Neutron 进行相关配置。
设置 Neutron Server 的配置文件`plugins/ml2/ml2_conf.ini`中的`network_vlan_ranges`选项，例如：

```
[ml2_type_vlan]
network_vlan_ranges = physnet3:100:200
```

network_vlan_ranges 的配置格式是：<physical_network>[:<vlan_min>:<vlan_max>] 
上述示例表示在 Neutron 中定义一个物理网络，并且该物理网络的范围是 VLAN 100 ~ VLAN 200。

当然，也可以配置多个物理网络，例如：

```
[ml2_type_vlan]
network_vlan_ranges = physnet3:100:200,physnet4:300:400
```
上述示例表示在 Neutron 中定义两个物理网络，每个物理网络的范围是 VLAN 100 ~ VLAN 200 和 VLAN 300 ~ VLAN 400。

也可以在一个物理网络中配置多个不连续的 VLAN 范围，例如：

```
[ml2_type_vlan]
network_vlan_ranges = physnet3:100:200,physnet3:300:400
```
上述示例表示在 Neutron 中定义一个物理网络，范围是 VLAN 100 ~ VLAN 200 和 VLAN 300 ~ VLAN 400。

通过设置 Neutron Server 中的 `tenant_network_types` 参数为 `vlan`，使得租户默认创建的网络
类型为 VLAN 网络，具体的 VLAN ID 会从定义的 VLAN 范围中轮询获得。也可通过指定 VLAN ID 创建网络。

### 预定义 VLAN 网络
  
 Neutron 同样支持通过预创建 VLAN 网络来对外提供服务。具体定义 VLAN 的方式和`支持 VLAN 池化` 中提到
的方式大体一致。需要指出的是，由于是预定义 VLAN 网络，OpenStack 管理员需要知道所有 VLAN 网络的 VLAN ID
和每个 VLAN 网络对应的 CIDR，从而在 Neutron 中预先创建这些网络。如果不同的 Tenant 需要共享不同的 VLAN
网络，可参考 [Neutron 中的 RBAC](../funcs/rbac_networks.md)一节。


### 总结

 无论是哪种 VLAN 网络模式，都可以配合使用网络节点，或者单独使用（不使用网络节点）。
下面详细介绍这两种情况：

1. 对于不使用网络节点的情况，即，将网关配置在物理设备上，这种 VLAN 网络由于没有网络节点。
因此丧失了较多的灵活性，也丧失网络节点上的多种功能，例如：L3 HA，LBaaS，VPN等。 
但是，正是由于网关配置在物理设备上，因此本种 VLAN 网络性能很高，经过测试，虚拟机的东西流量可以
达到物理网卡的限速。本种 VLAN 网络也有一个优点就是方便 Neutron 网络和企业内网的打通。
 
2. 对于使用网络节点的情况，即，将网关配置在网络节点上，这种 VLAN 网络
仍然可以使用网络节点提供的多种高级网络服务。但是网络性能会有所降低。


因此，对比以上各种情况，推荐：

- 池化 VLAN + 有网络节点
- 预定义 VLAN + 无网路节点

对比如下：

|  |交换机上配置网关|性能|功能|FloatingIP|DHCP|隔离性|三层 Overlap|三层和二层对应|
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|池化 VLAN + 网络节点|x|最优|完整 OpenStack 功能|√|√|高|√|x|
|预定义 VLAN + 无网络节点|√|最佳|无|x|可启在控制或计算节点|通过设备做隔离|x|√|



[1]: ../../images/architecture/vlan.png
[2]: ../../images/architecture/vlan_openstack.png
[3]: ../../images/architecture/vlan_no_network_node.png
[4]: ../../images/architecture/vlan_network_node.png




References:

1. https://zh.wikipedia.org/wiki/数据链路层
2. http://www.cnblogs.com/sammyliu/p/4626419.html
3. http://www.microhowto.info/tutorials/802.1q.html
