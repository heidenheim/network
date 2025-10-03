# OSPF 多区域 (Multi-Area OSPF)

1. 理解 OSPF 多区域的作用（可扩展性、LSA 范围控制）。
2. 掌握 ABR（Area Border Router） 的概念与作用。
3. 配置一台路由器同时属于多个 Area。
4. 验证 LSA 类型（特别是 Type 3 Summary LSA）。

## R1

```
R1(config)#router ospf 110
R1(config-router)#router-id 1.1.1.1

R1(config)#int lo0
R1(config-if)#ip add
R1(config-if)#ip address 1.1.1.1 255.255.255.255
R1(config-if)#no shu
R1(config-if)#ip ospf 110 area 0

R1(config)#int e0/0
R1(config-if)#ip address 12.1.1.1 255.255.255.0
R1(config-if)#ip ospf 110 area 0
R1(config-if)#ip ospf priority 1
```

## R2

```
R2(config)#router ospf 110
R2(config-router)#router-id 2.2.2.2

R2(config)#int e0/0
R2(config-if)#ip address 12.1.1.2 255.255.255.0
R2(config-if)#no shu
R2(config-if)#ip ospf 110 area 0
R2(config-if)#ip ospf priority 255

R2(config)#int e0/1
R2(config-if)#ip address 23.1.1.2 255.255.255.0
R2(config-if)#ip ospf 110 area 1
R2(config-if)#ip ospf priority 255
```

## R3

```
R3(config)#router ospf 110
R3(config-router)#router-id 3.3.3.3

R3(config)#int lo0
R3(config-if)#ip address 3.3.3.3 255.255.255.255
R3(config-if)#ip ospf 110 area 1

R3(config)#int e0/0
R3(config-if)#ip address 23.1.1.3 255.255.255.0
R3(config-if)#ip ospf 110 area 1
R3(config-if)#ip ospf priority 1
```

```
R2#show ip ospf  neighbor

Neighbor ID     Pri   State           Dead Time   Address         Interface
1.1.1.1           1   FULL/BDR        00:00:36    12.1.1.1        Ethernet0/0
3.3.3.3           1   FULL/BDR        00:00:37    23.1.1.3        Ethernet0/1

R2#show ip ospf database summary

            OSPF Router with ID (2.2.2.2) (Process ID 110)

                Summary Net Link States (Area 0)

  LS age: 88
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 3.3.3.3 (summary Network Number)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0x31EC
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 11

  LS age: 134
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 23.1.1.0 (summary Network Number)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0x6EA3
  Length: 28
  Network Mask: /24
        MTID: 0         Metric: 10


                Summary Net Link States (Area 1)

  LS age: 88
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 1.1.1.1 (summary Network Number)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0x8D98
  Length: 28
  Network Mask: /32
        MTID: 0         Metric: 11

  LS age: 134
  Options: (No TOS-capability, DC, Upward)
  LS Type: Summary Links(Network)
  Link State ID: 12.1.1.0 (summary Network Number)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0xFD1F
  Length: 28
  Network Mask: /24
        MTID: 0         Metric: 10

```


为什么要把 OSPF 网络划分为多个 Area？
1. 减小 LSDB（链路状态数据库）的规模
2. 减少 LSA 泛洪
3. 收敛更快
4. 层次化设计：通过 Area Border Router (ABR) 做汇

ABR 的作用是什么？
1. 连接不同的 OSPF Area（至少要有一个接口在 Area 0，还有一个接口在非 0 区域）。
2. 在区域之间传递路由信息：
- 把本区域的路由转换成 Type 3 Summary LSA，发送到其他区域。
- 把其他区域的路由引入本区域。
3. 可做路由聚合：在 ABR 上可以配置 `area x range` 对路由进行汇总，减少路由数量。

如果 R3 ping R1 的 Loopback 成功，它是通过哪种 LSA 学到的路由？
1. R1 的 Loopback (1.1.1.1/32) 在 Area 0 中生成 Type 1 Router LSA 和 Type 2 Network LSA。
2. ABR (R2) 把它总结成 Type 3 Summary LSA，再传递到 Area 1。
3. 所以 R3 是通过 Type 3 Summary LSA 学到的 R1 的 Loopback 路由。 