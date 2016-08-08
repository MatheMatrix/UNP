## BigSwitch BCF 对接

---

### 概述

#### UnitedStack 知识库相关文章

- [BigSwitch 调研](https://confluence.ustack.com/pages/viewpage.action?pageId=12780452)
- [UnitedStack有云联合Big Switch、戴尔推出全新云网络解决方案](https://www.ustack.com/news/unitedstackbig-switch/)

#### Big Cloud Fabric (BCF)

BCF 是一个开放网络 SDN 数据中心结构，能同时提供基于 VLAN 的物理及虚拟负载的管理 (P+V)，可基于原生的 Neutron ML2 Driver 与 OpenStack 无缝集成。通过在白盒交换机上安装 BCF 网络操作系统，BCF 控制器可实现对于数据中心网络结构的集中式控制；由控制器管理的本地分布式虚拟路由器实现了网络负载的分担；控制器中的 test path 工具可对数据包流经路径进行定位分析；控制器通过和 Neutron 数据库的周期性同步，保证了数据库的一致性。

#### BCF 逻辑架构

BCF 是由一套基于 Spine-Leaf 的网络拓扑架构组成的，并且此架构还可被延伸到运行于 Hypervisor 上的虚拟交换机 (IVS)。这些交换设备都是以层级模式部署的，并均由 BCF 控制器统一进行管理，而 OpenStack 则部署于物理服务器上，服务器端的拓扑信息通过 LLDP 来进行收集。通过 Leaf-Spine 间以及 Leaf-Leaf 间的链路聚合连接，确保了整体的可靠性。

![bcf_logic_view][1]

### OpenStack 集成 BCF

#### 硬件要求

| Hardware Component | Model |
| ------------------ | ----- |
| BCF controller | BCF controller Appliance |
| Fabric Switch: Spine | Dell S4048-ON, Dell S6000-ON, Accton AS6700-32X, Accton AS5710-54X, Accton AS6712-32X, Accton AS5712-54X |
| Fabric Switch: Leaf | Dell S4048-ON, Dell S6000-ON, Accton AS6700-32X, Accton AS5710-54X, Accton AS6712-32X, Accton AS5712-54X |
| Layer 2 switch for p-switch control connectivity | All ports connected to controllers and fabric switches must be in the same VLAN, with IPv6 packets forwarded or flooded in the VLAN. |

#### 软件要求

| Software Component | Model |
| ------------------ | ----- |
| BCF controller | Big Cloud Fabric 3.5.0 |
| Fabric Switch: Spine | Switch Light OS SWL-BCF-3.5.0 |
| Fabric Switch: Leaf | Switch Light OS SWL-BCF-3.5.0 |
| Fabric Swtich: vLeaf | Switch Light Virtual BCF-SL-VX-3.5.0 |
| Switch Light Virtual supported OS on compute host | Cent OS 7.1, Ubuntu 14.04 |
| Browser software tested with Big Cloud Fabric GUI | Chrome 37.x <br> Internet Explorer 10 <br> Safari 7.x <br> Firefox 32.x |
| OpenStack Release | Kilo |

#### 集成

BCF 与 OpenStack 的集成难度相对较低，只需在 OpenStack 控制节点安装 Big Switch OpenStack Installer (BOSI)，并根据需要修改 BOSI 目录下的部署配置文件，通过运行 BOSI 脚本，就可实现与 OpenStack 的集成，以及在各节点中用以提供三层路由功能的 Switch Light Virtual 安装。

### 主要特性

- 集中式控制器管理，降低管理复杂度：可通过基于网页的图形界面、传统的命令行交互甚至 REST API 的控制器 来进行对于网络设备的统一配置、自动化以及问题定位。
- 通过单一的可编程接口，BCF 控制器可实现和 OpenStack 的无缝集成。
- BCF 控制器与 Neutron 数据库周期性的同步，确保控制器中的配置与 Neutron 数据库达成一致。
- 支持分布式三层路由以及分布式 NAT (Floating IP & PAT)：传统中的 OVS 虚拟网桥被替换为 IVS 虚拟网桥，并由 BCF 控制器进行管理；计算节点上的虚拟机可通过 Floating IP 由本节点直接访问至外网 (Internet)，无需再经过网络节点。
- 强化的 test path 功能，可对数据包流向进行全方位（VM - vLeaf - Leaf - Spine - Leaf - vLeaf - VM）分析，使得网络问题的定位分析变得更加方便快捷。
- 控制平面与转发平面的分离，使得即使在 BCF 控制器不可达的情况下，已经部署和配置好的服务得以继续进行。

### 参考文档

[1] Big Cloud Fabric 3.5.0 User Guide  
[2] Big Cloud Fabric 3.5 Datasheet, 可参见 http://go.bigswitch.com/rs/974-WXR-561/images/0627-47BL_BigSwitch_BCF_3.5_DS_WEB.pdf?_ga=1.70209688.965085565.1456297903  
[3] Big Cloud Fabric Leaf-Spine Clos for Data Centers, 可参见 http://www.slideshare.net/bigswitchnetworks/big-cloud-fabric  
[4] Big Cloud Fabric 3.5.0 Deployment Guide  
[5] UnitedStack有云联合Big Switch、戴尔推出全新云网络解决方案, 可参见 https://www.ustack.com/news/unitedstackbig-switch/

[1]:../../images/ecosystem/bcf_logic_view.png "Figure 1. BCF Logical Architecture"