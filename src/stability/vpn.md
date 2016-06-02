## VPN稳定性的提升

### UnitedStack知识库文章

 - [云平台上部署山石网科『云界』](https://confluence.ustack.com/pages/viewpage.action?pageId=16098168)
 - [Kilo版IPSec strongswan的驱动实现](https://confluence.ustack.com/pages/viewpage.action?pageId=16096664)

### 简介

目前，我们在云平台上通过 OpenSwan 提供 IPSec 隧道功能。但是在 OpenSwan 对接硬件网络设备时，发现 IPSec 隧道不稳定，会发生严重的丢包现象，并且在隧道中断后无法自行恢复。经过对基于 StrongSwan 提供的 IPSec 隧道进行测试，其丢包现象也很明显，但可以在隧道中断后自行恢复连接，因此在软件实现层面，我们打算将 OpenSwan Drvier 换成 StrongSwan Driver；在和第三方对接方面，我们和山石网科合作，集成 VFW，通过 VFW 提供 IPSec 隧道的功能，经过长时间的测试，我们发现**使用 VFW 提供 IPSec 隧道时的丢包率是0.04%**。关于和 VFW 的对接可参考上述链接。

#### 测试数据

软件层面测试结果如下，其中均已ping包形式进行测试：

| 底层驱动 | 测试环境 | 测试时间 | 丢包率 | 中断后重连 |
| --- | --- | --- | --- | --- |
| OpenSwan | vRouter - vRouter | 2 天 | 19% | 支持 |
|   | vRouter - Firewall | 1 ~ 2 天 | 未知 | 不支持 |
| StrongSwan | vRouter - vRouter | 4 天 | 少于 0.01% | 支持/未中断 |
|   | vRouter - Firewall | 2 天 | 60% | 支持 |
