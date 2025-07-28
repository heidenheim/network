# 通告原则

1. BGP通过network, redistribute, aggregate聚合方式生成BGP路由后, 通过update报文将BGP路由传递给对等体.
2. BGP通过遵循以下原则,
    - 只发布最优路由
    - 从IBGP对等体获得的路由, 不会再发给IBGP对等体, 只会发布给所有的EBGP对等体(IBGP防环/水平分割)
    - 从EBGP对等体获取的路由, 会发布给所有对等体(EBGP和IBGP)
    - IBGP水平分割的意思, 从IBGP对等体获取的路由, 不会发送给IBGP对等体
    - BGP同步规则指的是, 当一台路由器从自己的IBGP对等体学习到一条BGP路由时(这类路由被称为IBGP路由), 它将不能使用该条路由或把这条路由通告给自己的EBGP对等体, 除非它又从IGP协议(eg: OSPF等, 也包含静态路由), 学习到这条路由, 也就是要求IBGP路由与IGP路由同步. 同步规则主要用于规避BGP路由黑洞.
    
***BGP路由通告原则, BGP的同步规则已经不再使用***

## IBGP和EBGP防环(Preventing loops)

Ebgp用as号防环，ibgp不能跨路设备传路由，所以R3和R1上没有路由，如果R3要学R1的路由，必须手动指定邻居。