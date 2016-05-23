# RBAC NETWORKS

---

## 简介

### UnitedStack 知识库相关文章
- [Liberty版中的RBAC网络](https://confluence.ustack.com/pages/viewpage.action?pageId=16099650)

### 传统共享网络模式
传统模式下，网络的共享模式只有两种：
- 某租户的私有网络，该网络仅对该租户或管理员租户可见，该租户或管理员租户对此网络有全部操作权限。
- 数据中心中的共享网络，只有管理员租户可创建此共享网络，该网络对全部租户可见，所有租户都可以使用该网络。

这样的使用方式其实是很有局限性的，例如同一数据中心中的某一组租户间需要共用一个网络，而并不想将该网络开放给这个组以外的其他租户，那么此时并没有一个好的办法来解决这样的问题。

### 基于RBAC的网络控制策略
通过引入Role Based Access Control(RBAC)对于网络的控制策略，一个网络可以被某一组租户共享，这大大方便了组内租户间的通信。

### 使用示例
- 在demo用户下查看当前网络列表
```
[root@liberty ~(keystone_demo)]# neutron net-list
+--------------------------------------+---------+--------------------------------------------------+
| id                                   | name    | subnets                                          |
+--------------------------------------+---------+--------------------------------------------------+
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24 |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public  | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6             |
+--------------------------------------+---------+--------------------------------------------------+
```
- 在admin用户下查看当前网络列表
```
[root@liberty ~(keystone_admin)]# neutron net-list
+--------------------------------------+---------+------------------------------------------------------+
| id                                   | name    | subnets                                              |
+--------------------------------------+---------+------------------------------------------------------+
| afc53e26-85f7-4130-8d93-9d64e79b8d4f | private | 8f436128-ab4f-44ec-8101-d69a88fae67e 10.0.0.0/24     |
| 63ae8ae2-d71f-40a9-b92f-2208ce44c8ab | public  | c9aa91b0-5624-4d3e-b418-a0c3f3f0ebd6 172.24.4.224/28 |
+--------------------------------------+---------+------------------------------------------------------+
```
- 在admin用户下创建一个网络，命名为“rbac-net”。通过查看admin用户中的网络列表，可查看到rbac-net，然而在demo用户中并不能看到这个网络
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
- 在admin用户下创建一个对象为rbac-net的rbac策略
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
- 此时若去demo用户中查看rbac-list，其中并没有，说明这个rbac策略属于admin用户；但是能发现在demo用户中多了一个rbac-net的网络，即rbac策略中的对象网络
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
- 尝试在demo用户中在rbac-net中创建一个子网，失败 – 策略不允许
```
[root@liberty ~(keystone_demo)]# neutron subnet-create rbac-net 10.0.0.0/24
rule:create_subnet on {'host_routes': <object object at 0x7fbbea4eaaa0>, 'prefixlen': <object object at 0x7fbbea4eaaa0>, 'name': '', 'enable_dhcp': True, u'network_id': u'cd6b992a-ac14-4524-979c-9c8291246b73', 'tenant_id': u'2ae1333f17094c4a83a11a02ba1aeddf', 'dns_nameservers': <object object at 0x7fbbea4eaaa0>, 'ipv6_ra_mode': <object object at 0x7fbbea4eaaa0>, 'allocation_pools': <object object at 0x7fbbea4eaaa0>, 'gateway_ip': <object object at 0x7fbbea4eaaa0>, u'ip_version': 4, 'ipv6_address_mode': <object object at 0x7fbbea4eaaa0>, u'cidr': u'10.0.0.0/24', u'network:tenant_id': u'cd1d7694e4aa4585b6ac303f089c56e0', 'subnetpool_id': <object object at 0x7fbbea4eaaa0>} by {'domain': None, 'project_name': u'demo', 'tenant_name': u'demo', 'project_domain': None, 'timestamp': '2016-04-29 10:14:50.552269', 'auth_token': '981d65a7066240af8850eb0065e990e1', 'resource_uuid': None, 'is_admin': False, 'user': u'90f23f60f68d4db680ff3ab2f95a7c11', 'tenant': u'2ae1333f17094c4a83a11a02ba1aeddf', 'read_only': False, 'project_id': u'2ae1333f17094c4a83a11a02ba1aeddf', 'user_id': u'90f23f60f68d4db680ff3ab2f95a7c11', 'show_deleted': False, 'roles': [u'_member_'], 'user_identity': '90f23f60f68d4db680ff3ab2f95a7c11 2ae1333f17094c4a83a11a02ba1aeddf - - -', 'tenant_id': u'2ae1333f17094c4a83a11a02ba1aeddf', 'request_id': 'req-27882d6d-18ce-4937-a1cc-8402fbe3774d', 'user_domain': None, 'user_name': u'demo'} disallowed by policy
```
- 在admin用户下基于rbac-net创建一个子网成功
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
- 在demo用户下基于rbac-net网络创建一个端口成功
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
经测试，根据rbac策略，此对象网络对不同租户可见，租户可共同使用此网络(如创建端口)，但唯有网络的属主才能对该网络及其子网做创建、更改和删除的操作。

### 局限性


### 潜在问题

