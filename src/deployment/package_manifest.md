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

##### openstack-neutron-vpnaas

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|openstack-neutron              |7.0.0                  |
|strongswan                     |5.3.2-1.el7.x86_64     |

##### openstack-neutron-lbaas      

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|openstack-neutron              |7.0.0                  |
|python-neutron-lbaas           |7.0.0-1.el7            |
|haproxy                        |1.6.0                  |

##### python-neutronclient         

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|python-babel                   |1.3-6.el7.noarch       |
|python-cliff                   |1.15.0-1.el7.noarch    |
|python-iso8601                 |0.1.11-1.el7.noarch    |
|python-keystoneclient          |1.7.2-1.el7.noarch     |
|python-netaddr                 |0.7.18-1.el7.noarch    |
|python-oslo-i18n               |2.6.0-1.el7.noarch     |
|python-oslo-serialization      |1.9.0-1.el7.noarch     |
|python-oslo-utils              |2.5.0-1.el7.noarch     |
|python-pbr                     |1.8.1-2.el7.noarch     |
|python-prettytable             |0.7.2-2.el7.centos.noarch|
|python-requests                |2.7.0-6.el7.noarch     |
|python-setuptools              |0.9.8-3.el7.noarch     | 
|python-simplejson              |3.3.3-1.el7.x86_64     |
|python-six                     |1.9.0-2.el7.noarch     |

##### openstack-neutron

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron)      |7.1.1.2-1.el7.centos.ustack|
|conntrack-tools                |1.4.2-9.el7.x86_64     |
|dibbler-client                 |1.0.1-0.RC1.2.el7.x86_64   |
|dnsmasq                        |2.66-12.el7.x86_64     |
|dnsmasq-utils                  |2.67-1.el7.centos.x86_64   |
|ipset                          |6.19-4.el7.x86_64      |
|iptables                       |1.4.21-13.el7.x86_64   |
|keepalived                     |1.2.13-7.el7.x86_64    |
|openstack-neutron-common       |7.1.1.2-1.el7.centos.ustack|
|radvd                          |1.9.2-9.el7.x86_64     |

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
|python-ncclient                |0.4.2-2.el7.noarch     | 

##### python-neutron-vpnaas

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|python(abi)                    |2.7                    |
|python-alembic                 |0.8.3-3.el7.noarch     |
|python-jinja2                  |2.7.2-2.el7.noarch     |
|python-netaddr                 |0.7.18-1.el7.noarch    |
|python-neutron                 |7.1.1.2-1.el7.centos.ustack.noarch|
|python-oslo-config             |2.4.0-1.el7.noarch     |
|python-oslo-db                 |2.6.0-3.el7.noarch     |
|python-oslo-log                |1.10.0-1.el7.noarch    |
|python-oslo-messaging          |2.5.2-1.el7.centos.ustack.com.noarch|
|python-oslo-serialization      |1.9.0-1.el7.noarch     |
|python-oslo-utils              |2.5.0-1.el7.noarch     |
|python-pbr                     |1.8.1-2.el7.noarch     | 
|python-requests                |2.7.0-6.el7.noarch     |
|python-six                     |1.9.0-2.el7.noarch     |
|python-sqlalchemy              |1.0.11-1.el7.x86_64    |

##### python-neutron-lbaas         

| 包名                      | 版本                |
|:--------------------------|:--------------------|
| pyOpenSSL                 | 0.15.1-1.el7.noarch |
| python(abi)               | 2.7                 |
| python-alembic            | 0.8.3-3.el7.noarch  |
| python-debtcollector      | 0.8.0-1.el7.noarch  |
| python-eventlet           | 0.17.4-4.el7.noarch |
| python-netaddr            | 0.4.9-1.el7.x86_64  |
| python-oslo-config        | 0.9.2-1.el7.noarch  |
| python-oslo-db            | 2.7.2-2.el7.noarch  |
| python-oslo-log           | 1.7.2-1.el7.noarch  |
| python-oslo-messaging     | 2.3.1-1.el7.noarch  |
| python-oslo-serialization | 0.7.18-1.el7.noarch |
| python-oslo-utils         | 2.5.0-1.el7.noarch  |
| python-pbr                | 1.8.1-2.el7.noarch  |
| python-pyasn1             | 0.1.8-2.el7.noarch  |
| python-pyasn1-modules     | 0.1.8-2.el7.noarch  |
| python-requests           | 2.7.0-6.el7.noarch  |
| python-six                | 1.9.0-2.el7.noarch  |
| python-sqlalchemy         | 1.0.11-1.el7.x86_64 |

##### python-neutron

| 包名                         | 版本  				         	   |
|:-----------------------------|:--------------------------------------------------|
| MySQL-python                 | 1.2.3-11.el7.x86_64                               |
| pyOpenSSL                    | 0.15.1-1.el7.noarch                               |
| python(abi)                  | 2.7                                               |
| python-alembic               | 0.8.3-3.el7.noarch                                |
| python-debtcollector         | 0.8.0-1.el7.noarch                                |
| python-eventlet              | 0.17.4-4.el7.noarch                               |
| python-greenlet              | 0.4.9-1.el7.x86_64                                |
| python-httplib2              | 0.9.2-1.el7.noarch                                |
| python-jinja2                | 2.7.2-2.el7.noarch                                |
| python-keystoneclient        | 1.7.2-1.el7.noarch                                |
| python-keystonemiddleware    | 2.3.1-1.el7.noarch                                |
| python-netaddr               | 0.7.18-1.el7.noarch                               |
| python-neutronclient         | 3.1.0-1.el7.noarch                                |
| python-novaclient            | 2.30.3-0.20160325012319.bc213fd.el7.centos.noarch |
| python-oslo-concurrency      | 2.6.0-1.el7.noarch                                |
| python-oslo-config           | 2.4.0-1.el7.noarch                                |
| python-oslo-context          | 0.6.0-1.el7.noarch                                |
| python-oslo-db               | 2.6.0-3.el7.noarch                                |
| python-oslo-i18n             | 2.6.0-1.el7.noarch                                |
| python-oslo-log              | 1.10.0-1.el7.noarch                               |
| python-oslo-messaging        | 2.5.2-1.el7.centos.ustack.com.noarch              |
| python-oslo-middleware       | 2.8.0-1.el7.noarch                                |
| python-oslo-policy           | 0.11.0-1.el7.noarch                               |
| python-oslo-rootwrap         | 2.3.0-1.el7.noarch                                |
| python-oslo-serialization    | 1.9.0-1.el7.noarch                                |
| python-oslo-service          | 0.9.0-1.el7.noarch                                |
| python-oslo-utils            | 2.5.0-1.el7.noarch                                |
| python-oslo-versionedobjects | 0.10.1-0.20160309220013.30f17e0.el7.centos.noarch |
| python-paste                 | 1.7.5.1-9.20111221hg1498.el7.noarch               |
| python-paste-deploy          | 1.5.0-10.el7.noarch                               |
| python-pbr                   | 1.8.1-2.el7.noarch                                |
| python-pecan                 | 1.0.2-2.el7.noarch                                |
| python-requests              | 2.7.0-6.el7.noarch                                |
| python-retrying              | 1.2.3-4.el7.noarch                                |
| python-routes                | 1.13-2.el7.noarch                                 |
| python-ryu                   | 3.26-1.el7.noarch                                 |
| python-six                   | 1.9.0-2.el7.noarch                                |
| python-sqlalchemy            | 1.0.11-1.el7.x86_64                               |
| python-stevedore             | 1.8.0-1.el7.noarch                                |
| python-webob                 | 1.4.1-2.el7.noarch                                |

##### openstack-neutron-openvswitch

| 包名                          | 版本       		|
|:----------------------------- |:----------------------|
|config(openstack-neutron-openvswitch) |7.1.1.2-1.el7.centos.ustack|
|openstack-neutron-common 	|7.1.1.2-1.el7.centos.ustack|
|openvswitch			|2.4.0-1.el7.x86_64	|
|python-openvswitch		|2.4.0-1.el7.noarch	|
