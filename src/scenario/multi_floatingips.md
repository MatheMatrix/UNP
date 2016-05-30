## 虚拟机多 FloatingIP 的使用

---

### 简介

  在 OpenStack Neutron 中，每个虚拟网卡只允许绑定一个 FloatingIP，因此对于在一个虚拟机中
使用多个 FloatingIP 的使用场景中，我们建议通过多虚拟网卡分别绑定 FloatingIP 实现该功能。

### 使用过程

  对于虚拟机如何添加多网卡，并分别绑定 FloatingIP 的过程这里不再赘述。本节详细讨论
**多网卡下的路由问题**。

  这里以添加了双网卡的 Ubuntu 15.05 虚拟机为例，详细讲解多网卡下如何配置路由。

  拓扑结构如下：

  ![multi_fips][1]

  如上述逻辑拓扑图所示，在 Neutron 中配置了多个 [External Networks](../architeucture/external_network.md)
的情景下，Neutron 通过 External Network A 连接 Internet，通过 External Network B 连接某内网。

  具体需求如下：
- 虚拟机默认连接 Internet
- 访问 30.30.30.0/24 时通过 eth1 访问

  实验虚拟机的所有网卡信息，和路由信息如下：

```
root@ooo:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:a5:c9:72 brd ff:ff:ff:ff:ff:ff
    inet 10.10.10.29/24 brd 10.10.10.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fea5:c972/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1400 qdisc pfifo_fast state UP group default qlen 1000
    link/ether fa:16:3e:7b:85:2d brd ff:ff:ff:ff:ff:ff
    inet 20.20.20.16/24 brd 20.20.20.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::f816:3eff:fe7b:852d/64 scope link
       valid_lft forever preferred_lft forever
root@ooo:~# ip route
default via 20.20.20.1 dev eth1
10.10.10.0/24 dev eth0  proto kernel  scope link  src 10.10.10.29
20.20.20.0/24 dev eth1  proto kernel  scope link  src 20.20.20.16
169.254.169.254 via 10.10.10.1 dev eth0
```

对于需求1 来说，从上面的路由信息可以看到，虚拟机的默认路由是`20.20.20.1`，因此，对于虚拟机默认连接 Internet 来说，
只需修改默认路由即可，执行如下命令：

```
ip route change default via 10.10.10.1
```

对于需求2 来说，需要通过 Linux 的静态路由实现，只需添加：

```
ip route add 30.30.30.0/24 dev eth1 src 20.20.20.16
```

添加静态路由后的完整路由信息如下：

```
root@ubuntu:~# ip route
default via 10.10.10.1 dev eth0
10.10.10.0/24 dev eth0  proto kernel  scope link  src 10.10.10.29
20.20.20.0/24 dev eth1  proto kernel  scope link  src 20.20.20.16
30.30.30.0/24 dev eth1  scope link  src 20.20.20.16
169.254.169.254 via 10.10.10.1 dev eth0
```

[1]: ../../images/scenario/multi_fips.png
