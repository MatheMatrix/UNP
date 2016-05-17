## DVR

---

### 简介

#### UnitedStack 知识库相关文章

 - [网络软件优化方案之 DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR -- Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)
 - [通过 Rally 测试 DVR 场景下的 Neutron 控制平面](https://confluence.ustack.com/pages/viewpage.action?pageId=16097482)

#### 实现目的

在传统的 Classic 集中式路由架构下，虚拟机的跨子网东西向流量、所有 FloatingIP 或 SNAT 南北向流量都需要经过虚拟路由器，即这样的：

东西跨子网流量



根据我们的经验，会产生一些很难通过调优或运维解决的问题：

