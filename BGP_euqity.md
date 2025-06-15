# BGP对等提关系
与OSPF、EIGRP等协议不同，BGP的会话是基于TCP建立的，建立BGP对等体系关系的的两台路由器并不要求必须直连。

BGP存在两种对等体系类型：EBGP和IBGP：
- EBGP（External BGP）：位于不同自制体系的BGP路由器之间的BGP的对等体关系。两台	路由器之间要建立EBGP对等体关系，必须满足两个要求：
1. 两个路由器所属AS不同（即AS号不同）
2. 配置EBGP时，Peer命令所指定的对等体IP地址要求路由可达，并且TCP直连能够正确建立.

- IBGP （Internal BGP）：位于相同自治系统的BGP之间的BGP邻居关系。

简单说如果两个AS号不同就是EBGP，相同就是IBGP。只要路由可达就意味着TCP179可以连接就能跨设备建立邻居，但是通过默认路由是不行的。

![](image\45465.png)

BGP虽然能跨设备建立邻居关系,但是默认严格执行TTL, 如果没有手工指定默认必须直连(一跳)才能成功建立邻居.
```
R1(config)#ip route 3.3.3.3 255.255.255.255 12.1.1.2
// 不建议写默认路由, 最好使用静态路由, 不然到时候建不起邻居, 但是又ping的通, 查错查的脑壳疼. debug ip tcp transactions, debug ip bgp
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#neighbor 3.3.3.3 remote-as 300
R1(config-router)#neighbor 3.3.3.3 update-source lo0
```

```
R3(config)#ip route 1.1.1.1 255.255.255.255 23.1.1.2
R3(config)#router bgp 300
R3(config-router)#bgp router-id 3.3.3.3
R3(config-router)#neighbor 1.1.1.1 remote-as 100
R3(config-router)#neighbor 1.1.1.1 update-source lo0
```
现在把环境搭建好以后R1和R3是可以互通的, 但是bgp的邻居关系虽然已经制定了, 但是依旧是建立不起来的, 因为BGP默认严格执行TTL, 默认需要直连才能建立邻居.

```
R1#ping 3.3.3.3
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 3.3.3.3, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms

R3#show ip bgp summary
BGP router identifier 3.3.3.3, local AS number 300
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4          100       0       0        1    0    0 never    Idle
```
在R1和R3的BGP里, 手工指定允许接受最大两跳的邻居.
```
R1(config-router)#neighbor 3.3.3.3 ebgp-multihop 2
R3(config-router)#neighbor 1.1.1.1 ebgp-multihop 2
```

有时候建立连接会比较慢
这种时候可以软重置一下bgp或者检查一下tcp 179的连接
```
R1#clear ip bgp * soft

R3#telnet 1.1.1.1 179 /source-interface loopback 0
```
现在BGP的邻居已经成功建立了.
```
R1#show ip bgp summary
BGP router identifier 1.1.1.1, local AS number 100
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
3.3.3.3         4          300      19      19        1    0    0 00:14:58        0
```

## 维护BGP
BGP不会周期性跟新路由, 仅在需要的时候更新, 由于公网的路由可能的动荡, 因此触发更新也会有一定的等待时间, IBGP peer为 5秒, EBGP peer为30秒, 而在这段时间内, BGP仍然可以进行路由信息的搜集, 所以BGP收敛会比较慢.

如果配置了关于BGP的路由策略, 生效方式有两种--**硬重置**和**软重置**
- 重置BGP会话
- 必须触发更新后该策略才会生效

硬重置
- clear ip bgp {neighor-address}
- clear ip bgp * {all}

软重置
- clear ip bgp * soft (in/out)
- in soft reconfig inbound updates
- out soft reconfig outbound updates
软重置仅用于出站或入站策略

查看BGP的路由
1. show ip bgp neighbor {address} received-routes 查看从BGP邻居收到的路由
2. show ip bgp neighbor {address} advertised-routes 查看发给BGP邻居的路由

### 关于使用环回接口作为BGP邻居

在普通EBGP的环境中, 是不建议使用环回接口作为邻居的, 但是:
1. 跨多跳的 EBGP 链路（有多个中间设备转发）
2. 使用 VPN、GRE Tunnel、MPLS 等逻辑隧道通信
3. 双链路冗余结构（防止单条物理链断开即邻居掉线）
4. 作为实验练习或者对等结构清晰可控

这些时候建议使用环回接口建立邻居.在普通场景下，EBGP 建议使用物理接口地址建立邻居（更简单、更直观）；
只有在多跳、逻辑隧道或特定设计需求下，才使用环回接口建 EBGP 邻居，并需要额外配置。

在IBGP里就没有EBGP这么麻烦只要AS号相同就可以直接建立邻居，但是不能直接跨设备建立邻居。建立IBGP邻居尽量使用环回接口，因为环回接口稳定

TCP更新源地址
缺省情况下，BGP使用报文出接口作为TCP连接的本地接口。
在部署IBGP对等体关系时，建议使用Loopback地址作为更新源地址。Loopback接口非常稳定，而且可以借助AS内的IBGP和冗余拓扑来保证可靠性。
如果更新源地址错误，将影响IBGP的邻居关系建立。
在部署EBGP对等体关系时，通常使用直连接口的ip地址作为源地址，如果使用Loopback接口建立EBGP对等体关系，则应注意更新源地址和EBGP多跳问题。
需要注意，用于建立BGP对等体的源地址，不可以再network宣告进BGP。否则会带来BGP邻居关系的震荡（华为则直接不传递）
bgp用环回接口建立邻居关系的额外规则
1. 静态路由
2. ip源头
3. 设置多跳数 不然默认255




# EBGP 建邻方式对比：物理接口 vs 环回接口

## 🧭 概述

在 BGP 中，EBGP 邻居通常默认要求直连（TTL=1）。虽然默认使用物理接口地址建邻，但也可以通过配置支持使用环回接口作为邻居地址。

---

## 🔄 建邻方式对比

| 项目 | 使用物理接口建立邻居 | 使用环回接口建立邻居 |
|------|-----------------------|-----------------------|
| 邻居地址 | 对端物理接口 IP（如 12.1.1.2） | 对端 Loopback 地址（如 3.3.3.3） |
| 是否直连 | 是，TTL=1 默认有效 | 否，必须配置多跳 |
| 必须命令 | 无 | `ebgp-multihop` + `update-source` |
| 路由要求 | 默认路由或直连即可 | 必须能访问对方 Loopback |
| TTL 要求 | 默认 TTL=1，自动满足 | 需手动放宽 TTL（如 ebgp-multihop 2） |
| 接口 flap 影响 | 物理接口断开即邻居 down | loopback 不受物理接口 flap 影响，邻居稳定 |
| 应用场景 | 简单拓扑、直连网络 | 多跳路径、运营商部署、冗余设计 |
| 管理复杂度 | 简单 | 略复杂，需更多配置 |

---

## ✅ 示例配置对比

### ▶ 使用物理接口建立 EBGP 邻居

#### R1（AS 100）

```cisco
router bgp 100
 bgp router-id 1.1.1.1
 neighbor 12.1.1.2 remote-as 300
```

# TCP更新源地址
- 一般而言在AS内部, 网络具备一定的冗余性. 在R1与R3之间, 如果采用直连接口建立IBGP邻居关系, 那么一旦接口或者直连链路发生故障, BGP会话也就中断了, 但事实上,由于冗余链路的存在, R1与R3之间的IP连通性其实并没有中断(仍然可以通过R4到达彼此)

![](image\150600.png)

1. 缺省情况下, BGP使用报文出接口作为TCP连接的本地端口.
2. 在部署IBGP对等体关系时, 建议使用Loopback地址作为更新源地址. Loopback接口非常稳定, 而且可以借助AS内的IGP和冗余拓扑来保证可靠性.
3. 如果更新源地址错误, 将影响IBGP的邻居关系建立.
4. 在部署EBGP对等体关系是, 通常使用直连接口的IP地址作为源地址, 如若使用Loopback接口建立EBGP对等体关系, 则应注意更新源地址和EBGP多跳问题.

*** 需要注意, 用于建立BGP对等体的源地址, 不可再network宣告进BGP. 否则会带来BGP邻居关系震荡 ***

![](image\150601.png)

```
R2#show ip bgp summary
BGP router identifier 2.2.2.2, local AS number 100
BGP table version is 1, main routing table version 1

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
1.1.1.1         4          100       6       4        1    0    0 00:02:11        0
4.4.4.4         4          100       8       7        1    0    0 00:03:35        0
23.1.1.3        4          200       9       9        1    0    0 00:05:57        0
```

首先建立好各个BGP邻居, 现在尝试把1.1.1.1宣告进BGP

```
R1(config)#router bgp 100
R1(config-router)#network 1.1.1.1 mask 255.255.255.255
```

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
 r>i  1.1.1.1/32       1.1.1.1                  0    100      0 i
```
可以看看到在BGP路由1.1.1.1前面有个'r', 说明RIB-failure，路由未被安装进全局路由表（RIB），原因可能是有更优的同前缀路由（如静态路由、OSPF 路由等）已存在。BGP 学到此路由但未被使用转发。

OSPF和EIGRP都有5种报文, BGP报文有:
1. open 报文 (router ID, AS number, 超时间隔默认180s, 路由能力 如是否支持IPv6 ...)
    - 建立会话, 当BGP对等体第一次建立连接时, 他们会互相发送open报文, 开始会员的建立过程.
    - 交换能力, open报文包含了一些关键的参数, 比如BGP版本号, 本地主机的AS号, 保持时间, BGP标识符(router ID). 这些信息用于双方协商会话的特性, 以确保能够正常通信.
    - 协商功能, OPEN报文还可以携带可选参数(Optional Paramenters), 用于协商BGP扩张功能(多协议支持, 路由反射, BGP社区等). 这些扩展功能的支持情况在OPEN报文的交换过程中确认.
    - 会话确认, 如果两个BGP对等体的open报文协商成功, 他们进入后续的BGP状态机步骤(如发送keepalive报文). 如果协商失败会话终止.

2. update 更新报文(NLRI)
    - 通过新路由, update报文用于通过新路由到BGP对等体. 这个报文包含一组网络前缀和与之关联的路径属性, 这些信息可以让对等体直到如何到达这些网络.
    - 撤销路由, update报文还可以用于撤销以前通过的路由. 如果谋个路由已经失效或不再可达, BGP对等体会通过update报文通知其他对等体, 告知该路由不再可用.
    - 更新路由信息, 当谋个路由的路径属性发生变化(下一跳, AS路径, MED值等), BGP会通过update报文向对等体通过这些更新后的信息.
    - 路径属性传递, update报文还包括一系列路径属性, 如AS路径, 下一跳, 本地优先级, local preference, 多口鉴别 MED, BGP社区. 这邪恶路径属性帮助BGP对等评估和选择最佳路径

3. Notification报文, 报错报文, 收到后会立即断开连接也会一直尝试重新连接直到修复为止.
    - 错误报告, 当BGP对等体在协议的运行过程中检测到错误(协议消息格式错误, 接收到无效的参数, 非法的路径属性等), 它会立即发送notification报文, 通知对等体出现了问题.
    - 终止对话, 发送notification报文后, BGP会话会立即关闭. notification报文的发送标志着BGP对等体之间的通信即将结束, 以防传播有问题的路由信息.
    - 错误代码和子代码, notification报文包含一个错误代码和可选的子代码, 用于精确描述检测到的问题类型. 例如 错误代码可以表示"消息头错误", open消息错误, update消息错误等.
    - 提供故障诊断信息, notification报文中还可以包含一些错误相关的附加数据, 这些数据有助于对等体或网络管理员进行故障诊断和排查.

4. keeplive报文, 用于维护BGP对等体之间的会话连接
    - 保持连接状态, keeplive报文用于确认BGP对等体之间的连接仍然有效, 通过定期发送keeplive报文, BGP对于等体能够互相告知对对方其会话连接是否正常, 没有发生中断.
    - 防止超时, 在BGP会话建立后, 如果一段时间内没有任何消息(update或keeplive报文)从对等体接收, 会话可能会被认为已断开. 通过定期发送keeplive报文, 可以防止会话由于长时间没有通信而超时断开.
    - 支持会话建立后不发送路由信息, 在BGP会话建立后, 即使没有需要通告的路由信息, BGP对等体仍然会定期交换keeplive报文, 以确保对等体之间的连接保持活动状态.
    - 简洁高效, keeplive报文非常短, 不携带任何路由信息, 其唯一目的是维持BGP会话的活跃状态, 因此它消耗的网络资源极少, 同时也能有效监控连接的状态.
    
