## vhost-net

---

### 简介

#### virtio-net

virtio 是 qemu 的半虚拟化驱动，Guest 使用 virtio driver 将请求发送给 virtio-backend。在整个路径中，需要进行两次用户态-内核态的拷贝。

![virtio][1]

#### vhost-net

vhost-net 在宿主内核中模拟了 virtio-net，这样不需要 QEMU 在用户台调用昂贵的 system call，避免内存拷贝。vhost-net 目前是 vhost 设备中最成熟的，其他 vhost 的开发例如 vhost-blk、vhost-scsi 的开发时间都要比 vhost-net 新。

![vhost][2]

vhost-net 驱动会创建一个名为 /dev/vhost-net 的字符型设备，当 QEMU 通过` -netdev tap,fd=*,id=hostnet0,vhost=on,vhostfd=* `这样的参数启动时，QEMU 会打开这个设备（你可以通过 lsof -p $PID 查看，$PID 为 QEMU 的进程号）并通过 ioctl 初始化设备。

初始化过程中 vhost 驱动会创建一个内核线程名为 vhost-$PID （$PID 为 QEMU 的进程号），这个线程是 vhost 的工作线程（worker thread），工作线程会始终等待 virtqueue 触发，然后处理 virtqueue 上的 buffer，然后送到 tap 设备的文件描述符。反过来，文件描述符的 polling 也是由工作线程完成的，也就是说 vhost-net 在内核模拟了 tx、rx 队列，而并没有完成整个 virtio PIC 设备的模拟，控制平面例如在线迁移、协商等依旧由 QEMU 实现。

![vhost-2][3]

### 效果

vhost-net 将带来明显好于 virtio-net 的吞吐和延迟，根据官方文档估计，可以提升 8 倍左右的吞吐和至少 10% 的延时的减少。

![latency][4]

### 使用

使用 vhost-net 需要至少 qemu-kvm-0.13 以上，UnitedStack<sup>®</sup> OpenStack 默认使用 qemu-kvm-1.5.3-105。宿主机内核和虚拟机内核需要分别打开 CONFIG_VHOST_NET=y 和 CONFIG_PCI_MSI=y。启动虚拟机时，QEMU 应当有 ` -netdev tap,fd=*,id=hostnet0,vhost=on,vhostfd=* ` 类似的参数。




 [1]: ../../images/system/virtio.png
 [2]: ../../images/system/vhost-net.png
 [3]: ../../images/system/virtio_linux_vhost.png
 [4]: ../../images/system/vhost_virtio_vs.jpg