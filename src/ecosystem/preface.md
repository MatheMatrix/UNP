# 生态系统

---

UnitedStack<sup>®</sup> UNP 中一个重要的组成部分是它的生态系统。由于 UnitedStack<sup>®</sup> UNP 是基于 OpenStack Neutron 的开放平台，因此社区为 OpenStack Liberty 开发的网络插件、方案基本都可以运行在 UnitedStack<sup>®</sup> UNP Perlis 上。与社区生态环境相比，UnitedStack<sup>®</sup> UNP 生态系统有这些特点：

 - 完善的企业级对接测试。为了确保集成或相关产品能够正确、稳定的运行在 UnitedStack<sup>®</sup> UNP 上，我们对相关产品的稳定性、高可用、性能、同步、一致性保证、并发与时许等诸多问题进行考量，特别是一些由于对接而产生的常见难点问题，例如：
   - 一致性保证：当外部系统集成到 OpenStack 后，一个常见的问题如何保证数据库的一致性。比如当发生一些错误、异常、时序问题后，SDN 系统的数据库能否和 OpenStack 数据库保持一致；

   - 同步：当外部系统集成到 OpenStack 后，此时存在以下几个操作网络资源的入口：
     - 直接对设备操作：例如对交换机操作
     - 对外部系统操作：例如对 SDN 系统操作
     - 对 OpenStack 操作：即在 OpenStack 面板上或通过 OpenStack API 操作资源
     
     例如目前常见的做法有屏蔽部分入口、通过定时同步或直接人工避免这类操作；
     
   - OpenStack Neutron 本身的并发锁并没有很高效的实现，很多地方往往通过数据库锁实现（例如配额实现上的两步提交、更新操作的行锁），但这些实现在高并发的压力下并不是特别稳定，在额外对接了 SDN 系统后，OpenStack Neutron 很可能由于事务的拉长额外导致一些问题；
     
 - 明确的规格说明。集成方案是否对最新版本有支持、是否有明确的发布与更新节奏、集成代码是否开放、具体的硬件规格、更新时是否会对既有系统产生影响等都会在 UnitedStack<sup>®</sup> UNP 中做出说明，解决方案落地时常见的忽略问题；
 - 部署架构的推荐。我们会综合 UnitedStack<sup>®</sup> OpenStack 的高可用设计和架构意见，结合相应第三方产品和厂商建议做出部署架构或使用模型的建议；
 - 对部分重要的社区无 CI 集成或未在社区公开集成的产品提供集成对接。例如部分国内厂商的产品往往没有直接提交的社区，但会在 UnitedStack<sup>®</sup> UNP 提供支持；
 - 迁移指南。对于既有的 OpenStack 运行平台，UnitedStack<sup>®</sup> UNP 会对部分情况，特别是迁入到 UnitedStack<sup>®</sup> UNP 生态系统的案例提供技术方案论证和支持。

目前我们把 UnitedStack<sup>®</sup> UNP 生态系统产品分为三类：

- 核心网络实现，这类产品会对核心网络资源例如二层网络、子网、路由提供替代实现；
- 高级网络服务，这类产品会对原有 OpenStack 的部分功能或高级服务提供替代或扩展；
- 网络硬件设备，这类产品是经过验证的在 UnitedStack<sup>®</sup> UNP 可以正确运行部分硬件列表，该列表提供已测试的硬件设备兼容性说明与推荐。

目前已支持的有以下产品：

 - 核心网络实现
   - Cisco ACI™
   - Cisco VTS™
   - BigSwitch BCF™
 - 高级网络服务
   - Cisco ICF
   - F5 BIG-IP
   - 山石云∙界 SG-6000-VM
 - 硬件设备
   - 英特尔<sup>®</sup> 以太网控制器
   - 英特尔<sup>®</sup> 至强 <sup>®</sup> 处理器
   - Mellanox ConnectX<sup>®</sup> 以太网控制器
   - 戴尔™ Networking S 系列交换机
   - 戴尔™ Networking N 系列交换机
   - Cisco Nexus<sup>®</sup> 交换机

目前处于测试中或列于测试计划的产品有：
 
| 产品 | 目前状态 |
| ---- | ------ |
| Huawei Agile Controller | 测试中 |
| H3C Virtual Converged Framework | 测试计划 |
| F5 BIGIP VE | 测试计划 |
| Citrix NetsSaler NCC（可对接 NetScaler MPX、VPX、SDX） | 6月份测试 |
| 6WIND Virtual Accelerator | 测试过之前版本，6月测试 L 版 |
| CPlane DCI Solution | 测试计划 |
| OneConveragence L4-L7 Integration | 测试计划 |
| Netronome | 测试计划 |
| Fortinet VGFW | 7月份测试 |
| Fortinet 硬件防火墙对接 | 7 月份测试 |

UnitedStack<sup>®</sup> UNP 是一个厂商中立的平台，原则上不会对任何厂商做非技术性的推荐，只对产品的集成效果作出经过我们评估的保障、并对集成效果提供可靠、专业的测试与分析。