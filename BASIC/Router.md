
# Router>


| 命令                  | 功能说明                 | 示例                                          | EI | Security |
| ------------------- | -------------------- | ------------------------------------------- | ----- | ----------- |
| **access-enable**   | 创建临时 ACL 条目          | `Router> access-enable`                     | ❌     | ✅           |
| **access-profile**  | 应用用户访问配置文件           | `Router> access-profile`                    | ❌     | ✅           |
| **clear**           | 清除缓存/统计              | `Router> clear line 1`                      | ✅     | ✅           |
| **connect**         | 打开一个终端连接             | `Router> connect 192.168.1.2`               | ✅     | ❌           |
| **credential**      | 从文件加载凭证              | `Router> credential import flash:creds.txt` | ❌     | ✅           |
| **crypto**          | 加密命令 (VPN/IPSec)     | `Router> crypto pki certificates`           | ❌     | ✅           |
| **disable**         | 退回用户模式               | `Router# disable`                           | ✅     | ✅           |
| **disconnect**      | 断开连接                 | `Router> disconnect 1`                      | ✅     | ❌           |
| **do-exec**         | 在配置模式下执行 EXEC        | `Router(config)# do show ip int brief`      | ✅     | ✅           |
| **enable**          | 进入特权模式               | `Router> enable`                            | ✅     | ✅           |
| **ethernet**        | 以太网参数                | `Router> ethernet ?`                        | ❌     | ❌           |
| **exit**            | 退出 EXEC 模式           | `Router> exit`                              | ✅     | ✅           |
| **help**            | 查看帮助                 | `Router> help`                              | ✅     | ✅           |
| **ip**              | IP SLA 命令            | `Router> ip sla monitor`                    | ✅     | ✅           |
| **ips**             | 入侵防御系统               | `Router> ips status`                        | ❌     | ✅           |
| **lat**             | 打开 LAT 连接 (老 DEC 协议) | `Router> lat host`                          | ❌     | ❌           |
| **lig**             | LISP Internet Groper | `Router> lig`                               | ❌     | ❌           |
| **lock**            | 锁定终端                 | `Router> lock`                              | ✅     | ✅           |
| **login**           | 登录                   | `Router> login`                             | ✅     | ✅           |
| **logout**          | 退出会话                 | `Router> logout`                            | ✅     | ✅           |
| **modemui**         | 启动 modem UI          | `Router> modemui`                           | ❌     | ❌           |
| **mrinfo**          | 显示多播邻居信息             | `Router> mrinfo 224.0.0.1`                  | ✅     | ❌           |
| **mstat**           | 多播 traceroute 统计     | `Router> mstat`                             | ✅     | ❌           |
| **mtrace**          | 多播路径跟踪               | `Router> mtrace 224.0.0.1`                  | ✅     | ❌           |
| **name-connection** | 给连接命名                | `Router> name-connection 1 TestSession`     | ❌     | ❌           |
| **pad**             | 打开 X.29 PAD 连接       | `Router> pad`                               | ❌     | ❌           |
| **ping**            | 连通性测试                | `Router> ping 8.8.8.8`                      | ✅     | ✅           |
| **ppp**             | 启动 PPP 协议            | `Router> ppp`                               | ✅     | ❌           |
| **release**         | DHCP 释放              | `Router> release dhcp`                      | ✅     | ❌           |
| **renew**           | DHCP 续租              | `Router> renew dhcp`                        | ✅     | ❌           |
| **resume**          | 恢复连接                 | `Router> resume 1`                          | ✅     | ❌           |
| **rlogin**          | rlogin 远程登录          | `Router> rlogin 10.1.1.1`                   | ❌     | ❌           |
| **routing-context** | 路由上下文                | `Router> routing-context`                   | ✅     | ❌           |
| **set**             | 设置系统参数               | `Router> set privilege level 15`            | ✅     | ✅           |
| **show**            | 显示运行信息               | `Router> show ip int brief`                 | ✅     | ✅           |
| **slip**            | 串口 SLIP 协议           | `Router> slip`                              | ❌     | ❌           |
| **ssh**             | SSH 连接               | `Router> ssh -l admin 192.168.1.1`          | ✅     | ✅           |
| **systat**          | 显示终端行信息              | `Router> systat`                            | ❌     | ❌           |
| **tclquit**         | 退出 Tcl shell         | `Router> tclquit`                           | ❌     | ❌           |
| **telnet**          | Telnet 连接            | `Router> telnet 192.168.1.1`                | ✅     | ✅           |
| **terminal**        | 设置终端参数               | `Router> terminal length 0`                 | ✅     | ✅           |
| **tn3270**          | IBM 3270 终端连接        | `Router> tn3270`                            | ❌     | ❌           |
| **traceroute**      | 路由跟踪                 | `Router> traceroute 8.8.8.8`                | ✅     | ✅           |
| **tunnel**          | 打开隧道连接               | `Router> tunnel`                            | ✅     | ❌           |
| **udptn**           | UDP 隧道               | `Router> udptn`                             | ❌     | ❌           |
| **waas**            | WAAS 命令 (加速服务)       | `Router> waas`                              | ❌     | ❌           |
| **where**           | 显示当前会话               | `Router> where`                             | ✅     | ✅           |
| **x28**             | X.28 PAD             | `Router> x28`                               | ❌     | ❌           |
| **x3**              | X.3 PAD 参数           | `Router> x3`                                | ❌     | ❌           |


**总结**

EI 必备命令
ping、traceroute、show、clear、enable、telnet、ssh、terminal length 0

Security 必备命令
crypto、ips、access-enable、access-profile、set、ssh、logging (在特权模式下用)


## Router#?


| 命令                         | 功能说明                              | 示例                                           | EI| Security|
| -------------------------- | --------------------------------- | -------------------------------------------- | ----- | ----------- |
| **configure**              | 进入全局配置模式                          | `Router# configure terminal`                 | ✅     | ✅           |
| **copy**                   | 复制文件（保存配置）                        | `Router# copy running-config startup-config` | ✅     | ✅           |
| **write**                  | 保存配置                              | `Router# write memory`                       | ✅     | ✅           |
| **reload**                 | 重启设备                              | `Router# reload`                             | ✅     | ✅           |
| **show**                   | 显示运行信息                            | `Router# show ip interface brief`            | ✅     | ✅           |
| **ping**                   | 连通性测试                             | `Router# ping 8.8.8.8`                       | ✅     | ✅           |
| **traceroute**             | 路径跟踪                              | `Router# traceroute 8.8.8.8`                 | ✅     | ✅           |
| **ssh**                    | SSH 登录                            | `Router# ssh -l admin 192.168.1.1`           | ✅     | ✅           |
| **telnet**                 | Telnet 登录                         | `Router# telnet 192.168.1.1`                 | ✅     | ✅           |
| **terminal**               | 设置终端参数                            | `Router# terminal length 0`                  | ✅     | ✅           |
| **clear**                  | 清除缓存/统计                           | `Router# clear arp`                          | ✅     | ✅           |
| **debug**                  | 开启调试                              | `Router# debug ip packet`                    | ✅     | ✅           |
| **undebug** / **no debug** | 关闭调试                              | `Router# undebug all`                        | ✅     | ✅           |
| **crypto**                 | 加密相关 (IPSec/VPN)                  | `Router# crypto isakmp sa`                   | ❌     | ✅           |
| **cts**                    | Cisco Trusted Security (TrustSec) | `Router# cts refresh-policy`                 | ❌     | ✅           |
| **ips**                    | 入侵防御系统                            | `Router# show ips sessions`                  | ❌     | ✅           |
| **logging**                | 日志控制                              | `Router# logging buffered 10000`             | ✅     | ✅           |
| **verify**                 | 验证文件完整性                           | `Router# verify flash:ios.bin`               | ✅     | ✅           |
| **dir**                    | 查看文件系统                            | `Router# dir flash:`                         | ✅     | ✅           |
| **delete**                 | 删除文件                              | `Router# delete flash:old_config`            | ✅     | ✅           |
| **rename**                 | 重命名文件                             | `Router# rename flash:old new`               | ✅     | ✅           |
| **mkdir / rmdir**          | 创建/删除目录                           | `Router# mkdir flash:configs`                | ✅     | ✅           |
| **more**                   | 显示文件内容                            | `Router# more flash:config.text`             | ✅     | ✅           |
| **archive**                | 管理配置归档                            | `Router# archive config`                     | ✅     | ✅           |
| **ip**                     | IP SLA/特性命令                       | `Router# ip sla monitor`                     | ✅     | ✅           |
| **mpls**                   | MPLS 相关命令                         | `Router# mpls ldp neighbor`                  | ✅     | ❌           |
| **xconnect**               | L2 VPN 配置检查                       | `Router# show xconnect all`                  | ✅     | ✅ (VPN)     |
| **webvpn**                 | WebVPN 相关                         | `Router# webvpn gateway`                     | ❌     | ✅           |
| **waas**                   | WAAS 加速服务                         | `Router# waas status`                        | ❌     | ✅           |


**总结**

EI (Enterprise Infrastructure) ：
configure terminal、copy run start、write memory、reload、show ip interface brief、ping、traceroute、terminal length 0、clear arp、debug/undebug、mpls、xconnect

Security ：
crypto (VPN) 、cts (TrustSec)、ips (入侵防御)、logging、archive、webvpn、verify (文件完整性)


### Router(config)#


| 命令                             | 功能说明                    | 示例                                                                            | EI | Security  |
| ------------------------------ | ----------------------- | ----------------------------------------------------------------------------- | ----- | ----------- |
| **hostname**                   | 设置设备名称                  | `Router(config)# hostname R1`                                                 | ✅     | ✅           |
| **enable secret**              | 设置特权加密密码                | `Router(config)# enable secret cisco123`                                      | ✅     | ✅           |
| **banner motd**                | 设置登录横幅                  | `Router(config)# banner motd #Unauthorized Access Prohibited#`                | ✅     | ✅           |
| **username**                   | 本地用户认证                  | `Router(config)# username admin privilege 15 secret 12345`                    | ✅     | ✅           |
| **line**                       | 进入线路配置 (console/vty)    | `Router(config)# line vty 0 4`                                                | ✅     | ✅           |
| **password**                   | 设置线路密码                  | `Router(config-line)# password cisco`                                         | ✅     | ✅           |
| **login**                      | 启用密码登录                  | `Router(config-line)# login`                                                  | ✅     | ✅           |
| **interface**                  | 进入接口配置                  | `Router(config)# interface g0/0`                                              | ✅     | ✅           |
| **ip address**                 | 配置接口 IP                 | `Router(config-if)# ip address 192.168.1.1 255.255.255.0`                     | ✅     | ✅           |
| **no shutdown**                | 启用接口                    | `Router(config-if)# no shutdown`                                              | ✅     | ✅           |
| **description**                | 接口描述                    | `Router(config-if)# description Uplink-to-SW1`                                | ✅     | ✅           |
| **ip route**                   | 配置静态路由                  | `Router(config)# ip route 0.0.0.0 0.0.0.0 192.168.1.254`                      | ✅     | ✅           |
| **router ospf**                | 启用 OSPF                 | `Router(config)# router ospf 1`                                               | ✅     | ❌           |
| **router eigrp**               | 启用 EIGRP                | `Router(config)# router eigrp 100`                                            | ✅     | ❌           |
| **router bgp**                 | 启用 BGP                  | `Router(config)# router bgp 65001`                                            | ✅     | ❌           |
| **mpls**                       | 配置 MPLS                 | `Router(config)# mpls ip`                                                     | ✅     | ❌           |
| **vrf**                        | VRF 配置 (多 VPN 支持)       | `Router(config)# ip vrf CUST1`                                                | ✅     | ✅           |
| **spanning-tree**              | 全局生成树配置                 | `Router(config)# spanning-tree mode rapid-pvst`                               | ✅     | ❌           |
| **standby**                    | HSRP 配置                 | `Router(config-if)# standby 1 ip 192.168.1.254`                               | ✅     | ❌           |
| **fhrp**                       | First Hop Redundancy 配置 | `Router(config)# fhrp version vrrp`                                           | ✅     | ❌           |
| **aaa new-model**              | 启用 AAA 框架               | `Router(config)# aaa new-model`                                               | ❌     | ✅           |
| **aaa authentication login**   | 定义认证方法                  | `Router(config)# aaa authentication login default local`                      | ❌     | ✅           |
| **tacacs-server**              | TACACS+ 配置              | `Router(config)# tacacs-server host 192.168.1.100`                            | ❌     | ✅           |
| **snmp-server**                | SNMP 配置                 | `Router(config)# snmp-server community public RO`                             | ✅     | ✅           |
| **logging**                    | 日志服务器                   | `Router(config)# logging 192.168.1.200`                                       | ✅     | ✅           |
| **ntp server**                 | NTP 时间同步                | `Router(config)# ntp server 192.168.1.10`                                     | ✅     | ✅           |
| **crypto isakmp policy**       | 配置 VPN ISAKMP           | `Router(config)# crypto isakmp policy 10`                                     | ❌     | ✅           |
| **crypto ipsec transform-set** | 配置 IPSec                | `Router(config)# crypto ipsec transform-set VPN-SET esp-aes esp-sha-hmac`     | ❌     | ✅           |
| **zone security**              | Zone-Based Firewall 配置  | `Router(config)# zone security INSIDE`                                        | ❌     | ✅           |
| **zone-pair security**         | Zone-Pair 配置            | `Router(config)# zone-pair security IN-OUT source INSIDE destination OUTSIDE` | ❌     | ✅           |
| **policy-map**                 | 定义 QoS/防火墙策略            | `Router(config)# policy-map QOS1`                                             | ✅     | ✅           |
| **class-map**                  | 匹配流量                    | `Router(config)# class-map match-any VOICE`                                   | ✅     | ✅           |
| **object-group**               | ACL 对象组                 | `Router(config)# object-group network SERVERS`                                | ❌     | ✅           |
| **access-list**                | 标准/扩展 ACL               | `Router(config)# access-list 100 permit tcp any any eq 80`                    | ✅     | ✅           |
| **ip access-group**            | 应用 ACL                  | `Router(config-if)# ip access-group 100 in`                                   | ✅     | ✅           |
| **appfw**                      | 应用防火墙策略                 | `Router(config)# appfw policy POLICY1`                                        | ❌     | ✅           |
| **webvpn**                     | SSL VPN 配置              | `Router(config)# webvpn gateway SSLVPN1`                                      | ❌     | ✅           |
| **utd**                        | Unified Threat Defense  | `Router(config)# utd enable`                                                  | ❌     | ✅           |


**总结**

1. EI 
    - 基础：hostname、interface、ip address、ip route、router ospf/eigrp/bgp
    - 高级：mpls、vrf、fhrp、spanning-tree、standby (HSRP)

2. Security 
    - 认证：aaa new-model、username、tacacs-server
    - 控制：access-list、object-group、policy-map、class-map
    - VPN：crypto isakmp、crypto ipsec、webvpn、utd
    - 防火墙：zone security、zone-pair security