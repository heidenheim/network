1. MED (Multi-Exit Discriminator，多出口鉴别符)
    - 作用范围：从 AS 之间传递，告诉对方 AS “如果要进入我 AS，走哪条链路更好”。
    - 默认值：0，值越小越优先。
    - 规则：MED 只在 同一个邻居 AS 的多条链路之间比较，不跨不同 AS 比较。
    - 典型应用：
        1) 一个 AS 有两个出口给相同的上游 ISP，可以通过 MED 引导外部流量进来时的入口选择。

配置示例：

```
R1(config)#route-map SET-MED permit 10
R1(config-route-map)#set metric 50

R1(config)#router bgp 100
R1(config-router)#neighbor 23.1.1.2 route-map SET-MED out
```

3. 学习重点对比
|属性|传播范围|控制方向|默认值|越大越优先还是越小越优先|
|:--:|:-----:|:----:|:----:|:--------------------:|
|Local|Preference|仅 IBGP|出站|100|越大越优先|
|MED|传给|EBGP|入站|0|越小越优先|