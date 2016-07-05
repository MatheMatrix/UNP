## F5 BIG-IP 对接

---

###UnitedStack 知识库相关链接

 - [F5 OpenStack 集成
](https://confluence.ustack.com/pages/viewpage.action?pageId=16105150)
 - [L4~L7 Integration 对比
](https://confluence.ustack.com/pages/viewpage.action?pageId=16095151&src=contextnavpagetreemode)

###F5 OpenStack Driver 主要特点

####Feature

注：本节内容主要来自：http://f5-openstack-docs.readthedocs.io/ 和 http://f5-openstack-lbaasv2-driver.readthedocs.io/

**OpenStack 版本支持表格**

| OpenStack release | F5® LBaaSv1 Plugin | F5® LBaaSv2 Plugin | BIG-IP® | RHEL | Ubuntu |
|:--------|:--------|:--------|:--------|:--------|:--------|
| Icehouse - Kilo | 7.x | N/A | 11.5.2+, 11.6.x, 12.0.x |6, 7 | 12, 14 |
| Liberty | 8.x	| 8.x | 11.5.2+, 11.6.x, 12.0.x |6, 7 | 12, 14 |
| Mitaka |9.x | N/A | 11.5.2+, 11.6.x, 12.0.x |6, 7 | 12, 14 |

**Heat 支持表格**

| OpenStack release | F5® Heat Plugins | F5® Heat Templates |
| :------ | :------ | :------ |
| Kilo	| 7.x	| 7.x |
| Liberty	| 8.x	| 8.x |
| Mitaka	| 9.x	| N/A |

**目前不支持的特性**

The following features are unsupported in 8.0.3; they will be introduced in future releases.

 - BIG-IP® vCMP®
 - Agent High Availability (HA) 
 - Differentiated environments 

**对 LBaaS v2 支持有限制的内容**

The features supported in 8.0.3 are a subset of the Neutron LBaaSv2 API delivered in the OpenStack Liberty release. The following restriction(s) apply:

| Object | Unsupported |
| :------ | :----------- |
| Loadbalancer| Statistics (e.g., neutron lbaas-loadbalancer-stats)|

**部署参考文档**

BIG-IP VE：
http://f5-openstack-docs.readthedocs.io/en/latest/guides/deploy_ve_openstack.html
BIG-IP：
http://f5-openstack-lbaasv2-driver.readthedocs.io/en/latest/map_f5-lbaasv2-user-guide.html

###集成测试

####部署安装包
```python
yum install -y python-pip git python-neutron-lbaas openstack-neutron-lbaas

curl -O -L https://github.com/F5Networks/neutron-lbaas/releases/download/v8.0.1/f5.tgz
sudo tar xvf f5.tgz -C /usr/lib/python2.7/site-packages/neutron_lbaas/drivers/

sudo pip install git+https://github.com/F5Networks/f5-openstack-lbaasv2-driver@v8.0.3
sudo pip install git+https://github.com/F5Networks/f5-openstack-agent@v8.0.3
```
 
注：如果是离线部署，需要安装以下依赖：
```python
https://github.com/F5Networks/f5-common-python
https://pypi.python.org/pypi/future
https://github.com/F5Networks/f5-icontrol-rest-python
```

####配置

**修改配置文件**
```python
neutron.conf
vi /etc/neutron/neutron.conf
...
[DEFAULT]
service_plugins = [already defined plugins],neutron_lbaas.services.loadbalancer.plugin.LoadBalancerPluginv2
...
 ```
```python
neutron_lbaas.conf
vi /etc/neutron/neutron_lbaas.conf
...
[service_providers]
service_provider = LOADBALANCERV2:F5Networks:neutron_lbaas.drivers.f5.driver_v2.F5LBaaSV2Driver:default
...
```
```python
f5-openstack-agent.ini
f5_external_physical_mappings = default:2.1:True
#使用vlan时，advertised_tunnel_types = 为空  目前只能使用vlan模式
advertised_tunnel_types =
```

**测试中遇到发现版本兼容性报错，后通过修改代码解决**

测试时遇到代码问题需要修改以下两处代码：

```python
Supported_version:
/usr/lib/python2.7/site-packages/f5/bigip/mixins.py
LazyAttributeMixin._check_supported_versions
        tmos_v = container._meta_data['bigip'].tmos_version
        if tmos_v not in attribute._meta_data['supported_versions']:
            error = "There was an attempt to access API which " \
                    "has not been implemented or supported " \
                    "in the device's TMOS version: {}".format(tmos_v)
+++++++++
-           raise UnsupportedTmosVersion(error)
+           #raise UnsupportedTmosVersion(error)
+++++++++
```
```python
tagMode:
/usr/lib/python2.7/site-packages/f5/bigip/tm/net/vlan.py
Interfaces.create()
        if 'tagged' in kwargs and kwargs['tagged'] is True:
            tup_par = ('tagMode', 'tagged')
            self._meta_data['required_creation_parameters'].update(tup_par)
++++++
+       kwargs['tagMode']='service'
++++++
        self._create(**kwargs)
 
        return self
```
 

**升级数据库**

```python
neutron-db-manage --config-file /etc/neutron/neutron.conf --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head
```

**重启neutron-server**

```python
systemctl restart neutron-server
```


####使用

**创建网络**
```bash
[root@server-48 ~(keystone_admin)]# neutron net-create test1
```

**创建子网**
```bash
[root@server-48 ~(keystone_admin)]# neutron subnet-create c50bad34-3903-4611-a94c-b7077fd8366b 192.168.1.0/24
```

**创建loadbalancer**
```bash
[root@server-48 ~(keystone_admin)]# neutron lbaas-loadbalancer-create 44dd715d-aa0d-4637-bbc5-0d024bec53cc
```
 
**创建listener**
```bash
[root@server-48 ~(keystone_admin)]# neutron lbaas-listener-create --loadbalancer 2e90c0a8-a7f3-49ab-b001-fb4269fb902c --protocol TCP --protocol-port 80
```
**创建pool**
```bash
[root@server-48 ~(keystone_admin)]# neutron lbaas-pool-create --lb-algorithm ROUND_ROBIN --listener d53cc5fa-b96f-4946-9f70-01bfaf20c2a5 --protocol TCP
```
**创建member**
```bash
[root@server-48 ~(keystone_admin)]# neutron lbaas-member-create --subnet 44dd715d-aa0d-4637-bbc5-0d024bec53cc --address 192.168.1.10 --protocol-port 80 2d2c4d87-ae1e-46ff-8436-e3e81be7db9f
```
**创建monitor**
```bash
[root@server-48 ~(keystone_admin)]# neutron lbaas-healthmonitor-create --delay 1 --max-retries 3 --timeout 1 --type PING  --pool  2d2c4d87-ae1e-46ff-8436-e3e81be7db9f
```
**查看loadbalancer**
```bash
[root@localhost ~(keystone_admin)]# neutron lbaas-loadbalancer-list
+--------------------------------------+------+-------------+---------------------+------------+
| id                                   | name | vip_address | provisioning_status | provider   |
+--------------------------------------+------+-------------+---------------------+------------+
| 6e57a2dd-efc8-4bfe-a358-718c1cc79fb9 |      | 20.0.0.7    | ACTIVE              | f5networks |
| 710b3390-d31d-439a-94c3-f02bceb7250a |      | 40.0.0.8    | ACTIVE              | f5networks |
| 8328b0c1-b416-46be-801e-dd74edafa135 |      | 20.0.0.21   | ACTIVE              | f5networks |
| d93e534b-f653-4df3-bb76-f70b057443d1 |      | 20.0.0.10   | ACTIVE              | f5networks |
+--------------------------------------+------+-------------+---------------------+------------+
```
**查看listener**
```bash
[root@localhost ~(keystone_admin)]# neutron lbaas-listener-list
+--------------------------------------+--------------------------------------+------+----------+---------------+----------------+
| id                                   | default_pool_id                      | name | protocol | protocol_port | admin_state_up |
+--------------------------------------+--------------------------------------+------+----------+---------------+----------------+
| a1fc82cd-27a9-4491-bf7e-d2df446945a8 | f08fec9c-144c-499e-86de-ca12ea65ae8d |      | TCP      |           443 | True           |
| 3051e29f-0394-4397-a20b-fea174079bcb | 84adc754-2bd2-45f0-ab41-dc74df5483fc |      | TCP      |            80 | True           |
+--------------------------------------+--------------------------------------+------+----------+---------------+----------------+
```
**查看pool**
```bash
[root@localhost ~(keystone_admin)]# neutron lbaas-pool-list
+--------------------------------------+------+----------+----------------+
| id                                   | name | protocol | admin_state_up |
+--------------------------------------+------+----------+----------------+
| 84adc754-2bd2-45f0-ab41-dc74df5483fc |      | TCP      | True           |
| f08fec9c-144c-499e-86de-ca12ea65ae8d |      | TCP      | True           |
+--------------------------------------+------+----------+----------------+
```
**查看member**
```bash
[root@localhost ~(keystone_admin)]# neutron lbaas-member-list 84adc754-2bd2-45f0-ab41-dc74df5483fc
+--------------------------------------+-----------+---------------+--------+--------------------------------------+----------------+
| id                                   | address   | protocol_port | weight | subnet_id                            | admin_state_up |
+--------------------------------------+-----------+---------------+--------+--------------------------------------+----------------+
| 081e5501-25b8-4e7f-aa46-390aa00d84c3 | 20.0.0.11 |            80 |      1 | 96e06339-db66-411d-8404-5576546b8e8f | True           |
| aacad08a-1d00-46b5-894f-0f60de76a44b | 20.0.0.12 |            80 |      1 | 96e06339-db66-411d-8404-5576546b8e8f | True           |
| fc47612a-7f37-4644-9577-2c35c7cffa0c | 20.0.0.13 |            80 |      1 | 96e06339-db66-411d-8404-5576546b8e8f | True           |
+--------------------------------------+-----------+---------------+--------+--------------------------------------+----------------+
```
从上面输出可以看到我们创建了3个member，分别是20.0.0.11、20.0.0.12和20.0.0.13，使用的是http协议
 
####测试
**测试拓扑**

![image_1amhtm96l1hs61qg8f61ptv1nm89.png-46.2kB][1]
 
我们已经提前创建好了上述member中所使用的3个虚机，如图所示：

![image_1amhtoi68b2vgem1mh81dq13fgm.png-32kB][2]

在3个服务器上使用以下命令模拟web服务
```bash
while true; do echo -e  "HTTP/1.0 200 OK\r\n\r\nServer-11" | sudo nc -l -p 80; done&
while true; do echo -e  "HTTP/1.0 200 OK\r\n\r\nServer-12" | sudo nc -l -p 80; done&
while true; do echo -e  "HTTP/1.0 200 OK\r\n\r\nServer-13" | sudo nc -l -p 80; done&
```
在client端检查到20.0.0.7的连通性

![image_1amhtoupk8715k42re9ld1rrq13.png-86.8kB][3]

在client端访问20.0.0.7，检查是否负载到3个member服务器

![image_1amhtp5ulabh1frj1puhaf1ait1g.png-112.4kB][4]

图中可以看到，在client端的访问可以负载到不同的member服务器。
下面是F5控制台上的截图

![image_1amhtppu3gvm16v81hmfu23uv91t.png-180.8kB][5]

**遇到的BUG**
 - 重启neutron-server后F5上的配置消失，只剩下相关的tenant。
   - 解决方法：重启F5设备


  [1]: http://static.zybuluo.com/MatheMatrix/r1719xovblzfgt8702vfue0b/image_1amhtm96l1hs61qg8f61ptv1nm89.png
  [2]: http://static.zybuluo.com/MatheMatrix/881m8nlqcmobjmsxdqge3vsv/image_1amhtoi68b2vgem1mh81dq13fgm.png
  [3]: http://static.zybuluo.com/MatheMatrix/cjj1ggp68kimvmhgjj8fnrkc/image_1amhtoupk8715k42re9ld1rrq13.png
  [4]: http://static.zybuluo.com/MatheMatrix/i60u4jrzo62z4mwd76w15j3l/image_1amhtp5ulabh1frj1puhaf1ait1g.png
  [5]: http://static.zybuluo.com/MatheMatrix/0gq2kat2nuc0o6rxurx1s6y2/image_1amhtppu3gvm16v81hmfu23uv91t.png

与华为 Agile Controller 对接

HUAWEI-f5

lbaas v1

reinstall rpms ...

change oslo.config to oslo_config
change oslo.messaging to oslo_messaging

agent:

change neutron common  to oslo_service
RpcCallback  __init__ cfg.CONF
min extra mb
service.launch(cfg.CONF, svc)

remove old f5-sdk