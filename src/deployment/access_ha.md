## 接入层高可用

### 简介

为提供更为稳定的云服务，UOS3.0建议客户服务器及交换机采用全冗余架构（服务器只有一个IMPI网卡，因此IPMI没有冗余）
以下是服务器冗余网卡说明：

![arch][1]
 
###交换机设备选型

为提供接入交换机的冗余能力，我们强烈建议服务器使用链路聚合做双上联，理论上所有支持vpc或支持堆叠的交换机都可以使用，如VPC（思科）、CSS（华为）、IRF（华三）、VLT or stacking（Dell）。

 - 核心交换机：推荐使用Dell S6000-ON
 - 万兆接入：推荐使用Dell S4048-ON
 - 千兆接入：推荐使用Dell S3048-ON
 
###交换机冗余架构图示

物理连接图

![phy][2]
 
逻辑架构图

![logical][3]

以下是各节点接入图示
控制节点接入图示

![control][4]

计算节点接入图示

![compute][5]

网络节点接入图示

![network][6]

存储节点接入图示

![storage][7]
 
交换机堆叠设置（以S6000为例：
S6000共32个40G端口，每个端口都可以作为一个stack-group，本例中以 0  1  30  31口作为堆叠口，共需4跟40G线缆（普通40G线缆即可，无需专用堆叠线）。
配置如下：
```
stack-unit 0 stack-group 0
stack-unit 0 stack-group 1
stack-unit 0 stack-group 30
stack-unit 0 stack-group 31
```

配置完成以后重启交换机，先启动的交换机作为master设备，后启动的交换机作为slave设备。

[1]: ../../images/deployment/QQ20160614-1@2x.png
[2]: ../../images/deployment/image2016-6-13_11-10-29.png
[3]: ../../images/deployment/image2016-6-13_11-7-12.png
[4]: ../../images/deployment/image2016-6-13_10-39-33.png
[5]: ../../images/deployment/image2016-6-13_10-46-40.png
[6]: ../../images/deployment/image2016-6-13_10-47-5.png
[7]: ../../images/deployment/image2016-6-13_10-47-20.png