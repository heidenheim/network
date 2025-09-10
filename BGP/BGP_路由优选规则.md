# 优选规则

当到达同一个目的网段存在多条路由时, BGP通过如下的次序进行路由优选:

- 越大越优
1. 优选 Weight 属性值最大的路由
2. 优选 Local Preference 属性值最大的路由

- 越小越优秀
1. 本地始发的BGP路由优于从其他对等体学习到的路由, 本地始发的路由优先级: 优选手工聚合 > 自动聚合 > network > import > 从对等体学习到
2. 优选AS_Path属性最短的路由
3. 优选Origin属性最优的路由. Origin属性值按优先级从高到低排序 IGB, RGB, Incomplete
4. 优选MED属性值最小的路由
5. 优选从EBGP对等体学来的路由(EBGP路由优先级高于IBGP路由)
6. 优选到Next_Hop的IGP度量值最小的路由
7. 优选最老的EBGP学习到的路由, 降低路由翻滚的影响 (对EBGP路由有效, 反射基本不使用, 不确定性过大)
8. 优选Router ID (Originator ID) 最小的设备通告的路由
9. 优选Cluster_List最短的路由

![](image/270701.png)

```
R1#show ip bgp
BGP table version is 5, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i  172.16.1.1/32    3.3.3.3                  0    100      0 300 i
 *>i                   2.2.2.2                  0    100      0 100 i
```

默认情况下, R1肯定会选Router ID小的 2.2.2.2, 可以手工修改来让R1选择3.3.3.3传来的路由.

### 修改权重

```
R1(config)#router bgp 200
R1(config-router)#neighbor 3.3.3.3 weight 100

R1#show ip bgp
BGP table version is 6, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  172.16.1.1/32    3.3.3.3                  0    100    100 300 i
 * i                   2.2.2.2                  0    100      0 100 i
```

### 修改本地优先
```
R1(config)#route-map LoP permit 10
R1(config-route-map)#set local-preference 500
R1(config-route-map)#router bgp 200
R1(config-router)#neighbor 3.3.3.3 route-map LoP in


R1#show ip bgp
BGP table version is 3, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i  172.16.1.1/32    2.2.2.2                  0    100      0 100 i
 *>i                   3.3.3.3                  0    500      0 300 i
```

PS, 如果本地自己就有同样一个路由, 那么肯定会优选本地的.

### 追加路由AS域

```
R4(config)#route-map AS permit 10
R4(config-route-map)#set as-path prepend 111

R4(config-route-map)#router bgp 100
R4(config-router)#neighbor 24.1.1.2 route-map AS out
```

```
R1#show ip bgp
BGP table version is 3, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  172.16.1.1/32    3.3.3.3                  0    100      0 300 i
 * i                   2.2.2.2                  0    100      0 100 111 i
```

### 重分布

Origin 起源路由 是优于重分布的, 重分布后属于不完全路由 后面会有"?"

```
R1#show ip bgp
BGP table version is 12, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  4.4.4.4/32       2.2.2.2                  0    100      0 100 ?
 r>i  24.1.1.0/24      2.2.2.2                  0    100      0 100 ?
 * i  172.16.1.1/32    2.2.2.2                  0    100      0 100 ?
 *>i                   3.3.3.3                  0    100      0 300 i
```

## 优选MED最小

可以在R4或R2上使用路由策略修改MED, 使得R3的路由更优

```
R4(config)#ip prefix-list MED permit 172.16.0.0/16
R4(config)#ip prefix-list MED permit 172.16.0.0/16 ge 17

R4(config)#route-map MED permit 10
R4(config-route-map)#match ip address prefix-list MED
R4(config-route-map)#set metric 100

R4(config-route-map)#router bgp 100
R4(config-router)#nei 24.1.1.2 route-map MED out
```

```
R1#show ip bgp
BGP table version is 16, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i  172.16.1.1/32    3.3.3.3                  0    100      0 300 i
 *>i                   2.2.2.2                100    100      0 100 i
```

路由默认是只对相同AS的路由比较MED值, 需要使用命令"bgp alwasys-compare-med"来开启对比不同AS的MED

```
R1(config)#router bgp 200
R1(config-router)#bgp always-compare-med

R1#show ip bgp
BGP table version is 17, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  172.16.1.1/32    3.3.3.3                  0    100      0 300 i
 * i                   2.2.2.2                100    100      0 100 i
```

## 优选从EBGP对等体学来的路由

![](image/BGP/280700.png)

```
R1(config)#router bgp 200
R1(config-router)#neighbor 3.3.3.3 route-reflector-client
```

把R1做RR反射, 让172.16.1.1成为R3的一条IBGP路由

```
R3#show ip bgp
BGP table version is 2, local router ID is 3.3.3.3
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i  172.16.1.1/32    24.1.1.4                 0    100      0 100 i
 *>                    35.1.1.5                 0             0 300 i
```

可以看到R3会优选R5传来的172.16.1.1 这条EBGP路由, **EBGP对等体通告的BGP路由**优于IBGP对等体通告的BGP路由

## 修改Metric值

在EIGRP中可以使用 offset-list 调整度量值(MED) 来影响BGP的优选

```
R1
ip access-list standard R2L0
permit 2.2.2.2

eigrp 90
offset-list R2L0 in 500
```

这样从2.2.2.2发过来的路由MED都在原有基础上增加了500

## 优选最老的 EBGP 学习到的路由

以及BGP还有优选 最小Router ID, 最小Cluster list, 最小IP地址, 不再赘述

## BGP 路由等价负载分担

如果以上所有的值都相同, 那么就可以设置BGP的负载均衡

- 在大型网路中, 到达同一目的地通常会存在多条有效BGP路由, BGP设备只会优选一条最优的BGP路由, 将该路由加载到路由表中使用, 这一特点往往会造成很多流量负载不均衡的情况

- 通过配置BGP负载分担, 可以使得设备同时将多条等价的BGP路由加载到路由表, 实现流量负载均衡, 减少网络拥塞

- 值得注意的是, 尽管配置了BGP负载分担, 设备依然只会在多条到达同一目的地的BGP路由中**优选一条路由**, 并将这条路由通告给其他对等体

- 在设备上使能BGP负载分担功能后, **只有满足条件的多条BGP路由才会成为等价路由, 进行负载分担**

### 等价负载分担的条件
1. Weight 相同
2. Local Preference 相同
3. 同为聚合路由或同为非聚合路由
4. AS Path 长度相同
5. Origin类型(IGP, EGP, Incomplete)相同
6. MED相同
7. 同为EBGP或IBGP路由
8. AS Path 属性相同


