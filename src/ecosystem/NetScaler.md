## NetScaler 和 OpenStack 集成

---

### 支持
LBaaS V1 Driver 支持openstack horizon，LBaas V2只支持命令行，不支持horizon。

特性列表

| LBaas V1 | LBaas V1 |
|:--------|:--------|
| Load Balancing |Load Balancing | 
|  | SSL Offload with certificates managed by Barbican	| 
|  |Certificate Bundles (includes intermediary Certification Authorities) | 
| | SNI support|

openstack硬件需求

| Component | Requirement |
|:--------|:--------|
| Virtual Network Interfaces |1 | 
| Virtual CPU | 8	| 
| Throughput |1 Gbps or 100 Mbps | 
| Storage space| 500GB|
|RAM | 8GB|

###部署

详细的部署链接请访问：

[MAS部署](http://docs.citrix.com/en-us/netscaler-mas/11-1/integrating-netscaler-mas-with-openstack-platform.html)

[VPS部署](http://docs.citrix.com/en-us/netscaler/11-1/deploying-vpx/install-vpx-on-kvm/provision-on-kvm-using-openstack.html)

[MAS下载](https://www.citrix.com/downloads/netscaler-mas.html)

[VPS下载](https://www.citrix.com/downloads/netscaler-gateway/product-software/netscaler-gateway-111-build-4714.html)

####安装MAS

也可以参考[链接](http://docs.citrix.com/en-us/netscaler/11-1/deploying-vpx/install-vpx-on-kvm/provision-on-kvm-using-virsh.html)

1. 将下载的压缩包上传到openstack控制节点中
2. 将压缩包解压
    
    `tar -zxvf MAS-KVM-11.1-47.14.tgz`

3. 修改HTML文件
   ```
   <?xml version="1.0" encoding="UTF-8"?>
<domain type='kvm'>
  <name>MAS2</name>
  <memory>8388608</memory>
  <vcpu>4</vcpu>
  <os>
    <type arch='x86_64'>hvm</type>
    <boot dev='hd'/>
  </os>
  <features>
    <acpi/>
    <apic/>
    <pae/>
  </features>
  <clock offset='utc'/>
  <on_poweroff>destroy</on_poweroff>
  <on_reboot>restart</on_reboot>
  <on_crash>restart</on_crash>
  <devices>
    <disk type='file' device='disk'>
      <driver name='qemu' type='qcow2'/>
      <source file='/home/image/MAS-KVM-11.1-47.14.qcow2'/>
      <target dev='hda' bus='ide'/>
    </disk>
    <controller type='ide' index='0'>
    </controller>
    <interface type='bridge'>
      <mac address='52:54:00:29:74:b4'/>
      <source bridge='br0'/>
      <target dev='vnet1' />
      <model type='virtio'/>
    </interface>
    <serial type='pty'/>
    <console type='pty'/>
    <graphics type='vnc' port='-1' autoport='yes' listen='0.0.0.0' keymap='en-us'>
      <listen type='address' address='0.0.0.0'/>
    </graphics>
  </devices>
</domain>
   ```
4. 启用虚机

   ```
   virsh define MAS2-KVM.xml
   virsh start MAS2
   ```

5. 初始化虚机

   初始化更详细步骤请参考:[http://docs.citrix.com/en-us/netscaler-mas/11-1/how-to-migrate-netscaler-mas-single-server-to-ha.html](http://docs.citrix.com/en-us/netscaler-mas/11-1/how-to-migrate-netscaler-mas-single-server-to-ha.html)
   
   按1进入MAS server配置
   
   ![][1]
   
   配置地址、掩码和网关即可，最后保存
   
   ![][2]
   
6. 配置完成
   ![][3]
   
###安装VPX

  
1. 将下载好的VPX压缩包上传到openstack控制节点
2. 解压

   `tar -zxvf NSVPX-KVM-11.1-47.14_nc.tgz`
   
3. 创建glance镜像
   ```
   [root@localhost ~(keystone_admin)]# glance image-create --name="NS-VPX-11" --property hw_disk_bus=ide  --container-format=bare --progress --disk-format=qcow2 --file /tmp/NSVPX-KVM-11.1-47.14_nc.qcow2
[=============================>] 100%
+------------------+--------------------------------------+
| Property         | Value                                |
+------------------+--------------------------------------+
| checksum         | 3ef4346089aae24e60c2df0ec562720d     |
| container_format | bare                                 |
| created_at       | 2016-07-13T03:42:42Z                 |
| disk_format      | qcow2                                |
| hw_disk_bus      | ide                                  |
| id               | 4f6bccf1-f652-4e89-8ca8-09b8d494e5fe |
| min_disk         | 0                                    |
| min_ram          | 0                                    |
| name             | NS-VPX-11                            |
| owner            | ab2105c189734f6292f30492eb97ae60     |
| protected        | False                                |
| size             | 527794176                            |
| status           | active                               |
| tags             | []                                   |
| updated_at       | 2016-07-13T03:42:44Z                 |
| virtual_size     | None                                 |
| visibility       | private                              |
+------------------+--------------------------------------+
[root@localhost ~(keystone_admin)]#
```

4. 在openstack中创建VPX实例
   ![][4]
5. 创建成功
   ![][5]

###MAS 和 openstack 集成

####MAS上操作

1. 在MAS上进入Orchestration > Cloud Orchestration > Cloud Platform
2. 在下拉菜单中选择openstack，然后选择Get Started
   ![][6]
3. 依次填入openstack控制地址、admin用户和密码、admin租户
   ![][7]
4. 在OpenStack Neutron LBaaS - Credentials Used by NetScaler Driver中选择
   ![][8]
5. 点击OK
 
####openstack上操作

1. 从MAS上下载Netscaler Driver
   ![][9]
2. 上传到openstack控制节点并解压

   `tar -xvzf <name_of_tar_file>`
   
3. 进入相应目录

   `cd <Release Name> (e.g. cd Kilo)`
 
4. 运行
   
   `./install.sh --ip=<NetScaler_MAS_IP> --password=<password> --protocol=<protocol>
Example: ./install.sh --ip=10.102.29.90 --password=xxxx --protocol=HTTP
`


####VPX注册


1. 使用openstack创建VPX虚机
   ![][10]
2. 首次登录VPX虚机
   ![][11]
3. 依次配置subnet IP、DNS和时区、导入license

   **注：配置完subnet IP后由于neutron的限制，无法与外界通信，需要修改addess_pairs，命令如下：**
   
   `neutron port-update 08ccf4de-d6e2-4d4d-bcdf-55532e93f32f  --allowed-address-pairs type=dict list=true ip_address=20.0.0.101 ip_address=20.0.0.102 ip_address=20.0.0.103（如需添加多个地址用空格分开）`
   
   **注：没有license的用户请联系citrix销售人员**
   
4. License注册成功
   ![][12]
   
####VPX部署

1. 登录MAS，进入Infrastructure》instances》netscaler VPX，点击ADD
   ![][13]
   ![][14]
   ![][15]
2. 添加service package

   进入Orchestration》Cloud Orchestration》service package
   
   ![][16]
   
   点击Add
   
   ![][17]
   
   关联实例
   
   ![][18]
   
   关联openstack租户
   
   ![][19]
   
   完成
   
   ![][20]
   
###测试负载均衡

1. 创建负载均衡器的命令请参考：[https://confluence.ustack.com/pages/viewpage.action?pageId=16105150](https://confluence.ustack.com/pages/viewpage.action?pageId=16105150)
2. 配置检查

   VIP检查
   
   ![][21]
   
   listener检查
   
   ![][22]
   
   pool检查
   
   ![][23]
   
   member检查
   
   ![][24]
   
   monitor检查
   
   ![][25]
   
  
###测试结果

####  HTTP测试结果
  
  ![][26]
  
#### HTTPS测试结果 注：此处为https透传，并不是ssl卸载。
   
   ![][27]
   
### SSL卸载测试
  
  注：SSL卸载需要手动在netscaler上删除原来的VIP，再新建相同地址的VIP
  
  新建VIP
  
  ![][28]
  
  关联virtual server
  
  ![][29]
  
  关联证书
  
  ![][30]
  
  查看所有member
  
  ![][31]
  
  查看特定member
  
  ![][32]
  
  
#### 源地址透传测试
  
  编辑SG
  
  ![][33]
  
  查看关联
  
  ![][33]
  
  ![][34]
  
  验证
  
  抓使用使用命令：tcpdump tcp port 80 -n -X -s 0
  
  ![][36]
  
#### netscaler HA测试
  
  HA配置
  
  ![][37]
  
  ![][38]
  
  ![][39]
  
  ![][40]
 
####测试过程

#####HA测试

1. 挂起主用netscaler，只启用备用netscaler

   ![][41]
   
2. 备用netscaler正常负载

   ![][42]
   
#####HA切换时间测试

1. 强制断电测试 

   挂起主用netscaler，primary切换到备用netscaler，丢包4个
   
   ![][43]
   
2. 手动切换测试

   ![][44]
   
   手动切换测试过程没有丢包
     
   
[1]: ../../images/ecosystem/netscaler/1.png
[2]: ../../images/ecosystem/netscaler/2.png
[3]: ../../images/ecosystem/netscaler/3.png
[4]: ../../images/ecosystem/netscaler/4.png
[5]: ../../images/ecosystem/netscaler/5.png
[6]: ../../images/ecosystem/netscaler/6.png
[7]: ../../images/ecosystem/netscaler/7.png
[8]: ../../images/ecosystem/netscaler/8.png
[9]: ../../images/ecosystem/netscaler/9.png
[10]: ../../images/ecosystem/netscaler/10.png
[11]: ../../images/ecosystem/netscaler/11.png
[12]: ../../images/ecosystem/netscaler/12.png
[13]: ../../images/ecosystem/netscaler/13.png
[14]: ../../images/ecosystem/netscaler/14.png
[15]: ../../images/ecosystem/netscaler/15.png
[16]: ../../images/ecosystem/netscaler/16.png
[17]: ../../images/ecosystem/netscaler/17.png
[18]: ../../images/ecosystem/netscaler/18.png
[19]: ../../images/ecosystem/netscaler/19.png
[20]: ../../images/ecosystem/netscaler/20.png
[21]: ../../images/ecosystem/netscaler/21.png
[22]: ../../images/ecosystem/netscaler/22.png
[23]: ../../images/ecosystem/netscaler/23.png
[24]: ../../images/ecosystem/netscaler/24.png
[25]: ../../images/ecosystem/netscaler/25.png
[26]: ../../images/ecosystem/netscaler/26.png
[27]: ../../images/ecosystem/netscaler/27.png
[28]: ../../images/ecosystem/netscaler/28.png
[29]: ../../images/ecosystem/netscaler/29.png
[30]: ../../images/ecosystem/netscaler/30.png
[31]: ../../images/ecosystem/netscaler/31.png
[32]: ../../images/ecosystem/netscaler/32.png
[33]: ../../images/ecosystem/netscaler/33.png
[34]: ../../images/ecosystem/netscaler/34.png
[35]: ../../images/ecosystem/netscaler/35.png
[36]: ../../images/ecosystem/netscaler/36.png
[37]: ../../images/ecosystem/netscaler/37.png
[38]: ../../images/ecosystem/netscaler/38.png
[39]: ../../images/ecosystem/netscaler/39.png
[40]: ../../images/ecosystem/netscaler/40.png
[41]: ../../images/ecosystem/netscaler/41.png
[42]: ../../images/ecosystem/netscaler/42.png
[43]: ../../images/ecosystem/netscaler/43.png
[44]: ../../images/ecosystem/netscaler/44.png
[45]: ../../images/ecosystem/netscaler/45.png
[46]: ../../images/ecosystem/netscaler/46.png
[47]: ../../images/ecosystem/netscaler/47.png
[48]: ../../images/ecosystem/netscaler/48.png
[49]: ../../images/ecosystem/netscaler/49.png
[50]: ../../images/ecosystem/netscaler/50.png
[51]: ../../images/ecosystem/netscaler/51.png
[52]: ../../images/ecosystem/netscaler/52.png
[53]: ../../images/ecosystem/netscaler/53.png
[54]: ../../images/ecosystem/netscaler/54.png
[55]: ../../images/ecosystem/netscaler/55.png
[56]: ../../images/ecosystem/netscaler/56.png
[57]: ../../images/ecosystem/netscaler/57.png
[58]: ../../images/ecosystem/netscaler/58.png
[59]: ../../images/ecosystem/netscaler/59.png
[60]: ../../images/ecosystem/netscaler/60.png
