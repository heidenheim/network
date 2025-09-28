如何管理、删除与排错 ZBF，并且对比传统 ACL/CBAC 与 ZBF 的区别。

# ZBF 的特点

1. 接口必须加入 Zone，否则默认属于 “self zone”。
2. 同一个 Zone 内部通信 不受限制。
3. 不同 Zone 之间通信 必须通过 zone-pair 明确允许。
4. 默认：没有配置 zone-pair → 流量被丢弃（即使是 ICMP）。

## 删除/重置 ZBF

1. 如果你想取消防火墙配置，需要逐步清除：
- 解除接口与 Zone 的绑定：
```
R1(config)#int e0/1
R1(config-if)#no zone-member security INSIDE
```

- 删除 zone-pair：

`R1(config)#no zone-pair security INSIDE-TO-OUTSIDE`

- 删除 policy-map 与 class-map：

```
R1(config)#no policy-map type inspect P1
R1(config)#no class-map type inspect C1
```

- 最后删除 Zone：

```
R1(config)#no zone security INSIDE
R1(config)#no zone security OUTSIDE
R1(config)#no zone security DMZ
```

- 注意：如果你只是删除了 Zone，但接口还在 zone-member 状态，会报错，需要先解除接口与 Zone 的关联。

### 验证命令

1. 查看 zone 配置：

`show zone security`

2. 查看 zone-pair：

`show zone-pair security`

3. 查看策略：

`show policy-map type inspect zone-pair sessions`


#### 确认是否完全删除：执行 show run | include zone，没有输出即表示 ZBF 已被完全清除。