## F5 BIG-IP 对接

---

参考：http://f5-openstack-lbaasv2.readthedocs.io/

离线部署

相关包：

https://github.com/F5Networks/neutron-lbaas/releases/download/v8.0.1/f5.tgz
https://github.com/F5Networks/f5-openstack-lbaasv2-driver
https://github.com/F5Networks/f5-openstack-agent
https://github.com/F5Networks/f5-common-python
https://pypi.python.org/packages/5a/f4/99abde815842bc6e97d5a7806ad51236630da14ca2f3b1fce94c0bb94d3d/future-0.15.2.tar.gz#md5=a68eb3c90b3b76714c5ceb8c09ea3a06
https://github.com/F5Networks/f5-icontrol-rest-python

安装

server:

install f5.tgz
install f5-openstack-lbaasv2-driver
change config
upgrade db
restart

agent:

install f5.tgz
install f5-openstack-lbaasv2-driver
install future
install f5-icontrol-rest-python
install f5-common-python
install f5-openstack-agent

remember that f5-common-python should be a 0.17 not development or some versions may not support

华为 Agile Controller

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