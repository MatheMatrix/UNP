## Agent重启对数据层的影响

---

### 简介

 对数据平面有潜在影响的 Neutron Agent
 
- Neutron L3 Agent
- Neutron OpenvSwitch Agent

#### 重启Neutron L3 Agent

Neutron L3 Agent 主要负责虚拟路由器的生命周期管理，绑定的所有公网 IP 都会通过虚拟路由器实现。
下面通过实验测试当虚拟机打流时，重启 Neutron L3 Agent 是否会造成流量中断。

##### 测试步骤

1. 创建虚拟机，并绑定 FloatingIP；
2. 在外部通过脚本 Ping 该 FloatingIP；
   测试脚本如下：

   ``` 
   #!/bin/bash

   while true
   cmd=$(ping 2.2.100.102 -c1 -w3|grep "time=")
   do
   sleep 1
   if [[ "bad"$cmd"bad" == "badbad" ]]; then
       echo `date` "ping lost" >> ping.history.log
   else
       echo `date` ${cmd} >> ping.history.log
   fi
   done
   ```

   `说明：本脚本通过每隔1秒 Ping FloatingIP ，超时时间3秒，如果丢包输出相关日志到指定文件，
    通过最后的日志文件观察是否有丢包现象。`

3. 重启 Neutron L3 Agent，观察脚本的输出

#####测试结果
 截取部分执行上述脚本后输出的日志文件，如下：
```
...
2016年 05月 22日 星期日 20:36:07 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.193 ms
2016年 05月 22日 星期日 20:36:08 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.189 ms
2016年 05月 22日 星期日 20:36:09 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.205 ms
2016年 05月 22日 星期日 20:36:10 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.181 ms
2016年 05月 22日 星期日 20:36:11 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.178 ms
2016年 05月 22日 星期日 20:36:12 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.123 ms
2016年 05月 22日 星期日 20:36:13 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.151 ms
2016年 05月 22日 星期日 20:36:14 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.181 ms
2016年 05月 22日 星期日 20:36:15 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.202 ms
2016年 05月 22日 星期日 20:36:16 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.291 ms
...
```

`说明：在上面的日志输出中之所以 icmp_seq=1 ，是因为在测试脚本中是每此只发一个 Ping 请求导致的。
 我们之所以使用上述这种方式测试 ICMP 的连通性，是因为 Linux Kernel 中的基于 Netfilter 的 conntrack 机制。
详细的资料可以参考(https://en.wikipedia.org/wiki/Netfilter)`

##### 测试结论
重启 Neutron L3 agent 时，不会造成 FloatingIP 的流量中断。


#### 重启Neutron OpenvSwitch Agent

 Neutron OpenvSwitch Agent（以下简称OVS Agent）负责虚拟机的所有通信。在以往的 Neutron 版本中，
当重启OVS Agent，都会重新刷新流表，在刷新流表时必然会造成该节点的所有虚拟机的任何流量中断，这是不可接受的。
对于该问题，我们通过动态维护 VLAN 的形式解决这一问题。
在L版 Neutron 代码中，OVS Agent 通过配置文件控制在重启 OVS Agent 时是否重新刷新流表。如无特殊用处，该选项必须关闭。
这是该选项：

> drop_flows_on_start=False

但是在实验过程中，当 drop_flows_on_start=False 时，此时重启 OVS Agent 时也造成虚拟机流量中断，这是由于重新
生成了 OVS Patch Port 导致的，代码如下：

    
    def setup_integration_br(self):
        '''Setup the integration bridge.
        '''
        self.int_br.set_agent_uuid_stamp(self.agent_uuid_stamp)
        # Ensure the integration bridge is created.
        # ovs_lib.OVSBridge.create() will run
        #   ovs-vsctl -- --may-exist add-br BRIDGE_NAME
        # which does nothing if bridge already exists.
        self.int_br.create()
        self.int_br.set_secure_mode()
        self.int_br.setup_controllers(self.conf)

        self.int_br.delete_port(self.conf.OVS.int_peer_patch_port)
        if self.conf.AGENT.drop_flows_on_start:
            self.int_br.delete_flows()
        self.int_br.setup_default_table()
    
由上面的代码可以看出，当设置 drop_flows_on_start=False ，会删除Patch Port。因此导致了测试中虚拟机流量的中断。
该 [Bug](https://bugs.launchpad.net/neutron/+bug/1514056) 在 Launchpad 上注册，Bug fix
的 [patch](https://review.openstack.org/#/c/299243/) 已 merge 到了Liberty 分支的代码中。

##### 测试步骤
1. 在两台计算节点上分别创建一台虚拟机；
2. 在其中一台虚拟机对另一台虚拟机进行 iPerf 测试；
3. 设置 drop_flows_on_start=True 
4. 同时重启两台计算节点的 Neutron OpenvSwitch Agent，观察 iPerf 流量是否有中断或者速率减少的现象。 
5. 设置 drop_flows_on_start=False
6. 同时重启两台计算节点的 Neutron OpenvSwitch Agent，观察 iPerf 流量是否有中断或者速率减少的现象。 

##### 测试结果
- 设置 drop_flows_on_start=True时 iPerf 测试结果

``` 
root@vm-233-111:~# iperf -c 10.10.10.5 -i 1 -t 100
------------------------------------------------------------
Client connecting to 10.10.10.5, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.10.4 port 36065 connected with 10.10.10.5 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec   234 MBytes  1.97 Gbits/sec
[  3]  1.0- 2.0 sec   208 MBytes  1.74 Gbits/sec
[  3]  2.0- 3.0 sec   301 MBytes  2.53 Gbits/sec
[  3]  3.0- 4.0 sec   261 MBytes  2.19 Gbits/sec
[  3]  4.0- 5.0 sec   304 MBytes  2.55 Gbits/sec
[  3]  5.0- 6.0 sec   133 MBytes  1.11 Gbits/sec
[  3]  6.0- 7.0 sec  22.5 MBytes   189 Mbits/sec
[  3]  7.0- 8.0 sec  0.00 Bytes  0.00 bits/sec
[  3]  8.0- 9.0 sec  0.00 Bytes  0.00 bits/sec
[  3]  9.0-10.0 sec   183 MBytes  1.53 Gbits/sec
[  3] 10.0-11.0 sec   212 MBytes  1.78 Gbits/sec
[  3] 11.0-12.0 sec   188 MBytes  1.58 Gbits/sec
[  3] 12.0-13.0 sec   224 MBytes  1.87 Gbits/sec
[  3] 13.0-14.0 sec   313 MBytes  2.62 Gbits/sec

```


- 设置 drop_flows_on_start=False 时 iPerf 测试结果


```
root@vm-233-111:~# iperf -c 10.10.10.5 -i 1 -t 100
------------------------------------------------------------
Client connecting to 10.10.10.5, TCP port 5001
TCP window size: 45.0 KByte (default)
------------------------------------------------------------
[  3] local 10.10.10.4 port 36067 connected with 10.10.10.5 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 1.0 sec   198 MBytes  1.66 Gbits/sec
[  3]  1.0- 2.0 sec   206 MBytes  1.72 Gbits/sec
[  3]  2.0- 3.0 sec   228 MBytes  1.91 Gbits/sec
[  3]  3.0- 4.0 sec   125 MBytes  1.05 Gbits/sec
[  3]  4.0- 5.0 sec   178 MBytes  1.49 Gbits/sec
[  3]  5.0- 6.0 sec   326 MBytes  2.73 Gbits/sec
[  3]  6.0- 7.0 sec   229 MBytes  1.92 Gbits/sec
[  3]  7.0- 8.0 sec   314 MBytes  2.63 Gbits/sec
[  3]  8.0- 9.0 sec   318 MBytes  2.67 Gbits/sec
[  3]  9.0-10.0 sec   309 MBytes  2.59 Gbits/sec
[  3] 10.0-11.0 sec   310 MBytes  2.60 Gbits/sec
[  3] 11.0-12.0 sec   300 MBytes  2.52 Gbits/sec
[  3] 12.0-13.0 sec   327 MBytes  2.74 Gbits/sec
[  3] 13.0-14.0 sec   332 MBytes  2.78 Gbits/sec
[  3] 14.0-15.0 sec   194 MBytes  1.63 Gbits/sec
[  3] 15.0-16.0 sec   200 MBytes  1.68 Gbits/sec
[  3] 16.0-17.0 sec   262 MBytes  2.20 Gbits/sec
[  3] 17.0-18.0 sec   228 MBytes  1.91 Gbits/sec
```

- 设置 drop_flows_on_start=True 时 Ping 测试结果

```
...
2016年 05月 22日 星期日 20:55:28 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.176 ms
2016年 05月 22日 星期日 20:55:29 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.152 ms
2016年 05月 22日 星期日 20:55:30 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.196 ms
2016年 05月 22日 星期日 20:55:31 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.191 ms
2016年 05月 22日 星期日 20:55:32 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.193 ms
2016年 05月 22日 星期日 20:55:33 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.182 ms
2016年 05月 22日 星期日 20:55:34 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.219 ms
2016年 05月 22日 星期日 20:55:35 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.185 ms
2016年 05月 22日 星期日 20:55:36 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.129 ms
2016年 05月 22日 星期日 20:55:37 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.190 ms
2016年 05月 22日 星期日 20:55:38 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.141 ms
2016年 05月 22日 星期日 20:55:39 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.135 ms
2016年 05月 22日 星期日 20:55:40 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.173 ms
2016年 05月 22日 星期日 20:55:41 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.124 ms
2016年 05月 22日 星期日 20:55:42 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.199 ms
2016年 05月 22日 星期日 20:55:43 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.277 ms
...

```

- 设置 drop_flows_on_start=False 时 Ping 测试结果

```
...
2016年 05月 22日 星期日 20:56:17 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.476 ms
2016年 05月 22日 星期日 20:56:18 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.215 ms
2016年 05月 22日 星期日 20:56:19 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.338 ms
2016年 05月 22日 星期日 20:56:20 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.228 ms
2016年 05月 22日 星期日 20:56:21 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.262 ms
2016年 05月 22日 星期日 20:56:22 CST 64 bytes from 2.2.100.102: icmp_seq=1 ttl=62 time=0.167 ms
2016年 05月 22日 星期日 20:56:24 CST 64 bytes from 2.2.100.102: icmp_seq=2 ttl=62 time=0.316 ms
...
```

##### 测试结论

1. 当 drop_flows_on_start=False 或者 True 时候，此时重启 OVS Agent 时不会造成 FloaingIP Ping 丢包。
2. 当 drop_flows_on_start=True 时，此时重启 OVS Agent 时，由于要重新生成流表规则，因此导致
虚拟机流量中断。
