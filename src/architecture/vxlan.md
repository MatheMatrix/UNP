# VxLan

---


### Overlay 网络
  Overlay 在网络技术领域，指的是一种网络架构上叠加的虚拟化技术模式，
其大体框架是对基础网路不进行大规模修改的条件下，实现应用在网络上的承载，
并能与其他网络业务隔离，并且以基于 IP 的基础网络技术为主。Overlay 技术
是在现有的物理网络之上构建一个虚拟网络，上层应用只与虚拟网络有关。
一个 Overlay 网络主要由三部分组成：

 - 边缘设备：指与虚拟机直接相连的设备
 - 控制平面：主要负责虚拟隧道的建立维护以及主机可达信息的通告
 - 转发平面：指承载 Overlay 报文的物理网络

### Overlay 主要技术标准和比较
  当前主流的 Overlay 技术主要有 VXLAN，GRE/NVGRE 和 STT。这三种二层 Overlay 
技术，大体思路都是将以太网报文承载到某种隧道层面，差异性在于选择和构造隧道
的不同，而底层均是 IP 转发。

 - VXLAN。 VXLAN 是将以太网报文封装在 UDP 传输层上的一种隧道转发模式，目的 UDP 端口号为4798；
为了使 VXLAN 充分利用承载网络路由的均衡性， VXLAN 通过将原始以太网数据头(MAC、IP、四层端口号等)的
 HASH 值作为 UDP 的号；采用24比特标识二层网络分段，称为 VNI(VXLAN Network Identifier)，
类似于 VLAN ID 作用；未知目的、广播、组播等网络流量均被封装为组播转发，
物理网络要求支持任意源组播(ASM)。
 - NVGRE。 NVGRE 是将以太网报文封装在 GRE 内的一种隧道转发模式；采用24比特标识二层网络分段，
称为 VSI(Virtual Subnet Identifier)，类似于 VLAN ID 作用；为了使 NVGRE 利用承载网络路由的均衡性，
 NVGRE 在GRE扩展字段 flow ID，这就要求物理网络能够识别到 GRE 隧道的扩展信息，
并以 flow ID 进行流量分担；未知目的、广播、组播等网络流量均被封装为组播转发。
 - STT。STT 利用了 TCP 的数据封装形式，但改造了 TCP 的传输机制，数据传输不遵循 TCP 状态机，
而是全新定义的无状态机制，将 TCP 各字段意义重新定义，无需三次握手建立 TCP 连接，因此称为无状态 TCP；
以太网数据封装在无状态 TCP；采用64比特 Context ID 标识二层网络分段；为了使 STT 充分利用承载
网络路由的均衡性，通过将原始以太网数据头(MAC、IP、四层端口号等)的 HASH 值作为无状态 TCP 的源端口号；
未知目的、广播、组播等网络流量均被封装为组播转发。

上述三种 Overlay 技术的比较:

|技术名称|支持者|支持方式|网络虚拟化方式|数据新增报文长度|链路HASH能力|
|:--:|:--:|:--:|:--:|:--:|:--:|
|VXLAN|Cisco/VMWARE/Citrix/Red Hat/Broadcom|L2 over UDP|VXLAN报头 24 bit VNI|50Byte(+原数据)|现有网络可进行L2 ~ L4 HASH|
|NVGRE|HP/Microsoft/Broadcom/Dell/Intel|L2 over GRE|NVGRE 报头 24 bit VSI|42Byte(+原数据)|GRE头的HASH 需要网络升级|
|STT|VMWare|无状态TCP，即L2在类似TCP的传输层|STT报头 64 bit Context ID|58 ~ 76Byte(+原数据)|现有网络可进行 L2 ~ L4 HASH|


### VXLAN 的实现

VXLAN 将二层数据帧封装成 UDP 包

 ![vxlan][1]

- 含义：

 - Outer MAC destination address (MAC address of the tunnel endpoint VTEP)
 - Outer MAC source address (MAC address of the tunnel source VTEP)
 - Outer IP destination address (IP address of the tunnel endpoint VTEP)
 - Outer IP source address (IP address of the tunnel source VTEP)
 - Outer UDP header：Src port 往往用于 load balancing，下文有提到；Dst port 即 VXLAN Port，默认值为 4789
 - VNID：表示该帧的来源虚机所在的 VXLAN 网段的 ID
- 特点：
 
 - VNID： 24-bits，最大 16777216。每个不同的 24-bits VNI 代表一个 VXLAN 网段。只有同一个网段中的虚机才能互相通信
 - VXLAN Port：目的 UDP 端口，默认使用 4789 端口。用户可以自己配置
 - 两个 VTEP 之间的 VXLAN tunnels 是无状态的
 - VTEP 可以在虚拟交换机上，物理交换机或者物理服务器上通过软件或者硬件实现
 - 使用多播来传送未知目的的、广播或者多播帧
 - VTEP 不可以对 VXLAN 包分段


### Neutron 对 VXLAN 的支持
  Neutron 通过 OpenvSwitch （以下简称 OVS）支持 VXLAN。OVS 在计算节点/网路节点的 br-tun 上建立多个 tunnel port ，
和其他节点的 tunnel port 之间建立点对点的 VXLAN Tunnel。 Tunnel Port 在 OVS 上的形式如下：

```
Bridge br-tun
    fail_mode: secure
    Port patch-int
        Interface patch-int
            type: patch
            options: {peer=patch-tun}
    Port "vxlan-c0a86444"
        Interface "vxlan-c0a86444"
            type: vxlan
            options: {df_default="true", in_key=flow,
                local_ip="192.168.100.233", out_key=flow,
                remote_ip="192.168.100.68"}
```

Neutron VXLAN network 通过使用 segmentation_id 作为VXLAN网络标识（VNI），类似于 VLAN ID，
来实现不同网络流量之间的隔离。



References：
1. [Data Center Overlay Technologies](http://www.cisco.com/c/en/us/products/collateral/switches/nexus-9000-series-switches/white-paper-c11-730116.html)
2. [Open vSwitch + GRE/VXLAN 组网](http://www.cnblogs.com/sammyliu/p/4627230.html)
3. [基于多租户的云计算Overlay网络](http://www.h3c.com.cn/About_H3C/Company_Publication/IP_Lh/2013/04/Home/Catalog/201309/796466_30008_0.htm)

[1]: ../../images/architecture/vxlan.jpg
