# BGP 状态机
|Peer状态名称|用途|
|:---:|:---|
|Idele|开始装备TCP的连接并监视远程对等体,启用BGP时,要准备足够的资源|
|Connect|正在进行TCP连接, 等待完成中, 认证都是在TCP建立期间完成的. 如果TCP丽娜姐建立失败则进去Active状态, 反复尝试连接|
|OpenSent|TCP丽娜姐已经建立成功, 开始发送Open包, Open包携带参数协商对等体的建立|
|OpenCongirm|参数, 能力特性协商成功, 自己发送keepalive包, 等待对方的keepalive包|
|establish|已经收到对方的keepalive包, 双方能力特性经协商发现一致, 开始使用update通告路由信息|

![](image\150603.png)

