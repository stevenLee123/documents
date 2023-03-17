# scala 基础

## 特征
> 面向对象
> 函数式编程
> 静态类型
> 扩展性
> 并发性

语法
*函数和方法*
val 语句可以定义函数，def 语句定义方法。

```scala
  def m(x: Int) = x + 3
  val f = (x: Int) => x + 3
```
*闭包（java中的lambda表达式）*
```scala
var factor = 3  
val multiplier = (i:Int) => i * factor
```

