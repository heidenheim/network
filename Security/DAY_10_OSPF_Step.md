
回顾并理解 Stub、Totally Stub、NSSA 的特点与使用场景。

在 EVE-NG 中搭建拓扑并进行配置验证。

学会通过 show ip route 和 show ip ospf database 验证不同区域的路由表情况。



```
R1(config)#router ospf 110
R1(config-router)#router-id 1.1.1.1

R1(config)#int lo0
R1(config-if)#ip address 1.1.1.1 255.255.255.255
R1(config-if)#no shu
R1(config-if)#ip ospf 110 area 0

R1(config)#int e0/0
R1(config-if)#ip address 12.1.1.1 255.255.255.0
R1(config-if)#no shu
R1(config-if)#ip ospf 110 area 0
```

```
R2(config)#router ospf 110
R2(config-router)#router-id 2.2.2.2
R2(config-router)#area 1 stub

R2(config)#int lo0
R2(config-if)#ip address 2.2.2.2 255.255.255.255
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 1

R2(config)#int e0/0
R2(config-if)#ip address 12.1.1.2 255.255.255.0
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 0

R2(config)#int e0/1
R2(config-if)#ip address 23.1.1.2 255.255.255.0
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 1
```

```
R3(config)#router ospf 110
R3(config-router)#router-id 3.3.3.3

R3(config)#int lo0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
R3(config-if)#no shu
R3(config-if)#ip ospf 110 area 1

R3(config)#int e0/0
R3(config-if)#ip address 23.1.1.3 255.255.255.0
R3(config-if)#no shu
R3(config-if)#ip o 110 a 1
```