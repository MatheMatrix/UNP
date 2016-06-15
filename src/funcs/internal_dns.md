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
