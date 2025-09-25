
![](image.png)

# ACL与基础防火墙

1. ACL 的作用: 访问控制, 流量过滤, 基础防火墙
2. 区分标准(standard)和扩展(extend) ACL, 以及使用场景
    - 标准 ACL：只匹配源 IP 地址，通常应用在靠近目标的位置。
    - 扩展 ACL：匹配源、目的 IP、协议、端口等，可以更精确控制流量，通常应用在靠近源的位置。
    - 方向：in / out，ACL 是接口相关的。
3. 扩展ACL实现常见的安全需求:
    - 阻止特定主机访问某个服务器
    - 阻止某些应用端口
    - 仅允许内部网段访问外部, 进制外部主动访问
4. 理解 ACL 在流量路径中的作用, 和 Stateful Firewall 的差别


## 1. 阻止访问TCP 80(HTTP)

```
R2(config)#ip access-list extended BLOCK-HTTP
R2(config-ext-nacl)#deny tcp host 10.1.1.10 host 10.3.3.10 eq 80
R2(config-ext-nacl)#permit ip any any
R2(config)#int e0/0
R2(config-if)#ip access-group BLOCK-HTTP in
```

## 2. 阻止所有外部访问内部, 仅允许内部访问外部

```
R2(config)#ip access-list extended INSIDE-TO-OUT
R2(config-ext-nacl)#permit ip 10.1.1.0 0.0.0.255 any
R2(config-ext-nacl)#deny ip any any
R2(config)#int e0/1
R2(config-if)#ip access-group INSIDE-TO-OUT out
```

## 3. 阻止 ICMP 攻击（例如阻止外部 ping 内部）

```
R2(config)#ip access-list extended BLOCK-ICMP
R2(config-ext-nacl)#deny icmp any 10.1.1.0 0.0.0.255
R2(config-ext-nacl)#permit ip any any
R2(config)#int e0/0
R2(config-if)#ip access-group BLOCK-ICMP in
```

### 总结

1.	为什么 ACL 被称为 无状态过滤？
    - 因为它不会自动跟踪连接，返回流量也必须显式允许。
2.	如果要实现 状态跟踪（stateful firewall），你觉得和 ACL 有什么不同？
3.	在企业网络中，ACL 通常用在哪些地方？
    - 边界路由器（隔离 Internet）
    - 核心交换机（隔离部门间访问）
    - VPN 隧道入口

验证命令
1. show access-lists — 查看 ACL hit count（确认规则是否生效）。
2. ping / telnet / curl — 模拟不同流量。
3. debug ip packet — 实验环境中可用，观察 ACL 丢弃情况。
