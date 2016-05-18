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

由于网络拓普架构并没有变化，所以性能相对 Legacy 并没有太大的变化


[1]: ../../images/architecture/scenario-classic-ovs-network2.png
[2]: ../../images/architecture/scenario-l3ha-ovs-network2.png
[3]: ../../images/architecture/l3_ha_proposal_dedicated_net.png
[4]: ../../images/architecture/l3-ha-rally.png

