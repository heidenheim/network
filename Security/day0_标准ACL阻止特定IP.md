### access-list 基本语法, 方向和应用接口
![](image/456511)

正常情况下PC1和PC是能够互通的.
```
PC1> ping 192.168.2.10

84 bytes from 192.168.2.10 icmp_seq=1 ttl=63 time=17.037 ms
84 bytes from 192.168.2.10 icmp_seq=2 ttl=63 time=0.458 ms
84 bytes from 192.168.2.10 icmp_seq=3 ttl=63 time=0.773 ms
84 bytes from 192.168.2.10 icmp_seq=4 ttl=63 time=0.340 ms
84 bytes from 192.168.2.10 icmp_seq=5 ttl=63 time=0.939 ms
```

在R1上写一条ACL
```
R1(config)#aaccess-list 10 deny 192.168.1.10 log //log为显示日志
R1(config)#aaccess-list 10 permit any
```
应用在R1的e0/0接口上
```
R1(config)#int e0/0
R1(config-if)#ip access-group 10 in //in方向是因为要控制从e0/0进来的流量而不是出去的.
```

现在再用PC1 ping PC2, 就已经不通了
```
PC1> ping 192.168.2.10

*192.168.1.1 icmp_seq=1 ttl=255 time=1.052 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=2 ttl=255 time=0.697 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=3 ttl=255 time=0.461 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=4 ttl=255 time=0.867 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=5 ttl=255 time=0.980 ms (ICMP type:3, code:13, Communication administratively prohibited)
```

并且在R1上会提示出拦截日志
```
R1#
*Jun 10 18:52:36.534: %SYS-5-CONFIG_I: Configured from console by console
R1#
*Jun 10 18:53:30.559: %SEC-6-IPACCESSLOGNP: list 10 denied 0 192.168.1.10 -> 192.168.2.10, 1 packet
R1#
```

使用命令 show access-lists 可以查看ACL的命中次数
```
R1#show access-lists
Standard IP access list 10
    10 deny   192.168.1.10 log (20 matches)
    20 permit any
```

知识点

|标准|ACL|只基于源 IP 控制流量|
|:--:|:-:|:----------------:|
|方向|in|是进入接口的流量，out 是离开接口的流量|
|应用接口|通常配置在流量进来的接口，更高效||

#### 扩展--使用ACL阻止特定端口服务.

**使用扩展ACL阻止ICMP**
```
R1(config)#access-list 100 deny icmp host 192.168.1.10 host 192.168.2.10 //阻止PC1 ping PC2
R1(config)#access-list 100 permit ip any any //允许所有

#应用到接口
R1(config)#int e0/0
R1(config-if)#ip access-group 100 in

R1#show access-lists 100
Extended IP access list 100
    10 deny icmp host 192.168.1.10 host 192.168.2.10
    20 permit ip any any
# 现在可以看到是没有命中的

```

```
PC1> ping 192.168.2.10

*192.168.1.1 icmp_seq=1 ttl=255 time=0.652 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=2 ttl=255 time=1.335 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=3 ttl=255 time=0.477 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=4 ttl=255 time=0.472 ms (ICMP type:3, code:13, Communication administratively prohibited)
*192.168.1.1 icmp_seq=5 ttl=255 time=0.533 ms (ICMP type:3, code:13, Communication administratively prohibited)
```

```
R1#show access-lists 100
Extended IP access list 100
    10 deny icmp host 192.168.1.10 host 192.168.2.10 (5 matches)
    20 permit ip any any
# 看到ACL已经命中5次
```

