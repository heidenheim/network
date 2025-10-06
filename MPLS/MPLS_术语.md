
# LSR 与 MPLS 域

- MPLS 域(MPLS Domain), 一系列连续的运行 MPLS 的网络设备构成了一个 MPLS 域
- LSR(Label Switching Router), 支持 MPLS 的路由器(所有支持 MPLS 的设备, 路由器, 交换机, 防火墙, etc. 都可以称为 LSR). 
    1. 位于 MPLS 域边缘, 连接其他网络的 LSR 叫做 LER(Label Edge Router)
    2. 位于 MPLS 域内部, 称为核心 LSR


## LSR 分类

- 入站 LSR(Ingress LSR), 通常是向 IP 报文压入 MPLS 头部并生成 MPLS 报文的 LSR
- 中转 LSR(Transit LSR), 通常是将 MPLS 报文进行例如标签置换操作, 并将报文继续在 MPLS 域中转发的 LSR
- 出站 LSR(Egress LSR), 通常是将 MPLS 报文中 MPLS 头部移除, 还原为 IP 报文的 LSR


|        |       |         |标签栈|        |       |
|--------|-------|---------|-------|------|-------|
|        |栈顶|            |栈底|   |              |
|二层头部|标签头部1|标签头部2|标签头部3|IP头部|报文载荷|

            |       |            ⬇
            |       |          |Label|EXP|**1**|TTL|
            |       |          |-----|---|-----|---|
            |       ⬇
            |    |Label|EXP|**0**|TTL|
            |    |-----|---|-----|---|
            ⬇
        |Label|EXP|**0**|TTL|
        |-----|---|-----|---|