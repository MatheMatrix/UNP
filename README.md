## 前言

---

### UnitedStack<sup>®</sup> United Networking Platform 简介

UnitedStack<sup>®</sup> United Networking Platform（以下简称 UnitedStack<sup>®</sup> UNP）是一个以 UnitedStack OpenStack 为中心，为用户和合作伙伴提供完整的网络功能与架构参考，能够为电信、企业和服务提供商带来专业、清晰的指导方案。UnitedStack® UNP 是通过一系列开源软件栈、商业解决方案集成、专业文档、配置说明、架构参考、用户案例定义的。

UnitedStack® UNP Perlis 的基础组成包含以下方面：
 - 云系统: UnitedStack® OpenStack
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

本文档将来可以作为所有其他文档的源头，会以完整、相互引用的方式达到每节都可以单独阅读，分别介绍：UnitedStack® UNP Perlis 所支持的架构、功能、提升、高级功能、稳定性、部署、系统、生态和用户使用场景。

### UnitedStack<sup>®</sup> UNP 的版本

#### 版本号说明

UnitedStack<sup>®</sup> UNP 的版本号参考了[语义化版本2.0.0](http://semver.org/lang/zh-CN/)，但考虑到 UnitedStack<sup>®</sup> UNP 并不是一个直接提供 API 服务的软件，我们将版本号修订语义更改为如下：
 - 主版本号：架构或核心实现的变化，抑或某些影响很大的改善或提升；
 - 次版本号：功能、性能等各方面提升或增强；
 - 修订号：问题的修正。

此外为了纪念为计算机科学事业做出重大贡献的计算机科学家们，我们采用历届图灵奖获得者的名字命名每次比较重要的 Release，例如第一版为 Perlis，以纪念第一届图灵奖的获得者 [Alan Perlis](https://en.wikipedia.org/wiki/Alan_Perlis)  在高级程序设计技术和编译器构造上作出的贡献。

#### 路线图

UnitedStack<sup>®</sup> UNP 的总体目标是打造一个结合 UnitedStack<sup>®</sup> OpenStack 使用的网络平台，会提供一个默认推荐标准架构，但针对用户多样性的需求和网络实现同时会给出建议和专业性的说明，以期能够完整的满足用户。

 根据目前网络发展现状和规划，UnitedStack<sup>®</sup> UNP 的 Release 计划如下
 
![plan][2]

对应版本号会在未来补充。


### 文档撰写目的

本文档会推荐一个默认标准架构，但是理想情况下当具备 OpenStack 基础的用户、架构师、实施运维人员阅读过本文档后应当可以对如何设计一个满足、贴近用户需求和场景的架构和方案，并对如何针对该场景下实施和部署有较为清晰的思路，如果所有硬件要求和平台要求均满足本文档的话，应当可以达到完成所需功能和达成目标性能的效果。


[1]: images/preface/UNPv0.1.png
[2]: images/preface/1-year-plan.png