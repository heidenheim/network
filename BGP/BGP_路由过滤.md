# BGP的路由过滤

- 重分布OSPF继承EBGP的AS_Path作为Tag, 用于还原BGP的AS_Path属性

- 路由过滤

```
1. 前缀列表--out/in

ip prefix-list CON permit 2.2.2.2/32
router bgp 200
neighbor 12.1.1.1 prefix-list CON out
允许前缀列表CON中匹配的ip往外传输

ip prefix-list CON deny 2.2.2.2/32
router bgp 100
neighbopr 12.1.1.2 prefix-list CON in
匹配前缀列表CON, 不接收该列表中的IP

2. 分发列表--out/in

ip prefix-list CON permit 2.2.2.2/32
router bgp 200
distreibute-list prefix CON out
重分布前缀列表CON

3. 路由策略 route-map--in/out

ip access-list standard CON
permit 2.2.2.2
*//使用ACL匹配路由*

ip prefix-list CON permit 22.22.22.22/32
*//使用前缀列表匹配路由*

route-map FILTER deny 10
match ip address CON
route-map FILTER permit 20
match ip address prefix-list CON

router bgp 200
neighbor 12.1.1.1 route-map FILTER out

4. 正则表达式(主要针对AS_Path的路由)

ip as-path access-list 5 deny _200$
ip as-path access-list 5 permit .*
*//拒绝AS200始发路由, 接收其AS的他路由*
router bgp 200
neighbor 12.1.1.2 filter-list 5 in

ip as-path access-list 4 permit _300$
//使用as-path访问控制列表匹配某个AS结尾的路由
route-map FILTER deny 10
match as-path 4
route-map FILTER permit 20
//在路由策略上匹配as-path access list 4 拒绝掉其中的路由
router bgp 100
neighbor 12.1.1.1 route-map FILTER in
```
正则表达式（Regular Expression，简称 regex 或 regexp）是一种用于描述、匹配和操作字符串模式的工具，常用于查找、替换、验证文本内容。

| 符号    | 含义说明                           | 示例                           |       |                    |
| ----- | ------------------------------ | ---------------------------- | ----- | ------------------ |
| `.`   | 匹配任意一个字符（除换行符）                 | `a.c` 可匹配 `abc`、`a1c`        |       |                    |
| `*`   | 匹配前一个字符 0 次或多次                 | `a*` 可匹配 `""`, `a`, `aa`     |       |                    |
| `+`   | 匹配前一个字符 1 次或多次                 | `a+` 可匹配 `a`, `aa`，不能匹配 `""` |       |                    |
| `?`   | 匹配前一个字符 0 次或 1 次               | `a?` 可匹配 `""`, `a`           |       |                    |
| `[]`  | 匹配括号内任意一个字符                    | `[abc]` 可匹配 `a`、`b`、`c`      |       |                    |
| `[^]` | 匹配**不在**括号内的任意字符               | `[^0-9]` 匹配非数字               |       |                    |
| \`    | \`                             | 或运算，匹配左边或右边的模式               | \`cat | dog`匹配`cat`或`dog\` |
| `()`  | 分组或提取子匹配                       | `(ab)+` 匹配 `ab`, `abab`, 等   |       |                    |
| `\d`  | 匹配数字，等价于 `[0-9]`               | `\d{3}` 匹配三位数字               |       |                    |
| `\w`  | 匹配字母、数字、下划线，等价于 `[a-zA-Z0-9_]` | `\w+` 匹配单词                   |       |                    |
| `\s`  | 匹配空白符（空格、换行、制表符）               | `\s+` 匹配多个空格                 |       |                    |

## ORF邻居按需发布路由

- **如果设备希望值接收自己需要的路由**, 但对端设备又无法针对每个与它连接的设备维护不同的出口策略. 此时, 可以通过配置BGP基于前缀ORF (**Outbound Route Filter, 出口路由过滤器**) 来满足两端设备的需求

![](image\BGP\280701.png)

假设AS100更新有10000条路由前缀, AS200只需要其中100条, 多余的9900条就浪费了CPU, 带宽以及其他配置.

- **BGP基于前缀的ORF能力**, 能将本端设备配置的基于前缀的入口策略通过路由刷新报文发送给BGP邻居. BGP邻居根据这些策略构造出口策略, 在路由发送时对路由进行过滤.
- 这样不仅避免了本端设备接收大量无用路由, 降低了本端设备的CPU使用率, 还有效减少了BGP邻居的配置工作, 降低了链路带宽的占用率.

如果要实现ORF, 要求两端都具有ORF能力, R2只希望R1通过1.1.1.1/32, R2向R1推送**ORF报文(路由刷新)**来实现这个目的

```
R2(config)#ip prefix-list ORF permit 2.2.2.2/32
R2(config)#ip prefix-list ORF deny 0.0.0.0/0 le 32
R2(config)#router bgp 200
R2(config-router)#neighbor 12.1.1.1 prefix-list ORF in
//使用前缀列表的BGP路由过滤, 不能忘记这条命令
R2(config-router)#neighbor 12.1.1.1 capability orf prefix-list send
```

```
R1(config)#router bgp 100
R1(config-router)#neighbor 12.1.1.2 capability orf prefix-list receive
```

在R2上开启发送, 同样也要在R1上开启接收, 之后就会过滤掉其他不需要的路由了.

```
R2#show ip bgp
BGP table version is 8, local router ID is 2.2.2.2
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   2.2.2.2/32       12.1.1.1                 0             0 100 i
 ```

通过抓包可以看到在路由刷新报文中会有一个 ORF information的内容

### 验证ORF配置
```
R1#show ip bgp neighbors 12.1.1.2 policy
 Neighbor: 12.1.1.2, Address-Family: IPv4 Unicast
 Locally configured policies:
  capability orf prefix-list receive
```

**capability orf prefix-list receive** 配置了ORF监狱前缀能力接收

```
R1#show ip bgp neighbors 12.1.1.2 advertised-routes
BGP table version is 3, local router ID is 1.1.1.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   1.1.1.1/32       0.0.0.0                  0         32768 i
 *>   2.2.2.2/32       0.0.0.0                  0         32768 i

Total number of prefixes 2
```

