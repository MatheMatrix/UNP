# LLDP

---

## 简介

链路层发现协议（Link Layer Discovery Protocol，LLDP）是一种数据链路层协议，
网络设备可以通过在本地网络中发送LLPDU（Link Layer Discovery Protocol Data Unit）
来通告其他设备自身的状态。是一种能够使网络中的设备互相发现并通告状态、交互信息的
协议。

### 封装格式

LLDP协议属于链路层协议，它的以太类型为0x88cc。一个标准的LLDP帧格式如下：

| 目的MAC地址 | 源MAC地址 | 以太类型 | Chassis ID TLV | Port ID TLV | Time to live TLV | 可选 TLV | End of LLDPDU TLV |
|:----------- |:--------- |:-------- |:-------------- |:----------- |:---------------- |:-------- |:----------------- |
| 01-80-C2-00-00-0E 或 01-80-C2-00-00-03 或 01-80-C2-00-00-00 | 源MAC | 0x88cc |   |   |   | 零或多个可选的TLV | 表示LLDP结束 |

##### 目的MAC地址

| 名称 | 目的MAC地址 | 意义 |
|:---- |:----------- |:---- |
| Nearest bridge | 01-80-C2-00-00-0E | 包被限制在本地网络中，无法被任何桥或路由设备转发 |
| Nearest non-TPMR bridge | 01-80-C2-00-00-03 | 包只被Two-Port MAC Relay (TPMR)转发，其他的任何桥或路由设备都不转发该数据包 |
| Nearest Customer Bridge | 01-80-C2-00-00-00 | 只在两个Customer Bridge之间传播 |


## 安装和部署

### 安装

需要安装 lldpd，加载了 UnitedStack 的 yum 源的集群中，可以安装

```
yum install lldpd
```

在公网环境中，需要通过 rpm 包安装，CentOS 7 版的下载地址：

```
http://download.opensuse.org/repositories/home:/vbernat/CentOS_7/x86_64/
```

### 开启服务

```
systemctl restart lldpd
```

## 查询结果

lldp 开启后，能看到与其邻接的设备信息，使用 lldpctl 命令查看，例：

```
# lldpctl
-------------------------------------------------------------------------------
LLDP neighbors:
-------------------------------------------------------------------------------
Interface:    em1, via: LLDP, RID: 3, Time: 0 day, 00:01:42
  Chassis:
    ChassisID:    mac f8:b1:56:77:e2:bb
  Port:
    PortID:       ifname Gi1/0/6
    MDI Power:    supported: yes, enabled: yes, pair control: no
      Device type:  PSE
      Power pairs:  signal
      Class:        unknown
      Power type:   2
      Power Source: PSE
      Power Priority: low
      PD requested power Value: 0
      PSE allocated power Value: 0
-------------------------------------------------------------------------------
Interface:    p3p1, via: LLDP, RID: 2, Time: 0 day, 00:01:50
  Chassis:
    ChassisID:    mac f8:b1:56:73:87:dd
    SysName:      DL4064SW01
  Port:
    PortID:       ifname Te1/0/26
    PortDescr:    Te1/0/26
-------------------------------------------------------------------------------
```
