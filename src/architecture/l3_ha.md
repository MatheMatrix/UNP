# L3 HA

---

### 简介

#### UnitedStack 知识库相关文章
 - [HA Router](https://confluence.ustack.com/display/SDN/HA+Router)
 - [测试环境部署](https://confluence.ustack.com/pages/viewpage.action?pageId=16095643)

#### 非 HA 的实现问题

在非 HA 场景下，网关是在某个网络节点上的单点，存在单点故障的问题。网络节点的宕机
会导致所有在该节点上的 Router 无法提供服务，虚拟机东西向跨子网和南北向通信中断。

网络节点上 Router 的实现如图：

![classic-router][1] 

#### 基于 VRRP 的 HA 解决方案

这种方案的解决思路是，对于同一个虚拟路由器，Router 会以主备(active/standby) 的模
式工作在多个节点，通过 VRRP 来实现主从同步和选主。

其中， VRRP 手通过 keepalived 进程来实现，这样在每个虚拟路由器内部，就会启动一个
keepalived 进程，定期发送组播来进行 VRRP 通告。为了实现 VRRP 组播与租户网络的隔离，
每个租户会自动隐式的创建一个 HA Network，这样在 router 的 namespace 中就会多一个
ha interface:

Router 具体实现如下图
![ha-rourer][2]

HA router 总体架构如图

![ha-router-architecture][3]

此外还有没有 开启 contrackd 同步 session，VRRP 主从切换之后丢失一部分现有的会话，但不会影响虚拟机
跟外部新的会话。对于丢失的那部分会话，虚拟机内部通过重传能够保持。

#### HA Router 创建过程
##### server-plugin

[1] 确定 router 是否能够被调度到两个或多个 L3 Agent 上

[2] 需要至少两个L2 Agent 来承载 HA Router

##### agent

[1] 创建router相关资源 (namespace, interface, net/iptables, route table)

[2] 生成 keepalived 配置文件

[3] 在 namespace 中启动 keepalived (配置 vip)

#### 控制平面压测

##### Server

开启HA Router之后，相比于Legacy控制平面增加更多的负担：

[1] 对于一个新的tenant，在首次创建是会进行HA Network的创建，在后续创建HA Network
的时候会进行HA Network的校验

[2] 在调度阶段，这将是一个大的Transaction，Scheduler会选择多个l3-agent进行调度，此时会
在HA Network内创建HA Port并与l3-agent绑定

[3] l3-agent收到create_router的请求会根据不同类型的router进行初始化（Kilo 版 L3 Agent 重构）

[4] HaRouter.initialize() 也是一个大的事务，包含keepalived配置文件的生成和
state_change_monitor进程的创建

[5] HaRouter.process() 阶段会创建keepalived进程(VRRP + Healthchecking)


采用 rally 的 create_and_update_routers 用例进行控制平面的压测，具体的 template:

```
{
    "NeutronNetworks.create_and_update_routers": [
        {
            "args": {
                "network_create_args": {},
                "subnet_create_args": {},
                "subnet_cidr_start": "1.1.0.0/30",
                "subnets_per_network": 2,
                "router_create_args": {
                    "external_gateway_info": {"network_id": "ccf351c9-826c-4d93-9dd4-625302bf3051"}
                },
                "router_update_args": {
                    "name": "_router_updated",
                }
            },
            "runner": {
                "type": "constant",
                "times": 50,
                "concurrency": 10
            },
            "context": {
                "network": {},
                "users": {
                    "tenants": 2,
                    "users_per_tenant": 3
                },
                "quotas": {
                    "neutron": {
                        "network": -1,
                        "subnet": -1,
                        "router": -1
                    }
                }
            }
        }
    ]
}
```

这个过程中会并发的创建Router(开启公网网关)，紧接创建网络和子网；接下来是一个
router_update操作，会将创建的子网跟router关联。

测试过程中监测neutron-server日志，server.log，看是否有报错的情况

##### Agent

Router 在网络节点上的体现为namespace，所以在网络节点上写脚本rs.monitor.sh监控
namespace里面的interface设备情况

```
#!/usr/bin/env bash
 
# file name: rs.monitor.sh
file=`hostname`".r_ns_monitor."`date '+%s'`
while true;
do
    for r_ns in `ip netns | grep router`;
    do
        echo -e $r_ns >> $file
        ip netns exec $r_ns ip addr >> $file
        echo -e >> $file
        ps -aux | grep ${r_ns:8} | grep -v "grep" >> $file
        echo -e >> $file
    done
    sleep 1
done
```

该脚本在 rally create_and_update_router 的时候在所有的网络节点执行，在 rally 执行结
束后停止该脚本，得到一个名称类似 "server-68.r_ns_monitor.1461313501"的文件

分析该文件中 router namespace 中的情况可知当时 router 在网络节点 namespace
interface 和与之相关联的进程(keepalived/neutron-ns-metadata-proxy)

##### 测试结果

rally 测试结果:

![l3-ha-rally][4]

通过分析 rs.monitor.sh log，router 中的 namespace 几乎都配置正确，同时与 router
先关联的进程 也能够正确的创建和回收

#### 性能测试

由于 L3 HA 网络拓扑架构相对于 Classic 并没有变化，虚拟路由器是集中式的，但其实现
和 DVR 一样基于内核路由和 Namespace，所以虚拟机东西向流量和虚拟机直接绑定
FloatingIP 的总体系统吞吐会下降，单个实例会加长 IO 路径，但对其自身吞吐影响不大。

#### 其它

##### HA 与 DVR Router 共存的问题

在 Liberty 稳定版本中，虚拟路由器不能同时具有 DVR 和 HA 的属性，一直问题，详见社区
[问题](https://blueprints.launchpad.net/neutron/+spec/dvr-support-ha)描述。

DVR 的高可用是针对与网路节点上的集中式 snat 的单点故障；同时应对需要在一个子网中
的虚拟机内部通过 keepalived 等方法做高可用的特殊需求。

该问题在社区 Mitaka 稳定版中解决，我们尝试进行代码 backport ，但存在较多的代码冲
突。

在当前的部署架构和默认配置下，默认 router 为非 HA，租户在创建 router 时可以选择为 DVR 或 HA。

目前 DVR 的 SNAT 高可用解决方案是采用 Pacemaker 监控网络节点的状况；当检测到某个网络
节点宕机时，会把该网络节点上的 DVR 路由器迁移到其余可用网络节点上，降低 SNAT 的故障时间。

##### HA Router 的使用场景

 * 租户需要在 虚拟机层自己维护 VIP 漂移的场景，并且 VIP 需要跟外部通信。例如，租户
 在同一个子网内的两个虚拟机上起 keepalived 进程，维护一个 VIP，同时需要通过router 的
 NAT 功能使该 VIP 对外提供服务。这种情况下，集中式路由能够在网关上写 NAT 规则就能够实现，
 而分布式路由的 NAT 规则需要随 VIP 漂移，由于 neutron 控制平面对虚拟机内部的 VIP 漂移
 无感知，所以分布式路由无法满足要求，也不能在每个节点上都配上该 VIP 的 NAT 规则，会发生
 IP冲突的问题

 * 租户需要使用 Service VM 做虚拟路由的场景，见
 [云平台上部署山石网科『云界』](https://confluence.ustack.com/pages/viewpage.action?pageId=16098168)
 也需要使用集中式路由

##### VRRP 与 Pacemaker 两种高可用方案的对比

两种方式都是业界广泛采用的 HA 实施方案，单在 L3 HA 这个场景中，设计思路上还是有所不同。

首先，VRRP 的监控方法是对于某个具体的 router 来说的，router 中的 keepalived 互相
同步，更接近真是的虚拟机通信状态，而 Pacemaker 的方法是对整个网络节点的状态监控；

其次，VRRP 的切换是通过 keepalived 主从切换时执行脚本来实现，纯数据平面的操作，不需要控制平面
地参与，而 Pacemaker 是通过迁移的方式来实现切换，在整个过程中的调用栈更长，在目的网络节点上也会
进行更多的操作，故障时间相对于 VRRP 要长。

##### L2 HA 与 L2 Population 共存的问题

L2 Population 是通过预先下发 port 可达性的方法来较少隧道网络 (VxLAN/GRE) 通信过程中可能发生的
组播，提高隧道网络的通信效率。由于采用 VRRP 的方式，主从切换是直接才数据平面发生，控制平面
通过轮询 router 状态的方式需要过一小段时间才能知道切换的信息，进而更新 l2population可达信息。
所以对于隧道网络，L3 HA 发生主从切换时，故障时间会变得稍长(30秒左右)。


##### 部署方式及配置

整体 Service Layout 相对于 Classic 没有发生变化，默认在 neutron.conf 中设置 l3_ha = False，
以避免和 DVR 冲突，租户可以在创建 router 时指定 router 的 HA 属性。

[1]: ../../images/architecture/scenario-classic-ovs-network2.png
[2]: ../../images/architecture/scenario-l3ha-ovs-network2.png
[3]: ../../images/architecture/l3_ha_proposal_dedicated_net.png
[4]: ../../images/architecture/l3-ha-rally.png

