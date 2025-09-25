
![](image.png)

# NAT（Network Address Translation） 的三种模式：

1. 静态 NAT：一对一转换
2. 动态 NAT：从池中动态分配
3. PAT（NAPT/Overload）：多对一（常见于家庭路由器）

配置 NAT, PAT 必须要配置接口

R1

```
R1(config)#int e0/0
R1(config-if)#ip address 192.168.1.254 255.255.255.0
R1(config-if)#ip nat inside
R1(config-if)#no shut

R1(config)#int e0/1
R1(config-if)#ip address 12.1.1.1 255.255.255.0
R1(config-if)#ip nat outside
R1(config-if)#no shut
```

## NAT

`R1(config)#ip nat inside source static 192.168.1.10 12.1.1.10`

## 动态 NAT

```
R1(config)#ip nat pool POOL1 12.1.1.20 12.1.1.30 netmask 255.255.255.0
R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)#ip nat inside source list 1 pool POOL1
```

## PAT

```
R1(config)#access-list 1 permit 192.168.1.0 0.0.0.255
R1(config)#ip nat inside source list 1 interface e0/1 overload
```

### 验证

```
R1#show ip nat translations
R1#show ip nat statistics
```

```
R1#show ip nat translations
Pro Inside global      Inside local       Outside local      Outside global
icmp 12.1.1.10:8438    192.168.1.10:8438  12.1.1.100:8438    12.1.1.100:8438
...
```
`show ip nat statistics` 这条命令是用来 查看 NAT 当前的总体运行状态和统计信息 

- 输出内容主要包括：
    1. NAT 映射条数
        - 当前有多少条静态 / 动态 NAT 转换被创建。
        - 动态映射数会随着流量而增加/减少。
    2. Inside/Outside 接口信息
        - 哪些接口被定义为 ip nat inside，哪些是 ip nat outside。
    3. NAT 池（pool）信息
        - 如果你配置了动态 NAT，能看到池的 IP 地址范围、已分配/剩余情况。
    4. Overload (PAT) 统计
        - 显示 NAT Overload 时，每个外部地址上分配的端口数量。
    5. 命中计数
        - NAT 转换成功/失败的次数（hits, misses）。