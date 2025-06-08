## BGP发展
1980年代出现EGP，1989年出现BGP-1，现在使用的是BGP4+支持多种地址族（BGP4只能支持IPv4，但是BGP4+可以支持IPv6，还有VPNv4，VPNv6，L2VPN）

BGP是一种实现AS之间的路由可达，并选择最佳路由（优选）的矢量路由协议。

特点:
BGP使用TCP作为其传输层协议（port 179），使用触发式路由更新，而不是周期性路由更新。
路由器之间会话基于TCP连接而建立
BGP能够承载大批量的路由协议信息，能够支撑大规模网络。
BGP提供了丰富的路由策略，能够灵活的进行路由选路，并能直到对等体按策略发布路由。
BGP能够支撑MPLS/VPN的应用，传递客户VPN路由。
BGP提供了路由聚合和路由衰减功能用于防止路由震荡，通过这两项功能有效地提高了网络稳定性。



	

运行BGP的路由器称为BGP speaker，或BGP路由器

两个建立BGP会话对策路由器互为等体（Peer），BGP对等体之间交换BGP路由表。

BGP路由器只发送增量的BGP路由更新，或进行触发式更新（不会周期性更新）。

BGP能够承载大批量的路由前缀，可在大规模网络中应用。



BGP安全性

常见BGP攻击主要有两种

	1. 建立非法BGP邻居关系，通过非法路条目，干扰正常路由表

	2. 发送大量非法BGP报文，路由收到上送CPU，导致CPU利用率升高



BGP认证

BGP使用认证和GTSM（Generalized TTL Security Mechanism 通过TTL 安全保护机制）两个方法保证BGP对等体间的交互安全。



BGP认证分为MD5认证和Keychain认证，对BGP对等体关系进行认证可以预防非法BGP邻居建立。

 R1  ←→  R2

←TCP 三次握手→

←   Open报文  →

← Update报文 →

←Keepalive报文→

←Notification报文→

BGP使用TCP作为传输层协议，为提高BGP的安全性，可以在建立TCP连接时进行MD5认证。BGP的MD5认证只是为了TCP连接设置MD5认证密码，由TCP完成认证。

其中Keychain是华为的私有技术。

BGP认证需要硬重置才可以使密码生效，如果BGP会话先行建立再去设置密码就没有意义了。

