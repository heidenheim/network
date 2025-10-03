BGP 在收到多条到同一前缀的路由时，会按以下顺序进行选路（从上到下，先满足则选定，不继续往下）：
1. Weight（思科私有属性，越大越优先）
2. Local Preference（局部优先级，越大越优先）
3. Originated by router itself（本地生成路由优先）
4. AS-PATH 长度（越短越优先）
5. Origin 类型（IGP > EGP > Incomplete）
6. MED（多出口判定，越小越优先，仅在相同邻居 AS 内比较）
7. eBGP 优于 iBGP
8. IGP cost to NEXT-HOP（到下一跳的 IGP 距离，越小越优先）
9. Router ID（数值小的优先）