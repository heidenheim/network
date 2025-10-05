
# IPsec(Internet Protocol Security)

IPsec 是一套在三层(网络层)上提供**加密与认证**的安全协议, 用于在点两个端点(如路由器, VPN网关)之间建立**安全隧道**.


**三大功能**

| 功能                 | 协议                                  | 作用               |
| ------------------ | ----------------------------------- | ---------------- |
| 认证（Authentication） | AH（Authentication Header）           | 验证数据来源与完整性，不加密内容 |
| 加密（Encryption）     | ESP（Encapsulating Security Payload） | 加密数据，提供保密性，可选认证  |
| 密钥交换               | IKE（Internet Key Exchange）          | 协商密钥与算法          |


## IKE(Internet Key Exchange)

IKE 是 IPsec 的控制平面协议. 用来协商安全参数, 交换密匙


**阶段 1：建立 IKE SA（安全关联）**

用于保护后续的 IPsec 协商通道。


- 主要任务：
    1. 认证对端身份（预共享密钥或证书）
    2. 协商加密算法, 哈希算法, DH 组
    3. 建立安全通道（ISAKMP SA）

两种模式

| 模式              | 特点   | 报文数量 | 用途          |
| --------------- | ---- | ---- | ----------- |
| Main Mode       | 安全性高 | 6 报文 | 常用于站点到站点    |
| Aggressive Mode | 快速   | 3 报文 | 常用于动态 IP 场景 |


**阶段 2：建立 IPsec SA**

用于实际加密业务流量（ESP/AH）。


- 协商内容：
    1. ESP/AH 协议类型
    2. 加密算法（AES、3DES）
    3. 认证算法（SHA、MD5）
    4. 生存期（lifetime）

- 模式：
    1. Quick Mode（3 报文）


###  IKEv1 报文流程图(Main Mode)

Initiator (R1)                      Responder (R2)
----------------------------------------------------------
1. SA proposal (encryption, hash, DH group)
2. SA response (chosen proposal)
3. DH public value + nonce
4. DH public value + nonce
5. Authentication (ID + hash)
6. Authentication (ID + hash)
===> 建立 IKE SA（ISAKMP SA）


之后进入阶段2

7. Quick mode 1: 选择 ESP/AH 参数
8. Quick mode 2: 交换密钥与 nonce
9. Quick mode 3: 确认建立 IPsec SA
===> 建立 IPsec SA（ESP 隧道）


#### 配置


**R1**

```
// 定义一条 IKEv1“提案”（编号10，数字越小优先级越高）。双方要至少有一条提案的参数一致，才能谈成。
R1(config)#crypto isakmp policy 10

// hash：用于 IKE 报文完整性校验（SHA-1）。
// encryption：用于 IKE 报文加密（AES）。
// authentication pre-share：对端身份用**预共享密钥（PSK）**验证。
R1(config-isakmp)#hash sha
R1(config-isakmp)#encryption aes
R1(config-isakmp)#authentication pre-share

// DH 组：用于 Diffie-Hellman 密钥交换的强度（组2=1024位，考试常用；更安全推荐 14=2048位）。
R1(config-isakmp)#group 2

// 阶段1 SA 的寿命（单位：秒）。到期会重协商。
R1(config-isakmp)#lifetime 86400

// 设置预共享密钥（PSK）并指明对端公网IP。两边的密钥字符串必须一致。
// 如果对端是动态地址/在NAT后面，会用到别的写法，这里先不展开。
R1(config)#crypto isakmp key CISCO123 address 200.1.1.2

# IKEv1 阶段1（建“控制通道”/ISAKMP SA）
-----------------------------------------------

// 定义 IPsec 数据面的加密与完整性算法：
    加密：esp-aes
    完整性：esp-sha-hmac
    mode tunnel：隧道模式（常见的站点到站点用法）。
R1(config)#crypto ipsec transform-set TS esp-aes esp-sha-hmac
R1(cfg-crypto-trans)#mode tunnel

# IPsec 阶段2（加密“业务数据”/IPsec SA）
------------------------------------------------

//创建一个名为 VPN 的 crypto map 实例（顺序号10）。
    set peer：对端公网 IP。
    set transform-set TS：阶段2用哪套算法。
    match address 100：哪些流量要用这条隧道（“有趣流量”）

R1(config)#crypto map VPN 10 ipsec-isakmp
% NOTE: This new crypto map will remain disabled until a peer
        and a valid access list have been configured.
// 系统提示“will remain disabled until a peer and a valid access list…” 就是说：没指定 peer 或 ACL 之前，这张 crypto map 还不能用。

R1(config-crypto-map)#set peer 200.1.1.2
R1(config-crypto-map)#set transform-set TS

R1(config-crypto-map)#match address ?
  <100-199>    IP access-list number
  <2000-2699>  IP access-list number (expanded range)
  WORD         Access-list name

R1(config-crypto-map)#match address 100
------------------------------------------------------

// 选择“哪些流量进隧道”（有趣流量选择器）
R1(config)#access-list 100 permit ip 10.1.1.0 0.0.0.255 10.2.2.0 0.0.0.255
---------------------------------------------------------

// 把上面配好的 “去哪儿（peer） + 怎么加密（transform-set） + 哪些流量（ACL）” 套到 公网口。
    这样当 有趣流量 从 e0/0 出去时，就会触发 IKE，建好隧道后把数据封装成 ESP 发给对端。
R1(config)#int e0/0
R1(config-if)#crypto map VPN
```


1. 把它们放回到“阶段流程”里看（按发生顺序）

    - PC1→PC2 的包经过 R1 e0/0 出口 → 命中 crypto map VPN 的 match address 100 → 这是有趣流量。

    - R1 发现“这流量要加密”，检查有没有现成的 IPsec SA；没有就发起 IKEv1 阶段1：
        1) 用 crypto isakmp policy 10 里的算法与对端“对表”；
        2) 用 crypto isakmp key ... 的 PSK 相互认证；
        3) 建立 ISAKMP SA（控制通道）。

    - 进入 阶段2（Quick Mode）：
        1) 用 transform-set TS 里的算法谈IPsec SA（数据面）；
        2) 选择器来自 ACL 100（10.1.1.0/24 ↔ 10.2.2.0/24）。

    - 隧道就绪 → 业务包被 ESP 封装/加密 → 发到 set peer（200.1.1.2）。


2. 已经看到的验证结果如何解读

    - show crypto isakmp sa = QM_IDLE
        → 阶段1 OK，阶段2也完成并空闲待用。

    - show crypto ipsec sa 里 #pkts encaps/decaps 增长
        → 真实业务正在被加密/解密。


3. 常见联动问题（你以后一看到就会想起）

    - NAT：要给“有趣流量”做 NAT 豁免，否则被改了地址就不再匹配 ACL 100，隧道不触发。

    - ZBF/ACL：要放行 UDP/500（IKE）、UDP/4500（NAT-T）、协议50（ESP）。

    - 路由：两端要能到达对端公网 IP（静态路由或默认路由）。

    - 参数不一致：任一边的 isakmp policy / transform-set 不匹配都会谈不成。

4. 超简速记（考试/面试背诵版）

    - 阶段1怎么谈：`crypto isakmp policy`（hash/encryption/auth/group/lifetime）+ `crypto isakmp key`

    - 阶段2怎么加密：`crypto ipsec transform-set`（esp-xxx + esp-yyy）

    哪些流量进隧道：`access-list` + `crypto map ... match address`

    - 对端是谁：`crypto map ... set peer x.x.x.x`

    - 从哪儿出去：`interface e0/0` → `crypto map VPN`


**R2**

```
R2(config)#crypto isakmp policy 10
R2(config-isakmp)#encryption aes
R2(config-isakmp)#hash sha
R2(config-isakmp)#authentication pre-share
R2(config-isakmp)#group 2
R2(config-isakmp)#lifetime 86400

R2(config)#crypto isakmp key CISCO123 address 200.1.1.1

R2(config)#crypto ipsec transform-set TS esp-aes esp-sha-hmac
R2(cfg-crypto-trans)#mode tunnel

R2(config)#crypto map VPN 10 ipsec-isakmp
% NOTE: This new crypto map will remain disabled until a peer
        and a valid access list have been configured.
R2(config-crypto-map)#set peer 200.1.1.1
R2(config-crypto-map)#set transform-set TS
R2(config-crypto-map)#match address 100
R2(config)#access-list 100 permit ip 10.2.2.0 0.0.0.255 10.1.1.0 0.0.0.255

R2(config)#int e0/0
R2(config-if)#crypto map VPN
```


```
R1#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status

IPv6 Crypto ISAKMP SA



















A. 先查 NAT 和 ZBF（两条速查命令）

在 R1、R2 各跑：
```
show run | inc ip nat|zone-member
show ip nat translations
```

若看到 ip nat inside source ... overload（尤其 ACL 1/标准 ACL）= 有 NAT；

若接口上还挂着 zone-member security = 有 ZBF 在工作。


B. 解决 NAT：给 VPN 流量做“豁免”

目标：10.1.1.0/24 ↔ 10.2.2.0/24 不做 NAT，其余照常 PAT。

R1（示例，R2 对称改 10.2.2.0→10.1.1.0）

任选一种方式；**方式1（扩展 ACL）**最简单。

方式1：扩展 ACL 直接控制 NAT

conf t
ip access-list extended NAT-EXEMPT
 deny   ip 10.1.1.0 0.0.0.255 10.2.2.0 0.0.0.255   ! VPN 流量不做 NAT
 permit ip 10.1.1.0 0.0.0.255 any                  ! 其他照常 PAT

! 用新的 ACL 重绑 NAT（先删旧的那条 NAT）
no ip nat inside source list 1 interface e0/0 overload
ip nat inside source list NAT-EXEMPT interface e0/0 overload
end


方式2：route-map 豁免（更通用）

conf t
ip access-list extended VPN-TRAFFIC
 permit ip 10.1.1.0 0.0.0.255 10.2.2.0 0.0.0.255
route-map NONAT permit 10
 match ip address VPN-TRAFFIC

! 先删旧 NAT，再用 route-map 的 NAT
no ip nat inside source list 1 interface e0/0 overload
ip nat inside source route-map NONAT interface e0/0 overload
end


在 R2 做对称配置（把 10.1.1.0 与 10.2.2.0 互换）。








R1#show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
200.1.1.2       200.1.1.1       QM_IDLE           1001 ACTIVE

IPv6 Crypto ISAKMP SA




R1#show crypto ipsec sa

interface: Ethernet0/0
    Crypto map tag: VPN, local addr 200.1.1.1

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (10.1.1.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (10.2.2.0/255.255.255.0/0/0)
   current_peer 200.1.1.2 port 500
     PERMIT, flags={origin_is_acl,}
    #pkts encaps: 4, #pkts encrypt: 4, #pkts digest: 4
    #pkts decaps: 4, #pkts decrypt: 4, #pkts verify: 4
    #pkts compressed: 0, #pkts decompressed: 0
    #pkts not compressed: 0, #pkts compr. failed: 0
    #pkts not decompressed: 0, #pkts decompress failed: 0
    #send errors 0, #recv errors 0

     local crypto endpt.: 200.1.1.1, remote crypto endpt.: 200.1.1.2
     plaintext mtu 1438, path mtu 1500, ip mtu 1500, ip mtu idb Ethernet0/0
     current outbound spi: 0x91EA7096(2448060566)
     PFS (Y/N): N, DH group: none

     inbound esp sas:
      spi: 0x2C166963(739666275)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 1, flow_id: SW:1, sibling_flags 80004040, crypto map: VPN
        sa timing: remaining key lifetime (k/sec): (4227682/3447)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     inbound ah sas:

     inbound pcp sas:

     outbound esp sas:
      spi: 0x91EA7096(2448060566)
        transform: esp-aes esp-sha-hmac ,
        in use settings ={Tunnel, }
        conn id: 2, flow_id: SW:2, sibling_flags 80004040, crypto map: VPN
        sa timing: remaining key lifetime (k/sec): (4227682/3447)
        IV size: 16 bytes
        replay detection support: Y
        Status: ACTIVE(ACTIVE)

     outbound ah sas:

     outbound pcp sas:
R1#





可选加固（保持两端一致）

这些不是必须，但更贴近考试/生产最佳实践。

conf t
! 更强的 DH 组（如 14）
crypto isakmp policy 10
 group 14
! 或使用 AES-256
 encryption aes 256
! IKEv1 的 SHA 这里保持 sha（即 SHA-1），若平台支持可考虑迁移 IKEv2 再上 sha256

! 周期探活（掉线能更快重建；命令略有平台差异，如不识别可忽略）
crypto isakmp keepalive 10 periodic
end





常见小疑问，顺手答一下

为什么第一包丢？
触发 Quick Mode、建 SA、或对端/网关 ARP 学习期间丢 1 包很常见。

QM_IDLE 是不是最终态？
是。表示阶段1已完成、阶段2（IPsec SA）也就绪，等待/处理业务流量。

怎么确认 crypto map 真在公网口？
show crypto map interface e0/0 应能看到 crypto map VPN。

如果你贴一份 show crypto ipsec sa 的片段，我可以帮你快速解读每一行（方向、SPI、加/解密算法、重放窗口、计数等），顺便看看是否需要调优。接下来就可以进 Day42：站点到站点 IPsec VPN 的多网段/路由联动场景了。









建议的小优化（两端都做）

（不是必须，但更贴近考试/生产最佳实践）

conf t
! 1) 开 PFS（推荐 14 或与你的 IKE 组一致）
crypto map VPN 10 set pfs group14

! 2) 统一 IPsec SA 生命周期（便于可预期轮转；双方一致）
crypto map VPN 10 set security-association lifetime seconds 3600 kilobytes 4608000

! 3) IKE 探活（平台支持就开）
crypto isakmp keepalive 10 periodic

! 4) 避免 TCP 分片（内网口）
interface e0/1
 ip tcp adjust-mss 1360
end
wr mem

继续自检（可选）

show crypto map interface e0/0 → 确认加密图贴在公网口。

show crypto isakmp policy → 两端加密/哈希/DH 是否一致。

show crypto session → 一眼看全栈状态（IKE/IPsec）。

一切就绪的话，你已经顺利完成 Day41：IKEv1 阶段1/2。准备好就可以上 Day42：站点到站点 IPsec（多子网 + 动态路由/GRE/DMVPN） 了，我可以直接给你“可复制”的完整配置模板与验证清单。