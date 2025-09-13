只要是动态路由协议, 都具有产生默认路由的功能, IGP中的OSPF, EIGRP, RIP都可以, 当然作为高级的BGP也可以

BGP产生默认路由的方式有三种

1. network通告IGP中的默认路由
2. default-information originate 产生默认路由
3. 针对邻居产生默认路由default-originate




```
R2(config)#interface tunnel 1
R2(config-if)#ip address 10.23.23.2 255.255.255.255
R2(config-if)#no shu
R2(config-if)#tunnel source 2.2.2.2
R2(config-if)#tunnel destination 3.3.3.3

R2(config)#ip route 10.23.23.3 255.255.255.255 12.1.1.1


R3(config)#int tunnel 1
R3(config-if)#ip address 10.23.23.3 255.255.255.255
R3(config-if)#no shu
R3(config-if)#tunnel source 3.3.3.3
R3(config-if)#tunnel destination 2.2.2.2

R3(config)#ip route 10.23.23.2 255.255.255.255 13.1.1.1