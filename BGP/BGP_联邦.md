# BGP 联邦原理

![](image/240703.png)

BGP联邦 Confederation

联邦将一个AS划分为若干个子AS. 每个子AS内部建立IBGP对等体, 子AS之间建立EBGP对等体

**配置联邦后, 原AS号将作为每个路由器的联邦ID**

**联邦外部AS仍认为联邦是一个整体打AS. 并不需要了解联邦内部具体的细节**

## 联邦术语

术语: 配置了联邦后, 产生了新的术语

- **联邦AS** 在外部AS看来的一个整体**大AS**
- **成员** 联邦内部划分的若干**小AS**(使用私AS号)

- **联邦IBGP对等体** 成员**AS号相同**的IBGP对等体 
- **联邦EBGP对等体** 成员**AS号不同**的EBGP对等体

注意联邦EBGP对等体与普通的EBGP对等体是由不同的, 体现在建立对等体时**OPEN报文携带的AS号不相同, 因此联邦EBGP需要特别配置, 联邦BGP则不需要**

**另外在联邦配置中另外注意的事项: 联邦EBGP多跳问题**

配置BGP联邦后, 就i打破了IBGP水平分割原则, R3从EBGP对等体学习到的路由传递给联邦IBGP对等体(R3给R4), 再传给联邦EBGP对等体(R5), 最后再传给EBGP对等体(R5传给R2)

### BGP联邦配置

```
R3(config)#router bgp 64512
R3(config-router)#bgp confederation identifier 300
R3(config-router)#neighbor 13.1.1.1 remote-as 100
R3(config-router)#neighbor 4.4.4.4 remote-as 64512
R3(config-router)#neighbor 4.4.4.4 update-source lo0
```

```
R1(config-if)#router bgp 100
R1(config-router)#bgp router-id 1.1.1.1
R1(config-router)#neighbor 13.1.1.3 remote-as 300
```

```
R4(config)#router bgp 64512
R4(config-router)#bgp confederation identifier 300

R4(config-router)#neighbor 3.3.3.3 remote-as 64512
R4(config-router)#neighbor 3.3.3.3 update-source lo0

R4(config-router)#bgp confederation peers 64513
R4(config-router)#neighbor 5.5.5.5 remote-as 64513
R4(config-router)#neighbor 5.5.5.5 update-source lo0
R4(config-router)#neighbor 5.5.5.5 ebgp-multihop
```

```
R5(config)#router bgp 64513
R5(config-router)#bgp router-id 5.5.5.5
R5(config-router)#bgp confederation identifier 300

R5(config-router)#bgp confederation peers 64512
R5(config-router)#neighbor 4.4.4.4 remote-as 64512
R5(config-router)#neighbor 4.4.4.4 update-source lo0
R5(config-router)#neighbor 4.4.4.4 ebgp-multihop

R5(config-router)#neighbor 25.1.1.2 remote-as 200
```

成员AS之间在配置联邦EBGP对等体时, **需要额外配置联邦EBGP对等体AS号**, 和用于真正的EBGP邻居做区分.

真正的EBGP对等体使用**联邦AS号(AS300)发送Open消息(R3和R1, R5和R2)**

联邦EBGP对等体使用**成员AS号(AS64512和64513)发送open消息(R4和R5)**

另外如果使用**环回接口**建立EBGP邻居时, 仍然要记得配置**EBGP多跳**

R2和R5都会有R1和R2宣告的所有路由, 打破了IBGP在AS内的水平分割原则**其实想想也可以理解, 联邦其实是在遵循EBGP对等体之间传递路由的规则**


#### BGP联邦内的路径属性 Next-hop

Nexthop下一跳在**联邦EBGP邻居之间传递并不会发送改变**

**这一点和普通的EBGP邻居之间传递Nexthop是不相同的,** 普通的EBGP邻居将该路由的Next Hop设置为自己的TCP连接源地址.


### BGP联邦内的路径属性-Med

![](image/240703.png)

```
R5#show ip bgp 172.16.1.1
BGP routing table entry for 172.16.1.1/32, version 3
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  (64512) 100
    13.1.1.1 (metric 30) from 4.4.4.4 (4.4.4.4)
      Origin IGP, metric 0, localpref 100, valid, confed-external, best
      rx pathid: 0, tx pathid: 0x0
```

R1传递过来的172.16.1.1 metric是30, 现在使用路由策略在R1上修改MED值

```
R1(config)#route-map MED permit 10
R1(config-route-map)#set metric 111

R1(config-route-map)#router bgp 100
R1(config-router)#neighbor 13.1.1.3 route-map MED out
```

```
R5#show ip bgp 172.16.1.1
BGP routing table entry for 172.16.1.1/32, version 4
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  (64512) 100
    13.1.1.1 (metric 30) from 4.4.4.4 (4.4.4.4)
      Origin IGP, metric 111, localpref 100, valid, confed-external, best
      rx pathid: 0, tx pathid: 0x0
```

现在验证R1传递过来的路由MED已经改为设置的111了. 在普通的EBGP对等体之间传递MED时, MED的规则是**不会跨AS传递**, 但是在联邦EBGP对等体之间则仍然携带.

### BGP联邦内的路径属性 Localpreference

```
R3(config)#route-map LP permit 10
R3(config-route-map)#set local-preference 500

R3(config-route-map)#rotuer bgp 64512
R3(config-route-map)#router bgp 64512
R3(config-router)#neighbor 13.1.1.1 route-map LP in
```

```
R5#show ip bgp 172.16.1.1
BGP routing table entry for 172.16.1.1/32, version 5
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     1
  Refresh Epoch 1
  (64512) 100
    13.1.1.1 (metric 30) from 4.4.4.4 (4.4.4.4)
      Origin IGP, metric 111, localpref 500, valid, confed-external, best
      rx pathid: 0, tx pathid: 0x0
```

可以验证到 之前Localpreference由之前的100更改为设置的500, 同样Local preference只会在联邦内保持不变, 不会传递出联邦区域.

### BGP 联邦防环

- 联邦的本质仍然是建立EBGP对等体来传递路由, 所以联邦的防环仍然使用EBGP的防环规则, 使用 AS-Path进行防环
- 但是普通的AS-Path用于真正的EBGP对等体, 是无法用在联邦内的
- 因此定义了另外两种AS-Path类型:
1. AS_Confed_Sequence
2. AS_confed_Set

#### AS_Path 的四种类型

1. AS_Set 无序
- 无序合集, 主要由BGP路由聚合产生

2. AS_Sequence 有序
- AS号顺序不能改变
- AS_Sequence 越短, 路由优先级越高

3. AS_Confed_Sequence
- 与 AS_SEQUENCE 类似，但仅限于联盟内部。
- 外部 AS 看到的路径中 不会显示联盟内部的 AS_CONFED_SEQUENCE，只会显示一个公共 AS（Confederation Identifier）

4. AS_Confed_Set
- 与 AS_CONFED_SEQUENCE 类似，但 AS_CONFED_SET 是无序的，用 方括号 [] 表示
- 主要在联盟内部进行路由聚合时出现。

![](image/270700.png)

默认的联邦内AS-Path是 AS_Confed_Sequence 联邦有序, 使用小括号 () 表示

在联邦内配置**手工**汇总AS_Path类型则为4. AS_Confed_Set 联邦无序, 使用中括号 [] 表示

### 联邦内的BGP 路由路劲属性

- 在联邦内部保留联邦外部路由的Next_Hop属性不变
- 公布给联邦的路由的MED属性在整个联邦内予以保留
- 路由的Localpreference属性在整个联邦内予以保留
- 在联邦范围内, 将成员AS压入AS_Path, **但不公布到联邦外**, 并使用类型3(联邦有序)和类型4(联邦无序)
-联邦内成员AS不参与AS_Path长度计算

## RR路由反射器与联邦的比较

|路由反射器|联邦|
|:-----:|:--------:|
|不需要更改现有的网络拓扑, 兼容性好|需要修改逻辑拓扑|
|配置方便, 客户机不知道自己是客户机|所有设备需要重新进行配置, 且所有设备必须支持联邦功能|
|集群与集群之间仍需全连接|联邦的子AS之是特殊的EBGP连接, 不需要全连接|
|在大型网络中应用广泛|应用较少|

1. 联邦需要重新划分区域, 对现网改动较大
2. 反射器在配置时, 只需要对RR进行配置, 客户机不需要做任何其他的操作, 联邦需要在所有路由器上进行配置
3. RR与RR之间需要IBGP全互联
4. 路由反射器应用较为广泛, 联邦应用较少
5. 如果在一个AS内部如果想要按照EBGP防环规则去繁殖路由传路由的话, 那就需要联邦

