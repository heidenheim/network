
# ZBF 的核心概念：Zone、Zone-Pair、Class-map、Policy-map。

    如何彻底删除 ZBF 配置（包括接口绑定、策略、类映射）。
    
    验证清理是否成功。

## 理论部分
1. 为什么要删除 ZBF 配置？
    - 在实验环境中经常需要测试不同场景，清除旧配置是必须的。
    - 部分接口需要恢复到无 Zone 状态，才能回到传统 ACL 或 CBAC。

2. 删除 ZBF 配置的步骤

- 接口解绑
```
R1(config)#int e0/0
R1(config-if)#no zone-member security DMZ
R1(config)#int e0/1
R1(config-if)#no zone-member security INSIDE
R1(config)#int e0/2
R1(config-if)#no zone-member security OUTSIDE
```

2. 删除 zone-pair
```
R1(config)#no zone-pair security IN-TO-OUT
R1(config)#no zone-pair security OUT-TO-IN
```

3. 删除 policy-map
```
R1(config)#no policy-map type inspect POLICY-IN-TO-OUT
R1(config)#no policy-map type inspect POLICY-OUT-TO-IN
```

4. 删除 class-map
```
R1(config)#no class-map type inspect CLASS-IN-TO-OUT
R1(config)#no class-map type inspect CLASS-OUT-TO-IN
```

5. 删除 Zone 本身
```
R1(config)#no zone security INSIDE
R1(config)#no zone security OUTSIDE
R1(config)#no zone security DMZ
```

### 验证是否完全删除

1. 查看接口绑定情况

`R1#show run | i zone-member`

如果没有任何输出，说明接口解绑成功。

2. 查看剩余的 Zone/Zone-Pair
```
R1#show zone security
R1#show zone-pair security
```

#### 如果只删除了 zone-pair，但没删除接口上的 zone-member，会发生什么？

- 现象：
    1. 如果接口依旧属于某个 Zone，但没有任何 Zone-Pair 去描述这两个 Zone 之间的流量关系，那么 所有跨 Zone 的流量都会被默认丢弃。
    2. 换句话说，接口绑定了 Zone，但没有“策略”，Cisco ZBF 的默认行为就是 deny all。

- 举例：
    1. INSIDE → OUTSIDE 的 zone-pair 删除了，但接口依然在 INSIDE 和 OUTSIDE。
    2. 结果：INSIDE 到 OUTSIDE 的 ping、HTTP、DNS 等 全部被丢弃。
    3. 但是 同一个 Zone 内的通信（比如两个 INSIDE 网段之间）是允许的，因为 intra-zone 默认 permit。


##### 在生产环境中，删除 ZBF 配置前需要注意什么？

1. 评估流量影响
    - 删除 zone-pair 或解绑接口 → 流量可能直接中断。
    - 在生产环境，必须先确认替代方案（例如 ACL、FW、NGFW 已经接管）。

2. 逐步迁移
    - 可以先在 维护窗口 做测试。

    - 或者先把某个接口移出 zone，观察流量是否符合预期，再逐步清理。
3. 备份配置
    - `copy running-config startup-config`
    - 或 `show run` 保存一份到外部文件。
        - 这样即使删除错了，也能迅速回滚。

4. 验证工具
    - 用 ping、telnet、curl 等验证关键服务是否能正常通信。
    - 建议配合 show log / debug policy-map 检查是否有流量被丢弃。

5. 替代安全策略
    - 删除 ZBF 前要确认是否需要 ACL/IPS/外部防火墙接替安全控制，否则网络可能变成“裸奔”。

___________________________________________________________________________________

掌握彻底删除 ZBF 的命令步骤。

学会验证是否仍有残留配置。

理解 ZBF 删除不完整可能带来的网络影响。

理论知识

ZBF 配置的三个主要组成部分

zone security 定义安全区域

zone-pair 定义区域间流量与 policy-map 绑定

接口 zone-member 将接口加入安全区域

删除顺序（推荐）

先从接口上移除 zone-member

删除 zone-pair

删除 policy-map、class-map（可选）

最后删除 zone security

为什么要按顺序？

如果直接删除 zone security，而接口仍绑定 zone-member，配置会报错。

如果只删 zone-pair，不移除接口的 zone-member，接口依旧属于某个区域 → 默认拒绝任何非同一区域流量。

实验步骤

假设当前配置：

```
zone security INSIDE
zone security OUTSIDE
zone-pair security IN-TO-OUT source INSIDE destination OUTSIDE
 service-policy type inspect POLICY1

int e0/0
 zone-member security INSIDE

int e0/1
 zone-member security OUTSIDE
```

删除操作：
! 第一步：先移除接口的 zone 成员

```
R1(config)#int e0/0
R1(config-if)#no zone-member security INSIDE
R1(config)#int e0/1
R1(config-if)#no zone-member security OUTSIDE
```

! 第二步：删除 zone-pair

`R1(config)#no zone-pair security IN-TO-OUT`

! 第三步：删除 policy-map / class-map（可选）

```
R1(config)#no policy-map type inspect POLICY1
R1(config)#no class-map type inspect CLASS1
```

! 第四步：删除 zone

```
R1(config)#no zone security INSIDE
R1(config)#no zone security OUTSIDE
```

验证方法

检查接口：

`R1#show run interface e0/0`


确认没有 zone-member security。

检查 zone：

`R1#show zone security`


若已删除，应无输出或显示 "no zones configured"。

检查 zone-pair：

`R1#show zone-pair security`


应为空。