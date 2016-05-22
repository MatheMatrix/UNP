## 控制平面测试

---

### UnitedStack知识库相关文章

 - [QA提升之Rally](https://confluence.ustack.com/display/SDN/Rally)

### 简介

 Rally拥有API测试、压力测试的功能，常用于模拟高并发场景下的压力测试。
 Rally 测试OpenStack在高并发下API的相应时间和请求成功率，从而测试出OpenStack在一定规模下的性能。 
 它的工作原理如下：

 ![rally][1]


 下面给出一个Rally的测试模板和测试结果：
 ```
  {
     "NeutronNetworks.create_and_delete_networks": [
         {
             "args": {
                 "network_create_args": {}
             },
             "runner": {
                 "type": "constant",
                 "times": 100,
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


 ![rally_result][2]

 完整的 Rally 测试结果可以[参考这里](../../attachment/rally.html)


 [1]: ../../images/stability/rally.png
 [2]: ../../images/stability/rally_result.png
