### Linux Packet Generator

----

#### Pktgen 简介

  pktgen 是一个位于 Linux 内核层的高性能网络发包工具。主要用来测试网络驱动和
网卡设备，支持多线程，能够产生随机 MAC 地址、IP 地址、UDP 端口号的数据包。

#### Pktgen 使用

##### 加载内核模块

  通过命令 `modprobe pktgen` 加载 pktgen 内核模块，加载成功后，通过命令 `lsmod |grep pktgen`
检查是否加载成功，如下：

```
[root@localhost ~(keystone_admin)]# lsmod |grep pktgen
pktgen                 49089  0
```
 
  加载成功后，在目录 `/proc/net/` 下会生成 `pktgen` 文件夹，该文件夹下会有若干线程配置文件，文件
个数和 CPU 核心数相同，如下（测试机器为 32 颗 CPU）：

```
[root@localhost ~(keystone_admin)]# cat /proc/net/pktgen/
kpktgend_0   kpktgend_11  kpktgend_14  kpktgend_17  kpktgend_2   kpktgend_22  kpktgend_25  
kpktgend_28  kpktgend_30  kpktgend_5   kpktgend_8
kpktgend_1   kpktgend_12  kpktgend_15  kpktgend_18  kpktgend_20  kpktgend_23  kpktgend_26  
kpktgend_29  kpktgend_31  kpktgend_6   kpktgend_9
kpktgend_10  kpktgend_13  kpktgend_16  kpktgend_19  kpktgend_21  kpktgend_24  kpktgend_27  
kpktgend_3   kpktgend_4   kpktgend_7   pgctrl
```

##### 基本使用

  为了方便配置，定义如下工具函数：

```
pgset() {
	local result
	echo $1 > $PGDEV
	result=`cat $PGDEV | fgrep "Result: OK:"`
	if [ "$result" = "" ]; then
		cat $PGDEV | fgrep Result:
	fi
}

pg() {
	echo inject > $PGDEV
	cat $PGDEV
}
```

  绑定 pktgen 的 0 号线程到 eth0 网卡，并发包

```
PGDEV=/proc/net/pktgen/kpktgend_0
  echo "Removing all devices"
 pgset "rem_device_all"
  echo "Adding eth0"
 pgset "add_device eth0"


# device config
# delay 0 means maximum speed.

CLONE_SKB="clone_skb 1000000"
# NIC adds 4 bytes CRC
PKT_SIZE="pkt_size 1420"

# COUNT 0 means forever
COUNT="count 0"
DELAY="delay 0"

PGDEV=/proc/net/pktgen/eth0
  echo "Configuring $PGDEV"
 pgset "$COUNT"
 pgset "$CLONE_SKB"
 pgset "$PKT_SIZE"
 pgset "$DELAY"
 pgset "dst 130.140.25.121"
 pgset "dst_mac fa:16:3e:fd:6a:5d"

# Time to run
PGDEV=/proc/net/pktgen/pgctrl

 echo "Running... ctrl^C to stop"
 pgset "start"
 echo "Done"
```

##### 观察发包情况

  通过 `sar` 命令观察发包情况：

```
平均时间     IFACE   	    rxpck/s   	    txpck/s    rxkB/s         txkB/s    rxcmp/s   txcmp/s   rxmcst/s
11时43分17秒 tap33d6c75c-b8 1471258.00      1.00       2040221.05      0.58      0.00      0.00      0.00
11时43分18秒 tap33d6c75c-b8 925928.00       1.00       1284001.72      0.58      0.00      0.00      0.00
11时43分19秒 tap33d6c75c-b8 1270854.00      1.00       1762317.07      0.58      0.00      0.00      0.00
11时43分20秒 tap33d6c75c-b8 1513016.00      1.00       2098127.66      0.58      0.00      0.00      0.00
```

##### References

1. [HOWTO for the linux packet generator](https://www.kernel.org/doc/Documentation/networking/pktgen.txt)
2. [pktgen](https://wiki.linuxfoundation.org/networking/pktgen)
