# 查看本路由器的 OSPF 数据库（所有 LSA）

![](../image/Security/140901.png)

`show ip ospf database`

```
R2#show ip ospf database

            OSPF Router with ID (2.2.2.2) (Process ID 110)

                Router Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum Link count
1.1.1.1         1.1.1.1         83          0x80000005 0x005BD5 2
2.2.2.2         2.2.2.2         82          0x80000005 0x00E04F 3
3.3.3.3         3.3.3.3         84          0x80000005 0x0069AB 2

                Net Link States (Area 0)

Link ID         ADV Router      Age         Seq#       Checksum
192.168.1.2     2.2.2.2         87          0x80000001 0x0009B0
192.168.2.2     2.2.2.2         88          0x80000001 0x00624E
```

# Router LSA (Type 1)

`R2# show ip ospf database router`

```
R2#show ip ospf database router

            OSPF Router with ID (2.2.2.2) (Process ID 110)

                Router Link States (Area 0)

  LS age: 261
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 1.1.1.1
  Advertising Router: 1.1.1.1
  LS Seq Number: 80000005
  Checksum: 0x5BD5
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 1.1.1.1
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.1.2
     (Link Data) Router Interface address: 192.168.1.1
      Number of MTID metrics: 0
       TOS 0 Metrics: 10


  LS age: 260
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 2.2.2.2
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000005
  Checksum: 0xE04F
  Length: 60
  Number of Links: 3

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 2.2.2.2
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.2.2
     (Link Data) Router Interface address: 192.168.2.2
      Number of MTID metrics: 0
       TOS 0 Metrics: 10

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.1.2
     (Link Data) Router Interface address: 192.168.1.2
      Number of MTID metrics: 0
       TOS 0 Metrics: 10


  LS age: 261
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 3.3.3.3
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000005
  Checksum: 0x69AB
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 3.3.3.3
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.2.2
     (Link Data) Router Interface address: 192.168.2.3
      Number of MTID metrics: 0
       TOS 0 Metrics: 10
```

# Network LSA (Type 2)

`R2# show ip ospf database network`

```
R2#show ip ospf database network

            OSPF Router with ID (2.2.2.2) (Process ID 110)

                Net Link States (Area 0)

  LS age: 332
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 192.168.1.2 (address of Designated Router)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0x9B0
  Length: 32
  Network Mask: /24
        Attached Router: 2.2.2.2
        Attached Router: 1.1.1.1

  LS age: 333
  Options: (No TOS-capability, DC)
  LS Type: Network Links
  Link State ID: 192.168.2.2 (address of Designated Router)
  Advertising Router: 2.2.2.2
  LS Seq Number: 80000001
  Checksum: 0x624E
  Length: 32
  Network Mask: /24
        Attached Router: 2.2.2.2
        Attached Router: 3.3.3.3
```

# Summary LSA (Type 3)

`R2# show ip ospf database summary`

```
R2#show ip ospf database summary

            OSPF Router with ID (2.2.2.2) (Process ID 110)
```

# AS External LSA (Type 5)

`R2# show ip ospf database external`

```
R2#show ip ospf database external

            OSPF Router with ID (2.2.2.2) (Process ID 110)
```

## 查看指定 Router-ID 产生的 LSA

`R2# show ip ospf database router 3.3.3.3`

```
R2#show ip ospf database router 3.3.3.3

            OSPF Router with ID (2.2.2.2) (Process ID 110)

                Router Link States (Area 0)

  LS age: 502
  Options: (No TOS-capability, DC)
  LS Type: Router Links
  Link State ID: 3.3.3.3
  Advertising Router: 3.3.3.3
  LS Seq Number: 80000005
  Checksum: 0x69AB
  Length: 48
  Number of Links: 2

    Link connected to: a Stub Network
     (Link ID) Network/subnet number: 3.3.3.3
     (Link Data) Network Mask: 255.255.255.255
      Number of MTID metrics: 0
       TOS 0 Metrics: 1

    Link connected to: a Transit Network
     (Link ID) Designated Router address: 192.168.2.2
     (Link Data) Router Interface address: 192.168.2.3
      Number of MTID metrics: 0
       TOS 0 Metrics: 10
```

## 查看链路状态信息（LSA 的详细内容）

`R2# show ip ospf database router 3.3.3.3 detail`