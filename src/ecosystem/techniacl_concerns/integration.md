## 集成方法

### 简介

#### UnitedStack 知识库相关文章

 - [BigSwitch集成](https://confluence.ustack.com/pages/viewpage.action?pageId=14027997)
 - [VTS部署及联调文档](https://confluence.ustack.com/pages/viewpage.action?pageId=12059454)
 - [Cisco ACI 集成测试配置方法](https://confluence.ustack.com/pages/viewpage.action?pageId=14025104)

#### 技术说明

本节不讨论 Plugin、Service Plugin 等的配置问题，只讨论软件部署的问题。目前常见的部署方法有几种：

 1. 提供 CentOS 7 下的 RPM 包
 - 提供 CentOS 7 下的集成源码和 Spec 文件
 - 提供集成相关文件，需要手动部署
 - 只需要部署镜像，例如 qcow2 文件
 - 提供一系列脚本或工具自动集成或部署

其中 1 适用于软件稳定，只需要部署和配置的情况；2 适用于需要对集成代码联合开发、重新打包的情况；5 适用于软件稳定、对接测试完善只需要部署和配置的情况；4 只适用于一些提供高级服务（NFV）的场景；不推荐 3 的场景，因为无法维护后续的升级和回归等问题。