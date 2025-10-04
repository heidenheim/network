
# Switch>


| 命令                          | 功能说明                 | 示例                                | EI 重点 | Security 重点 |
| --------------------------- | -------------------- | --------------------------------- | ----- | ----------- |
| **enable**                  | 进入特权 EXEC 模式         | `Switch> enable`                  | ✅     | ✅           |
| **disable**                 | 退出特权模式               | `Switch# disable`                 | ✅     | ✅           |
| **exit / logout**           | 退出会话                 | `Switch> exit`                    | ✅     | ✅           |
| **ping**                    | ICMP 测试              | `Switch> ping 192.168.1.1`        | ✅     | ✅           |
| **traceroute**              | 路径跟踪                 | `Switch> traceroute 8.8.8.8`      | ✅     | ✅           |
| **ssh**                     | SSH 登录远程设备           | `Switch> ssh -l admin 10.1.1.1`   | ✅     | ✅           |
| **telnet**                  | Telnet 登录远程设备        | `Switch> telnet 10.1.1.1`         | ✅     | ✅           |
| **show**                    | 查看信息 (受限)            | `Switch> show sessions`           | ✅     | ✅           |
| **lock**                    | 锁定终端                 | `Switch> lock`                    | ❌     | ✅           |
| **login**                   | 用户登录                 | `Switch> login`                   | ✅     | ✅           |
| **where**                   | 查看当前会话               | `Switch> where`                   | ✅     | ✅           |
| **clear**                   | 重置功能 (如清缓存)          | `Switch> clear line 1`            | ✅     | ✅           |
| **set**                     | 设置临时参数               | `Switch> set privilege level 1`   | ❌     | ✅           |
| **resume**                  | 恢复挂起的会话              | `Switch> resume 1`                | ✅     | ❌           |
| **disconnect**              | 断开会话                 | `Switch> disconnect 1`            | ✅     | ❌           |
| **connect**                 | 打开会话                 | `Switch> connect 192.168.1.2`     | ✅     | ❌           |
| **mrinfo / mstat / mtrace** | 多播相关测试               | `Switch> mrinfo 224.0.0.1`        | ✅     | ❌           |
| **crypto**                  | 加密相关命令 (受限)          | `Switch> crypto pki certificates` | ❌     | ✅           |
| **ppp**                     | 启动 PPP               | `Switch> ppp`                     | ❌     | ❌           |
| **slip**                    | 启动 SLIP (老串口协议)      | `Switch> slip`                    | ❌     | ❌           |
| **pad / x28 / x3**          | X.25 协议相关            | `Switch> pad`                     | ❌     | ❌           |
| **tunnel / udptn**          | 隧道/UDP 隧道            | `Switch> tunnel`                  | ❌     | ❌           |
| **lig**                     | LISP Internet Groper | `Switch> lig`                     | ❌     | ❌           |
| **systat**                  | 显示终端行信息              | `Switch> systat`                  | ❌     | ❌           |
| **tclquit**                 | 退出 Tcl shell         | `Switch> tclquit`                 | ❌     | ❌           |
| **name-connection**         | 给连接命名                | `Switch> name-connection 1 TEST`  | ❌     | ❌           |
| **routing-context**         | 路由上下文 (受限)           | `Switch> routing-context`         | ❌     | ❌           |


**总结**

EI 必备
enable、disable、ping、traceroute、ssh、telnet、show

Security 必备
enable、ssh、crypto、lock、login

其余命令（PPP、SLIP、X.25、LISP 等）属于过时协议或特殊环境，实际 CCNA/CCNP/CCIE 学习和工作中很少会用。


## Switch#


| 命令                  | 功能说明                     | 示例                                                                                                 | EI 重点 | Security 重点 |
| ------------------- | ------------------------ | -------------------------------------------------------------------------------------------------- | ----- | ----------- |
| **hostname**        | 设置设备名称                   | `Switch(config)# hostname SW1`                                                                     | ✅     | ✅           |
| **enable secret**   | 设置加密的特权密码                | `Switch(config)# enable secret MyPass`                                                             | ✅     | ✅           |
| **banner motd**     | 设置登录横幅                   | `Switch(config)# banner motd #No Unauthorized Access!#`                                            | ✅     | ✅           |
| **username**        | 配置本地用户                   | `Switch(config)# username admin privilege 15 secret cisco123`                                      | ✅     | ✅           |
| **line**            | 配置控制台/VTY线路              | `Switch(config)# line vty 0 4`                                                                     | ✅     | ✅           |
| **password**        | 配置线路密码                   | `Switch(config-line)# password cisco`                                                              | ✅     | ✅           |
| **login**           | 启用登录验证                   | `Switch(config-line)# login`                                                                       | ✅     | ✅           |
| **interface**       | 进入接口配置                   | `Switch(config)# interface g0/1`                                                                   | ✅     | ✅           |
| **ip address**      | 设置三层接口 IP                | `Switch(config-if)# ip address 192.168.1.1 255.255.255.0`                                          | ✅     | ✅           |
| **no shutdown**     | 启用接口                     | `Switch(config-if)# no shutdown`                                                                   | ✅     | ✅           |
| **description**     | 接口描述                     | `Switch(config-if)# description Uplink-to-Core`                                                    | ✅     | ✅           |
| **vlan**            | 创建 VLAN                  | `Switch(config)# vlan 10`                                                                          | ✅     | ❌           |
| **name**            | VLAN 命名                  | `Switch(config-vlan)# name HR`                                                                     | ✅     | ❌           |
| **vtp mode**        | 设置 VTP 模式                | `Switch(config)# vtp mode transparent`                                                             | ✅     | ❌           |
| **vtp domain**      | 设置 VTP 域                 | `Switch(config)# vtp domain LAB`                                                                   | ✅     | ❌           |
| **vtp password**    | 设置 VTP 密码                | `Switch(config)# vtp password lab123`                                                              | ✅     | ❌           |
| **spanning-tree**   | 配置生成树协议                  | `Switch(config)# spanning-tree mode rapid-pvst`                                                    | ✅     | ❌           |
| **udld**            | 配置 UDLD (单向链路检测)         | `Switch(config)# udld enable`                                                                      | ✅     | ❌           |
| **lacp**            | 配置链路聚合                   | `Switch(config)# interface range g0/1-2`<br>`Switch(config-if-range)# channel-group 1 mode active` | ✅     | ❌           |
| **port-channel**    | EtherChannel 配置          | `Switch(config)# interface port-channel 1`                                                         | ✅     | ❌           |
| **port-security**   | 接口端口安全                   | `Switch(config-if)# switchport port-security`                                                      | ❌     | ✅           |
| **access-list**     | 定义 ACL                   | `Switch(config)# access-list 10 permit 192.168.1.0 0.0.0.255`                                      | ✅     | ✅           |
| **ip access-group** | 应用 ACL                   | `Switch(config-if)# ip access-group 10 in`                                                         | ✅     | ✅           |
| **object-group**    | 对象组 (ACL 强化)             | `Switch(config)# object-group network SERVERS`                                                     | ❌     | ✅           |
| **aaa new-model**   | 启用 AAA 框架                | `Switch(config)# aaa new-model`                                                                    | ❌     | ✅           |
| **tacacs-server**   | TACACS+ 配置               | `Switch(config)# tacacs-server host 10.1.1.10`                                                     | ❌     | ✅           |
| **snmp-server**     | SNMP 配置                  | `Switch(config)# snmp-server community public RO`                                                  | ✅     | ✅           |
| **ntp server**      | NTP 时间同步                 | `Switch(config)# ntp server 192.168.1.100`                                                         | ✅     | ✅           |
| **logging**         | 配置日志服务器                  | `Switch(config)# logging 192.168.1.200`                                                            | ✅     | ✅           |
| **crypto**          | 加密/安全 (如 MACsec)         | `Switch(config)# crypto key generate rsa`                                                          | ❌     | ✅           |
| **cts**             | Cisco TrustSec           | `Switch(config)# cts role-based enforcement`                                                       | ❌     | ✅           |
| **dot1x**           | 802.1X 配置                | `Switch(config)# dot1x system-auth-control`                                                        | ❌     | ✅           |
| **mka**             | MACsec Key Agreement     | `Switch(config)# mka policy default`                                                               | ❌     | ✅           |
| **policy-map**      | QoS/策略配置                 | `Switch(config)# policy-map QOS1`                                                                  | ✅     | ✅           |
| **class-map**       | 匹配流量                     | `Switch(config)# class-map match-any VOICE`                                                        | ✅     | ✅           |
| **service-policy**  | 应用策略                     | `Switch(config-if)# service-policy input QOS1`                                                     | ✅     | ✅           |
| **track**           | 对象追踪 (常配合 HSRP/VRRP)     | `Switch(config)# track 1 interface g0/1 line-protocol`                                             | ✅     | ❌           |
| **fhrp**            | 配置 FHRP (HSRP/VRRP/GLBP) | `Switch(config)# fhrp version vrrp`                                                                | ✅     | ❌           |


**总结**

EI 必备 (交换/三层)

    hostname、interface、vlan、vtp、spanning-tree、lacp、port-channel、udld、fhrp

Security 必备 (安全/管理)

    aaa new-model、username、tacacs-server、dot1x、mka、port-security、crypto、cts

    ACL/对象组：access-list、ip access-group、object-group


### Switch(config)#


| 命令                  | 功能说明                     | 示例                                                                                                 | EI | Security |
| ------------------- | ------------------------ | -------------------------------------------------------------------------------------------------- | ----- | ----------- |
| **hostname**        | 设置设备名称                   | `Switch(config)# hostname SW1`                                                                     | ✅     | ✅           |
| **enable secret**   | 设置加密的特权密码                | `Switch(config)# enable secret MyPass`                                                             | ✅     | ✅           |
| **banner motd**     | 设置登录横幅                   | `Switch(config)# banner motd #No Unauthorized Access!#`                                            | ✅     | ✅           |
| **username**        | 配置本地用户                   | `Switch(config)# username admin privilege 15 secret cisco123`                                      | ✅     | ✅           |
| **line**            | 配置控制台/VTY线路              | `Switch(config)# line vty 0 4`                                                                     | ✅     | ✅           |
| **password**        | 配置线路密码                   | `Switch(config-line)# password cisco`                                                              | ✅     | ✅           |
| **login**           | 启用登录验证                   | `Switch(config-line)# login`                                                                       | ✅     | ✅           |
| **interface**       | 进入接口配置                   | `Switch(config)# interface g0/1`                                                                   | ✅     | ✅           |
| **ip address**      | 设置三层接口 IP                | `Switch(config-if)# ip address 192.168.1.1 255.255.255.0`                                          | ✅     | ✅           |
| **no shutdown**     | 启用接口                     | `Switch(config-if)# no shutdown`                                                                   | ✅     | ✅           |
| **description**     | 接口描述                     | `Switch(config-if)# description Uplink-to-Core`                                                    | ✅     | ✅           |
| **vlan**            | 创建 VLAN                  | `Switch(config)# vlan 10`                                                                          | ✅     | ❌           |
| **name**            | VLAN 命名                  | `Switch(config-vlan)# name HR`                                                                     | ✅     | ❌           |
| **vtp mode**        | 设置 VTP 模式                | `Switch(config)# vtp mode transparent`                                                             | ✅     | ❌           |
| **vtp domain**      | 设置 VTP 域                 | `Switch(config)# vtp domain LAB`                                                                   | ✅     | ❌           |
| **vtp password**    | 设置 VTP 密码                | `Switch(config)# vtp password lab123`                                                              | ✅     | ❌           |
| **spanning-tree**   | 配置生成树协议                  | `Switch(config)# spanning-tree mode rapid-pvst`                                                    | ✅     | ❌           |
| **udld**            | 配置 UDLD (单向链路检测)         | `Switch(config)# udld enable`                                                                      | ✅     | ❌           |
| **lacp**            | 配置链路聚合                   | `Switch(config)# interface range g0/1-2`<br>`Switch(config-if-range)# channel-group 1 mode active` | ✅     | ❌           |
| **port-channel**    | EtherChannel 配置          | `Switch(config)# interface port-channel 1`                                                         | ✅     | ❌           |
| **port-security**   | 接口端口安全                   | `Switch(config-if)# switchport port-security`                                                      | ❌     | ✅           |
| **access-list**     | 定义 ACL                   | `Switch(config)# access-list 10 permit 192.168.1.0 0.0.0.255`                                      | ✅     | ✅           |
| **ip access-group** | 应用 ACL                   | `Switch(config-if)# ip access-group 10 in`                                                         | ✅     | ✅           |
| **object-group**    | 对象组 (ACL 强化)             | `Switch(config)# object-group network SERVERS`                                                     | ❌     | ✅           |
| **aaa new-model**   | 启用 AAA 框架                | `Switch(config)# aaa new-model`                                                                    | ❌     | ✅           |
| **tacacs-server**   | TACACS+ 配置               | `Switch(config)# tacacs-server host 10.1.1.10`                                                     | ❌     | ✅           |
| **snmp-server**     | SNMP 配置                  | `Switch(config)# snmp-server community public RO`                                                  | ✅     | ✅           |
| **ntp server**      | NTP 时间同步                 | `Switch(config)# ntp server 192.168.1.100`                                                         | ✅     | ✅           |
| **logging**         | 配置日志服务器                  | `Switch(config)# logging 192.168.1.200`                                                            | ✅     | ✅           |
| **crypto**          | 加密/安全 (如 MACsec)         | `Switch(config)# crypto key generate rsa`                                                          | ❌     | ✅           |
| **cts**             | Cisco TrustSec           | `Switch(config)# cts role-based enforcement`                                                       | ❌     | ✅           |
| **dot1x**           | 802.1X 配置                | `Switch(config)# dot1x system-auth-control`                                                        | ❌     | ✅           |
| **mka**             | MACsec Key Agreement     | `Switch(config)# mka policy default`                                                               | ❌     | ✅           |
| **policy-map**      | QoS/策略配置                 | `Switch(config)# policy-map QOS1`                                                                  | ✅     | ✅           |
| **class-map**       | 匹配流量                     | `Switch(config)# class-map match-any VOICE`                                                        | ✅     | ✅           |
| **service-policy**  | 应用策略                     | `Switch(config-if)# service-policy input QOS1`                                                     | ✅     | ✅           |
| **track**           | 对象追踪 (常配合 HSRP/VRRP)     | `Switch(config)# track 1 interface g0/1 line-protocol`                                             | ✅     | ❌           |
| **fhrp**            | 配置 FHRP (HSRP/VRRP/GLBP) | `Switch(config)# fhrp version vrrp`                                                                | ✅     | ❌           |


**总结**

EI 方向重点

    基础：hostname、interface、ip address、ip route、router ospf/eigrp/bgp

    高级：mpls、vrf、fhrp、spanning-tree、standby (HSRP)

Security 方向重点

    认证：aaa new-model、username、tacacs-server

    控制：access-list、object-group、policy-map、class-map

    VPN：crypto isakmp、crypto ipsec、webvpn、utd

    防火墙：zone security、zone-pair security