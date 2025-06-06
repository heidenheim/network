![](image/091944.png)

在实际环境中，常常有这样的物理意义上的环路，环路会造成：
1. **广播风暴** 在相同链路上重复转发帧，消耗链路的大部分容量。
2. **MAC地址表不稳定** 由于帧循环导致交换机的MAC地址表持续更新错误条目，导致帧被发送到错误的位置。
3. **多点帧拷贝** 帧循环的副作用，其中一个帧的多个副本被传递给预定的主机，使主机感到困惑。

![](image/091944.png)

## STP设置端口状态，形成树状结构防环
Spanning Tree Protocols STP 通过将每个交换机端口置于*转发状态*或*阻塞状态*来防止环路。
处于*阻塞状态*的接口除了STP消息外不会处理任何帧。阻塞的接口*不转发*用户帧，*不学习*接收到的帧的MAC地址，也*不处理*接收到的用户帧。

STP是一种公有协议，除了STP还有其他的协议用于防止环路</br>
 STP   802.1D  low    slow       one 
 PVST+ cisco   high   slow       one of every VLAN
 RSTP  802.1W  medium fast       one
 Rapid PVST+   cisco  very high fast multi instance
</br>

## BPDU </br>

Byte   Field
2      协议 `代表上层协议（BPDU），该值总为0`</br>
1      版本 `802.1D的总为0`
1      TYPE `“配置BPDU”为0，“TCN BPDU”为80`
1      标志 `LSB最低有效位表示TC标志；MSB最高有效位表示TCA标志`
8      根ID `根网桥的ID`
4      路径开销 `到达根桥的STP cost`
8      网桥ID `BPDU发送桥的ID`
2      端口ID `BPDU发送网桥的端口ID (优先级+端口号)` 
2      Meeasge age `从根网桥出发的BPDU之后的秒数,每一跳减去1`
2      Max age `当一段时间未收到BPDU, 生存期到达MAX age时,网桥认为该端口连接的链路发生故障.默认20s`
2      Hello时间 `根网桥连续发送的BPDU之间的时间间隔. 默认2s`
2      转发延迟 `在监听和学习状态所停留的时间间隔. 默认15s`