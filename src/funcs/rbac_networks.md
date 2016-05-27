# RBAC NETWORKS

---

## 简介

### UnitedStack 知识库相关文章
- [Liberty 版中的 RBAC 网络](https://confluence.ustack.com/pages/viewpage.action?pageId=16099650)

### 传统共享网络模式
传统模式下，网络的共享模式只有两种：
- 某租户的私有网络，该网络仅对该租户或管理员租户可见，该租户或管理员租户对此网络有全部操作权限。
- 数据中心中的共享网络，只有管理员租户可创建此共享网络，该网络对全部租户可见，所有租户都可以使用该网络。

这样的使用方式其实是很有局限性的，例如同一数据中心中的某一组租户间需要共用一个网络，而并不想将该网络开放给这个组以外的其他租户，那么此时并没有一个好的办法来解决这样的问题。

### 基于 RBAC 的网络控制策略
Role-based Access Control (RBAC) 是一种用于定义用户角色及其权限的机制，这对于大规模网络环境的管理提供了更安全和便捷的保障。通过引入 RBAC 对于网络的角色控制机制，一个网络可以被某一组租户共享，这大大方便了组内租户间的通信，也达到了对组外租户隔离以确保安全性的目的。与此同时，RBAC 机制也提供了对传统共享网络模式的兼容，通过指定目标租户选项为“*”，该网络对象则可对数据中心所有用户可见，即相当于传统模式下的共享网络。

### 使用示例
根据[社区命令行手册](http://docs.openstack.org/cli-reference/neutron.html) [1]，neutron rbac 相关命令使用方式有如下几种：
```
usage: neutron rbac-create [-h] [-f {html,json,shell,table,value,yaml}]
                           [-c COLUMN] [--max-width <integer>] [--noindent]
                           [--prefix PREFIX] [--request-format {json}]
                           [--tenant-id TENANT_ID] --type {qos-policy,network}
                           [--target-tenant TARGET_TENANT] --action
                           {access_as_external,access_as_shared}
                           RBAC_OBJECT
```
这里需要解释一下的是，qos-policy 参数是在 [Mitaka 版本](http://specs.openstack.org/openstack/neutron-specs/specs/mitaka/rbac-qos-policies.html) [2] 中新增的特性；与此同时，access_as_external 参数虽然在 Liberty 版本中可见，但实际上起并未真正实现。因此，这两个参数在当前版本(UNP1.0)中暂不能被使用。
```
usage: neutron rbac-delete [-h] [--request-format {json}] RBAC_POLICY
```
```
usage: neutron rbac-list [-h] [-f {csv,html,json,table,value,yaml}]
                         [-c COLUMN] [--max-width <integer>] [--noindent]
                         [--quote {all,minimal,none,nonnumeric}]
                         [--request-format {json}] [-D] [-F FIELD] [-P SIZE]
                         [--sort-key FIELD] [--sort-dir {asc,desc}]
```
```
usage: neutron rbac-show [-h] [-f {html,json,shell,table,value,yaml}]
                         [-c COLUMN] [--max-width <integer>] [--noindent]
                         [--prefix PREFIX] [--request-format {json}] [-D]
                         [-F FIELD]
                         RBAC_POLICY
```
```
usage: neutron rbac-update [-h] [--request-format {json}]
                           [--target-tenant TARGET_TENANT]
                           RBAC_POLICY
```
以下就 RBAC 的实际使用及其产生的效果，做一个简单的测试：
- 在 demo 用户下查看当前网络列表
```
[root@liberty ~(keystone_demo)]# neutron net-list
+--------------------------------------+---------+--------------------------------------------------+
| id                                   | name    | subnets                                          |
+--------------------------------------+---------+--------------------------------------------------+
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24 |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public  | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6             |
+--------------------------------------+---------+--------------------------------------------------+
```
- 在 admin 用户下查看当前网络列表
```
[root@liberty ~(keystone_admin)]# neutron net-list
+--------------------------------------+---------+------------------------------------------------------+
| id                                   | name    | subnets                                              |
+--------------------------------------+---------+------------------------------------------------------+
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24     |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public  | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6 172.24.4.224/28 |
+--------------------------------------+---------+------------------------------------------------------+
```
- 在 admin 用户下创建一个网络，命名为 “rbac-net”。通过查看 admin 用户中的网络列表，可查看到 rbac-net，然而在 demo 用户中并不能看到这个网络
```
[root@liberty ~(keystone_admin)]# neutron net-create rbac-net
Created a new network:
+---------------------------+--------------------------------------+
| Field                     | Value                                |
+---------------------------+--------------------------------------+
| admin_state_up            | True                                 |
| id                        | cd6b992a-ac14-4524-979c-9c8291246b73 |
| mtu                       | 0                                    |
| name                      | rbac-net                             |
| provider:network_type     | vxlan                                |
| provider:physical_network |                                      |
| provider:segmentation_id  | 100                                  |
| router:external           | False                                |
| shared                    | False                                |
| status                    | ACTIVE                               |
| subnets                   |                                      |
| tenant_id                 | cd1d7694e4aa4585b6ac303f089c56e0     |
+---------------------------+--------------------------------------+
[root@liberty ~(keystone_admin)]# neutron net-list
+--------------------------------------+----------+------------------------------------------------------+
| id                                   | name     | subnets                                              |
+--------------------------------------+----------+------------------------------------------------------+
| cd6b992a-ac14-4524-979c-9c8291246b73 | rbac-net |                                                      |
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private  | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24     |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public   | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6 172.24.4.224/28 |
+--------------------------------------+----------+------------------------------------------------------+
```
- 在 admin 用户下创建一个对象为 rbac-net 的 RBAC 策略
```
[root@liberty ~(keystone_admin)]# neutron rbac-create --tenant-id cd1d7694e4aa4585b6ac303f089c56e0 --target-tenant 2ae1333f17094c4a83a11a02ba1aeddf --type network --action access_as_shared cd6b992a-ac14-4524-979c-9c8291246b73
Created a new rbac_policy:
+---------------+--------------------------------------+
| Field         | Value                                |
+---------------+--------------------------------------+
| action        | access_as_shared                     |
| id            | 88a81f5f-206f-4173-b3e9-25bd55bbb1d7 |
| object_id     | cd6b992a-ac14-4524-979c-9c8291246b73 |
| object_type   | network                              |
| target_tenant | 2ae1333f17094c4a83a11a02ba1aeddf     |
| tenant_id     | cd1d7694e4aa4585b6ac303f089c56e0     |
+---------------+--------------------------------------+
[root@liberty ~(keystone_admin)]# neutron rbac-list
+--------------------------------------+--------------------------------------+
| id                                   | object_id                            |
+--------------------------------------+--------------------------------------+
| 88a81f5f-206f-4173-b3e9-25bd55bbb1d7 | cd6b992a-ac14-4524-979c-9c8291246b73 |
+--------------------------------------+--------------------------------------+
```
- 此时若去 demo 用户中查看 rbac-list，其中并没有，说明这个 RBAC 策略属于 admin 用户；但是能发现在 demo 用户中多了一个 rbac-net 的网络，即 RBAC 策略中的对象网络
```
[root@liberty ~(keystone_demo)]# neutron rbac-list
[root@liberty ~(keystone_demo)]# neutron net-list
+--------------------------------------+----------+--------------------------------------------------+
| id                                   | name     | subnets                                          |
+--------------------------------------+----------+--------------------------------------------------+
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private  | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24 |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public   | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6             |
| cd6b992a-ac14-4524-979c-9c8291246b73 | rbac-net |                                                  |
+--------------------------------------+----------+--------------------------------------------------+
```
- 在 admin 用户下基于 rbac-net 创建一个子网成功
```
[root@liberty ~(keystone_admin)]# neutron subnet-create rbac-net 192.168.0.0/24
Created a new subnet:
+-------------------+--------------------------------------------------+
| Field             | Value                                            |
+-------------------+--------------------------------------------------+
| allocation_pools  | {"start": "192.168.0.2", "end": "192.168.0.254"} |
| cidr              | 192.168.0.0/24                                   |
| dns_nameservers   |                                                  |
| enable_dhcp       | True                                             |
| gateway_ip        | 192.168.0.1                                      |
| host_routes       |                                                  |
| id                | bb443108-2592-4276-ad73-48107ecba654             |
| ip_version        | 4                                                |
| ipv6_address_mode |                                                  |
| ipv6_ra_mode      |                                                  |
| name              |                                                  |
| network_id        | cd6b992a-ac14-4524-979c-9c8291246b73             |
| subnetpool_id     |                                                  |
| tenant_id         | cd1d7694e4aa4585b6ac303f089c56e0                 |
+-------------------+--------------------------------------------------+
```
- 在 demo 用户下基于 rbac-net 网络创建一个端口成功
```
[root@liberty ~(keystone_demo)]# neutron port-create rbac-net
Created a new port:
+-----------------------+-----------------------------------------------------------------------------------------------------------+
| Field                 | Value                                                                                                     |
+-----------------------+-----------------------------------------------------------------------------------------------------------+
| admin_state_up        | True                                                                                                      |
| allowed_address_pairs |                                                                                                           |
| binding:vnic_type     | normal                                                                                                    |
| device_id             |                                                                                                           |
| device_owner          |                                                                                                           |
| dns_assignment        | {"hostname": "host-192-168-0-5", "ip_address": "192.168.0.5", "fqdn": "host-192-168-0-5.openstacklocal."} |
| dns_name              |                                                                                                           |
| fixed_ips             | {"subnet_id": "bb443108-2592-4276-ad73-48107ecba654", "ip_address": "192.168.0.5"}                        |
| id                    | 56fcc4d7-8a26-4634-b4da-9d749b3d41f2                                                                      |
| mac_address           | fa:16:3e:f3:a1:50                                                                                         |
| name                  |                                                                                                           |
| network_id            | cd6b992a-ac14-4524-979c-9c8291246b73                                                                      |
| security_groups       | e957c137-cf72-4e9c-b3d1-f246488d7885                                                                      |
| status                | DOWN                                                                                                      |
| tenant_id             | 2ae1333f17094c4a83a11a02ba1aeddf                                                                          |
+-----------------------+-----------------------------------------------------------------------------------------------------------+
```
经测试，根据 RBAC 策略，此对象网络对不同租户可见，租户可共同使用此网络(如创建端口)，但唯有网络的属主或管理员才能对该网络及其子网做创建、更改和删除的操作。

### 局限性
- RBAC 机制当前仅适用于网络对象资源，暂不支持对于其他资源的角色控制。
- 在指定目标租户时仅可指定一个租户，对于多个租户的共享组，需要创建多个 RBAC 策略。
- 非管理员用户在与其他租户共享一个网络时，无法看到或删除其他租户下创建的端口 [3]。

### 潜在问题
- 在创建 RBAC 策略，并以目标租户名字为参数进行指定时，该 RBAC 策略可被成功创建，但目标租户并不能正常使用该网络对象资源。只有将目标租户 UUID 作为参数进行指定时，基于该 RBAC 策略才真正生效。相关 [bug](https://bugs.launchpad.net/neutron/+bug/1585082) 已提出，需进一步讨论。
- 目标租户选项参数默认为非必要参数，且根据[社区文档](http://specs.openstack.org/openstack/neutron-specs/specs/liberty/rbac-networks.html) [4] 来看默认值为“*”，即默认对数据中心中所有租户共享。但实测时发现没有参数被传入，默认值为空，并伴有报错。相关 [bug](https://bugs.launchpad.net/neutron/+bug/1578997) 已在 Launchpad 上提出，但在新的补丁被合并前，此选项仍需用户自己指定参数，且为必选项。

### 参考文档
[1] "Networking service command-line client", 可参见 http://docs.openstack.org/cli-reference/neutron.html  
[2] "Role-based Access Control for QoS policies", 可参见 http://specs.openstack.org/openstack/neutron-specs/specs/mitaka/rbac-qos-policies.html  
[3] "Role-Based Access Control for networks", 可参见 http://docs.openstack.org/liberty/networking-guide/adv-config-network-rbac.html  
[4] "Role-based Access Control for Networks", 可参见 http://specs.openstack.org/openstack/neutron-specs/specs/liberty/rbac-networks.html
