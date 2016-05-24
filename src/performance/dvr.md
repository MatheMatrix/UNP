## DVR场景下虚拟机的性能

---

### 测试目的
测试在DVR场景下，不同宿主机上的多对虚拟机东西流量性能。本次测试大体分为两个方面：
 - 系统在高负载下的测试情况
 - 系统在正常负载下的测试情况

### 测试拓扑

 ![topology][1]

### 测试配置

#### 物理机配置

|-|机器型号|CPU核数|网卡型号|内存|内核版本|系统版本|
|:--:|:--:|:--:|:--:|:--:|:--:|
|Node A|Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz|32|Intel Corporation Ethernet 10G 2P X520 Adapter|94G|3.10.0-327.13.1.el7.x86_64|CentOS Linux release 7.2.1511 (Core)|
|Node B|Intel(R) Xeon(R) CPU E5-2630 v3 @ 2.40GHz|32| Intel Corporation Ethernet 10G 2P X520 Adapter|94G|3.10.0-327.13.1.el7.x86_64|CentOS Linux release 7.2.1511 (Core)|

#### 虚拟机配置

|Memory_MB | Disk | Ephemeral| VCPUs|
|:--:|:--:|:--:|:--:|:--:|
|4096      | 40   | 0        | 4     |

`说明：所有测试使用的虚拟机配置均相同`

### 测试工具
  - [iPerf](https://iperf.fr/)
  - [Stress](../stability/stress.md)

### 测试结果

 ![dvr_stress][2]


`说明：测试中的一对一指的是同时有一对虚拟机进行 Iperf 测试，依次类推。`

### 关于如何达到本次测试用到的系统负载

 本次测试通过使用[Stress](../stability/stress.md)给系统增加负载，其中，系统负载通过[uptime](http://linux.die.net/man/1/uptime)命令得到。
具体的负载数据和 stress 的命令如下：

|--|系统负载|
|:--:|:--:|
|0 works|2.53|
|5 works|31.26|
|20 works|47.27|
|30 works|53.99|
|50 works|67.46|
|80 works|100.96|

`说明：使用到的 Stress 命令：stress --cpu 20 --io 4 --vm [workers] -vm-bytes 1280M --timeout 10d 。
本次使用到的测试机器是 32 核心，当系统负载为 31.26 时，每颗 CPU 接近满负载（31.26 / 32 * 100% = 97%）。`

### 测试结论

 从本次测试可以得出初步得出以下几点结论：
 - 虚拟机的东西流量性能随着系统负载的增加，而明显下降
 - 在多组虚拟机（小于等于五组）进行 IPerf 流量测试时，每组虚拟机的性能不会明显下降




 [1]: ../../images/performance/topology.png
 [2]: ../../images/performance/dvr_stress.png
