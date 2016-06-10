# 部署

---

部署建议是 UnitedStack<sup>®</sup> UNP 对客户落地内容的重要体现，UnitedStack<sup>®</sup> UNP 与其他 OpenStack 厂商提供的网络方案的重要区别就是架构和场景的灵活性与针对性。UnitedStack<sup>®</sup> UNP 的部署方案主要分为两类：

 - Vendor-Free 方案
 - Vendor-Related 方案

前者主要是由 UnitedStack<sup>®</sup> 自身完全借助开源项目诸如 Open vSwitch、Linux 等基于标准硬件（例如标准的 IP 交换机、万兆以太网卡）或开放标准的硬件（例如 ONIE 交换机）实现的架构，目前主要是 Open vSwitch Driver 的纯软实现，未来可能会有基于诸如 DragonFlow、Open Daylight 等项目实现的 SDN 方案，纯软或软硬结合的方案。特点是完全开源、成本很低、供应商选择丰富。

后者主要是由 UnitedStack<sup>®</sup> 的合作伙伴诸如 Cisco<sup>®</sup>、Big Switch Networks<sup>®</sup> 等专业网络厂商实现的网络方案，特点是能够实现针对性的一些架构如物理、虚拟网络同时管理、更高的性能等。目前这部分的部署实践会撰写在生态部分。

本节内容目前主要针对 Open vSwitch Driver，提供完整、可靠、经过验证的企业级部署方案与实践，针对的场景与包含内容如下：

 1. VXLAN + VLAN 自服务模型（默认架构）；
 - 纯 VLAN 预配置模型；
 - 接入层高可用配置；
 - 企业内网接入所需配置；
 - UnitedStack 远程支持接入；
 - 最佳性能硬件推荐；
 - 部署前的检测与测试建议。

所有自服务模型都会参照架构篇的默认架构所开启服务进行部署，例如：

 - DVR、ARP Responder
 - L2 Population（仅在 VXLAN 下有效）
 - VLAN 外部网络

为了节省篇幅，默认架构会给出 Neutron 的全部完整配置和部署过程，而其他模型只会给出与默认配置的区别。

值得注意的是本文档给出了 VXLAN + VLAN 自服务的模型，因为混合使用或者单独使用 VXLAN 或单独使用 VLAN 的自服务配置方法和都是混合使用其实都是一样，唯一需要做的只是前端提交时需要注意默认的网络类型。