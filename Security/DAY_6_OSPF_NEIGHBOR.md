OSPF 邻居建立的五个阶段
1. Down → 刚启动，没收到邻居 Hello。
2.Init → 收到邻居的 Hello，但还没在对方的 Hello 里看到自己。
3. 2-Way → 双方互相在 Hello 里看到对方，形成双向通信（广播网络里只有 2-Way 的才会进入 DR/BDR 选举）。
4. ExStart → 开始交换 DBD（数据库描述包），选出 master/slave。
5. Full → LSA 数据库同步完成，邻居建立成功。

DR/BDR 选举规则
1. 只在 多访问网络（broadcast / NBMA） 上进行（例如以太网）。
2. 先比较 OSPF 接口优先级（ip ospf priority），高者当选。
3. 优先级相同 → 比较 Router ID，高者当选。
4. 一旦 DR 选出，不会轻易重新选举（除非 down 掉）。