
![](image-1.png)

R1

```
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#neighbor 2.2.2.2 remote-as 100
R1(config-router)#neighbor 2.2.2.2 update-source lo0
R1(config-router)#neighbor 2.2.2.2 next-hop-self
R1(config-router)#neighbor 2.2.2.2 route-reflector-client


R1(config-router)#neighbor 3.3.3.3 remote-as 100
R1(config-router)#neighbor 3.3.3.3 update-source lo0
R1(config-router)#neighbor 3.3.3.3 next-hop-self
R1(config-router)#neighbor 3.3.3.3 route-reflector-client

R1(config-router)#neighbor 4.4.4.4 remote-as 100
R1(config-router)#neighbor 4.4.4.4 update-source lo0
R1(config-router)#neighbor 4.4.4.4 next-hop-self
```

R2

```
R2(config)#router bgp 100
R2(config-router)#bgp router-id 2.2.2.2
R2(config-router)#neighbor 1.1.1.1 remote-as 100
R2(config-router)#neighbor 1.1.1.1 update-source lo0
R2(config-router)#neighbor 1.1.1.1 next-hop-self
R2(config-router)#network 22.22.22.22 mask 255.255.255.255
```

R3

```
R3(config)#router bgp 100
R3(config-router)#bgp router-id 3.3.3.3
R3(config-router)#neighbor 1.1.1.1 remote-as 100
R3(config-router)#neighbor 1.1.1.1 update-source lo0
R3(config-router)#neighbor 1.1.1.1 next-hop-self
R3(config-router)#network 33.33.33.33 mask 255.255.255.255
```

R4

```
R4(config)#router bgp 100
R4(config-router)#bgp router-id 4.4.4.4
R4(config-router)#neighbor 1.1.1.1 remote-as 100
R4(config-router)#neighbor 1.1.1.1 update-source lo0
R4(config-router)#neighbor 1.1.1.1 next-hop-self
R4(config-router)#network 44.44.44.44 mask 255.255.255.255
```

现在R1成为RR(路由反射器)让R2和R3成为反射客户端, 可以小心使用`debug ip bgp updates`观察转发状态

# BGP Confederation(联邦)

建立BGP联邦需要两条命令
1. `bgp confederation identifier x` 确认公认联邦AS-Path
2. `bgp confederation peers x` 确认自己在联邦内部的AS-Path

R1

```
R1(config)#router bgp 100
R1(config-router)#bgp confederation identifier 100
R1(config-router)#bgp confederation peers 110
```

R2

```
R2(config)#router bgp 100
R2(config-router)#bgp confederation identifier 100
R2(config-router)#bgp confederation peers 110
```

R3

```
R3(config)#router bgp 100
R3(config-router)#bgp confederation identifier 100
R3(config-router)#bgp confederation peers 120
```

R4

```
R4(config)#router bgp 100
R4(config-router)#bgp confederation identifier 100
R4(config-router)#bgp confederation peers 120
```

## 选路策略与流量工程 

### 提高本地偏好(Local_Preference) 

```
ip prefix-list PFX-R3 seq 5 permit 3.3.3.3/32
route-map SET-LOCALPREF permit 10
 match ip address prefix-list PFX-R3
 set local-preference 200

router bgp 65000
 neighbor 10.0.12.1 route-map SET-LOCALPREF in
```

### AS-Path prepend(出口上游流量控制)

```
route-map PREPEND permit 10
 set as-path prepend 65000 65000 65000

router bgp 65000
 neighbor 203.0.113.1 route-map PREPEND out   ! 对外网关
验证：远端看到的 AS-PATH 会有重复的 65000，路径会被降低优先级。
```

### 改写 NEXT_HOP / NEXT_HOP SELF

```
router bgp 65000
 neighbor 10.0.12.1 next-hop-self
```

验证：show ip bgp 中 next-hop 变为对等的 IP（用于 RR 将 next-hop 保持正确连通性）。

### 使用 COMMUNITY 做分组策略

```
ip community-list standard 1 permit 65000:100
route-map TAG-SET permit 10
 set community 65000:100 additive

router bgp 65000
 neighbor 10.0.12.1 route-map TAG-SET out
然后在接收端用 community 做 route-map 筛选/设定 local-pref。
```

•	iBGP 不转发 learned iBGP 路由（不是 RR 的情况）→ 记住 iBGP 不会转发通过 iBGP 学到的路由，除非 RR 或者通过 confederation 的特殊情形。
•	Next-Hop 不可达 → 在 RR 场景中常见；用 next-hop-self 或确保 IGP/静态路由使 next-hop 可到达。
•	Route Reflector 未转发更新 → 检查 neighbor ... route-reflector-client 是否配置在 RR 上，同时确认 BGP 会话正常（show ip bgp neighbors）。
•	Confed 配置错乱 → 检查 bgp confederation identifier 与 bgp confederation peers 是否一致且相互匹配。
________________________________________
可选扩展（如果还想更“进攻性”）
•	把 RR 与 route-reflection cluster-id 与 client cluster-id 混合实验，观察 cluster-id 在 split-horizon 行为中的作用。
•	在多 RR 的场景下模拟 RR 间的不一致 policy 导致的路由回环 / 持久性问题并排查。
•	引入 MPLS L3VPN（下一步学习建议），观察 BGP 用于 VPNv4 路由的区别（route-target, extended community）。
