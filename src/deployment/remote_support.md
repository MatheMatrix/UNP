## UnitedStack 远程支持接入

### 简介


项目实施完毕后，UnitedStack<sup>®</sup> United Networking Platform(UNP) 以远程接入方式提供技术支持及运维，
为了保证数据的安全性和兼容性，建议采用 IPsec VPN 方式与客户连通。
目前 UnitedStack<sup>®</sup> United Networking Platform(UNP) UC
中心使用了两个厂商的防火墙与客户打通 VPN ，DELL sonicwall 和 Fortinet。
理论上讲客户可以采用任何厂商的防火墙与 UnitedStack UC（Unified Center） 防火墙建立VPN，
但是由于不同厂商由于兼容性存在问题，因此建议客户使用我们推荐的防火墙。

### DELL SonicWALL 防火墙
[DELL SonicWALL](www.sonicwall.com) 防火墙是我们长期使用的防火墙，性能稳定，配置简单。
以下是 SonicWALL 防火墙的IPsec VPN 配置方法：

1. 创建10.12.0.0地址段

 ![wall1][1]

2. 创建10.12.8.0地址段，把刚才创建的地址段拉到地址组中，稍后创建 VPN 时会用到。

 ![wall2][2]

3. 创建地址组

 ![wall3][3]

4. 创建vpn

 ![wall4][4]
 ![wall5][5]
 ![wall6][6]
 ![wall7][7]

5. 开放策略

 策略无需单独开放，Dell SonicWALL 防火墙在创建创建 vpn 的时候默认创建了。
 ![wall8][8]

### 推荐使用Fortinet防火墙
Fortinet 防火墙是我们为不能与 Dell SonicWALL 防火墙建立 VPN 而新建的另外一组防火墙，Fortinet 防火墙功能强大、可靠，且配置相对简单。
以下是 Fortinet 防火墙的 IPsec VPN 配置方法：

1. 添加地址对象

 ![net1][9]

2. 阶段1配置

 ![net2][10]

3. 阶段2配置

 ![net3][11]

4. 添加静态路由

 ![net4][12]

5. 添加策略

 - 入向策略：

  ![net5][13]
 
 - 出向策略：

 ![net6][14]

[1]: ../../images/deployment/sonicwall1.png
[2]: ../../images/deployment/sonicwall2.png
[3]: ../../images/deployment/sonicwall3.png
[4]: ../../images/deployment/sonicwall4.png
[5]: ../../images/deployment/sonicwall5.png
[6]: ../../images/deployment/sonicwall6.png
[7]: ../../images/deployment/sonicwall7.png
[8]: ../../images/deployment/sonicwall8.png
[9]: ../../images/deployment/net1.png
[10]: ../../images/deployment/net2.png
[11]: ../../images/deployment/net3.png
[12]: ../../images/deployment/net4.png
[13]: ../../images/deployment/net5.png
[14]: ../../images/deployment/net6.png
