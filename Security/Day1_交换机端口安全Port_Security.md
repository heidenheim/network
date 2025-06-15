- 理解什么是 Port Security
- 学会配置 Sticky MAC 地址绑定
- 限制每个端口的允许设备数
- 验证端口被非法设备触发后自动 Shutdown

![](image\110601)

```
SW1(config)#int e0/0
SW1(config-if)#switchport mode access
SW1(config-if)#switchport port-security
//启用端口安全功能
SW1(config-if)#switchport port-security maximum 1
//安全端口最大允许1个MAC地址
SW1(config-if)#switchport port-security mac-address sticky
//自动学习当前连接设备的MAC并绑定
SW1(config-if)#switchport port-security violation ?
  protect   Security violation protect mode
  //丢弃非法流量，不提示, 不记录日志, 不关端口, 安静模式（最温和）
  restrict  Security violation restrict mode
  //丢弃非法流量 + 计数报警, 记录日志, 不关端口,想监控但不断线
  shutdown  Security violation shutdown mode
  //直接关闭端口（err-disabled）, 记录日志, 关闭端口, 高安全场景
SW1(config-if)#switchport port-security violation shutdown
//违规时自动关闭端口

SW1(config)#int e0/2
SW1(config-if)#no switchport
SW1(config-if)#ip address 192.168.1.1 255.255.255.0
SW1(config-if)#no shu
```

设置PC的IP地址
```
PC1> ip 192.168.1.10 24 192.168.1.1
Checking for duplicate address...
PC1 : 192.168.1.10 255.255.255.0 gateway 192.168.1.1

PC2> ip 192.168.1.20 24 192.168.1.1
Checking for duplicate address...
PC2 : 192.168.1.20 255.255.255.0 gateway 192.168.1.1
```