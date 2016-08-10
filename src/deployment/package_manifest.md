### Package Manifest

------

#### 简介

 本节主要描述 UnitedStack<sup>®</sup> UNP 中使用到的所有 Neutron 软件及其依赖的软件包。

#### Neutron 软件包

| 包名                          | 版本       			     | 注释               |
|:----------------------------- |:-----------------------------------|:-------------------|
| openstack-neutron-vpnaas      | 7.0.0-1.el7.noarch                 | 初始版本 (UNP 1.0) |
| openstack-neutron-lbaas       | 7.0.0-1.el7.noarch                 | 初始版本 (UNP 1.0) |
| python-neutronclient          | 3.1.0-1.el7.noarch                 | 初始版本 (UNP 1.0) |
| openstack-neutron             | 7.1.1.2-1.el7.centos.ustack.noarch | 初始版本 (UNP 1.0) |
| openstack-neutron-common      | 7.1.1.2-1.el7.centos.ustack.noarch | 初始版本 (UNP 1.0) |
| openstack-neutron-ml2         | 7.1.1.2-1.el7.centos.ustack.noarch | 初始版本 (UNP 1.0) |
| python-neutron-vpnaas         | 7.0.0-1.el7.noarch                 | 初始版本 (UNP 1.0) |
| python-neutron-lbaas          | 7.0.0-1.el7.noarch                 | 初始版本 (UNP 1.0) |
| python-neutron                | 7.1.1.2-1.el7.centos.ustack.noarch | 初始版本 (UNP 1.0) |
| openstack-neutron-openvswitch | 7.1.1.2-1.el7.centos.ustack.noarch | 初始版本 (UNP 1.0) |

#### 每个 Neutron 软件包的依赖包及版本信息
`注：'-' 表示该软件包不限制版本，'>=' 表示该软件包版本需大于或者等于某版本`。

##### openstack-neutron-vpnaas

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|openstack-neutron              |>= 7.0.0               |
|strongswan                     |5.3.2-1.el7.x86_64     |

##### openstack-neutron-lbaas      

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|openstack-neutron              |>= 7.0.0               |
|python-neutron-lbaas           |7.0.0-1.el7          |
|haproxy                        |1.6.0                  |

##### python-neutronclient         

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|python-babel                   |-                      |
|python-cliff                   |>= 1.14.0              |
|python-iso8601                 |-                      |
|python-keystoneclient          |>= 1.6.0               |
|python-netaddr                 |-                      |
|python-oslo-i18n               |>= 1.5.0               |
|python-oslo-serialization      |>= 1.4.0               |
|python-oslo-utils              |>= 2.0.0               |
|python-pbr                     |-                      |
|python-prettytable             |>= 0.6                 |
|python-requests                |>= 2.5.2               |
|python-setuptools              |-                      | 
|python-simplejson              |-                      |
|python-six                     |>= 1.9.0               |

##### openstack-neutron

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron)      |7.1.1.2-1.el7.centos.ustack|
|conntrack-tools                |-                      |
|dibbler-client                 |-                      |
|dnsmasq                        |-                      |
|dnsmasq-utils                  |-                      |
|ipset                          |-                      |
|iptables                       |-                      |
|keepalived                     |-                      |
|openstack-neutron-common       |7.1.1.2-1.el7.centos.ustack|
|radvd                          |-                      |

##### openstack-neutron-common

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron)      |7.1.1.2-1.el7.centos.ustack|
|python-neutron                 |7.1.1.2-1.el7.centos.ustack|

##### openstack-neutron-ml2

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron-ml2)  |7.1.1.2-1.el7.centos.ustack|
|openstack-neutron-common       |7.1.1.2-1.el7.centos.ustack|
|python-ncclient                |-                      | 

##### python-neutron-vpnaas

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|python(abi)                    |2.7                    |
|python-alembic                 |-                      |
|python-jinja2                  |-                      |
|python-netaddr                 |>= 0.7.12              |
|python-neutron                 |>= 7.0.0               |
|python-oslo-config             |>= 1.4.0               |
|python-oslo-db                 |>= 1.1.0               |
|python-oslo-log                |>= 1.0.0               |
|python-oslo-messaging          |>= 1.4.0.0             |
|python-oslo-serialization      |>= 1.0.0               |
|python-oslo-utils              |>= 1.0.0               |
|python-pbr                     |-                      | 
|python-requests                |-                      |
|python-six                     |-                      |
|python-sqlalchemy              |-                      |

##### python-neutron-lbaas         

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|pyOpenSSL                      |-                      |
|python(abi)                    |2.7                    |
|python-alembic                 |>= 0.7.2               |
|python-barbicanclient          |>= 3.0.1               |
|python-eventlet                |-                      |
|python-netaddr                 |>= 0.7.12              |
|python-neutron                 |>= 7.0.0               |
|python-oslo-config             |>= 1.9.3               |
|python-oslo-db                 |>= 1.7.0               |
|python-oslo-log                |>= 1.0.0               |
|python-oslo-messaging          |>= 1.8.0               |
|python-oslo-serialization      |>= 1.4.0               |
|python-oslo-utils              |>= 1.4.0               |
|python-pbr                     |-                      |
|python-pyasn1                  |-                      |
|python-pyasn1-modules          |-                      |
|python-requests                |-                      |
|python-six                     |-                      |
|python-sqlalchemy              |>= 0.9.7               |

##### python-neutron

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|MySQL-python                   |-                      |
|python(abi)                    |2.7                    |
|python-alembic                 |>= 0.8.0               |
|python-debtcollector           |>= 0.3.0               |
|python-eventlet                |>= 0.17.4              |   
|python-greenlet                |>= 0.3.2               | 
|python-httplib2                |>= 0.7.5               | 
|python-jinja2                  |>= 2.6                 |
|python-keystoneclient          |>= 1.6.0               |
|python-keystonemiddleware      |>= 2.0.0               |
|python-netaddr                 |>= 0.7.12              |
|python-neutronclient           |>= 2.6.0               |
|python-novaclient              |>= 2.26.0              |
|python-oslo-concurrency        |>= 2.3.0               |
|python-oslo-config             |>= 2:2.1.0             |
|python-oslo-context            |>= 0.2.0               |
|python-oslo-db                 |>= 2.0                 |
|python-oslo-i18n               |>= 1.5.0               | 
|python-oslo-log                |>= 1.8.0               | 
|python-oslo-messaging          |>= 1.16.0              |  
|python-oslo-middleware         |>= 2.4.0               | 
|python-oslo-policy             |>= 0.5.0               | 
|python-oslo-rootwrap           |>= 2.0.0               | 
|python-oslo-serialization      |>= 1.4.0               | 
|python-oslo-service            |>= 0.6.0               | 
|python-oslo-utils              |>= 2.0.0               | 
|python-oslo-versionedobjects   |>= 0.6.0               |
|python-paste     		|-			|
|python-paste-deploy 		|>= 1.5.0		|
|python-pbr			|-			|
|python-pecan 			|>= 1.0.0		|
|python-requests 		|>= 2.5.2		|		 
|python-retrying 		|>= 1.2.3		|		 
|python-routes 			|>= 1.12.3		|
|python-ryu 			|>= 3.23.2 		|  
|python-six 			|>= 1.9.0		|
|python-sqlalchemy		|>= 0.9.7		| 
|python-stevedore 		|>= 1.5.0		|  
|python-webob 			|>= 1.2.3		|

##### openstack-neutron-openvswitch

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron-openvswitch) |7.1.1.2-1.el7.centos.ustack|
|openstack-neutron-common 	|7.1.1.2-1.el7.centos.ustack|
|openvswitch			|-			|
|python-openvswitch		|-			|
