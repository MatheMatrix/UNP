### Internal DNS

---

#### 简介

 Neutron 提供了 Internal DNS 的功能，用来虚拟机之间通过彼此的域名进行访问。

#### Internal DNS 工作原理

 在 Neutron 中，通过 [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html) 提供
虚拟机的 DHCP 和 DNS 服务。当虚拟机的域名解析(一般地，该文件是 /etc/resolv.conf)指向运行着 Dnsmasq 
进程的地址时，虚拟机即可通过 Internal DNS 完成本网络内虚拟机的域名访问。

 一个典型的 Dnsmasq 进程如下：

```
dnsmasq --no-hosts --no-resolv --strict-order --except-interface=lo 
--pid-file=/var/lib/neutron/dhcp/19a5179f-d2ec-4238-a8de-df3a91d6807c/pid
--dhcp-hostsfile=/var/lib/neutron/dhcp/19a5179f-d2ec-4238-a8de-df3a91d6807c/host
--addn-hosts=/var/lib/neutron/dhcp/19a5179f-d2ec-4238-a8de-df3a91d6807c/addn_hosts
--dhcp-optsfile=/var/lib/neutron/dhcp/19a5179f-d2ec-4238-a8de-df3a91d6807c/opts
--dhcp-leasefile=/var/lib/neutron/dhcp/19a5179f-d2ec-4238-a8de-df3a91d6807c/leases
--dhcp-match=set:ipxe,175
--bind-interfaces
--interface=tapd675bcf3-74
--dhcp-range=set:tag0,1.1.1.0,static,86400s
--dhcp-lease-max=256
--conf-file=/etc/neutron/dnsmasq-neutron.conf
--domain=openstacklocal
```

Internal DNS 是通过 `--addn-hosts` 完成的，该文件的作用和`/etc/hosts`相同，当访问该文件中的域名
时，重定向到相应地址。在上述示例中，`--addn-hosts`文件的内容如下：

```
1.1.1.5	host-1-1-1-5.openstacklocal. host-1-1-1-5
1.1.1.1	host-1-1-1-1.openstacklocal. host-1-1-1-1
1.1.1.2	host-1-1-1-2.openstacklocal. host-1-1-1-2
1.1.1.3	host-1-1-1-3.openstacklocal. host-1-1-1-3
```

虚拟机可通过上述域名访问相应主机，如下所示：

```
$ ping host-1-1-1-3
PING host-1-1-1-3 (1.1.1.3): 56 data bytes
64 bytes from 1.1.1.3: seq=0 ttl=64 time=0.204 ms
64 bytes from 1.1.1.3: seq=1 ttl=64 time=0.185 ms
^C
--- host-1-1-1-3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.185/0.194/0.204 ms
$ ping host-1-1-1-3.openstacklocal.
PING host-1-1-1-3.openstacklocal. (1.1.1.3): 56 data bytes
64 bytes from 1.1.1.3: seq=0 ttl=64 time=0.149 ms
^C
--- host-1-1-1-3.openstacklocal. ping statistics ---
1 packets transmitted, 1 packets received, 0% packet loss
round-trip min/avg/max = 0.149/0.149/0.149 ms
```

#### Neutron 中配置 DNS

##### 修改 Neutron Server 相关配置项

在 `/etc/neutron/neutron.conf` 中配置如下参数：
```
# Domain to use for building the hostnames
dns_domain = my_domain.org.
```

注意：上述的 `dns_domain` 参数一定要以`.`结尾。修改配置后，重启 Neutron Server。

##### 指定虚拟网卡创建虚拟机

 - 指定`dns_name`创建虚拟网卡
```
[root@server-233 neutron(keystone_admin)]# neutron port-create 1978224b-86fd-41ba-8c83-dd85a3acc95d --dns_name=my
Created a new port:
+-----------------------+-----------------------------------------------------------------------------------------+
| Field                 | Value                                                                                   |
+-----------------------+-----------------------------------------------------------------------------------------+
| admin_state_up        | True                                                                                    |
| allowed_address_pairs |                                                                                         |
| binding:host_id       |                                                                                         |
| binding:profile       | {}                                                                                      |
| binding:vif_details   | {}                                                                                      |
| binding:vif_type      | unbound                                                                                 |
| binding:vnic_type     | normal                                                                                  |
| device_id             |                                                                                         |
| device_owner          |                                                                                         |
| dns_assignment        | {"hostname": "my", "ip_address": "10.10.10.31", "fqdn": "my.my_domain.org."}            |
| dns_name              | changzhi                                                                                |
| fixed_ips             | {"subnet_id": "01ffc296-359d-4022-abab-e56986f3870f", "ip_address": "10.10.10.31"}      |
| id                    | 32c91693-87f2-49a2-950d-4f6a54e4eda4                                                    |
| mac_address           | fa:16:3e:25:be:09                                                                       |
| name                  |                                                                                         |
| network_id            | 1978224b-86fd-41ba-8c83-dd85a3acc95d                                                    |
| security_groups       | 4616d6e2-3491-43cc-9411-9463b20d1a11                                                    |
| status                | DOWN                                                                                    |
| tenant_id             | ef979882f1954a0fa4ce7daf244aa557                                                        |
+-----------------------+-----------------------------------------------------------------------------------------+
```
 - 根据上述虚拟网卡，创建虚拟机
```
nova boot --image fa572ab6-64d1-4994-8498-4c5eebf8eb18 --flavor shaker-flavor --nic port-id=32c91693-87f2-49a2-950d-4f6a54e4eda4 vm
```

执行完成上述操作后，会在该网络的 Dnsmasq 的`addn-hosts`文件中生成如下信息：
```
10.10.10.31	my.my_domain.org. my
```
此时，同一网络内的虚拟机可通过上述域名信息对该虚拟机进行访问：
```
root@vm-snat3:/var/lib/dhcp# ping my.my_domain.org.
PING my.my_domain.org (10.10.10.31) 56(84) bytes of data.
64 bytes from my.my_domain.org (10.10.10.31): icmp_seq=1 ttl=64 time=1.01 ms
^C
--- my.my_domain.org ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.018/1.018/1.018/0.000 ms
root@vm-snat3:/var/lib/dhcp# ping my
PING my.openstacklocal (10.10.10.31) 56(84) bytes of data.
64 bytes from my.my_domain.org (10.10.10.31): icmp_seq=1 ttl=64 time=0.339 ms
^C
--- my.openstacklocal ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.339/0.339/0.339/0.000 ms
```
