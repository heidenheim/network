|起源名称|标记|描述|
| :---: |:---|:---:|
|IGP    |i   |如果路由是由始发的BGP路由器使用network或路由汇总命令注入到BGP的,那么该BGP的Origin属性为IGP|
</br> Origin是BGP的***公认***且***必须***属性之一. 如上所表示, 根据路由被引入BGP的方式不同, 存在三种不同类型的Origin.
</br> 当去往同一个目的地存在多条不同的Origin属性的路由时,在其他条件都相同的情况下, BGP将按Origin的顺序优选路由: IGP > EBGP > Incomplete
