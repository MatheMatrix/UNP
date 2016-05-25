## 社区 API 支持

### 简介

#### UnitedStack 知识库相关文章

 - [UOS开发者手册](https://confluence.ustack.com/pages/viewpage.action?pageId=3643445)

#### 技术说明

随着数年的 OpenStack 发展，Neutron 已经形成了一套相对较为稳定的核心 API 和不断发展的扩展 API，我们主要考察的是与网络资源的实现相关的 API 及相关属性，以及如果发往控制器的资源含有未知属性会如何处理。

此外，社区目前提供两种 API 模型的支持，分别为 ML2 和 GBP，完整的考察应当包含着两种 API 模型的支持。

### ML2

#### 核心 API 资源：

 - Network

| Parameter | Style |	Type |	Description |
| --------- | ----- |   ---- |  ----------- |
| admin_state_up |	plain |	xsd:boolean |	The administrative state of the network, which is up (true) or down (false). |
| id |	plain |	csapi:UUID |	The UUID of the network. |
| changed_at |	plain |	xsd:dateTime |	Time at which the network has been created. |
| name |	plain |	xsd:string |	The network name. |
| shared |	plain |	xsd:boolean |	Indicates whether this network is shared across all tenants. By default, only administrative users can change this value. |
| status |	plain |	xsd:string |	The network status. |
| subnets |	plain |	xsd:list |	The associated subnets. |
| tenant_id |	plain |	csapi:UUID |	The UUID of the tenant. |
| qos_policy_id |	plain |	csapi:UUID |	The UUID of the QoS policy. |
| router_external |	plain |	xsd:boolean |	Indicates whether this network is externally accessible. |
| updated_at |	plain |	xsd:dateTime |	Time at which the network has been updated. |
| mtu |	plain |	xsd:int |	The MTU of a network resource. |
| availability_zones |	plain |	xsd:list |	The availability zone for the network. |
| availability_zone_hints |	plain |	xsd:list |	The availability zone candidate for the network. |

 - Subnet

|Parameter	| Style |	Type |	Description |
| --------- | ----- | ---------- | ------------ |
|name	| plain |	xsd:string |	The subnet name. |
|network_id	| plain |	csapi:UUID |	The UUID of the attached network. |
|created_at	| plain |	xsd:dateTime |	Time at which subnet has been created. |
|updated_at	| plain |	xsd:dateTime |	Time at which subnet has been updated. |
|tenant_id	| plain |	csapi:UUID |	The UUID of the tenant who owns the network. Only administrative users can specify a tenant UUID other than their own. |
|allocation_pools	| plain |	xsd:list |	The start and end addresses for the allocation pools. |
|start	| plain |	xsd:string |	The start address for the allocation pools. |
|end	| plain |	xsd:string |	The end address for the allocation pools. |
|gateway_ip	| plain |	xsd:string |	The gateway IP address. |
|ip_version	| plain |	xsd:int |	The IP version, which is 4 or 6. |
|cidr	| plain |	xsd:string |	The CIDR. |
|id	| plain |	csapi:UUID |	The UUID of the subnet. |
|enable_dhcp	| plain |	xsd:boolean |	Set to true if DHCP is enabled and false if DHCP is disabled. |
|host_routes	| plain	| xsd:list	| A list of host route dictionaries for the subnet. |
|dns_nameservers	| plain |	xsd:list |	The DNS server. |
|destination	| plain |	xsd:string |	The destination for static route. |
|nexthop	| plain |	xsd:string |	The next hop for the destination. |
|ipv6_ra_mode	| plain |	xsd:string |	The IPv6 RA mode, which is dhcpv6-stateful, dhcpv6-stateless, or slaac. |
|ipv6_address_mode	| plain |	xsd:string |	The IPv6 address mode, which is dhcpv6-stateful, dhcpv6-stateless, orslaac. |

 - Port

| Parameter |	Style |	Type |	Description |
| --------- | ----- | ---------- | ------------ |
| status |	plain |	xsd:string |	The port status. Value is ACTIVE or DOWN. |
| name |	plain |	xsd:string |	The port name. |
| allowed_address_pairs |	plain |	xsd:list |	A set of zero or more allowed address pairs. An address pair consists of an IP address and MAC address. |
| ip_address |	plain |	xsd:string |	The IP address. |
| mac_address |	plain |	xsd:string |	The MAC address. |
| created_at |	plain |	xsd:dateTime |	Time at which port has been created. |
| updated_at |	plain |	xsd:dateTime |	Time at which port has been updated. |
| admin_state_up |	plain |	xsd:boolean |	The administrative state of the port, which is up (true) or down (false). |
| network_id |	plain |	csapi:UUID |	The UUID of the attached network. |
| tenant_id |	plain |	csapi:UUID |	The UUID of the tenant who owns the network. Only administrative users can specify a tenant UUID other than their own. |
| extra_dhcp_opts |	plain |	xsd:list |	A set of zero or more extra DHCP option pairs. An option pair consists of an option value and name. |
| opt_value |	plain |	xsd:string |	The extra DHCP option value. |
| opt_name |	plain |	xsd:string |	The extra DHCP option name. |
| device_owner |	plain |	xsd:string |	The UUID of the entity that uses this port. For example, a DHCP agent. |
| fixed_ips |	plain |	xsd:list |	The IP addresses for the port. Includes the IP address and UUID of the subnet. |
| subnet_id |	plain |	csapi:UUID |	The UUID of the subnet to which the port is attached. |
| ip_address |	plain |	xsd:string |	The fixed IP address of the port. |
| id |	plain |	csapi:UUID |	The UUID of the port. |
| security_groups |	plain |	xsd:list |	The UUIDs of any attached security groups. |
| device_id |	plain |	csapi:UUID |	The UUID of the device that uses this port. For example, a virtual server. |
| port_security_enabled |	plain |	xsd:boolean |	The port security status. The status is enabled (true) or disabled (false). |

#### 扩展 API 资源

 - metering
 - qos
 - routes
 - router
 - floatingip
 - security-group
 - subnetpools

#### 高级服务

 - LBaaS
 - VPNaaS
 - FWaaS

### GBP

GBP 非本文重点，其核心概念及可映射到 ML2 的资源为（注意并非完全映射，仅为帮助理解）：

 - Policy Target	Port
 - Policy Target Group	Subnet
 - L2 Policy	Network
 - L3 Policy	Router
 - Service Chain Nodes
 - Service Chain Spec
 - Service Chain Instance

其大致模型如下图。

 ![GBP][1]



 [1]: ../../../images/ecosystem/gbp-white-paper-c11-733126_0.jpg

