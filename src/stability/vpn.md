## VPN稳定性的提升

### UnitedStack知识库文章

 - [云平台上部署山石网科『云界』](https://confluence.ustack.com/pages/viewpage.action?pageId=16098168)

### 简介

目前，我们在云平台上通过OpenSwan提供IPSec隧道功能。但是在OpenSwan对接硬件网络设备时，
发现IPSec隧道不稳定，会发生严重的丢包现象。对于此，在软件实现层面，我们打算将OpenSwan Drvier
换成StrongSwan Driver；在和第三方对接方面，我们和山石网科合作，集成VFW，通过VFW提供IPSec
隧道的功能，经过长时间的测试，我们发现**使用VFW提供IPSec隧道时的丢包率是0.04%**。关于
和VFW的对接可参考上述链接。
