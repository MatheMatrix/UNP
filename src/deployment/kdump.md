## Kdump

---

### 简介

kdump 是一种基于 kexec 的 Linux 内核崩溃捕获机制，能够让 Linux kernel 在系统 crash 的时候产生 crash dumps，在 kernel crash 前产生一个内存镜像，通过工具可以分析 crash 的原因。这个文件是一个 Executable and Linkable Format (ELF) 文件，可以通过 /dev/oldmem raw 设备或 /proc/vmcore 访问。

### 安装配置 Kdump

#### 安装 kdump

```
yum install kexec-tools
```

#### 配置 kdump 保留的最小内存

在 `/etc/grub2.cfg` 中配置当前使用的 kernel 的 `crashkernel` 使用的内存大小，推荐 2 GB 以上内存最小保留内存为 160 MB，每 4 KB RAM 增加 2 bits

```
crashkernel=128M
```

#### 配置 kdump

配置文件 `/etc/kdump.conf`

```
path /var/crash
core_collector makedumpfile -l --message-level 1 -d 31
```

开启服务

```
systemctl enable kdump.service
systemctl start kdump.service
```

### 测试 kdump

检查服务是否开启

```
# systemctl is-active kdump
active
```

强制触发 kernel crash **( 注意：此操作会导致系统触发 crash 并重启，请勿在生产环境尝试 )**

```
echo 1 > /proc/sys/kernel/sysrq
echo c > /proc/sysrq-trigger
```

系统会在 kdump 配置路径 ( 默认 `/var/crash/` 目录 ) 拷贝文件名如：`address-YYYY-MM-DD-HH:MM:SS/vmcore` 的文件。

### 分析 Core Dump

将上面产生的 vmcore 文件放在一个具有相同内核的服务器中进行调试，**不要在生产环境中调试**，同时确保 `/etc/yum.conf` 中 `exclude` 不包含 `kernel`，`/etc/yum.repos.d/CentOS-Debuginfo.repo`中 `base-debuginfo` 部分 `enabled` 为 1。

```
yum install crash
yum install yum-utils
wget -O /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-Debug-7 http://vault.centos.org/RPM-GPG-KEY-CentOS-Debug-7
debuginfo-install kernel
```

**注意：** kernel-debuginfo 在国内无镜像站，可以提前从 [http://debuginfo.centos.org/7/x86_64/](http://debuginfo.centos.org/7/x86_64/) 下载 rpm 包，使用以下替代方式安装：

```
yum install kernel-debuginfo-3.10.0-327.el7.x86_64.rpm kernel-debuginfo-common-x86_64-3.10.0-123.el7.x86_64.rpm
```

使用 `crash` 分析 kernel crash 的原因：

```
crash /usr/lib/debug/lib/modules/3.10.0-327.el7.x86_64/vmlinux vmcore

crash 7.1.2-3.el7_2.1
Copyright (C) 2002-2014  Red Hat, Inc.
Copyright (C) 2004, 2005, 2006, 2010  IBM Corporation
Copyright (C) 1999-2006  Hewlett-Packard Co
Copyright (C) 2005, 2006, 2011, 2012  Fujitsu Limited
Copyright (C) 2006, 2007  VA Linux Systems Japan K.K.
Copyright (C) 2005, 2011  NEC Corporation
Copyright (C) 1999, 2002, 2007  Silicon Graphics, Inc.
Copyright (C) 1999, 2000, 2001, 2002  Mission Critical Linux, Inc.
This program is free software, covered by the GNU General Public License,
and you are welcome to change it and/or distribute copies of it under
certain conditions.  Enter "help copying" to see the conditions.
This program has absolutely no warranty.  Enter "help warranty" for details.

GNU gdb (GDB) 7.6
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-unknown-linux-gnu"...

      KERNEL: /usr/lib/debug/lib/modules/3.10.0-327.el7.x86_64/vmlinux
    DUMPFILE: vmcore  [PARTIAL DUMP]
        CPUS: 40
        DATE: Mon Aug  8 15:02:49 2016
      UPTIME: 45 days, 03:42:44
LOAD AVERAGE: 0.63, 0.43, 0.25
       TASKS: 934
    NODENAME: server-64
     RELEASE: 3.10.0-327.el7.x86_64
     VERSION: #1 SMP Thu Nov 19 22:10:57 UTC 2015
     MACHINE: x86_64  (2299 Mhz)
      MEMORY: 63.8 GB
       PANIC: "SysRq : Trigger a crash"
         PID: 122956
     COMMAND: "bash"
        TASK: ffff88084e667300  [THREAD_INFO: ffff88078f2a0000]
         CPU: 14
       STATE: TASK_RUNNING (SYSRQ)

crash>
```

## 参考文档

* Installing and Configuring Kdump,  https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Crash_Dump_Guide/chap-installing-configuring-kdump.html

* 深入探索 Kdump，第 1 部分：带你走进 Kdump 的世界， https://www.ibm.com/developerworks/cn/linux/l-cn-kdump1/
