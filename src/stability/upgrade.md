## 升级对API停服时间减少

---


### UnitedStack知识库文章

 - [数据库升级](https://confluence.ustack.com/pages/viewpage.action?pageId=6652628)

### 简介
对于 Neutron 升级，一般采用以下的步骤：
1. 升级 Neutron 软件包
2. 升级数据库
3. 升级 Neutron Server
4. 升级网络节点
5. 升级计算节点

### 数据库升级
Neutron Server 服务本身是无状态且高可用的，造成 Neutron Server downtime 的只有数据库的升级，因此，
如何减少数据库升级时造成的 downtime 显得尤为重要。
相比较之前的 Neutron 升级，Liberty Neutron 数据库维护两个 alembic 分支： expand 分支和 contract 分支。
 - expand 分支：该分支中的数据库迁移操作可以在 Neutron Server 运行状态中执行，因为该
分支的数据库升级对服务没有影响。这些操作包括：创建表，创建字段等等。
 - contract 分支：如果正在运行着 Neutron Server，该分支的升级操作可能对服务有影响。
这些操作包括：字段、表的删除，数据的迁移。

### 升级 Neutron Server
Liberty 之前的 Neutron Server 升级如下表示：
1. systemctl stop neutron-server (on all nodes)
2. neutron-db-manage upgrade head
3. systemctl start neutron-server (on all nodes)

Liberty 之后的 Neutron Server 升级可以如下表示：
1. neutron-db-manage upgrade --expand
2. systemctl stop neutron-server (on all nodes)
3. neutron-db-manage upgrade --contract
4. systemctl start neutron-server (on all nodes)

因此，只有当执行`neutron-db-manage upgrade --contract` 的时候才会造成 Neutron Server 的 downtime。

### Oslo Versioned Objects
在 Liberty Neutron 中 ，通过 [Oslo Versioned Objects](http://docs.openstack.org/developer/oslo.versionedobjects/) （以下简称OVO）
定义网络资源。OVO 提供了一种基于版本的数据模型，通过它定义的数据模型和能够和外部 API 或者数据库模型解耦，
最终实现 Neutron 升级 0 downtime。OVO 更为强大的能力在于通过 Lazy-Loading 将数据先缓存在 OVO 的数据库中，
通过内部机制在同步到后端数据库中。目前，OVO 作为 Neutron 的 
[BP](https://blueprints.launchpad.net/neutron/+spec/adopt-oslo-versioned-objects-for-db) 还需编写大量的代码，
还在进一步的开发中。
