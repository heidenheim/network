# BGP路由表
```
R2#show ip bgp
BGP table version is 3, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>i  11.11.11.11/32   1.1.1.1                  0    100      0 i
 *>   22.22.22.22/32   0.0.0.0                  0         32768 i
R2#
```
看到BGP table version is 3, BGP 路由表中路由更新的“内部版本号”，也就是说，这是 BGP 自己维护的“路由表变动次数”。
- network 路由的目的网络地址以及网络掩码
- nextHop 下一跳地址
- 如果存在同个目的地多条路由, 则路由都将进行罗列, 但是每个目的地只有会有一条优选路由
``` show ip bgp ipv4-address {mask | mask-length}```
查看BGP路由详细的详细展示
```
R1#show ip bgp 22.22.22.22 255.255.255.255
BGP routing table entry for 22.22.22.22/32, version 3
Paths: (1 available, best #1, table default)
  Not advertised to any peer
  Refresh Epoch 1
  Local
    2.2.2.2 from 2.2.2.2 (2.2.2.2)
      Origin IGP, metric 0, localpref 100, valid, internal, best
      rx pathid: 0, tx pathid: 0x0
R1#show ip bgp 22.22.22.22 255.255.255.255 | ?
  append    Append redirected output to URL (URLs supporting append operation
            only)
  begin     Begin with the line that matches
  count     Count number of lines which match regexp
  exclude   Exclude lines that match
  format    Format the output using the specified spec file
  include   Include lines that match
  redirect  Redirect output to URL
  section   Filter a section of output
  tee       Copy output to URL

R1#show ip bgp 22.22.22.22 255.255.255.255 |
```

- '*' 可用的路由(不一定最优) 
- 's' 被一致的路由条目, 例如总了路由汇总, 抑制了明细
- 'd' 被惩罚(dampening)的路由, 路由收到了惩罚, 虽然该路由当前可能正常, 但是在惩罚结束前不会被通告
- 'H' 被惩罚(dampening)的路由, 路由可能出现了故障(down), 有历史信息, 但没有最佳路由
- 'r' 路由没有被装在RIB表中, 例如由于AD值等原因导致
- 'S' stale 过期路由
- '>' BGP计算出的最优路由
- ' '/'i' 为空表示路由从EBGP另据获取, 为i表示从IBGP学习到
*IBGP的AD为200, EBGP的Ad为20*

```
R1#show ip route bgp
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      22.0.0.0/32 is subnetted, 1 subnets
B        22.22.22.22 [200/0] via 2.2.2.2, 00:13:03
```


