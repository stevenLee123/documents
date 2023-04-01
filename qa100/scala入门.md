# scala 基础

## 特征
> 面向对象
> 精简、高级
> 函数式编程
> 静态类型
> 扩展性
> 并发性
## 语法
### **scala程序与java程序对比**
java： java源代码 =》javac编译器=》 java字节码.class文件=》类加载器=》jvm =》解释执行
scala： scala源码=》 scalac编译器=》java字节码.class文件=》类加载器=》jvm=》解释执行
scala和java可以无缝互调

### **数据类型（强类型语言）**
Byte 8位带符号类型
Short 16位带符号整数
Int 32wei带符号整数
Long 64位带符号整数
char 16位无符号unicode字符
String char类型的序列
Float 32位单精度浮点数
Double 64位双精度浮点数
Boolean true或这false

Any 所有类型的父类相当于Object
AnyVal 所有数值类型的父类，Double，Boolean，Int，Unit（类似于java中的void），Short，Long的父类
AnyRef 所有引用类型的父类，String，数组，对象类型的父类
Null 所有引用类型的子类，他的值只有一个null
Nothing 所有类型的子类，一般结合异常使用

### **类型转换**
值类型的类型转化
自动类型转换：将范围小的类型转化为范围大的类型
强制类型转换
可能造成精度缺失
```scala
var a =  3
var b = a + 2.1
//强制类型转换
//转double
val c:Int = (a+ 2.1).toInt
//转string。2.13版本之后不推荐
var b = a + ""
//使用插值表达式
var b = s"${a}"
var c = a.toString()
//字符串转数字
var m = b.toInt
```



语法
### **输出语句**
```scala
println("world")
print("hello")
print(1,2,3,"hello")
```

### **数据录入**

```scala
val str = StdIn.readLine()
//接受数值输入，如果不是整型会报错
val i= StdIn.readInt()
//必须是整型
val l = StdIn.readLong()
//浮点
val f = StdIn.readFloat()
```

### **常量**
整型常量   10
浮点型常量  10.1
字符串常量  “hello”
字符常量    'a'
空常量      null
### **变量**
```scala
//val修饰的变量不可重新赋值
val name:String = "tom"
//var 修饰的变量可重新赋值
var name:String = "tom"
name = "jim"
//使用自动类型推断,scala会自动推断变量类型
var age = 30
```
### **字符串**
```scala
//双引号定义
var name = "hadoop"
//使用插值表达式
var name  = "zhangsan"
var age = 23
var sex = "male"
//插值表达式，name=zhangsan,age=23,sex=male
var result = s"name=${name},age=${age},sex=${sex}
//三引号实现
val sql = """
      select 
       * 
      from
       user where username = "zhangsan"
      """
```
### **惰性赋值**
```scala
lazy val sql2 = """
      select
           *
      from
           user where username = "zhangsan"
      """
    //lazy val sql2: String // unevaluated 
```

关于分号：scala中单行语句最后的分号可以不写，如果是多行代码写在一起，中间的分毫不能省略，最后一条语句可以不写

### **运算符**
scala中没有三目运算符，使用if 替换 其他与java一致
```scala
//打印三次hello
print("hello "*3)
//取余算法
a % b = a -a/b*b = -10-(-10/3)*3 = -1 
print(-10%3)
a % b = a -a/b*b = 10-(10/-3)*3 = 1 
print(10% -3)
```
比较运算符
scala中比较值是否相等时使用`==`或`!=`来进行比较，比较引用地址是否相等是使用eq（方法来进行比较）
```scala
var s1 = "abc"
var s2 = "abc"
print(s1==s2)  //true
print(s1.eq(s2)) //true
var s2 = s1 +""
print(s1.eq(s2))//false
```
二进制 0b开头
十进制 自然数值
八进制 0开头
十六进制 0x开头
原码，反码，补码
正数原反补都一样,计算机存储的是补码形式
负数
原码：十进制转二进制（最高位伪1）
反吗：符号位不变，其他位按位取反
补码：在反码基础上加1即可
位运算：
`&` 按位与
`｜` 按位或
`^` 按位异或
`~` 按位取反
`<<` 按位左移动 
`>>` 按位右移动
~3 0000 0011 =》   1111 1100 （按位取反的值，补码）  =》 1111 1011（补码减去1求反码） ==》 1000 0100 （反码取反求原码） =4
使用异或交换两个数,一个数同时异或两次相同的另一个数的到这个数本身
```scala
var a= 20
var b = 30
var tmp = a^b
a = a ^ tmp
b = b ^ tmp
```
### 流程控制
if-else: 与java基本一致，if else允许接受返回值
```scala
var result = if (a> b) a else b
```
块表达式：与java类似，能携带返回值,最后一个表达式的结果会被返回
```scala
var a = {
      print(1+1)
      1+1
      }
```
for循环
```scala
var nums = 1 to 10
for(i <- nums) { 
  println("hello scala"+i)
 }  
 //简写
for(i<- 1 to 10) println("hello scala" + i)
//打印三行五颗星
//****
//****
//****
for(i<- 1 to 3) for(j <- 1 to 5) if(j == 5) println() else print("*")
```
for循环守卫(java中的普通for循环)
```scala
for(i<- array if expression)
for(i<- 1 to 10 if i% 3 == 0) println(i)
```
yield 推导式,将循环中的某些值进行返回
```scala
var nums = for(i <- 1 to 10) yield  i * 10
```

while循环,do while循环
```scala
while ( i<= 10){
      println(i)
      i = i+1
      }

  do{
     | print(i)
     | i = i+1
     | }while(i <=10)
```
scala 中的break和continue
使用import scala.util.control.Breaks._  zk
```scala
//包裹整个for，相当于java的break
import scala.util.control.Breaks._

breakable{
       for(i<- 1 to 10)
           if(i == 5) break() else println(i)
      }

for(i<- 1 to 10){
      breakable{
       if(i%3==0) break() else println(i)
      }
 }
//包裹for的循环体,相当于java的continue
```
```scala
//打印九九乘法表
for(i <- 1 to 9) for(j <- 1 to i) if(j < i) print(s"${j} * ${i} =  ${j * i}     ") else println(s"${j} * ${i} =  ${j * i}  ")
//合并for的简化版本
for(i <- 1 to 9;j<- 1 to i)if(j < i) print(s"${j} * ${i} =  ${j * i}  ") else println(s"${j} * ${i} =  ${j * i}  ")
```
1 * 1 =  1  
1 * 2 =  2     2 * 2 =  4  
1 * 3 =  3     2 * 3 =  6     3 * 3 =  9  
1 * 4 =  4     2 * 4 =  8     3 * 4 =  12     4 * 4 =  16  
1 * 5 =  5     2 * 5 =  10     3 * 5 =  15     4 * 5 =  20     5 * 5 =  25  
1 * 6 =  6     2 * 6 =  12     3 * 6 =  18     4 * 6 =  24     5 * 6 =  30     6 * 6 =  36  
1 * 7 =  7     2 * 7 =  14     3 * 7 =  21     4 * 7 =  28     5 * 7 =  35     6 * 7 =  42     7 * 7 =  49  
1 * 8 =  8     2 * 8 =  16     3 * 8 =  24     4 * 8 =  32     5 * 8 =  40     6 * 8 =  48     7 * 8 =  56     8 * 8 =  64  
1 * 9 =  9     2 * 9 =  18     3 * 9 =  27     4 * 9 =  36     5 * 9 =  45     6 * 9 =  54     7 * 9 =  63     8 * 9 =  72     9 * 9 =  81 

### 方法，函数
**方法**
```scala
// 完整
 def getMax(a:Int,b:Int):Int ={
      return if(a >b) a else b
      }
//简化写法,返回值类型可以不用写
def getMax(a:Int,b:Int)= if(a > b) a else b  
//使用
var max = getMax(10,20)
```
一般情况下方法返回值类型可以省略，scala会自动推断，**递归方法返回值不可省略**
```scala
def factorial(n:Int):Int = if(n == 1) 1 else n * factorial(n-1)
var num = factorial(5)
```

惰性方法
当记录方法的返回值的变量呗声明位lazy时，方法的执行将会被推迟，直到第一次使用值时，方法才会执行
lazy不能修饰var类型的变量
场景：当打开数据库连接时，可以使用lazy
```scala
def getSum(a:Int,b:Int) = a+b
lazy val sum = getSum(1,2)
print(sum)
```
方法参数
默认参数
```scala
ef getSum(a:Int=10,b:Int=20) = a +b
//可以不传参
var sum  = getSum()
//带名参数
var sum  = getSum(1)  //21
var sum  = getSum(a = 1)  //21
var sum  = getSum(b = 1)   //11
```
变长参数
```scala
def getSum(a:Int*) = a.sum //数组求和方法
var sum1 = getSum1() //0
var sum1 = getSum1(1,2,3,4) //10
```

方法调用
```scala
//后缀调用
Math.abs(-10)
//中缀调用,多个用（，）拼接
getSum1 (10, 20)
//操作符即方法
1+1
//方法只有一个参数时使用{}
Math.abs{
      print("绝对值")
      -10
      }
//无括号，不存在参数，小括号可以不写,scala3版本之后不允许这样写了     
 def hello() = println("hello scala")
//Unit返回值的方法叫过程，可以不写等号
def hello() { println("hello scala")}
```

**函数**







