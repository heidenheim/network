
掌握 ZBF 的服务策略（Service Policy）

理解基于应用的检查（Deep Packet Inspection）

学习如何允许部分流量通过（例如 DNS、HTTP、ICMP）

动手实验：配置多个 class-map 和 policy-map，实现精细化控制

1. ZBF 三大要素
    - class-map 匹配流量类别(协议, ACL, 应用)
    - policy-map 定义对这些流量的动作(inspect / drop / pass)
    - zone-pair 把 policy-map 应用在 zone 到 zone 的方向

2. 三种动作区别
    - inspect 有状态检测，返回流量会自动允许（适合 TCP/UDP 协议）
    - drop 丢弃报文 = deny
    - pass 放行，但无状态（返回流量也必须显式允许，否则会被丢弃）。

## pass 和 inspect 的根本区别

1. inspect（有状态）
    1) 会跟踪连接（Session Table）。
    2) 当 Inside 发起请求时，ZBF 自动允许返回的流量。
    3) 适合 TCP/UDP 应用（HTTP、DNS、ICMP 等）。

2. pass（无状态）
    - 只是单纯放行 单方向流量，不会记录会话。
    - 返回流量必须 单独写规则，否则会被丢弃。
    - 更像是一个单向 “permit”。

### 简单说：
1. inspect = permit + 建立状态表（自动允许回流）。
2. pass = permit only（回流要自己管）。

#### Inside → Outside 只允许 HTTP，为什么外部返回能通过？

你在 policy-map 里配置了：

```
class-map match protocol http
  inspect
```

当 Inside 客户端发起 HTTP 请求时，ZBF 在 会话表 里记录下：

- 源 IP/端口（Inside）
- 目的 IP/端口（Outside，TCP/80）
- 返回的 HTTP 响应符合这个会话，就会被自动允许。

⚡ 所以返回流量能过，是因为 inspect 建立了状态表，让回流自动放行。

##### 如果改成 pass，返回的 HTTP 响应还能否通过？

- 答案：不能。

- 原因：
    1. pass 只在 Inside → Outside 放行了请求。
    2. Outside → Inside 的响应流量没有匹配规则，会被 ZBF 丢弃。
    3. 除非你再建一个 zone-pair（OUTSIDE → INSIDE）并配置相应的 pass。

👉 换句话说：

- inspect = 一条规则管双向。
- pass = 必须两条规则，分别配置 出站 和 回流。

###### 总结

1. inspect：有状态，自动允许回流。
2. pass：无状态，只单方向放行，回流需要额外配置。
