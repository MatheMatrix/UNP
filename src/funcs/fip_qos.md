### FloatingIP QoS

---

#### 简介

目前社区 OpenStack Neutron 所实现的 QoS 功能只针对挂载在 OpenvSwitch Bridge 上的虚拟网卡
进行的限速，最终通过调用如下的 OpenvSwitch 命令实现 QoS 的功能：

```
ovs-vsctl set interface tap0 ingress_policing_rate=1000
ovs-vsctl set interface tap0 ingress_policing_burst=100
```

本文主要说明针对 FloatingIP 进行的 QoS。

默认的，对于新创建的 FloatingIP 限速是 1024 Kb (1Mbit)，对于虚拟路由器的 1:SNAT 来说，
默认限速是 5Mb，即：虚拟路由器 SNAT 流量的共享带宽是 5Mb（目前无法通过 API 修改）。

实现该功能的 Neutron 软件包版本：

 - python-neutron-7.1.1.1-2.el7.centos.ustack.noarch
 - openstack-neutron-7.1.1.1-2.el7.centos.ustack.noarch

#### 开启 FloatingIP QoS

 修改配置文件：

 - 修改 `neutron.conf` 中 section 为 [DEFAULT] 中的 `enable_fip_qos` 选项为 `True`。即：打开 FloatingIP QoS 功能。
 - 修改 `l3_agent.conf` 中section 为 [DEFAULT] `default_router_gw_rate_limit` 选项为 5 (若无特殊需要，无需修改，该选项默认值即为 5)。即：路由器开启公网网关的共享带宽为 5Mb。

升级影响：

 对于使用过 7.1.1.1 之前版本的 Neutron 来说，升级到 7.1.1.1 版本后，不影响之前的 FloatingIP 使用（无限速），
对于所有的 FloatingIP 都可通过 API 调整 FloatingIP 的 QoS。

#### 实现原理

通过 Linux TC (Traffic Control) 实现针对 FloatingIP 的 QoS。

通常，要对网卡进行流量控制的配置，需要进行如下的步骤：

 - 为网卡配置一个队列；
 - 在该队列上建立分类；
 - 根据需要建立子队列和子分类；
 - 为每个分类建立过滤器。

针对 FloatingIP 的 QoS，使用 TC 中的 HTB(Hierarchical Token Bucket) 队列，与其他复杂的队列
类型相比，HTB 具有功能强大，配置简单等优点。在 TC 中，使用 `major:minor` 这样的句柄来表示队列
和类别，其中 `major` 和 `minor` 都是数字。

在 TC 中使用下列的缩写表示相应的带宽:

 - Kbps : kilobytes per second，千字节每秒 ;
 - Mbps : megabytes per second，兆字节每秒 ;
 - Kbit : kilobits per second，千比特每秒 ;
 - Mbit : megabits per second， 兆比特每秒 。

流量的处理由三种对象控制，分别是：qdisc，class，filter

##### QDISC (队列)
QDisc 是理解流量控制的基础。无论何时，内核如果需要通过某个网络接口发送数据包，

它都需要按照为这个接口配置的 qdisc 把数据包加入队列。然后内核会尽可能多得从 qdisc 里面取出数据包，
把它们交给网络适配器驱动模块。
##### CLASS (策略)
class 用来表示控制策略，很多时候，可能要对不通的 IP 实行不同的流量控制策略，
这时候我们就要用不同的 class 来表示不同的控制策略了

##### FILTER (过滤器)
filter 用来将用户划入到具体的控制策略中（即不同的class中），比如我们要对 a 和 b 两个 IP 实行不同的控制策略 A 和 B ，
这时我们可以用 filter 将 a 划入控制策略 A，将 b 划入控制策略 B， filter 划分的标志位可用 u32 打标功能或者 iptables 的 set-mark 功能来实现。

#### 实现流程图

![process][1]

流程图说明：
 1. qg 设备代表桥接在外部网桥上的虚拟网路设备，qr 设备代表虚拟机的默认网关；
 2. 红线代表数据流进入虚拟机时的流量走向；
 3. 蓝线代表数据流离开虚拟机时的流量走向；
 4. ifb 设备用于 qg 设备流量的重定向；
 5. qg 设备提供 ingress redirect 和 egress ratelimit 的功能，ifb 设备只提供 ratelimit 的功能。


### API 使用和测试

FloatingIP 的 QoS 目前支持通过 Neutron-client 形式修改带宽，命令如下：
```
[root@server-233 ~(keystone_admin)]# neutron update-floatingip-ratelimit eb0fa229-9b42-4db3-8789-d2d94ed57fbb 20480
Update floating IP eb0fa229-9b42-4db3-8789-d2d94ed57fbb rate limit 20480
```

通过 Curl 的形式修改带宽：

REQ:
```
curl -g -i -X PUT \
http://10.0.44.233:9696/v2.0/floatingips/eb0fa229-9b42-4db3-8789-d2d94ed57fbb/update_floatingip_ratelimit.json \
-H "User-Agent: python-neutronclient" \
-H "Content-Type: application/json" \
-H "Accept: application/json" \
-H "X-Auth-Token: {SHA1}28aacfc5724b41b6d3ef84728fc5655de778a26a" \
-d '{"floatingip": {"rate_limit": 20480}}'
```

RESP BODY: 
```
{
    "floating_network_id": "8a93ea75-87cd-4677-ba02-8c5989ad0843",
    "router_id": "8db51fa5-22c7-4944-a4c5-de99e62ad58f",
    "fixed_ip_address": "10.10.10.22",
    "floating_ip_address": "2.2.2.94",
    "tenant_id": "ef979882f1954a0fa4ce7daf244aa557",
    "status": "ACTIVE",
    "port_id": "06e18bdb-c35d-4914-9e4a-f763231484d6",
    "id": "eb0fa229-9b42-4db3-8789-d2d94ed57fbb",
    "rate_limit": 20480
}
```

#### 测试过程和结论

通过在绑定了 FloatingIP 的虚拟机中下载文件，观察下载速度从而验证 TC 规则是否正确限速。

在 FloatingIP 带宽为 2048Kb （2Mb）情况下，虚拟机中 wget 文件的速度为：
```
root@vm-snat7:~# wget http://10.0.44.233:8000/data
--2016-07-11 19:16:57--  http://10.0.44.233:8000/data
Connecting to 10.0.44.233:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 104857600 (100M) [application/octet-stream]
Saving to: 'data.2'

data.2                      36%[=============>             ]  36.15M   232KB/s   eta 55s
```
通过上述结果观察到当前虚拟机下载文件的速度为 230KB/s 左右，符合预期。

通过 API 修改该 floatingip 的带宽
值为 20480 Kb （20 Mb），重新下载该文件观察下载速度：
```
root@vm-snat7:~# wget http://10.0.44.233:8000/data
--2016-07-11 19:19:58--  http://10.0.44.233:8000/data
Connecting to 10.0.44.233:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 104857600 (100M) [application/octet-stream]
Saving to: 'data.4'

data.4                       9%[==>                        ]   9.14M  2.29MB/s   eta 42s
```
通过上述结果观察到当前虚拟机下载文件的速度为 2.29MB/s 左右，符合预期。


[1]: ../../images/funcs/tc.png
