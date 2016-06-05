# Cisco ACI with OpenStack

## 知识总览

### Cisco ACI 简介

Cisco Application Centric Infrastructure (ACI) policy-based 架构，通过集中式的控
制器和交换机组成 fabric。ACI 提供可编程的 API 接口，使得能够与 OpenStack 或其它
CMS 集成。

ACI fabric 提供了一个基于 VxLAN 集成的二三层 overlay 网络，将虚拟机的数据包封装
从计算节点卸载到 Leaf 交换机上；得益于硬件网络设备，ACI fabric 能够高性能和稳定
的提供服务。


### OpFlex 简介

OpFlex 是一个开发可扩展的策略协议，用来将 ACI 中定义的策略转化为其它设备能够实现
的功能。例如，通过 OpFlex 协议可以将 ACI 中的策略模型转换到计算节点上的虚拟交换机
(Open vSwitch) 上来实现，完成 OpenStack 中基础的网络功能例如 NAT。

OpFlex-based OpenStack 驱动支持两种不同的部署方式。

 * 结合 Neutron API 和 ML2，提供网络、子网、路由器、子网等基本的功能

 * 结合 Group-Based Policy 的 Native Driver，提供的功能跟 ACI 的 Policy 模型更接近

## 物理架构

ACI fabric 包含一个基于 Nexus 9000 Spine/Leaf 拓扑架构，与 OpenStack 则还需要一组物理
服务器来运行 OpenStack 。

ACI External Routed Network 提供 OpenStack 内部虚拟机跟外部网络三层连接的功能。

![aci-opflex-phy-arch][1]

## OpFlex Architecture

### OpFlex ML2 Architecture

Neutron 中 ML2 通过 TypeDriver 和 MechanismDriver 来定义和实现网络功能。通常的网
络类型包括vlan，vxlan，gre 等。在这个架构中，新增加了一种网络类型 opflex ,具体实
际的数据包封装方式是 vlan 还是 vxlan 在计算节点上的配置文件中定义。

| Neutron Object | APIC Object Mapping | Description |
|:-------------- |:------------------- |:----------- |
| Project | Tenant (fvTenant) | The project is directly mapped to a Cisco APIC tenant. |
| Network | EPG (fvAEPg) Bridge domain (fvBD) | Network creation or deletion triggers both EPG and bridge domain configurations. The Cisco ACI fabric acts as a distributed Layer 2 fabric, allowing networks to be present anywhere. |
| Subnet | Subnet (fvSubnet) |The subnet is a 1:1 mapping. ｜
| Security Group and Rule |  |Security groups are fully supported as part of the solution. However, these resources are not mapped to Cisco APIC, but are instead enforced through IP tables as they are in traditional OpenStack deployments. |
| Router | Contract (vzBrCP) Subject (vzSubj) Filter (vzFilter) | Contracts are used to connect EPGs and define routed relationships. The Cisco ACI fabric also acts as a default gateway. The Layer 3 agent is not used. |
| Network: external | Outside | An outside EPG, including the router configuration, is used. |
| Port | Static path binding (fvRsPathAtt) | When a virtual machine is attached, a static EPG mapping is used to connect a specific port and VLAN combination on the top of the rack (ToR). |

增加了一个 MechanismDriver，来把 Neutron 资源转换成 ACI 的 Policy 数据模型，接受
到的的请求转发到 APIC 上。


### OpFlex GBP Native Architecture

Group-Based Policy 是一个OpenStack的一个API 框架，提供了一个 intent-driven model
(意向性驱动的模型)，独立于底层基础设施来描述应用需求。

与以网络为中心的构建方式（Layer 2 domains etc.）不同，GBP 引入Group (组)的概念，
通过Policy（策略）来定义Group之间的连接、安全和网络服务。
 
GBP 以 service plugin的方式运行在 neutron-server的进程空间，未来可能会剥离出成为
一个单独的server。

| GBP Object | Cisco APIC Mapping |Description |
|:---------- |:------------------ |:---------- |
| Policy target | Endpoint | A basic addressable unit in the architecture. It corresponds to an individual network endpoint (generally a virtual network interface card [vNIC]). |
| Policy group | Endpoint group (fvAEPg) | A basic unit of isolation consisting of a set of policy targets with the same policy. |
| Policy classifier | Filter (vzFilter) | A means of filtering network traffic including protocol, port range, and direction (in or out or bidirectional). |
| Policy action | - | An action to take when a particular rule is applied. The APIC driver supports Allow as well as Redirect actions. |
| Policy rule | Subject (vzSubj) | A classifier-action pair defining a specific policy behavior. |
| Policy rule set | Contract (vzBrCP) | A group of policy rules that can be provided or consumed by a number of policy groups. |
| Layer 2 policy | Bridge domain (fvBD) | A set of groups within the same switching domain. Layer 2 policies can contain one or more subnets. |
| Layer 3 policy | Context (fvCtx) | Private network containing a potentially overlapping set of IP addresses. Layer 3 policies in GBP contain IP supernets that are divided into subnets. |


ACI 提供了一个Driver 作为 GBP 的 Nativa Driver 将 GBP 的资源转换成 ACI 的 Policy
数据模型，接受到的的请求转发到 APIC 上。

### ML2 与 GBP Native 对比

|   | GBP | ML2 |
|:-- |:--- |:--- |
| Model | Multiple EPG per BD | Limited to 1:1 EPG to BD |
| Security view in APIC | All contract rules pushed from OpenStack to APIC | Security groups pushed straight to host (ie. Not visible in APIC) |
| Network API* | ML2 model not available. This may impact higher level tools that require network/router model such as Cloud Foundry | GBP model not available |
| Service chaining | Easy use of APIC service chaining and device packages | Limited use of device packages.  Easier of use of LBaaS |

![gbp_ml2][3]

### OpFlex Architecture

每个计算节点上运行 neutron-opflex-agent，通过 RPC 获取 neutron-server 发送的 endpoint
相关的更新消息，并把 endpoint 信息写入本地文件中；OpFlex Proxy 运行在 Leaf 交换机上，
agent-ovs 通过其获取 Policy 相关数据，同时通过 OpenFlow 协议管理 ovs 网桥上的流表。

![opflex-arch][2]

### OpFlex 分布式网络服务
 * NAT：计算节点上的 Open vSwitch 网桥实现了公网访问的 Floating IP 和 SNAT，虚拟机对
 外部网络的访问会先进行地址转换，然后路由到 APIC 中定义的 External Network 中

 * Layer 3 Forwarding：计算节点上的 Open vSwitch 网桥实现了三层路由和转发
 
 * DHCP：计算节点上的 agent-ovs 完成了之前由 neutron-dhcp-agent 提供的功能，将 DHCP 
 Discovery，Offer，Request 和 Acknowledge 控制在计算节点内部，实现分布式的 DHCP 服务

 * Metadata Proxy：在计算节点运行 metadata-agent，通过 agent-ovs 下发流表将 matadata
 请求重定向，实现分布式 Metadata Proxy
 
![opflex][4]

### 在 Horizon 的演示

GBP所有的操作均在 Horizon 的 Policy Tab 下面，直接在 Compute 和 Network 下面操作将无法通信。

![h1][5]

首先需要创建一个 L3 policy, 对应于一个VRF，这里需要输入 IP pool，相当于一个 base 的cidr。同时还要输入掩码长度，比如输入 24，那么待会儿创建 L2 Policy的时候就会创建一个网络和一个包含在 base cidr 中 24 位掩码的子网。

![h2][6]

接着，创建L2 Policy ，选中我们刚刚创建的那个L3 Policy

![h2][7]

上述过程中，我们就创建了一个网络和子网，接着可以创建 rule 供接下来的 Group 使用 ，关于 rule 可以参考 UnitedStack 知识库相关文章。

![h3][8]

接下来为了容纳虚拟机，就需要创建 Group。选中刚创建的 L2 policy。创建的 Group 的同时可以应用一些规则。

![h4][9]

进入这个 Group，Create Member(创建虚拟机)，虚拟机的网络选刚创建的 Group。

![h5][10]



[1]: ../../images/ecosystem/aci-opflex-phy-arch.png
[2]: ../../images/ecosystem/opflex-arch.png
[3]: ../../images/ecosystem/aci2.png
[4]: ../../images/ecosystem/aci3.png
[5]: ../../images/ecosystem/horizon1.png
[6]: ../../images/ecosystem/horizon2.png
[7]: ../../images/ecosystem/horizon3.png
[8]: ../../images/ecosystem/horizon4.png
[9]: ../../images/ecosystem/horizon5.png
[10]: ../../images/ecosystem/horizon6.png