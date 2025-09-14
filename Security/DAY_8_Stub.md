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