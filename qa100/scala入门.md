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
scala中函数式一个对象
类似于方法，函数也有参数列表和返回值，
函数定义不需要def
无需指定返回值类型

```scala
//定义
val getSum3 = (a:Int,b:Int) => a + b
//使用
val sum = getSum(11,121)
```

方法属于类或对象
> 以将函数对象赋值给一个变量，运行时，它是加载到jvm中的方法区
>可以将一个函数对象赋值给一个变量，运行时，它是加载到jvm堆内存中
> 函数式一个对象，继承自FunctionN，函数对象有apply，curried，toString，tupled这些方法，方法则没有
*函数是对象，方法属于对象*
方法转化为函数

```scala
def add(a:Int,b:Int)=a+b
//赋值给变量，方法名+ 空格+ _
val a = add _
val sum = a(1,2)  //3
```
n阶乘法表
```scala
def printMT(n:Int) ={
      for(i <- 1 to n; j<- 1 to i) if(j < i) print(s"${j} * ${i} =  ${j * i}  ") else println(s"${j} * ${i} =  ${j * i}  ")
      }
def printMT(n:Int) = for(i <- 1 to n; j<- 1 to i) print(s"${j} * ${i}  = ${j * i}" + (if(i == j)"\r\n" else "\t"))  
//函数实现
val printMT = (n:Int) => for(i <- 1 to n; j<- 1 to i) print(s"${j} * ${i}  = ${j * i}" + (if(i == j)"\r\n" else "\t"))
```
### 类和对象
scala是一种函数式面向对象语言，支持面向对象编程思想的，也有类和对象的概念，我们可以基于scala语言来开发面向对象的应用程序
面相对象的特征：封装、继承、多态

创建类
```scala
class Customer {

  var name:String = _

  var sex:String = _

  def sayHello(msg:String) = println(msg)

}
```

权限修饰
`private` ：自身使用或伴生对象使用
`private[this]`:仅自身使用伴生对象也无法使用
`protected`:允许子类使用
默认：所有范围内都可访问

构造器
```scala
class Person2(val name:String = "zhangsan",val age:Int = 23) {
    println("construct is called")

}
//调用
  val p2 = new Person2()
  println(s"p2:${p2.name},${p2.age}")
  val p3 = new Person2("lisi",27)
  println(s"p3:${p3.name},${p3.age}")
  val p4 = new Person2(age = 30)
  println(s"p4:${p4.name},${p4.age}")
```
辅助构造器
```scala
class Person3(val name:String,val address:String) {
  //  辅助构造器
    def this(arr:Array[String]){
      //第一行必须调用主构造器
      this(arr(0),arr(1))
    }
}
```
单例对象
```scala
object Dog {
  val leg_num = 4
  //程序入口
  def main(args: Array[String]): Unit = {
    println(Dog.leg_num)
    Dog.bite()
  }
  //成员方法类似于java的静态方法
  def bite()= println("wang-" * 15)
}
```
app特质
```scala
//直接继承app实现程序入口
object Demo02 extends App {
  println("hello scala")
}
```
伴生对象
java中会有些类既有实例成员又有静态成员
> 如果一个class和object具有相同的名字，这个object称为伴生对象，class称为伴生类
> 伴生对象和伴生类在同一个scala源文件中
> 伴生对象和伴生类可以相互访问private属性

```scala
class Generals {
    def toWar()={
      println(s"go to the war with ${Generals.armsName}")
    }
}
//伴生对象，静态
object Generals{
    private val armsName = "qinglongdao"
}
//使用
 val g = new Generals
  g.toWar()
```
`private[this]`  只能在当前类中访问伴生对象也无法直接访问

```scala
class Person4 {
  private[this] var name:String = _
  def getName() = name
  def setName(name:String) = this.name = name
}
object Person4{
  //private[this]变量 无法直接使用
  def  printPerson(p:Person4) = println(p.getName())

  def main(args: Array[String]): Unit = {
    val person4 = new Person4
    person4.setName("zhangsan")
    Person4.printPerson(person4)
  }
}
```

apply方法：scala支持创建对象按的时候免new动作，需要通过伴生对象来实现
```scala
class Person5(var name:String,var age:Int) {
    println("main construct")

}

object Person5{
  //scala会自动调用apply
  def apply(name: String, age: Int): Person5 = new Person5(name, age)

  def main(args: Array[String]): Unit = {
//    val p = new Person5("zhangsan",27)
    //免new操作
    val p = new Person5("zhangsan",27)

    print(p.name,p.age)
  }
}
```
日期转换示例
```scala
object DateUtilsDemo {
  object DateUtils{
      var sdf:SimpleDateFormat = null

      def date2String(date:Date,pattern:String) = {
        sdf = new SimpleDateFormat(pattern)
        sdf.format(date)
      }

      def  string2Date(dateString:String,pattern:String) ={
        sdf =  new SimpleDateFormat(pattern)
        sdf.parse(dateString)
      }
  }

  def main(args: Array[String]): Unit = {
    println(DateUtils.date2String(new Date(), "yyyy-MM-dd HH:mm:ss"))
    println(DateUtils.string2Date("2022-12-20","yyyy-MM-dd"))
  }
}
```

### 继承
class/object A extends B

类继承与java一致

单例对象继承
```scala
object Demo03 {

  class Person6(){
    var name:String = _
    var age:Int = _
    def hello() = println("hell scala")
  }

  object Student extends Person6{

  }
  def main(args: Array[String]): Unit = {
    Student.name = "zhangsan"
    println(Student.name)
    Student.hello()
  }
}
```
**方法和属性重写**
使用override重载属性,重载的属性值必须时val标记的，不能时var标记的
要使用父类中的成员变量可以使用super.method
scala编译器不允许使用super.val
object Demo04 {
  class Person7 {
    val name: String = "zhangsan"
    val age: Int = 23

    def sayHello() = println("person say hello scala")
  }
  class Student extends Person7{
    //使用override重载属性,重载的属性值必须时val标记的，不能时var标记的
    override val name = "lisi"
    //父类中用var修饰的方法不能被override
//    override var age = 30
    override val age = 30

    override def sayHello(): Unit = {
      //scala编译器不允许使用super.val
//      println(super.name)
      super.sayHello()
      println("hello student")
    }

  }

  def main(args: Array[String]): Unit = {
    val s = new Student
    println(s.name,s.age)
    s.sayHello()
  }
}

### isInstanceOf 和asInstanceOf
多态：用相同的接口去表示不同的实现
isInstanceOf: 判断对象是否是指定类型或指定类型的子类
asInstanceOf: 将对象转换为指定类型
```scala
object Demo05 {

  class Person{

  }
  class Student extends Person{
    var name:String = _
    var age:Int = _
    def sayHello() = println("person say hello sacla")
  }

  def main(args: Array[String]): Unit = {
    //多态，父类引用指向子类独享
    var person:Person = new Student
    if(person.isInstanceOf[Student]){
      val s = person.asInstanceOf[Student]
      s.sayHello()
    }
  }
}
```
### getClass 和classOf
精准判断对象类型并转换
```scala
object Demo06 {

  class Person

  class Student extends Person{
    var name:String = _
    def sayHello() = println("student say hello scala")
  }

  def main(args: Array[String]): Unit = {
        var p:Person  = new Student
        println(p.isInstanceOf[Person])  //true
        println(p.isInstanceOf[Student]) //true
        println(p.getClass == classOf[Person])  //false
        println(p.getClass == classOf[Student])  //true
  }
}
```

### 抽象类
用abstract 关键值定义的类
```scala
object Demo07 {
  abstract class Shape{
    //抽象方法
    def area:Double
  }

  class Square(var edge:Double) extends Shape {
    override def area: Double = {
      edge * edge
    }
  }

  class Rectangle(var length:Double,var width:Double) extends Shape {
    override def area: Double = {
      length * width
    }
  }

  class Circle(var raius:Double) extends Shape{
    override def area: Double = raius *raius * Math.PI
  }

  def main(args: Array[String]): Unit = {
    val s1 = new Square(5)
    println(s"s1.area:${s1.area}")
    val s2 = new Rectangle(10,5)
    println(s"s2.area:${s2.area}")
    val s3 = new Circle(5)
    println(s"s3.area:${s3.area}")
  }
}
```
### 抽象字段
```scala
object Demo08 {

  abstract class Person{
    //抽象字段
    var name:String
    val age:Int
  }

  class  Student extends Person{
    override var name: String = "zhangsan"
    override val age: Int = 23
  }

  def main(args: Array[String]): Unit = {
    val s = new Student
    println(s.name,s.age)
  }
} 
```
### 匿名内部类
语法
new objectName(){}
```scala
object Demo09 {

  abstract class Person{
    def sayHello()
  }
  def show(p:Person) = p.sayHello()

  def main(args: Array[String]): Unit = {
    //匿名内部类，直接使用
    new Person {
      override def sayHello(): Unit = println("person say hello scala")
    }.sayHello()
    //匿名内部类
    show(new Person {
      override def sayHello(): Unit = println("person say hello in method")
    })
    //使用lamda
    show(() => println("person say hello in method"))
  }
}
```
示例代码：
```scala
object Demo10 {
  abstract class Animal {
    var name: String = ""
    var age: Int = 0

    def run() = println("animal run")

    //抽象方法
    def eat()

  }

  class Cat extends Animal {
    override def eat(): Unit = println("cat eat fish")

    def catchMouse() = println("cat catch mouse")
  }

  class Dog extends Animal {
    override def eat(): Unit = println("dog eat meat")

    def lookHome() = println("dog look home")
  }

  def main(args: Array[String]): Unit = {
    val cat:Animal = new Cat
    cat.name = "jeff cat"
    cat.age = 2
    println(cat.name,cat.age)
    cat.run()
    cat.eat()

    if(cat.isInstanceOf[Cat]){
      val cat2 = cat.asInstanceOf[Cat]
      cat2.catchMouse()
    }else{
      println("not a cat")
    }
  }
}
```

### trait 特质
在不影响当前继承体系的情况下，对某些类或对象进行功能加强，可以使用特制 trait实现
提高代码的复用性
提高代码的扩展性和可维护性
类与特质之间是继承关系，只不过类与类之间是单继承，类与特质之间可以是单继承也可以是多继承
scala的特质可以有普通字段，抽象字段，普通方法，抽象方法
语法：
trait traitName{

}
class ClassName extends  traitName1  with traitName2{

}
类继承单个特质
```scala
object Demo11 {
  trait Logger{
    def log(msg:String)
  }

  class ConsoleLogger extends Logger{
    override def log(msg: String): Unit = println(msg)
  }

  def main(args: Array[String]): Unit = {
    val cl = new ConsoleLogger
    cl.log("hello scala trait")
  }
}
```
类继承多个特质
```scala
object Demo12 {

  trait MessageSender{
    def send(msg:String)
  }

  trait MessageReceiver{
    def receive()
  }

  class  MessageWoker extends MessageSender with MessageReceiver{
    override def send(msg: String): Unit = println(s"发送消息：${msg}")

    override def receive(): Unit = println(s"消息已收到")
  }

  def main(args: Array[String]): Unit = {
      val mw = new MessageWoker
      mw.send("hello scala trait")
      mw.receive()
  }
}
```
object 继承trait
```scala
object Demo13 {
  trait Logger{
    def log(msg:String)
  }

  trait  Warning{
    def warn(msg:String)
  }

  object ConsoleLogger extends Logger with Warning{
    override def log(msg: String): Unit = println(s"控制台日志信息：${msg}")

    override def warn(msg: String): Unit = println(s"控制台警告信息：${msg}")
  }

  def main(args: Array[String]): Unit = {
    ConsoleLogger.log("this is a log")
    ConsoleLogger.warn("this is a warn")
  }
}
```
trait 中的成员(抽象成员变量和方法必须override)
```scala

object Demo14 {
  trait Hero{
    var name = "关羽"
    //抽象字段
    var arms:String

    def eat() = println("guanyu eat meat")
    //抽象方法
    def toWar()
  }

  class Generals extends Hero{
    override var arms: String = "青龙偃月刀"

    override def toWar(): Unit = print(s"${name} go to war,use ${arms}")
  }

  def main(args: Array[String]): Unit = {
    val g = new Generals
    g.eat()
    g.toWar()
  }
}
```

对象混入trait
    //使用对象混入,临时拥有特质
    val u1 = new User with Logger
```scala
object Demo15 {

  trait Logger{
    def log(msg:String) = print(msg)
  }

  class User{

  }

  def main(args: Array[String]): Unit = {
    //使用对象混入,临时拥有特质
    val u1 = new User with Logger
    u1.log("this is a log")
  }
}
```
### trait 使用适配器模式
当特质中存在多个抽象方法，而具体的使用类只需要用某一个方法时，可以在中间加上一个抽象类，将所有的方法进行一个空的实现，然后让最终的使用类继承该抽象类的，实现需要使用的方法
```scala
object Demo16 {

  trait PlayLOL{
    def top()
    def mid()
    def adc()
    def support()
    def jungle()
    def schoolChild()
  }

  abstract class Player extends PlayLOL {
    override def top(): Unit = {

    }

    override def mid(): Unit = {

    }

    override def adc(): Unit = {

    }

    override def support(): Unit = {

    }

    override def jungle(): Unit = {

    }

    override def schoolChild(): Unit = {}
  }

  class GreenHand extends Player{
    override def schoolChild(): Unit = println("schoolChild")

    override def support(): Unit = {
      println("support")
    }
  }

  def main(args: Array[String]): Unit = {
    val g = new GreenHand
    g.support()
    g.schoolChild()
  }
}
```
### 使用trait实现模版方法
设计程序时知道算法具体的步骤和执行顺序，但是具体的步骤未知，可以使用模版方法
在scala中可以使用trait或抽象类实现模版类
```scala
object Demo17 {
  trait Template {
    def code()

    def getRuntime = {
      val start = System.currentTimeMillis()
      code()

      val end = System.currentTimeMillis()
      end- start
    }
  }

  class ForDemo extends Template {
    override def code(): Unit = {
      for( i <- 1 to 10000){
        println(s"hello scala${i}")
      }
    }
  }

  def main(args: Array[String]): Unit = {
    val f = new ForDemo
    println(f.getRuntime)
  }
}
```

trait实现责任链模式
多个trait中出现同一个方法，且该方法最后都调用了父类中的某个方法，当类继承这多个trait后，旧可以依次调用多个trait中的此同一个方法，形成一个调用链
**叠加特质，按从右往左的顺序执行trait的方法，然后执行，当子特质中的方法执行完毕后，最后执行父特质中的方法**
```scala
object Demo18 {
  trait Handler{
    def handle(data:String) = {
      println("具体的数据处理")
      println(data)
    }
  }
  trait  DataValidHandler extends Handler{
    override def handle(data: String): Unit = {
      println("验证数据")
      super.handle(data)

    }
  }
  trait SingatureValidHandler extends Handler{
    override def handle(data: String): Unit = {
      println("检查数据")
      super.handle(data)
    }
  }
  //按从右往左的顺序执行trait的方法，然后执行，当子特质中的方法执行完毕后，最后执行父特质中的方法
  //叠加特质
  class Payment extends DataValidHandler with SingatureValidHandler{
    def pay(data:String): Unit ={
      println("用户发起支付")
      //会按照顺序调用trait的方法
      super.handle(data)
    }
  }

  def main(args: Array[String]): Unit = {
    val p = new Payment
    p.pay("章三给李四转100元")
  }
}
```
trait的构造机制
每个特质只有一个无参构造器
遇到一个类继承另一个类及多个trait的情况，当创建类的实例时，它的构造器执行顺序如下：
1. 执行父类的构造球
2. 按照从左到右的顺序，依次执行trait的构造器
3. 如果trait有父trait，先执行父trait的构造器
4. 如果有多个trait有相同的父trait，则父trait的构造器只初始化一次
5. 执行子类构造器

```scala
object Demo19 {
 trait Logger{
   println("执行logger构造器")
 }

  trait MyLogger extends Logger{
    println("执行MyLogger构造器")
  }

  trait TimeLogger extends Logger{
    println("执行TimeLogger构造器")
  }
  class Person{
    println("执行Person构造器")
  }

  class  Student extends Person with MyLogger with TimeLogger{
    println("执行Student构造器")
  }

  def main(args: Array[String]): Unit = {
    val s = new Student
  }

  /**
   * 执行Person构造器
     执行logger构造器
     执行MyLogger构造器
     执行TimeLogger构造器
     执行Student构造器
   */
}
```
trait 继承class
特质会将class的成员都继承下来
```scala
object Demo20 {

  class Message{
    def printMsg() = println("hello scala")
  }
  trait Logger extends Message

  class ConsoleLogger extends Logger

  def main(args: Array[String]): Unit = {
    val cl = new ConsoleLogger
    cl.printMsg()
  }
}
```
综合实例
```scala 
object Demo21 {

  abstract class Programmer {
    var name: String = _
    var age: Int = _

    def eat()

    def skill()
  }

  class JavaProgrammer extends Programmer {
    override def eat(): Unit = println("java eat")

    override def skill(): Unit = println("java language")
  }

  class PythonProgrammer extends Programmer {
    override def eat(): Unit = println("python eat")

    override def skill(): Unit = println("python language")
  }

  trait BigData {
    def learningBigData() = {
      println("learning bigdata skill: hadoop,zookeeper,hbase,hive,sqoop,scala,spark")
    }
  }
  class PartJavaProgrammer extends JavaProgrammer with BigData {
    override def eat(): Unit = {

      print("java and bigdata eat")
    }

    override def skill(): Unit = {
      super.skill()
      super.learningBigData()
    }
  }
  class PartPythonProgrammer extends JavaProgrammer with BigData {
    override def eat(): Unit = {

      print("python and bigdata eat")
    }

    override def skill(): Unit = {
      super.skill()
      super.learningBigData()
    }
  }
  def main(args: Array[String]): Unit = {
    val jp = new JavaProgrammer
    jp.eat()
    jp.skill()
    var pp = new PythonProgrammer
    pp.eat()
    pp.skill()
    var bjp = new PartJavaProgrammer
    bjp.eat()
    bjp.skill()
    var pjp = new PartPythonProgrammer
    pjp.eat()
    pjp.skill()
  }
}
```
### 包
关键字`package`修饰
package目录可以与源码文件目录不一致
建议包嵌套不超过3层
三种包写法
合并版，分解版以及使用串联式

### 样例类

### 样例对象

### 案例--计算器






















