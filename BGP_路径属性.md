# BGP路径属性

任何一条BGP路由都拥有多个路径属性.

当路由器将BGP路由通告给它的对等体时, 一并通告的还有路由所携带的各个路径属性.

BGP的路径属性将影响路由优选.

## 路径属性分类
|公认必遵|公认任意|可选过渡|可选非过度|
|:-----:|:-----:|:-------|:--------|
|Origin|Local_Prefence|Aggregator|MED|
|AS_Path|Atomic_aggregate-address|Aggregator|Cluster-List|
|Next_hop|address||Originator-ID|
||||*Weight|

BGP路径属性分为两大类, 一大类叫做**公认属性 well-know**, 一大类叫**可选属性Optional**

公认属性是所有BGP路由器都必须能够识别的属性
1. 公认属性又分为两小类:
    - 公认必遵(Well-Know Mandatory), 必须包括在每个update消息里
    - 公认任意(Well-Know Discretionary), 可能包括在某些update消息里, 也可以不包括

可选属性不需要都被BGP路由器所识别
2. 可选属性也可以分为两小类:
    - 可选过渡(Optional Transitive), BGP设备不识别此类属性依然会接收该类属性并通告给其他对等体.
    - 可选非过渡(Optional Non-transitive), BGP设备不识别此类属性会忽略该属性, 且不会通告给其他对等体.

## Weight 介绍
 - Weight(权重值)是思科设备的私有属性, 取值范围:0 - 65535, 该值越大, 则路由越优先.
 - 路由器本地始发的路由*默认权重值为32768*, 从其他BGP邻居学习到的为*0*
 - Weight 只能在路由器本地配置, 而且只影响本设备的路由优选. **该属性不会传递给任何BGP对等体**
 - 该属性仅在本地有效. 当BGP路由表中存在到相同目的地的路由时, 将优先选择 Weight 值高的路由
 - Weight 值是思科BGP的**第一条选路规则

 ![](image/190600.png)

 在R1上部署路由策略(import策略), 将R2传递1.1.1.0/24的 weight 值设为300, 将R3传递的1.1.1.0/24的 weight 值设为200, 如此一来关于 1.1.1.0/24, R1肯定会优选R2传递的路由

 ![](image/190601.png)

 R2与R3都传递一条8.8.8.8的路由给R1, 各项数值都相同, weight为0, R1选取R2作为路劲

 ```
 R1#show ip bgp
BGP table version is 2, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 * i  8.8.8.8/32       3.3.3.3                  0    100      0 i
 *>i                   2.2.2.2                  0    100      0 i
```

现在在R1上把R3的 weight 改为100

```
R1(config)#router bgp 100
R1(config-router)#neighbor 3.3.3.3 weight 100

R1#clear ip bgp * soft
R1#show ip bgp
BGP table version is 4, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  8.8.8.8/32       3.3.3.3                  0    100    100 i
 * i                   2.2.2.2                  0    100      0 i

```

软重置一下BGP, 可以看到由于修改了weight, R1选路路径变为了R3. **Weight仅对本地有效**


## Local Preference 本地优先级

- Local Preference 本地优先级属性, 是**公认任意属性**, 可用于告诉AS内的路由器, **哪条路径是离开本AS的首选路径**.
- Local Preference 属性越大则BGP路由越优, 缺省的Local Preference值为100.
- **该属性只能被传递给IBGP对等体**, 而不能传递给EBGP对等体. (**即本地优先级属性仅在AS内具有意义**而不能出AS外)