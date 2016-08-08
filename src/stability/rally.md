## 控制平面测试

---

### UnitedStack知识库相关文章

 - [QA提升之Rally](https://confluence.ustack.com/display/SDN/Rally)

### 简介

 Rally 拥有 API 测试、压力测试的功能，常用于模拟高并发场景下的压力测试。
 Rally 测试 OpenStack 在高并发下 API 的相应时间和请求成功率，从而测试出 OpenStack 在一定规模下的性能。 
 它的工作原理如下：

 ![rally][1]

 Rally 的部署方式如下：

 1. 在 Rally 中注册当前以及部署完成的 OpenStack 集群:

    ```
    $ rally deployment create --fromenv --name=existing
    +--------------------------------------+----------------------------+------------+------------------+--------+
    | uuid                                 | created_at                 | name       | status           | active |
    +--------------------------------------+----------------------------+------------+------------------+--------+
    | 28f90d74-d940-4874-a8ee-04fda59576da | 2015-01-18 00:11:38.059983 | existing   | deploy->finished |        |
    +--------------------------------------+----------------------------+------------+------------------+--------+
    ```
 2. 检查当前 OpenStack 集群各个服务的工作状态：

    ```
    root@devstack:/opt/stack# rally deployment check
    keystone endpoints are valid and following services are available:
    +-------------+----------------+-----------+
    | services    | type           | status    |
    +-------------+----------------+-----------+
    | __unknown__ | compute_legacy | Available |
    | __unknown__ | volumev2       | Available |
    | cinder      | volume         | Available |
    | glance      | image          | Available |
    | keystone    | identity       | Available |
    | neutron     | network        | Available |
    | nova        | compute        | Available |
    +-------------+----------------+-----------+
    ```

 3. Rally 通过 `json` 或者 `yaml` 的模板文件，定义一个测试用例，如：

    JSON 模板文件：
    ```
    {
        "NeutronNetworks.create_and_delete_networks": [
            {
                "args": {
                    "network_create_args": {}
                },
                "runner": {
                    "type": "constant",
                    "times": 10,
                    "concurrency": 10
                },
                "context": {
                    "users": {
                        "tenants": 3,
                        "users_per_tenant": 3
                    },
                    "quotas": {
                        "neutron": {
                            "network": -1
                        }
                    }
                }
            }
        ]
    }
    ```

    YAML 模板文件：

    ```
    ---
      NeutronNetworks.create_and_delete_networks:
        -
          args:
            network_create_args: {}
          runner:
            type: "constant"
            times: 100
            concurrency: 10
          context:
            users:
              tenants: 3
              users_per_tenant: 3
            quotas:
              neutron:
                network: -1
    ```

 下面是测试完成后的结果，生成的 html 文件如下：

 ![rally_result][2]

 完整的 Rally 测试结果可以[参考这里](../../attachment/rally.html)

### 存在的问题
目前当运行 Rally 测试时有可能遇到 IpAddressGenerationFailure 的 [Bug](https://bugs.launchpad.net/neutron/+bug/1562887)。



 [1]: ../../images/stability/rally.png
 [2]: ../../images/stability/rally_result.png
