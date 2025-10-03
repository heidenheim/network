# OSPF的特殊区域

## OSPF 区域
1. 普通区域（Normal Area）
- 支持所有 LSA 类型 (1–5, 7)

2. Stub Area
- 不接受外部路由（LSA 5 被 ABR 屏蔽）
- ABR 会给该区域下发一条默认路由（LSA 3）

3. Totally Stub Area
- 除了屏蔽外部路由，还屏蔽其他区域的 Summary LSA，只保留默认路由

3. NSSA (Not-So-Stubby Area)
- 特殊的 Stub，可以引入外部路由，但使用 LSA 7，在 ABR 转换为 LSA 5

## LSA 类型总结
1. Type 1：Router LSA（区域内的路由器信息）
2. Type 2：Network LSA（由 DR 产生）
3. Type 3：Summary LSA（ABR 产生，跨区域路由）
4. Type 4：ASBR Summary LSA（ABR 产生，指明 ASBR 的路由）
5. Type 5：External LSA（ASBR 产生的外部路由）
6. Type 7：NSSA External LSA（NSSA 内部的外部路由）

### 配置 Stub Area

1. 在大型 OSPF 网络里，外部路由 (Type 5 LSA) 非常多。
2. 如果一个分支路由器根本不需要知道所有外部路由，只要知道怎么出去就行，这时 Stub 能屏蔽外部路由，只用一条默认路由代替，减轻 CPU/内存/LSA 泛洪压力。

在 ABR 和所有 Area 1 内部路由器上加上 `area 1 stub`

```
# R1 (ABR)
R1(config)# router ospf 110
R1(config-router)# area 1 stub

# R2 (Area 1 内部路由器)
R2(config)# router ospf 110
R2(config-router)# area 1 stub
```

### 配置 Totally Stub Area

1. 在一些更小的分支（比如只有几台电脑的办公室），不仅不需要外部路由，连其他区域的 Summary 路由 (Type 3/4) 也用不上。
2. Totally Stub 可以进一步简化路由表，只留下默认路由，配置和维护更轻松。

在 ABR 上额外加一个 no-summary 参数，其他路由器仍然用普通的 stub。

```
# R1 (ABR)
R1(config)# router ospf 110
R1(config-router)# area 1 stub no-summary

# R2 (Area 1 内部路由器)
R2(config)# router ospf 110
R2(config-router)# area 1 stub
```

### 配置 NSSA（Not-So-Stubby Area）

1. Stub 的限制是不能引入外部路由，但有些分支必须引入外部路由（例如：分支有一条通往 Internet 的出口）。
2. NSSA 解决这个矛盾：允许分支引入外部路由，但在区域内用 Type 7 LSA，到了 ABR 再转成 Type 5，这样既保留 Stub 的简化优势，又能引入本地外部路由。

在 ABR 和该区域内所有路由器上配置 nssa。

如果要在 NSSA 内部引入外部路由，用 redistribute。

```
# R3 (ABR)
R3(config)# router ospf 110
R3(config-router)# area 2 nssa

# R4 (NSSA 内部路由器)
R4(config)# router ospf 110
R4(config-router)# area 2 nssa

# 如果 R4 想引入外部路由
R4(config)# router ospf 110
R4(config-router)# redistribute connected subnets
```

# LSA 分类

1. 区域内拓扑: Type 1&2
2. 区域间路由: Type 3&4
3. 外部路由:   Type 5&7

Type 1(Router): 所有路由器都会产生, 在区域内建立区域拓扑

Type 2(Network): 只有DR才会产生, 在区域内描述广播/多路访问网段

Type 3(Summary): 由边界路由器(ABR)产生, 跨区域传递区域间路由

Type 4(ASBR Summary): 由ABR产生, 跨区域让其他路由器找到ASBR

Type 5(External): 由ASBR(链接外部网络的边界路由器)产生, 除了Stub/NSSA的整个OSPF域传递, 引入的外部路由

Type 7(NSSA External): NSSA内的ASBR产生, 在NSSA区域内传递, 后续由ABR转换为Tpye 5传递, 是NSSA内引入的外部路由

## Stub 的需求



## Totally Stub 的需求



## NSSA 的需求

