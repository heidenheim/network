
# Layer7 Inspection -- HTTP policy-map，自定义 URL 控制

这一部分是 ZBF（Zone-Based Firewall）的进阶用法 

inspect http 只是检查协议合法性

现在要实现 应用层的细粒度控制（例如基于 URL 的过滤）


## **Layer7 Inspection** 概念

普通的 ZBF inspect 只能判断协议是否正常,不会理解内容

而 Layer7 inspection 可以深入应用层

对 HTTP 的 URL / header / method 等进行检查和控制

- 举例
    1. 允许访问 example.com 但阻止访问 facebook.com
    2. 阻止下载 .exe 文件
    3. 只允许 HTTP GET 不允许 POST


- 关键命令

    ```
    class-map type inspect http match-all BLOCK_FACEBOOK
    match request uri regex "facebook"
    ```

    1. `type inspect http` → 表示这是 HTTP 专用 class-map
    2. `match request uri regex` → 匹配 URL 路径或域名
    2. `"facebook"` → 用正则表达式匹配


**定义 HTTP policy-map**

```
policy-map type inspect http HTTP_POLICY
 class BLOCK_FACEBOOK
  drop log
 class class-default
  inspect
```

- `drop log` → 丢弃并记录日志
- `inspect` → 正常允许并检查其他流量


**在 ZBF policy-map 中调用**

必须把 HTTP policy-map 绑定到 ZBF 的 Layer7 policy

```
policy-map type inspect INSIDE_TO_OUT
 class class-default
  inspect
   service-policy HTTP_POLICY
```


### 配置


![](../../image/Security/04102502.png)


```
conf t
!
! ================= 基础接口/IP/NAT/DHCP =================
!
interface e0/0
 ip address dhcp
 ip nat outside
 no shutdown
!
interface e0/1
 ip address 10.1.1.254 255.255.255.0
 ip nat inside
 no shutdown
!
! PAT (NAT Overload)
ip access-list standard NAT_INSIDE
 permit 10.1.1.0 0.0.0.255
ip nat inside source list NAT_INSIDE interface e0/0 overload
!
! DHCP（给内网PC发地址）
ip dhcp excluded-address 10.1.1.250 10.1.1.254
ip dhcp pool USER
 network 10.1.1.0 255.255.255.0
 default-router 10.1.1.254
 dns-server 8.8.8.8
!
! ================= ZBF：定义 Zone 并绑定接口 =================
!
zone security INSIDE
zone security OUTSIDE
!
interface e0/1
 zone-member security INSIDE
interface e0/0
 zone-member security OUTSIDE
!
! ================= HTTP L7：正则 + HTTP类 + 子策略 =================
!
! 1) 定义正则：命中 URI/域名里含 "youtube"
parameter-map type regex YOUTUBE
 pattern "youtube"
!
! 2) HTTP 专用 class：命中则视为要拦
class-map type inspect http match-all BLOCK_YOUTUBE
 match request uri regex YOUTUBE
! （可改为：match host header regex YOUTUBE 以按Host匹配域名）
!
! 3) HTTP 子策略：命中后 reset；未命中HTTP默认allow
policy-map type inspect http HTTP_POLICY
 class BLOCK_YOUTUBE
  reset
!
! ================= L4：识别HTTP + 兜底有状态 =================
!
! 4) 识别HTTP（用于父策略里挂HTTP子策略）
class-map type inspect match-all HTTP_TRAFFIC
 match protocol http
!
! 5) 兜底的有状态基础（DNS/HTTPS/ICMP等）
class-map type inspect match-any STATEFUL_BASE
 match protocol tcp
 match protocol udp
 match protocol icmp
!
! 6) 父策略：对HTTP做inspect并挂HTTP子策略；其他基础流量inspect；杂流量drop
policy-map type inspect FIREWALL_POLICY
 class HTTP_TRAFFIC
  inspect
  service-policy http HTTP_POLICY
 class STATEFUL_BASE
  inspect
 class class-default
  drop
!
! ================= 把父策略应用到流向（Zone-Pair） =================
!
zone-pair security IN_TO_OUT source INSIDE destination OUTSIDE
 service-policy type inspect FIREWALL_POLICY
!
end
write memory

```


简要解释

接口/NAT/DHCP：让内网 10.1.1.0/24 通过 e0/0 出网；PC 自动获得 IP/DNS。

Zones & zone-member：把 e0/1 放进 INSIDE，e0/0 放进 OUTSIDE。只有进入了 Zone 的接口，其流量才受 ZBF 控制。

HTTP L7 子策略：

parameter-map type regex 定义可重用的正则；

class-map type inspect http 仅用于 HTTP 解析；

policy-map type inspect http 是 HTTP 专用子策略，这里不能写 class-default 或 drop，用 reset 拦截命中的 HTTP 请求。

父策略（L3/L4）：

HTTP_TRAFFIC → inspect 并 service-policy http HTTP_POLICY 挂上 L7；

STATEFUL_BASE 让 非HTTP 的 TCP/UDP/ICMP（DNS、HTTPS、Ping 等）也走有状态回流；

class-default drop 拦住其他未知协议。

zone-pair：把父策略应用在 INSIDE→OUTSIDE 的方向上。

⚠️ 提醒：该 HTTP L7 检查只对明文 HTTP 有效；HTTPS 看不到 URI/Host，无法用这个方法拦。若要拦 HTTPS，请考虑 DNS 层拦截、URL 过滤云库 或 SSL 解密代理 等方案。


验证

```
show zone-pair security
show class-map type inspect
show class-map type inspect http
show policy-map type inspect http
show policy-map type inspect | s FIREWALL_POLICY
show policy-map type inspect zone-pair sessions
```


回滚

```
conf t
no zone-pair security IN_TO_OUT
no policy-map type inspect FIREWALL_POLICY
no class-map type inspect match-any STATEFUL_BASE
no class-map type inspect match-all HTTP_TRAFFIC
no policy-map type inspect http HTTP_POLICY
no class-map type inspect http match-all BLOCK_YOUTUBE
no parameter-map type regex YOUTUBE
!
interface e0/1
 no zone-member security INSIDE
interface e0/0
 no zone-member security OUTSIDE
no zone security INSIDE
no zone security OUTSIDE
end
```