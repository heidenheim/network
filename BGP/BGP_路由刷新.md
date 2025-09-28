# 路由刷新 Router Refresh

路由刷新有着两个方向一个为 in 另一个 out

![](../image/BGP/82164.png)

首先在R1和R2上写好静态路由 1.1.1.1 和 2.2.2.2

在IBGP中R1与R2建立IBGP邻居关系，R1把11.11.11.11宣告进IBGP

```
R1(config-if)#router bgp 100
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#neighbor 2.2.2.2 remote-as 100
R1(config-router)#neighbor 2.2.2.2 update-source lo0
R1(config-router)#network 11.11.11.11 mask 255.255.255.255
```

```
R2(config-if)#router bgp 100
R2(config-router)#bgp router-id 2.2.2.2
R2(config-router)#neighbor 1.1.1.1 remote-as 100
R2(config-router)#neighbor 1.1.1.1 update-source lo0
```

在R2可以看到这条路由的下一跳是1.1.1.1

```
R2#show ip bgp
BGP table version is 2, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  11.11.11.11/32   1.1.1.1                  0    100      0 i
```

现在来设置一个路由策略只是把下一跳改为12.1.1.1

```
R2(config)#route-map RR permit 5
R2(config-route-map)#set ip next-hop 12.1.1.1

R2(config)#router bgp 100
R2(config-router)#neighbor 1.1.1.1 route-map RR in
R2#clear ip bgp * //重置bgp让规则生效

R2#show ip bgp
BGP table version is 2, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  11.11.11.11/32   12.1.1.1                 0    100      0 i
```

在R2上使用的路由策略 route-map 是 in 方向, 是因为这条11.11.11.11的路由是R2从R1学习到的, 对R2来说这是一条外来的路由, 那么就是in 入方向的.

同理如果想要在R1上修改这条路由的下一跳, 就是 out 出方向, 因为这是R1要传递给其他邻居的路由.

```
R1(config)#route-map RR permit 5
R1(config-route-map)#set ip next-hop 3.3.3.3

R1(config)#router bgp 100
R1(config-router)#neighbor 2.2.2.2 route-map RR out
```

在BGP中, router refresh报文是一种用于动态更新BGP路由表的机制, 而无需中断现有的BGP会话, 它允许BGP对等体请求对方重新发送符合某些策略的路由信息. router refresh报文可以在**入方向(inbound)**和**出方向(outbound)**中使用

1. 入方向报文
    - 重新应用入站路由策略, 当一个BGP对等体发送入方向的router refresh 报文时, 它是请求对等体重新发送路由更新, 以便在本地重新应用入站的路由策略(前缀列表, 路由映射, 社区过滤器等)
    - 更新接收到的路由信息, 入方向的router refresh报文确保对等体重新发送符合当前策略的路由信息. 这在管理大型网络时非常有用, 因为可以即时更新路由而不影响网络稳定性.
    
2. 出方向报文
    - 重新应用出战路由策略, 出方向的router refresh报文用于请求对等体重新评估并发送符合当前出战路由策略的路由信息. 假设对等体改变了出站的策略(修改了发送的BGP属性, 如AS路径或下一跳), 通过发送出方向的router refresh报文, 可以要求对等体重新发送符合这些新策略的路由信息
    - 保证对等体接收最新的路由策略更新, 出方向的router refresh报文确保所有的对等体接收到的路由信息都符合当前的出站策略. 这在网络拓扑或策略发生变化时尤为重要, 因为可以即使传播最新的路由决策.

    
硬重置
clear ip bgp ['neighbor'] 会中断BGP会话

软重置
clear ip bgp [neighbor] in / out