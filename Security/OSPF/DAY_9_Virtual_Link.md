# 虚拟链路

**不重要**, 在非骨干区没有连接到骨干区时才会采用虚拟链路, 相当于BGP的跨设备建立邻居.

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

R2(config)#int lo0
R2(config-if)#ip address 2.2.2.2 255.255.255.255
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 1

R2(config)#int e0/0
R2(config-if)#ip address 12.1.1.2 255.255.255.0
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 1

R2(config)#int e0/1
R2(config-if)#ip address 23.1.1.2 255.255.255.0
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 2
```

```
R3(config)#router ospf 110
R3(config-router)#router-id 3.3.3.3

R3(config-if)#ip address 3.3.3.3 255.255.255.255
R3(config-if)#no shu
R3(config-if)#ip ospf 110 area 2

R3(config)#int e0/0
R3(config-if)#ip address 23.1.1.3 255.255.255.0
R3(config-if)#no shu
R3(config-if)#ip ospf 110 a 2
```

```
R1(config)#router ospf 110
R1(config-router)#area 1 virtual-link 2.2.2.2

R2(config)#router ospf 110
R2(config-router)#area 1 virtual-link 1.1.1.1
```
设置虚拟链路

验证`show ip ospf virtual-links`

