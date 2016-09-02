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
default reboot
```

开启服务

```
systemctl enable kdump.service
systemctl start kdump.service
```

**注意：重启系统后 kdump 生效**

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

### kdump.conf

```
# Configures where to put the kdump /proc/vmcore files
#
# This file contains a series of commands to perform (in order) when a
# kernel crash has happened and the kdump kernel has been loaded.  Directives in
# this file are only applicable to the kdump initramfs, and have no effect if
# the root filesystem is mounted and the normal init scripts are processed
#
# Currently only one dump target and path may be configured at once
# if the configured dump target fails, the default action will be preformed
# the default action may be configured with the default directive below.  If the
# configured dump target succedes
#
# Basics commands supported are:
# raw <partition>	- Will dd /proc/vmcore into <partition>.
#			  Use persistent device names for partition devices,
#			  such as /dev/vg/<devname>.
#
# nfs <nfs mount>	- Will mount fs and copy /proc/vmcore to
#			  <mnt>/var/crash/%HOST-%DATE/, supports DNS.
#
# ssh <user@server>	- Will scp /proc/vmcore to
#			  <user@server>:/var/crash/%HOST-%DATE/, supports DNS
#			  NOTE: make sure user has necessary write
#			  permissions on server
#
# sshkey <path>		- Will use the sshkey to do ssh dump
#			  Specifies the path of the ssh key you want to use
#			  when do ssh dump, the default value is
#			  /root/.ssh/kdump_id_rsa.
#
# <fs type> <partition> - Will mount -t <fs type> <partition> /mnt and copy
#		 	  /proc/vmcore to /mnt/var/crash/%DATE/.
#			  NOTE: <partition> can be a device node, label or uuid.
#			  It's recommended to use persistent device names
#			  such as /dev/vg/<devname>.
#			  Otherwise it's suggested to use label or uuid.
#
# path <path> 		- "path" represents the file system path in which
#                         vmcore will be saved. If a dump target is specified
#                         in kdump.conf, then "path" is relative to the
#                         specified dump target. Interpretation of path
#                         changes a bit if user has not specified a dump
#                         target explicitly in kdump.conf. In this case,
#                         "path" represents the absolute path from root.
#                         And dump target and adjusted path are arrived
#                         at automatically depending on what's mounted
#                         in the current system.
#                         Ignored for raw device dumps.  If unset, will
#                         default to /var/crash.
#
# core_collector <command> <options>
#			- This allows you to specify the command to copy
#			  the vmcore.  You could use the dump filtering
#			  program makedumpfile, the default one, to retrieve
#			  your core, which on some arches can drastically
#			  reduce core file size. See /sbin/makedumpfile --help
#			  for a list of options. Note that the -i and -g
#			  options are not needed here, as the initrd will
#			  automatically be populated with a config file
#			  appropriate for the running kernel.
#			  Default core_collector for raw/ssh dump is:
#			  "makedumpfile -F -l --message-level 1 -d 31".
#			  Default core_collector for other targets is:
#			  "makedumpfile -l --message-level 1 -d 31".
#			  For core_collector format details please refer to
#			  kexec-kdump-howto.txt or kdump.conf manpage.
#
# kdump_post <binary | script>
# 			- This directive allows you to run a specified
# 			  executable just after the memory dump process
# 			  terminates. The exit status from the dump process
# 			  is fed to the kdump_post executable, which can be
# 			  used to trigger different actions for success or
# 			  failure.
#
# kdump_pre <binary | script>
#			- works just like the kdump_post directive, but instead
#			  of running after the dump process, runs immediately
#			  before.  Exit status of this binary is interpreted
#			  as follows:
#			  0 - continue with dump process as usual
#			  non 0 - reboot the system
#
# extra_bins <binaries | shell scripts>
# 			- This directive allows you to specify additional
# 			  binaries or shell scripts you'd like to include in
# 			  your kdump initrd. Generally only useful in
# 			  conjunction with a kdump_post binary or script that
# 			  relies on other binaries or scripts.
#
# extra_modules <module(s)>
# 			- This directive allows you to specify extra kernel
# 			  modules that you want to be loaded in the kdump
# 			  initrd, typically used to set up access to
# 			  non-boot-path dump targets that might otherwise
# 			  not be accessible in the kdump environment. Multiple
# 			  modules can be listed, separated by a space, and any
# 			  dependent modules will automatically be included.
#
# default <reboot | halt | poweroff | shell | dump_to_rootfs>
#			- Action to preform in case dumping to intended target
#			  fails. If no default action is specified, "reboot"
#			  is assumed default.
#			  reboot: If the default action is reboot simply reboot
#				  the system and loose the core that you are
#				  trying to retrieve.
#			  halt:   If the default action is halt, then simply
#				  halt the system after attempting to capture
#				  a vmcore, regardless of success or failure.
#			  poweroff: The system will be powered down
#			  shell:  If the default action is shell, then drop to
#				  an shell session inside the initramfs from
#				  where you can try to record the core manually.
#				  Exiting this shell reboots the system.
#				  Note: kdump uses bash as the default shell.
#			  dump_to_rootfs: If non-root dump target is specified,
#				  the default action can be set as dump_to_rootfs.
#				  That means when dump to target fails, dump vmcore
#				  to rootfs from initramfs context and reboot.
#
# force_rebuild <0 | 1>
#			- By default, kdump initrd only will be rebuilt when
#			  necessary. Specify 1 to force rebuilding kdump
#			  initrd every time when kdump service starts.
#
#override_resettable <0 | 1>
#			- Usually a unresettable block device can't be dump target.
#			Specifying 1 means though block target is unresettable, user
#			understand this situation and want to try dumping. By default,
#			it's set to 0, means not to try a destined failure.
#
# dracut_args <arg(s)>
#			- Pass extra dracut options when rebuilding kdump
#			  initrd.
#
# fence_kdump_args <arg(s)>
#			- Command line arguments for fence_kdump_send (it can contain
#			all valid arguments except hosts to send notification to).
#
# fence_kdump_nodes <node(s)>
# 			- List of cluster node(s) separated by space to send fence_kdump
# 			notification to (this option is mandatory to enable fence_kdump).
#

#raw /dev/vg/lv_kdump
#ext4 /dev/vg/lv_kdump
#ext4 LABEL=/boot
#ext4 UUID=03138356-5e61-4ab3-b58e-27507ac41937
#nfs my.server.com:/export/tmp
#ssh user@my.server.com
#sshkey /root/.ssh/kdump_id_rsa
path /var/crash
core_collector makedumpfile -l --message-level 1 -d 31
#core_collector scp
#kdump_post /var/crash/scripts/kdump-post.sh
#kdump_pre /var/crash/scripts/kdump-pre.sh
#extra_bins /usr/bin/lftp
#extra_modules gfs2
#default shell
#force_rebuild 1
#dracut_args --omit-drivers "cfg80211 snd" --add-drivers "ext2 ext3"
#fence_kdump_args -p 7410 -f auto -c 0 -i 10
#fence_kdump_nodes node1 node2
```

## 参考文档

[Installing and Configuring Kdump](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Crash_Dump_Guide/chap-installing-configuring-kdump.html)

[深入探索 Kdump，第 1 部分：带你走进 Kdump 的世界](https://www.ibm.com/developerworks/cn/linux/l-cn-kdump1/)
