## 前言

---

### UnitedStack<sup>®</sup> United Networking Platform 简介

UnitedStack<sup>®</sup> United Networking Platform 是一个以 UnitedStack OpenStack 为中心，为用户和合作伙伴提供完整的网络功能与架构参考，能够为电信、企业和服务提供商带来专业、清晰的指导方案。UnitedStack® UNP 是通过一系列开源软件栈、商业解决方案集成、专业文档、配置说明、架构参考、用户案例定义的。

UnitedStack® UNP Perlis 的基础组成包含以下方面：
 - 云系统: UnitedStack OpenStack
 - 操作系统: CentOS 7
 - 网络抽象: Neutron
 - 网络实现
  - Open vSwitch Driver
  - BigSwitch BCF
  - Cisco VTS
  - Cisco ACI™
 - 硬件交换机依赖
  - L3 Switch
  - ONIE Switch
  - Nexus Switch
 - 软件交换机依赖
  - Open vSwitch
  - BigSwitch<sup>®</sup> IVS
 - 第三方高级网络服务集成
  - 山石云∙界
 - 生态建设
  - Cisco<sup>®</sup>
  - Hillstone<sup>®</sup>
  - Big Switch Networks<sup>®</sup>
  - F5 Networks<sup>®</sup>
 - 其他开源软件
  - HAProxy
  - Openswan

其组织结构如图所示：

![unp_arch][1]

本文档将来可以作为所有其他文档的源头，会以完整、相互引用的方式达到每节都可以单独阅读，分别介绍：UNP Perlis 所支持的架构、功能、提升、高级功能、稳定性、部署、系统、生态和用户使用场景。

### UnitedStack<sup>®</sup> United Networking Platform 的版本

#### 版本号说吗



### 文档撰写目的

本文档不会直接说明那种组合方式是最好的，但是理想情况下当具备 OpenStack 基础的用户、架构师、实施运维人员阅读过本文档后应当可以对如何满足、贴近用户需求和场景设计架构和确定方案，对如何针对该场景下实施和部署有较为清晰的思路，如果所有硬件要求和平台要求均满足本文档的话，应当可以达到完成所需功能和达成目标性能的效果。


[1]: images/revision/UNPv0.1.png


