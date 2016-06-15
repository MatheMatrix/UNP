### 保持 DNS Nameservers 一致性

---

#### 简介

在 Neutron 的 Subnets 中，当创建子网指定 DNS Nameservers 或者
更新 Subnet 的 DNS Nameservers 属性时，保持了 DNS Nameservers 顺序的一致性。
也就是说，Neutron Subnet 中的 DNS Nameservers 的顺序会和虚拟机获得的
DNS Nameservers 顺序保持一致。

#### 更新 DNS Nameservers

```

changzhi@stack:~/devstack$ neutron subnet-show 1a2d261b-b233-3ab9-902e-88576a82afa6
+------------------+--------------------------------------------+
| Field            | Value                                      |
+------------------+--------------------------------------------+
| allocation_pools | {"start": "10.0.0.2", "end": "10.0.0.254"} |
| cidr             | 10.0.0.0/24                                |
| dns_nameservers  | 1.1.1.1                                    |
|                  | 2.2.2.2                                    |
|                  | 3.3.3.3                                    |
| enable_dhcp      | True                                       |
| gateway_ip       | 10.0.0.1                                   |
| host_routes      |                                            |
| id               | 1a2d26fb-b733-4ab3-992e-88554a87afa6       |
| ip_version       | 4                                          |
| name             |                                            |
| network_id       | a404518c-800d-2353-9193-57dbb42ac5ee       |
| tenant_id        | 3868290ab10f417390acbb754160dbb2           |
+------------------+--------------------------------------------+

changzhi@stack:~/devstack$ neutron subnet-update 1a2d261b-b233-3ab9-902e-88576a82afa6 \
--dns_nameservers list=true 3.3.3.3 2.2.2.2 1.1.1.1

changzhi@stack:~/devstack$ neutron subnet-show 1a2d261b-b233-3ab9-902e-88576a82afa6
+------------------+--------------------------------------------+
| Field            | Value                                      |
+------------------+--------------------------------------------+
| allocation_pools | {"start": "10.0.0.2", "end": "10.0.0.254"} |
| cidr             | 10.0.0.0/24                                |
| dns_nameservers  | 3.3.3.3                                    |
|                  | 2.2.2.2                                    |
|                  | 1.1.1.1                                    |
| enable_dhcp      | True                                       |
| gateway_ip       | 10.0.0.1                                   |
| host_routes      |                                            |
| id               | 1a2d26fb-b733-4ab3-992e-88554a87afa6       |
| ip_version       | 4                                          |
| name             |                                            |
| network_id       | a404518c-800d-2353-9193-57dbb42ac5ee       |
| tenant_id        | 3868290ab10f417390acbb754160dbb2           |
+------------------+--------------------------------------------+
```

当更新了 DNS Nameservers 的顺序，对于本 Subnet 中已经存在的虚拟机
不会立刻按照新的顺序更新 DNS Nameservers。只有当 Subnet 中的虚拟机 DHCP 或者进行 DHCP 续约时，
虚拟机才会按照新的顺序获取 DNS Nameservers。
比如，以 Cirros 镜像的虚拟机为例，当前虚拟机的 resolv.conf 如下：
```
$ cat /etc/resolv.conf
search openstacklocal
nameserver 1.1.1.1
nameserver 2.2.2.2
nameserver 3.3.3.3
```
重启网络服务后，该文件内容如下：
```
$ cat /etc/resolv.conf
search openstacklocal
nameserver 3.3.3.3
nameserver 2.2.2.2
nameserver 1.1.1.1
```
由上可知，当按照特定顺序更新 DNS Nameservers 时，在虚拟机更新网络服务后，虚拟机会获得特定顺序的 DNS Nameservers。
