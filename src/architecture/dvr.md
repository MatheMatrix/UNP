## DVR

---

### 简介

#### UnitedStack 知识库相关文章

 - [网络软件优化方案之DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=14024938)
 - [Neutron DVR —Dive into Human Defined DVR](https://confluence.ustack.com/pages/viewpage.action?pageId=16096933)
 - [通过Rally测试DVR场景下的Neutron控制平面](https://confluence.ustack.com/pages/viewpage.action?pageId=16097482)

#### 实现目的

 传统的 Open vSwitch Driver 实现的网关是集中式的，也就是说虚拟机的跨子网东西流量和外网的南北向流量都是回过网络节点这个中央节点的，参考下面两幅图分别说明了东西流量和南北流量：
 
 
 
 