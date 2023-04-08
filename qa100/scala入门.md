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
Char 16位无符号unicode字符
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
//三引号实现，引号内的所有字符都会被保留下来，包括换行
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
当记录方法的返回值的变量被声明为lazy时，方法的执行将会被推迟，直到第一次使用值时，方法才会执行
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

scala中下层包可以直接访问上层包中的类和对象，上层访问下层包的类使用import
包对象：和包名保持一致
```scala
package com {

  package steven {
    object Demo22 {
      def main(args: Array[String]): Unit = {
        print("ssss")
        scala.hello()
      }
    }
  }

  package object scala {
    var name = "shangsan"

    def hello() = println(name)
  }
}
```
包的可见性：private[com]
```scala
private[com] var name = _ //允许name对com包可见
```
导包
```scala
//导入包下的所有内容最后用_
import java.util._
//导入多个类
import java.util.{ArrayList,HashSet}
//重命名
import java.util.{HashSet=>JavaHashSet}
//除掉包中的类(除去HashSet)
import java.util.{HashSet=>_, _}
```

### 样例类
case class([var/val(默认)] column1:type){}
> 可以用作枚举
> 可以作为没有任何参数的消息传递
//默认实现了apply,toString,equals,hashCode,copy方法
```scala
object Demo23 {
  case class Person(name:String = "zhangsan",var age:Int = 23){}


  def main(args: Array[String]): Unit = {
    //可以不加new,默认系统会实现apply
      val p = Person()
      println(s"before : name:${p.name},age:${p.age}")
    //默认实现了toString
    println(p)
    //默认修饰符为val，不允许修改
//      p.name = "lisi"
    p.age=27
    println(s"after : name:${p.name},age:${p.age}")
    println(p)
    val p3 = Person(age = 27)
    //equals//true,底层实现了equals，会比较各个属性的值是否相同
    println(p == p3)
    //重写hashCode，hashcode是相同的
    println(s"p.hash:${p.hashCode()},p3.hash:${p3.hashCode()}",p.hashCode() == p3.hashCode())
    //默认实现了copy方法
    val p2 = p.copy()
    println(p2)
  }
}
```
枚举举例
```scala
object Demo24 {
  trait Sex

  case object Male extends Sex
  case object Female extends Sex

  case class Person(var name:String,var sex:Sex)

  def main(args: Array[String]): Unit = {
    val p = Person("zhangsan",Female)
    println(p)
  }
}
```
样例类实现简单计算器
```scala
object Demo25 {

  case class Calculate(a: Int, b: Int) {
    def add() = a + b

    def substract() = a - b

    def multiply() = a * b

    def divide() = a / b
  }

  def main(args: Array[String]): Unit = {
    val c = Calculate(10,2)
    println(c.add())
    println(c.substract())
    println(c.multiply())
    println(c.divide())
  }
}
```
### 数组
定长数组
通过[]指定
```scala
object Demo26 {
  def main(args: Array[String]): Unit = {
    //int默认值为0
    val arr1 = new Array[Int](10)
    arr1(0) = 10
    println(arr1(0))

    val arr2  = Array("java","scala")
    println(arr2.length)
    println(arr2.size)
  }

}
````

变长数组
使用Arraybuffer
```scala
object Demo27 {
  def main(args: Array[String]): Unit = {
    val array1 = ArrayBuffer[Int]()
    var array2 = ArrayBuffer("hadoop","storm","spark")
    println(array1)
    println(array2)
  }
}
```
增删改元素
+= 添加单个元素
-= 删除耽搁元素
++= 追加一个数组到变长数组中
--= 移除变长数组中的指多个元素
```scala
object Demo28 {

  def main(args: Array[String]): Unit = {
    val array1 = ArrayBuffer[String]("hadoop", "spark", "flink")
    array1 += "flume"
    println(array1)
    array1 -="hadoop"
    println(array1)
    array1 ++= Array("sqoop","hive")
    println(array1)
    array1 --= Array("sqoop","hive")
    println(array1)
  }
}
//ArrayBuffer(hadoop, spark, flink, flume)
//ArrayBuffer(spark, flink, flume)
//ArrayBuffer(spark, flink, flume, sqoop, hive)
//ArrayBuffer(spark, flink, flume)

```
遍历
```scala
object Demo29 {
  def main(args: Array[String]): Unit = {
    val array = Array(10,2,3,4,5)
    //索引遍历
    for(i <- 0 to array.length-1) println(array(i))
    //util ,包左不包右
    for(i <- 0 until array.length) println(array(i))

    for(i <- array.indices) println(array(i))
    //直接获取元素
    for(i <- array) println(i)
  }
}
```
常用方法
sum（） 求和
max（）求最大值
min（）求最小值
sorted（）排序
reverse（）反转
```scala
object Demo30 {
  def main(args: Array[String]): Unit = {
    val array = Array(10,2,3,4,5)
    println(s"sum:${array.sum}")
    println(s"sum:${array.max}")
    println(s"sum:${array.min}")
    val array2 = array.sorted
    for( i <- array2) println(i)
    val array3 = array.reverse
    for( i <- array3) println(i)
    //降序
    val array4 = array.sorted.reverse
    for( i <- array4) println(i)
  }
}
```

### 元组
存储多个不同类型的值
创建
val name = (a,b,c)
//只存在两个元素的创建
val name = a -> b
```scala
object Demo31 {
  def main(args: Array[String]): Unit = {
    //元组的两种创建方式
    val tuple1 = ("zhangsan",23)
    val tuple2 = "lisi" -> 24
    println(tuple1)
    println(tuple2)
  }
}
```
访问
```scala
object Demo32 {
  def main(args: Array[String]): Unit = {
    //元组的两种创建方式
    val tuple1 = ("zhangsan","male")
    val tuple2 = "lisi" -> "male"
    //通过_获取元素
    println(s"name:${tuple1._1}")
    println(s"sex:${tuple1._2}")
    //使用迭代器
    val it = tuple2.productIterator
    for(i <- it) println(i)
  }
}
```

### 列表
可重复，有序
不可变列表：元素长度和长度都不可变 List
创建
```scala
object Demo33 {
  def main(args: Array[String]): Unit = {
    //不可变列表
   val list = List(1,2,3,4)
    val list2 = Nil
    val list3 = -1 :: -2 :: Nil
    println(list)
    println(list2)
    println(list3)
  }
}
```
可变列表 ListBuffer
创建
```scala
object Demo34 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val array1 = new ListBuffer[Int]
    //设置可变列表初始值
    val array2 = ListBuffer(1,2,4,5)

    println(array1)
    println(array2)
  }
}
```
可变列表的操作 添加，删除元素，批量添加删除元素，转化为不可变列表和数组
```scala
object Demo35 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val list1 = ListBuffer(1,2,4)

    println(list1(0))
    //添加一个元素
    list1 +=5
    //添加多个元素
    list1 ++= List(1,3,4)
    //删除找到的第一个元素
    list1 -= 4
    //删除第一个元素
    list1 --= List(1,5,3)
    println(list1)
    //转换为不可变列表
    val list2 = list1.toList
    println(list2)
    val arr = list1.toArray
    println(arr.mkString("Array(", ", ", ")"))
  }
}
```
其他操作
```scala
object Demo36 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val list1 = ListBuffer(1,2,3,4)
    //第一个元素
    println(list1.head)
    //判断是否为空
    println(list1.isEmpty)
    val list2 = List(4,5,6)
    //拼接list
    val list3 = list1 ++ list2
    println(list3)
    //除了首个元素的其他元素
    println(list1.tail)
    //反转
    println(list1.reverse)
    //获取元素前三个元素，前缀元素
    println("===="+ list1.take(5))
    //获取后缀元素，前三个元素是前缀，其他的都是后缀元素
    println(list1.drop(10))
  }
}
```
扁平化 flatten
将嵌套列表中的所有元素单独放到一个新的列表中
```scala
object Demo37 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val list1 = List(List(1, 2, 3), List(4, 5, 6), List(7, 8, 9))
    println(list1)
    //扁平化
    val list2 = list1.flatten
    println(list2)

  }
}
```
拉链与拉开
拉链：将两个列表组合成一个元素为元组的列表
拉开：将一个包含元组的列表，拆解成包含两个列表的元组
```scala
object Demo38 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val names = List("zhangsan","lisi","wangwu")
    val ages = List(23,24,25)
    //拉链操作,list转元组列表
    val list1 = names.zip(ages)
    println(list1)
    //拉开操作,元组进行拆分
    val tuple1 = list1.unzip
    println(tuple1)
  }
}
```
列表转换为字符串
toString转换
mkString，将元素以指定的分隔符拼接
```scala
object Demo39 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val names = List("zhangsan","lisi","wangwu")
    println(names.toString())
    println(names.mkString(","))
  }
}
```
列表求交集intersect，并集union，差集diff
```scala
object Demo40 {
  def main(args: Array[String]): Unit = {
    //可变列表
    //空可变列表
    val list1 = List(1,2,3,4)
    val list2 = List(3,4,5,6)
    //并集，不去重
    val list3 = list1.union(list2)
    println(list3)
    //去重
    val list31 = list3.distinct
    println(list31)
    //交集
    val list4 = list1.intersect(list2);
    //差集
    val list5 = list1.diff(list2)
    println(list5)
  }
}
```
### set集合
无序，唯一
不可变集Set
```scala
object Demo41 {
  def main(args: Array[String]): Unit = {
    //不可变集
    val set1 = Set[Int](10)
    println(s"set1:${set1}")
    val set2 = Set(1,3,4,5,6,5)
    println(s"set2:${set2}")
    //获取大小
    println(s"size:${set1.size}")
    println(s"size:${set2.size}")
    //遍历
    for(i <- set2) println(i)
    //移除元素
    val set3 = set2 - 3
    println(s"set3:${set3}")
    //拼接set
    val set4 = set2 ++ set1
    println(s"set4:${set4}")
    //拼接list
    val set5 = set2 ++ List(7,8,9)
    println(s"set5:${set5}")
  }
}
```
可变集 mutable.Set
注意先导入包 ：`import scala.collection.mutable.Set`
```scala

import scala.collection.mutable.Set

object Demo41 {
  def main(args: Array[String]): Unit = {
    //可变集
    val set1 = Set[Int](1,2,3,4)
    //添加
    set1 += 5
    //添加集
    set1 ++= Set(6,7,8)
    //添加list
    set1 ++= List(9,10,11)
    //移除
    set1 -= 1
    //移除集合
    set1 --= List(2,3,4)
    println(set1)
  }
}
```

### 映射，map
键值对，键唯一
不可变map
```scala
object Demo42 {
  def main(args: Array[String]): Unit = {
    //scala默认导入的是不可变map
    val map1 = Map("zhangsan" -> 20,"lisi"-> 25)
    println(map1)
    val map2 = Map(("zhangsan",23),("lisi",24))
    println(map2)
  }
}
```

可变map 基本操作
```scala
import scala.collection.mutable.Map
object Demo43 {
  def main(args: Array[String]): Unit = {
    //scala默认导入的是不可变map
    val map1 = Map("zhangsan" -> 20,"lisi"-> 25)
    //根据键获取值
    println(map1("zhangsan"))
    //获取所有的键
    println(map1.keys)
    //获取所有的值
    println(map1.values)
    //遍历
    for((key,value) <- map1) println(key,value)
    //获取并设置默认值
    println(map1.getOrElse("王五",-1))
    //不可变map可以使用这种方式
    val map3 = map1 + "wangwu" ->5
    println(map3)
    map1 += "wangwu" -> 5
    //移除键值对
    map1 -= "lisi"
    println(map1)
    //添加
    map1 += ("lisi"->30)
    //修改键值对
    map1("zhangsan") = 26
    println(map1)

    val map2 = Map(("zhangsan",23),("lisi",24))
    println(map2)
  }
}
```
### 迭代器
scala针对每一类集合都提供了一个迭代器iterator，用来访问集合，提供以下两个方法：
hashNext
next
**迭代器是有状态的**
```scala
object Demo44 {
  def main(args: Array[String]): Unit = {
    val list1 = List(1,2,3,4,5)
    val it = list1.iterator

    while(it.hasNext){
      println(it.next())
    }
    //会抛出异常java.util.NoSuchElementException
    it.next()
  }
}
```

### 函数式编程
函数式编程指方法的参数列表可以接收函数对象
**foreach**
```scala
object Demo45 {
  def main(args: Array[String]): Unit = {
    val list1 = List(1,2,3,4)
    //完整版本
    list1.foreach((a:Int)=> {println(a)})
    //简化版本，类型推断
    list1.foreach(a => println(a))
    //使用_简化函数定义,如果函数参数只在函数提中出现一次，并且函数体没有涉及到复杂的使用（如嵌套），可以使用_简化函数定义
    list1.foreach(println(_))
  }
}
```
**map映射**
```scala
//集合映射
object Demo46 {
  def main(args: Array[String]): Unit = {
    val list1 = List(1,2,3,4)
    //普通方式
    val list2 = list1.map((x:Int)=>{ "*" * x})
    println(list2)
    //类型推断
    val list3 = list1.map(x => "*" *x)
    println(list3)
//    使用 下划线代替简单函数
    val list4 = list1.map("*" * _)
    println(list4)
  }
}
```

**flatMap**
```scala
//flatMap扁平化操作
object Demo47 {
  def main(args: Array[String]): Unit = {
   val list = List("hadoop hive spark flink flume","kudu hbase sqoop storm")
   //先map，然后flatten
    val list2 = list.map(x => x.split(" ")).flatten
    println(s"list2:${list2}")
    //使用flatMap
    val list3 =  list.flatMap((x:String) => x.split(" "))
    println(s"list3:${list3}")
    //简化
    val list4 = list.flatMap(x => x.split(" "))
    println(list4)
    //最终的简化版本,_代替函数
    val list5 = list.flatMap(_.split(" "))
    println(list5)
  }
}
```

**filter过滤**
```scala

//flatMap扁平化操作
object Demo49 {
  def main(args: Array[String]): Unit = {
    val list = (1 to 9).toList
    //   val list = List(1 to 9)
    println(list)
    //基础版本
    val list2 = list.filter(x => x % 2 == 0)
    println(list2)
    //_代替函数
    val list3 = list.filter(_ % 2 == 0)
    println(list3)
  }
}
```
**排序sorted**
```scala
//排序
object Demo50 {
  def main(args: Array[String]): Unit = {
    val list = List(3,1,2,9,7)
    println(list)
    //默认升序
    val list2 = list.sorted
    println(list2)
    //降序
    val list3 = list.sorted.reverse
    println(list3)
    val list11 = List("01 hadoop", "02 flume","03 hive","04 spark")
    //自定义排序sortBy，按字母排序
    val list12 = list11.sortBy(x => x.split(" ")(1))
    println(list12)
    //自定义排序sortWith，降序
    val list22 = List(2,3,1,6,5)
    val list23 = list22.sortWith((a,b) => a > b)
    println(list23)
    //使用_代替函数
    val list24 = list22.sortWith(_ > _)
    println(list24)
  }
}
```
**分组groupBy**
```scala
//分组
object Demo51 {
  def main(args: Array[String]): Unit = {
    //元组list
    val list = List("刘德华" -> "男","刘亦菲"-> "女","胡歌"->"男")
    //按性别分组,_2表示取性别
    val map1 = list.groupBy(x => x._2)
    println(map1)
    //分组统计数量
    val map2 = map1.map(x => x._1 -> x._2.size)
    println(map2)
    //groupBy简化版本
    val map11 = list.groupBy(_._2)
    println(map11)
  }
}
```

**聚合reduce，fold**
reduce
```scala

//聚合reduce
object Demo52 {
  def main(args: Array[String]): Unit = {
    val list = (1 to 10).toList

    //reduce,聚合求和
    val list2 = list.reduce((x,y) => x+y )
    println(list2)
    //简化操作
    val list3 = list.reduce(_ + _)
    println(list3)
    //reduceLeft
    val list4 =  list.reduceLeft(_ + _)
    val list44 = list.sum
    println(list44)
    print(list4)
    val list5 = list.reduceRight(_ + _) //55
    println(list5)
    //相减，从左到右
    val list6 = list.reduceLeft(_ - _ ) //-53
    println(list6)
    //相减，从右到左 9-10 = -1 , 8 - -1 = 9 ,7 -9 = -2 , 6 --2 = 8, 5 -8 = -3 , 4- -3 = 7 , 3 - 7 = -4, 2 - -4 = 6 ,1 -6 = -5
    val list7  = list.reduceRight(_ - _)  //-5
    println(list7)
  }
}
```
fold
```scala
//聚合folder
object Demo53 {
  def main(args: Array[String]): Unit = {
   //fold 求和
    val list = (1 to 2).toList
    //带初始化值的累加
    val list1 = list.fold(100)((x,y) => x+y)
    println(list1) //155
    val list2 = list.fold(100)(_ + _)
    println(list2)

    //foldLeft
    val list3 = list.foldLeft(100)(_ + _)
    println(list3)
    // foldRight
    val list4 = list.foldRight(100)(_ + _)
    println(list4)
    // fold 求差
    val list11 = list.fold(100)(_ - _) //100 -（1+2+3...+10） = 45
    println(list11)
    //100 -1 -2 = 97
    val list12 = list.foldLeft(100)(_ - _) //97
    println(list12)
    // 2-100 = -98 , 1- -98 = 99
    val list13 = list.foldRight(100)(_ - _) //98
    println(list13)
  }
}
```
综合实例代码
```scala
//综合代码
object Demo54 {
  def main(args: Array[String]): Unit = {
    val list = List(("zhangsan", 37, 90, 100), ("lisi", 90, 73, 81), ("wangwu", 60, 90, 76), ("zhaoliu", 59, 21, 72), ("tianqi", 100, 100, 100))
    //   val chineseList = list.filter(x => x._2 >= 60)
    val chineseList = list.filter(_._2 >= 60)
    println(s"chineseList${chineseList}")
    val totalScoreList = list.map(x => x._1 -> (x._2 + x._3 + x._4))
    println(s"totalScore${totalScoreList}")
    val sortedScoreList = totalScoreList.sortWith((x,y) => x._2 > y._2)
    val sortedScoreList2 = totalScoreList.sortWith(_._2 > _._2)
    println(s"sortedScoreList${sortedScoreList}")
    println(s"sortedScoreList2${sortedScoreList2}")
  }
}
```

### 模式匹配
* 判断固定值
* 类型查询
* 快速获取数据
match case
```scala
//简单模式匹配
object Demo55 {
  def main(args: Array[String]): Unit = {
    println("please input:")
    val s = StdIn.readLine()
    //模式匹配
//    val result = s match {
//      case "hadoop" => "大数据分布式存储和计算框架"
//      case "zookeeper" => "大数据分布式协调服务框架"
//      case "spark" => "大数据分布式内存计算框架"
//      case _ => "未匹配"
//    }
//    println(result)

    println("-" * 15)
    //简化写法
    s match {
      case "hadoop" => println("大数据分布式存储和计算框架")
      case "zookeeper" => println("大数据分布式协调服务框架")
      case "spark" => println("大数据分布式内存计算框架")
      //使用_相当于java的default
      case _ => println("未匹配")
    }
  }
}
```
匹配类型,根据类型来进行匹配
```scala
  object Demo56 {
  def main(args: Array[String]): Unit = {
    //    val a:Any = "hadoop"
    val a: Any = 1.3
    val result = a match {
      case x: String => s"${x}是一个string"
      case x: Int => s"${x}是一个Int"
      case x: Double => s"${x}是一个Double"
      case _ => "未匹配"
    }
    println(result)

    //简写版本 _代替变量x
    val result2 = a match {
      case _: String => "string"
      case _: Int => "Int"
      case _: Double => "Double"
      case _ => "未匹配"
    }
    println(result2)
  }
}
```
守卫
case语句中可以添加if条件判断
```scala
//模式匹配守卫
object Demo57 {
  def main(args: Array[String]): Unit = {
    println("please input：")
    val s = StdIn.readInt()
    s match {
      //添加守卫
      case x if x >= 0 && x <= 3 => println("[0-3]")
      case x if x >= 4 && x <= 8 => println("[4-8]")
      case _ => println("未匹配")
    }
  }
}
```
样例对象
```scala
//模式匹配样例类
object Demo58 {
  case class Customer(name:String,age:Int)

  case class Order(id:Int)

  def main(args: Array[String]): Unit = {
//    val c:Any = Customer("zhangsan",23)
    val c:Any = Order(24)

    c match {
      case Customer(a,b) => println(s"Customer:name=${a},age=${b}")
      case Order(a) => println(s"Order:id=${a}")
      case _ => println("未匹配")
    }
  }
}
```
数组匹配
```scala
//模式匹配数组
object Demo59 {
  case class Customer(name:String,age:Int)

  case class Order(id:Int)

  def main(args: Array[String]): Unit = {
    val arr1 = Array(1,2,4)
    val arr2 = Array(0)
    val arr3  = Array(1,2,3,5,5)
    arr3 match {
      case Array(1,x,y) => println(s"长度为3，首元素为1,其他元素x=${x},y=${y}")
      case Array(0) => println(s"长度为1，首元素为0,")
      case Array(1,_*) => println(s"长度任意，首元素为0")
      case _ => println("未匹配")
    }
  }
}
```
列表匹配
```scala

//模式匹配list
object Demo60 {
  case class Customer(name:String,age:Int)

  case class Order(id:Int)

  def main(args: Array[String]): Unit = {
    val list1 = List(1,2,30)
    val list2 = List(0,1,2,3)
    val list3 = List(11,22)



    list1 match {
      case List(0) => println("匹配只包含一个0的列表")
      case List(0,_*) => println("匹配以0开头，其他无所谓的列表")
      case List(x,y) => println(s"匹配包含两个元素的列表，${x},${y}")
      case _ => println("未匹配")
    }

    list1 match {
      case 0::Nil => println("匹配只包含一个0的列表")
      case 0::tail => println("匹配以0开头，其他无所谓的列表")
      case x::y::Nil => println(s"匹配包含两个元素的列表，${x},${y}")
      case _ => println("未匹配")
    }
  }
}
```
元组匹配
```scala
//模式匹配元组
object Demo61 {
  case class Customer(name:String,age:Int)

  case class Order(id:Int)

  def main(args: Array[String]): Unit = {
    val tuple1 = (1,4,5)
    val tuple2 = (3,4,5)
    tuple1 match {
      case (1,x,y) => println(s"元组长度为3，以1开头，其他元素${x},${y}")
      case (x,y,5) => println(s"元组长度为3，以5结尾，其他元素${x},${y}")
      case _ => println("未匹配")
    }
  }
}
```
变量声明模式匹配
```scala
//模式匹配-变量声明
object Demo62 {
  case class Customer(name:String,age:Int)

  case class Order(id:Int)

  def main(args: Array[String]): Unit = {
    val arr1 = (1 to 10).toArray
    //匹配第二个第三个第五个元素
    val Array(_,x,y,_,z,_*) = arr1
    println(x,y,z)

    val list1 =(1 to 10).toList
    val List(a,b,_*) = list1
    println(a,b)
    val a1 :: b1::tail = list1
    println(a1,b1)
  }
}
```

模式匹配-for表达式
```scala
//模式匹配-for表达式
object Demo63 {
  case class Customer(name: String, age: Int)

  case class Order(id: Int)

  def main(args: Array[String]): Unit = {
    val map1 = Map("zhangsan" -> 23, "lisi" -> 24, "wangwu" -> 23)
    //if实现
    for ((k, v) <- map1 if v == 23) println(k, v)
    //固定值实现
    for((k,23) <- map1) println(k,23)
  }
}
```

### Option类型
解决空指针异常的问题
```scala
//Option
object Demo64 {
  def divide(a:Int,b:Int) = {
    if(b== 0){
      None
    }else{
      Some(a/b)
    }
  }

  def main(args: Array[String]): Unit = {
    //算术异常
    val result1 = divide(4,2)
    println(s"${result1}")
    //通过模式匹配实现
    result1 match {
      case Some(x) => println(s"result:${x}")
      case None => println("除数不能为0")
    }
    //通过getOrElse实现

    println(result1.getOrElse("除数不能为0"))
  }
}
```

### 偏函数
一次性的函数，PartialFunction实例的对象，一般结合集合使用
```scala
//偏函数
object Demo65 {
  def main(args: Array[String]): Unit = {
    val pf:PartialFunction[Int,String] = {
      case 1 => "1"
      case 2 => "2"
      case 3 => "3"
      case 4 => "4"
      case 5 => "5"
      case  _ => "其他"
    }

    println(pf(1))
    println(pf(2))
    println(pf(3))
    println(pf(10))
  }
}
```
```结合map使用
//偏函数
object Demo66 {
  def main(args: Array[String]): Unit = {
    val list = (1 to 10).toList
    //偏函数结合map使用,{}内是一个偏函数
    val list2 = list.map{
      case x if x>=1 && x <=3 =>"[1-3]"
      case x if x>=4 && x <=8 =>"[4-8]"
      case _ =>"(8-*]"
    }
    println(list2)
  }
}
```



### 正则表达式
使用：
1. 使用regex类来定义正则表达式
2. 构造一regex对象，直接使用String类的r方法
3. 建议使用三引号来表示正则表达式，不然就得对正则表达式中的特殊字符进行转义

校验邮箱
```scala
//正则表达式，校验邮箱
object Demo67 {
  def main(args: Array[String]): Unit = {
    //正则规则
    val regex = """.+@.+\..+""".r
    val email = "12312312@qq.com"
    if(regex.findAllMatchIn(email).size!=0){
      println(true)
    }else{
      println(false)
    }
  }
}
```
过滤不合法
```scala
//正则表达式，校验邮箱
object Demo68 {
  def main(args: Array[String]): Unit = {
   val list = List("12345@google.com","12321dsdfas@126.com","zhanga@qq.cn","zhanga@abc")
    val regex =""".+@(.+)\..+""".r
//    val filterList = list.filter(x => regex.findAllMatchIn(x).isEmpty)
    val filterList = list.filter(regex.findAllMatchIn(_).isEmpty)
    println(filterList)
    //使用偏函数
    val list2 = list.map{
      //case中的正则匹配,取字符串中的某一段
      case x @ regex(company) => x -> company
      case x => x -> "未匹配"
    }
    println(list2)
  }
}
```
### 异常处理
捕获与抛出
```scala
//异常处理
object Demo69 {
  def main(args: Array[String]): Unit = {
    try{
      val i = 10/0
    }catch {
      case ex:ArithmeticException => {
        println("算术异常")
        ex.printStackTrace()
      }
      case exception: Exception => {
        println("其他异常")
        exception.printStackTrace()
      }
    }finally {
      println("other ....")
    }
    println("other1 ....")
    throw new Exception("a exception")
  }
}
```

### 提取器
提取器指unapply()方法，解构对象
apply构造对象
```scala
//提取器unapply
object Demo70 {

  class Student(var name: String, var age: Int)

  object Student {
    def apply(name: String, age: Int) = new Student(name, age)
    //提取器
    def unapply(s: Student) = {
      if (s == null) {
        None
      } else {
        Some(s.name, s.age)
      }
    }
  }

  def main(args: Array[String]): Unit = {
    val s1 = new Student("zhangsan", 23)
    val s2 = new Student("list", 24)
    println(s1.name, s1.age)
    val result = Student.unapply(s1)
    println(result)
    s1 match {
      case Student(name,age) => println(s"${name},${age}")
      case _ => println("未匹配")
    }
  }
}
```
### Source读取数据
Source单例对象中提供了很多便捷的方法
简单按行读取
```scala
import scala.io.Source
object Demo71 {
  def main(args: Array[String]): Unit = {
    val source = Source.fromFile("./datas/1.txt")

    val lines:Iterator[String] = source.getLines()
    val list:List[String] = lines.toList
    for(data <- list) println(data)
    source.close()
  }
}
````
按字符读取
```scala
import scala.io.Source

//按字符读取
object Demo72 {
  def main(args: Array[String]): Unit = {
    //默认utf-8
    val source = Source.fromFile("./datas/1.txt","utf-8")
    val iter: BufferedIterator[Char] = source.buffered
    //按字符读取
    //    while(iter.hasNext){
    ////      println(iter.next())
    //      print(iter.next())
    //    }
    //将数据读取到字符串中
    val string = source.mkString
    println(string)
    source.close()
  }
}
```
读取词法单元
```scala
//读取词法单元和数字
object Demo73 {

  def main(args: Array[String]): Unit = {
    val source  = Source.fromFile("./datas/3.txt")
    val string = source.mkString
    println(string)
    /**
     * 按空白字符进行切分
     */
    val strArray:Array[String] = string.split("\\s+")
    val intArray:Array[Int] = strArray.map(_.toInt).map(_ +1)
    for(i <- intArray) println(i)
  }
}
```
从url读取数据
```scala
//从url中读取数据
object Demo74 {

  def main(args: Array[String]): Unit = {
    val source  = Source.fromURL("https://portal.dxy.cn/")
    val str1 = source.mkString
    println(str1)
    //从字符串中读取
    val source1  = Source.fromString("sssssss")
    println(source1.mkString)
  }
}
```
读取二进制文件
```scala
//读取图片数据
object Demo75 {

  def main(args: Array[String]): Unit = {
    //读取一张文件
    val file = new File("./datas/wallE.jpeg")
    val fis = new FileInputStream(file)
    val bytes = new Array[Byte](file.length().toInt)
    val length = fis.read(bytes)
    println(length)
  }
}
```
文件写入（java api）
```scala
import java.io.{File, FileInputStream, FileOutputStream}

//写入数据
object Demo76 {

  def main(args: Array[String]): Unit = {
    val fos = new FileOutputStream("./datas/5.txt")
    fos.write("hello java \r\n".getBytes())
    fos.write("hello scala".getBytes())
    fos.close()
  }
}
```
序列化与反序列化
继承serializable接口或直接使用样例类（case类实现了Serializable接口）
```scala
//序列化和反序列化
object Demo77 {

  //继承Serializable
//  class Person(var name: String, var age: Int) extends Serializable
//  使用样例类
  case class Person(var name: String, var age: Int)

  def main(args: Array[String]): Unit = {
    //序列化
        val person = new Person("zhangsan",23)
        val oos = new ObjectOutputStream(new FileOutputStream("./datas/6.txt"))
        oos.writeObject(person)
        oos.close()
    //反序列化
    val ois = new ObjectInputStream(new FileInputStream("./datas/6.txt"))
    val p:Person = ois.readObject().asInstanceOf[Person]
    println(p.name,p.age)
  }
}
```
综合实例：学员成绩表
```scala
//序列化和反序列化
object Demo78 {

  class Student(var name:String,var ch:Int,var math:Int,var en:Int){
    def getSum() = ch +math + en
  }


  def main(args: Array[String]): Unit = {
    //读取文件解析
    val source = Source.fromFile("./datas/student.txt")
    val iterator:Iterator[Array[String]] = source.getLines().map(_.split("\\s+"))
    val studentList = ListBuffer[Student]()
    for ( s <- iterator){
      studentList += new Student(s(0),s(1).toInt,s(2).toInt,s(3).toInt)
    }
    source.close()
    //排序
    val sortList = studentList.sortBy(_.getSum()).reverse.toList
    //写入新的文件
    val writer = new BufferedWriter(new FileWriter("./datas/sortStu.txt"))
    for(s <- sortList){
      writer.write(s"${s.name} ${s.ch} ${s.math} ${s.en} ${s.getSum()}\t\n")
    }
    writer.close()
    println(studentList)
  }
}
```

### 作为值的函数与匿名函数 --高阶函数
函数的参数列表可以接收另一个函数对象
分类
* 值函数
```scala
//值函数
object Demo79 {

  def main(args: Array[String]): Unit = {
      val list1 = (1 to 10 ).toList
    //值函数
    val func = (x:Int) =>{"*" * x}

    val list2 = list1.map(func)
    println(list2)
  }
}
```
* 匿名函数
```scala
//匿名函数
object Demo80 {

  def main(args: Array[String]): Unit = {
     val list1 = (1 to 10).toList
    //匿名函数
    val list2 = list1.map((x:Int) => "*" * x)
    //简化
    val list21 = list1.map("*" * _)
    println(list21)
  }
}
```

* 柯里化
将原来接受多个参数的方法转换为多个只有一个参数的参数列表过程  
```scala
//柯里化
object Demo81 {
  //普通函数
  def merge1(s1: String, s2: String) = s1 + s2

  //柯里化写法
  def merge2(s1: String, s2: String)(f1: (String, String) => String) = f1(s1, s2)

  def main(args: Array[String]): Unit = {

    val str1 = merge1("abc", "xyz")
    println(str1)
    //传入函数
    val str2 = merge2("abc", "dxy")(_ + _)
    println(str2)
    val str3 = merge2("abc","dxy")(_.toUpperCase()+ _)
    println(str3)
  }
}
```


* 闭包
可以访问不在当前作用域范围数据的一个函数
```scala
//闭包
object Demo82 {

  def main(args: Array[String]): Unit = {
    val x = 10
    //可以访问不在当前作用域的数据的函数，闭包
    val getSum = (y: Int) => {
      x + y
    }
    println(getSum(12))
  }
}
```

* 控制抽象
函数A的参数列表需要接受一个函数B，函数B没有输入也没有输出，则A称为控制抽象函数
```scala
//控制抽象
object Demo83 {
  def main(args: Array[String]): Unit = {
    //控制抽象，参数是无参数无返回值的函数
    val myShop = (f1:()=>Unit ) =>{
      println("weclome in")
      f1()
      println("thanks for coming")
    }
    //使用
    myShop(() => {
      println("start shopping")
      println("buy computer")
      println("buy cellphone")
      println("stop shopping")
    })
  }
}
```
综合案例计算器
```scala
//计算
object Demo84 {
  //通过定义四个方法进行实现
  def add(a:Int,b:Int) = a+b
  def substract(a:Int,b:Int) = a-b
  def multiply(a:Int,b:Int) = a*b
  def divide(a:Int,b:Int) = a/b

  //柯里化
  def calculate(a:Int,b:Int)(func:(Int,Int) => Int)  = func(a,b)

  def main(args: Array[String]): Unit = {
    val a =10
    val b = 2
    println(add(a,b))

    //传入函数对象，柯里化
    println(calculate(a,b)(_ + _))
  }
}
```

### 隐式转换 implicit 
scala 2.10版本中支持，自动将某个类型数据转化为另一个类型
在object单例对象中使用implicit 关键字修饰方法
当对象调用类中不存在的方法时，编译器会自动对对象进行隐式转换（隐式转换的方法签名参数要与指定方法保持一致）
```scala
//隐式转换
object Demo85 {

  class RichFile(file: File){
    //定义read方法读取数据到字符串中
    def read() = Source.fromFile(file).mkString
  }
  object ImplicitDemo{

    implicit  def file2RichFile(file: File) = new RichFile(file)
  }


  def main(args: Array[String]): Unit = {
    //手动导入隐式转换
    //首先找file类是否有read（）方法，有就用
    //如果没有，查看该类型有没有隐式转换，有就进行类型转换，查看转换后的对象是否有指定的方法，有则直接调用转换后的对象的指定方法
    //没有直接报错
    import ImplicitDemo.file2RichFile

    val file = new File("./datas/1.txt")

    println(file.read())
  }
}
```
自动隐式转换
```scala
//隐式转换,自动导入
object Demo86 {

  class RichFile(file: File){
    //定义read方法读取数据到字符串中
    def read() = {
      val source = Source.fromFile(file)
      val str = source.mkString
      source.close()
      str
    }
  }


  def main(args: Array[String]): Unit = {
    //定义隐式转换方法将普通File对象转换为RichFile,自动 导入
    implicit  def File2RichFile(file: File) = new RichFile(file)
    val file = new File("./datas/1.txt")

    println(file.read())
  }
}
```


### 隐式参数 implicit 
方法可以带一个标记为implicit的参数列表，调用方法是，参数列表可以不用给初始化值，编译器会自动查找缺省的值
手动导入隐式参数
```scala
//隐式参数
object Demo87 {

  //给name加上前缀和后缀信息
  def show(name: String)(implicit delimit: (String, String)) = delimit._1 + name + delimit._2

  //隐式值
  object ImplicitParam{
    implicit val delimit_default: (String, String) = "<<<" ->">>>"
  }

  def main(args: Array[String]): Unit = {
    //手动导入隐式参数
    import ImplicitParam.delimit_default
    println(show("zhangsan"))
    println(show("lisi")("===","==="))
  }
}
```
自动导入隐式参数
```scala
//隐式参数
object Demo88 {

  //给name加上前缀和后缀信息
  def show(name: String)(implicit delimit: (String, String)) = delimit._1 + name + delimit._2

  def main(args: Array[String]): Unit = {
    //自动导入隐式参数
    implicit val delimit: (String, String) =   "<<<"-> ">>>"
    println(show("zhangsan"))
    println(show("lisi")("===","==="))
  }
}
```
综合案例：获取列表元素平均值
```scala
//隐式参数
object Demo89 {
  //获取列表平均值
  class RichList(list: List[Int]){
      def getAvg() = {
        if(list.size == 0){
          None
        }else
          Some(list.sum/list.size)
      }
  }

  def main(args: Array[String]): Unit = {
    val list = (1 to 10).toList
    //定义隐式转换方法
    implicit def RichList(list: List[Int]) = new  RichList(list)
    println(list.getAvg())
  }
}
```

### 递归
方法自己调用自己
递归实现阶乘
n! = n*(n-1)!
```scala
//递归求阶乘
object Demo90 {

  //递归
  def factorial(n:Int):Int = if(n == 1) 1 else n * factorial(n-1)

  def main(args: Array[String]): Unit = {
    println(factorial(5))
  }
}
```

斐波那契数列
```scala
//递归求斐波那契数列
object Demo91 {

  def fbnq(n:Int):Int = if(n == 1|| n==2) 1 else fbnq(n-1)+ fbnq(n-2)

  def main(args: Array[String]): Unit = {
      println(fbnq(10))
  }
}
```
递归打印文件目录
```scala
//递归打印文件目录
object Demo91 {

  def printFileIndex(file:File):Unit ={
    if(file.isDirectory){
      val files = file.listFiles()
      if(files.nonEmpty){
        for( f <- files) printFileIndex(f)
      }
    }else{
      println(s"${file.getPath}")
    }
  }

  def main(args: Array[String]): Unit = {
    println(printFileIndex(new File("/Users/lijie3/Documents/data")))
  }
}
```

### 泛型
用[]表示
泛型方法
```scala
//泛型方法
object Demo92 {

  //不使用泛型
  def getMiddleEment(arr:Array[Int]) = arr(arr.length/2)
  def getMiddleEment1(arr:Array[Any]) = arr(arr.length/2)
  //使用自定义泛型方法
  def getMiddleElement2[T](arr:Array[T]) = arr(arr.length/2)
  def main(args: Array[String]): Unit = {
    println(getMiddleEment(Array(1,2,3,4,5)))
    println(getMiddleEment1(Array('a','b','c')))
    println(getMiddleElement2(Array('a','b','c')))
  }
}
```
//泛型类
泛型放在类的定义上
```scala
//泛型类
object Demo93 {
  //泛型类
  class Pair[T](var a:T,var b:T)

  def main(args: Array[String]): Unit = {
    val pair = new Pair[Int](1, 2)
    println(s"${pair.a},${pair.b}")
  }
}
```
泛型特质
泛型定义到特质的声明上来
```scala
//泛型特质
object Demo94 {
  //泛型特质
  trait Logger[T]{
    val a:T
    def show(b:T)
  }

  object ConsoleLogger extends Logger[String]{
    override val a: String = "zhangxueyou"

    override def show(b: String): Unit = println(b)
  }
  def main(args: Array[String]): Unit = {
    println(ConsoleLogger.a)
    println(ConsoleLogger.show("hello scala"))
  }
}
```
泛型上界
`[T <: type]`
泛型下界
`[T >: type]`
上下界
`[T >:typ1 <: type2]`
```scala
//泛型上下界
object Demo95 {
    class Person

  class Student extends Person
  class Policeman extends Person


  //定义泛型的上界为Person
  def demo[T <: Person](arr:Array[T]) = println(arr)
  //定义下界
  def demo2[T >: Policeman](arr:Array[T]) = println(arr)
  //定义上下界
  def demo2[T >: Policeman <: Person](arr:Array[T]) = println(arr)


  def main(args: Array[String]): Unit = {
    demo(Array(new Person,new Person))
    //报错
//    demo(Array(new Object,new Person))
    demo(Array(new Student,new Person))
  }
}
```
非变： 类A和类B之间是父子类关系，Pair[A]与pair[B]没有任何关系  class pair[T]
协变： 类A和类B之间是父子类关系，Pair[A]与pair[B]也是父子关系 class pair[+T]
逆变： 类A和类B之间是父子类关系，Pair[A]与pair[B]是子父类的关系 class pair[-T]
```scala
//泛型非变 协变 逆变
object Demo96 {
  class Super

  class Sub extends Super

  class Temp1[T]

  class Temp2[+T]

  class Temp3[-T]


  def main(args: Array[String]): Unit = {
    val t1: Temp1[Sub] = new Temp1[Sub]
    //非变，Sub和super没有关系，报错
    //    val t2:Temp1[Super] = t1
    val t3: Temp2[Sub] = new Temp2[Sub]
    //协变
    val t4: Temp2[Super] = t3
    //逆变
    val t5:Temp3[Sub] = new Temp3[Sub]
    //逆变，报错
//    val t6:Temp3[Super] = t5
    val t7:Temp3[Super] = new Temp3[Super]
    val t8:Temp3[Sub] = t7
  }
}
````
列表去重排序
```scala
import java.io.{BufferedWriter, FileWriter}
import scala.io.Source

//文件内容去重排序
object Demo97 {

  def main(args: Array[String]): Unit = {
    val source = Source.fromFile("./datas/7.txt")
    val list1 = source.mkString.split("\\s+").toList
    val list2 = list1.map(_.toInt)
    val set:Set[Int]  = list2.toSet
    val list3:List[Int] = set.toList.sorted
    println(list1)
    val bw = new BufferedWriter(new FileWriter("./datas/8.txt"))
    for(i <- list3){
      bw.write(i.toString)
      bw.newLine()
    }
    bw.close()
    source.close()
  }
}
```

### scala的集合
Traversable :顶层trait
Iterable：迭代器trait
Set/Seq（IndexedSeq,LinearSeq）/Map 常用集合
不可变集合不需要主动导入包，可变集合需要手动导入包
* 创建Traversable
```scala
object Demo99 {
  def main(args: Array[String]): Unit = {
    //空Int集合
    val t1 = Traversable.empty[Int]
    val t2 = Traversable[Int]()
    val t3:Traversable[Int] = Nil
    //比较集合中的数据
    println(t1== t2)
    println(t1== t3)
    println(t2== t3)
    //比较集合的地址值
    println(t1 eq t3)
    println(t1 eq t2)
    println(t2 eq t3)

    //使用伴生对象
    val t4:Traversable[Int]  = List(1,2,3).toTraversable
    val t5:Traversable[Int]  = Set(1,2,3).toTraversable
    val t6:Traversable[Int] = Traversable(11,22,33,44)
    println(t4)
    println(t5)
    println(t6)
    //ist(1, 2, 3)
    //Set(1, 2, 3)
    //List(11, 22, 33, 44)

  }
}
```
traversable转置
```scala
//traversable 集合转置
object Demo100 {

  def main(args: Array[String]): Unit = {

    val t1:Traversable[Traversable[Int]]  = Traversable(Traversable(1,4,7),Traversable(2,5,8),Traversable(3,6,9))
//    转置
    val t2:Traversable[Traversable[Int]] = t1.transpose

    println(t1)
    println(t2)

  }
}
```
拼接集合使用concat方法实现
```scala
//traversable 拼接
object Demo101 {


  def main(args: Array[String]): Unit = {
    val t1 = Traversable(11,22,33)
    val t2 = Traversable(23,45)
    val t3 = Traversable( 100,200)
    val t4 = Traversable.concat(t1, t2, t3)
    println(t4)
  }
}
//List(11, 22, 33, 23, 45, 100, 200)
```
* 利用偏函数筛选元素 collct方法
```scala
//traversable 拼接
object Demo102 {


  def main(args: Array[String]): Unit = {
    val t1 = (1 to 10).toTraversable //底层是Vector
    val t2 = Traversable(1,2,3,4,5,6,7,8,9,10) //底层是List

    val pf:PartialFunction[Int,Int] = {
      case x if x% 2 == 0 =>x
    }
    val t3 = t1.collect(pf)
    println(t3) //Vector(2, 4, 6, 8, 10)
    val t4 = t2.collect({
      case x if x % 2 == 0 => x
    })
    println(t4) //List(2, 4, 6, 8, 10)
  }
}
```
* 计算集合元素的阶乘 ，scan/scanLeft()/scanRight()
```scala
//traversable 拼接
object Demo103 {
  def main(args: Array[String]): Unit = {
    val t1  = Traversable(1,2,3,4,5)
    //使用scan代替递归调用
    //x 前一次阶乘的值
    //y 下一个要计算的数据
    val t2 = t1.scan(1)((x:Int,y:Int) => x * y)
    val t3 = t1.scanLeft(2)(_ * _)
    println(t2)
    println(t3)
  }
}
```
* 获取集合中的指定元素
head/last/headOption/lastOption/find/slice
```scala
//集合基本方法 
object Demo104 {
  def main(args: Array[String]): Unit = {
    val t1  = Traversable(1,2,3,4,5)
    println(t1.head)
    println(t1.last)
    println(t1.headOption)
    println(t1.lastOption)
    //找到第一个满足条件的
    println(t1.find( _ %2 == 0))
    //从index =1 开始到index 3-1 = 2 左闭右开
    println(t1.slice(1,3))
  }
}
```
* 判断集合元素是否满足要求 forall全部满足/exists一个满足即可
```scala
//集合基本方法
object Demo105 {
  def main(args: Array[String]): Unit = {
    val t1 = Traversable(1,2,3,4,5,6)
    //全部是偶数返回true
    println(t1.forall(_%2==0))
    //只要有一个数是偶数返回true
    println(t1.exists(_%2==0))
    //返回的是一个集合
    println(t1.filter(_%2==0))
  }
}
```
* 聚合函数 count/sum/product(乘积)/max/min
```scala
//集合基本方法
object Demo106 {
  def main(args: Array[String]): Unit = {
    val t1 = Traversable(1,2,3,4,5,6)
//    count
    println(t1.count(_%2 != 0))
    //所有数据的个数
    println(t1.count(_ != null))
    println(t1.sum)
    //乘积
    println(t1.product)
    println(t1.max)
    println(t1.min)
  }
}
```
集合类型转换
```scala
//集合类型转换
object Demo107 {
  def main(args: Array[String]): Unit = {
    val t1 = Traversable(1,2,3,4,5,6)
    val arr = t1.toArray
    val set = t1.toSet
    val list  = t1.toList
    println(arr,set,list)
  }
}
```

填充元素 fill()/iterate()/range()
```scala
//集合填充
object Demo108 {
  def main(args: Array[String]): Unit = {
    //fill填充
    val t1 = Traversable.fill(5)("hello scala")
    println(t1)
    println(Traversable.fill(5)(Random.nextInt(100)))
    //生成一个二维数组，数组长度为5，每个数组元素的长度为2
    println(Traversable.fill(5,2)("hello scala"));
    //1是初始化值，5是获取元素个数，*10是步长
    println(Traversable.iterate(1,5)(_ * 10))
    //1是第一个数的值，21是最大值（不包含），5是步长
    println(Traversable.range(1,2,5))
    //步长是1
    println(Traversable.range(1,21))
  }
}
```
随机序列
```scala
//集合 随机序列
object Demo109 {
  case class Student(name:String,age:Int)

  def main(args: Array[String]): Unit = {
    val names:List[String] = List("a","b","c","d","e")
    val r:Random = new Random()
    val t1:Traversable[Student] = Traversable.fill(5)(new Student(names(r.nextInt(names.size)),r.nextInt(10)+20))
    //排序
    val t2 = t1.toList.sortBy(_.age).reverse
    val t3 = t1.toList.sortWith((a,b) => a.age > b.age)
    val t4 = t1.toList.sortWith(_.age > _.age)

    println(t2)
    println(t3)
    println(t4)
  }
}
```

### Iterable 特质
集合迭代的两种方式
iterator()主动迭代的方式
foreach（）被动迭代方式
```scala
//集合 iterable
object Demo110 {
  def main(args: Array[String]): Unit = {
    val list1 = List(1,2,3,4,5)
    //iterator()迭代
    val it = list1.iterator
    while(it.hasNext) {
      println(it.next())
    }
    for(i <- list1) println(i)
    list1.foreach((x:Int) => println(x))
    //简化写法
    list1.foreach( println(_))
  }
}
```
分组遍历
```scala
//集合 iterable
object Demo111 {

  def main(args: Array[String]): Unit = {
    val it = (1 to 13).toIterable
    val it2 = it.grouped(5)
    while (it2.hasNext){
      val ints = it2.next()
      println(ints)
      val it3 = ints.iterator
      while (it3.hasNext){
        println(it3.next())
      }
    }
    //底层拿到的是Vector
    //Vector(1, 2, 3, 4, 5)
    //Vector(6, 7, 8, 9, 10)
    //Vector(11, 12, 13)
  }
}
```
按照索引生成元组
```scala
//集合 iterable
object Demo112 {

  def main(args: Array[String]): Unit = {
    val list1 = Iterable("a","b","c","d","e")
    //生成元组
    val list2:Iterable[(String,Int)] = list1.zipWithIndex
    println(list2)
    //索引放在前面
    val tuples = list2.map(x => x._2 -> x._1)
    println(tuples)

  }
}
```
判断集合是否相同 sameElements（）：元素一样且迭代顺序一致返回true
```scala
import scala.collection.immutable.{HashSet, TreeSet}

//集合 sameElements():集合的元素和迭代顺序保持一致返回true
object Demo113 {

  def main(args: Array[String]): Unit = {
    val list1 = Iterable("a","b","c","d","e")
    println(list1.sameElements(Iterable("a","b","c","d","e"))) //true
    println(list1.sameElements(Iterable("d","e","a","b","c"))) //false
    println(list1.sameElements(Iterable("a","b","c","d"))) //false
    val hashset = HashSet("a","b","c","d","e")
    val treeSet = TreeSet("a","b","c","d","e")
    println(hashset.sameElements(treeSet)) //true,treeSet默认会对元素进行升序排列
  }
}
```
### Seq集合
按照一递归的顺序排列的元素列表
可重复，有索引
子类
IndexedSeq -> Vector，Range，NumericRange，String
LinearSeq -> List,Queue,Stack,Stream
```scala
//集合 Seq
object Demo114 {

  def main(args: Array[String]): Unit = {
   val seq = (1 to 5).toSeq
    println(seq)
      //Range 1 to 5
      获取长度及元素
    println(seq.length,seq.size)
    //按index查询数据
    println(seq(2))
    println(seq.apply(2))
    //查找元素index
    //查找元素索引
    println(seq2.indexOf(2))
    println(seq2.lastIndexOf(2))
    println(seq2.indexWhere(x => x <5 && x %2 == 0))
    //从index =2 开始查找
    println(seq2.indexWhere(x => x <5 && x %2 == 0,2))
    println(seq2.lastIndexWhere(x => x <5 && x %2 == 0))
    println(seq2.indexOfSlice(Seq(3,2))) //找不到返回-1
    //从索引3开始查找
    println(seq2.indexOfSlice(Seq(1,2),3))  
     //判断集合是否包含某个数据或某个序列
    val seq3 = (1 to 10).toSeq
    //是否以指定序列开头
    println(seq3.startsWith(Seq(1,2)))
    println(seq3.startsWith(Seq(1,3)))
    println(seq3.endsWith(Seq(9,10)))
    println(seq3.endsWith(Seq(8,10)))
    println(seq3.contains(3))
    println(seq3.containsSlice(Seq(1,2)))
    println(seq3.containsSlice(Seq(1,3)))
    //修改指定元素 update()/patch()
    val seq4 = (1 to 5).toSeq
    //2 索引位置 10 要修改的值
    val s2 = seq4.updated(2,10)
    println(s2)
    // 从index= 1开始替换 ,Seq(10,20)要替换的序列，3 替换多少个元素
    val s3 = seq4.patch(1,Seq(10,20),3) //Vector(1, 10, 20, 5)
    println(s3)
  }
}
```
### stack 栈 后进先出
```scala
//集合 Stack
object Demo116 {

  def main(args: Array[String]): Unit = {
    //添加的顺序是5，4，3，2，1
    val s1 = mutable.Stack(1, 2, 3, 4, 5)
    println(s1) //Stack(1, 2, 3, 4, 5)
    println(s1.top)
    println(s1.push(6)) //Stack(6, 1, 2, 3, 4, 5)
    println(s1.pushAll(Seq(11,22,33))) //Stack(33, 22, 11, 1, 2, 3, 4, 5)
    println(s1.pop())//33
    //清空
    println(s1.clear())
  }
}
```
可变栈ArrayStack
```scala
//集合 Stack
object Demo117 {

  def main(args: Array[String]): Unit = {
    //加入顺序是5，4，3，2，1
   val s1 = mutable.ArrayStack(1,2,3,4,5)
    println(s1)
    //复制栈顶再押入
    s1.dup()
    println(s1)
    //方法执行之后栈中的数据会恢复到执行之前
    s1.preserving({
      //方法内清除
      s1.clear()
      println("do clear")
    })
    println(s1)
  }
}
```

### Queue 队列 先进后出
```scala
//集合 Queue
object Demo118 {

  def main(args: Array[String]): Unit = {
    val q1 = mutable.Queue(1,2,3,4,5)
    println(q1) //Queue(1, 2, 3, 4, 5)
    //添加
    q1.enqueue(6)
    println(q1)
    //添加
    q1.enqueue(7,8,9)
    println(q1)
    //移除
    println(q1.dequeue())
    println(q1)
    //移除第一个奇数
    println(q1.dequeueFirst(_%2!=0))
    //移除满足条件的所有
    println(q1.dequeueAll(_%2 == 0))
    println(q1)
  }
}
```
### Set集合 不包含重复元素
HashSet 唯一无序
LinkedHashSet 唯一有序
TreeSet 唯一，排序
```scala

//集合 Set
object Demo119 {

  def main(args: Array[String]): Unit = {
    //唯一，升序
    val s1 = SortedSet(1,12,3,41,5)
    println(s1)

    //唯一，无序
    val s2 = HashSet(1,12,3,41,5)
    println(s2)

    //唯一，有序，存入与取出一样的顺序
    val s3 = mutable.LinkedHashSet(1,12,3,41,5)
    println(s3)
  }
}
```
### Map 键值对
HashMap
SortedMap
ListMap
TreeMap
```scala
//集合 Map
object Demo120 {

  def main(args: Array[String]): Unit = {
    val m1 = Map("a" ->1 ,"b" -> 2 ,"c"->3)
    //遍历
    for((k,v) <- m1) println(k,v)
    m1.foreach(println(_))
    println(m1.filterKeys(_=="a")) //Map(a -> 1)
  }
}
```

统计字符个数
```scala
//集合 统计字符串中字符出现的次数
object Demo121 {

  def main(args: Array[String]): Unit = {
    println("please input :")
    val line = StdIn.readLine()
    val m1 = mutable.Map[Char,Int]()
    val chars = line.toCharArray
    for (k <- chars){
      if(!m1.contains(k)){
        m1 += (k ->1)
      }else{
        m1 +=(k -> (m1.getOrElse(k,1)+1))
      }
    }
    m1.foreach(println(_))
  }
}
```

### Actor并发编程 2.12之后的版本已经被移除，后续版本都是用AKKA来代替Actor
基于事件模型的并发机制，不共享数据，依赖消息传递的一种并发编程模式，避免资源争夺，死锁等问题
Actor.start() 类似于java的Thread.start()启动线程
自动执行act()方法，类似于java中Thread.run()方法的自动调用
向Actor发送消息，使用偏函数发送消息，（receive只能接收一次消息，可以通过while（true）死循环来进行持续消息发送）
发送消息可以使用异步不带返回消息，同步带返回消息，异步带返回消息的方式来发送消息
可以使用loop{react{}}来复用线程，实现消息循环接收
执行exit（）退出

### Akka并发编程框架
构建高并发，分布式和可扩展的基于事件驱动的应用工具包，使用scala库开发
提供异步非阻塞，高性能的事件驱动编程模型
Akka也是机遇Actor来实现
* ActorSystem：负责创建和监督Actor
* 实现Actor类实现receive方法，preStart（）,在Actor对象构建后执行，在Actor声明周期中仅执行一次
* 加载Actor
* 嗲用ActorSystem.actorOf加载Actor
Actor Path
可以访问本地Actor ： `akka://actorSystemname/user/actorname`
远程Actor `akka.tcp://my-sys@ip:port/user/Actorname`
简单消息发送实现：
```xml
<dependency>
            <groupId>com.typesafe.akka</groupId>
            <artifactId>akka-actor_2.12</artifactId>
            <version>2.8.0</version>
        </dependency>
```
```scala
//提交消息
case class SubmitTaskMessage(msg:String)
//提交成功消息
case class SuccessSubmitTaskMessage(msg:String)

//消息接收actor
object ReceiverActor extends Actor{
  override def receive: Receive = {
    case SubmitTaskMessage(msg) => {
      println(s"i am receiver, i get a message:${msg}")
      sender ! SuccessSubmitTaskMessage("this is a back message")
    }
  }
}
//消息发送actor
object SenderActor extends Actor{
  override def receive: Receive = {
    case "start" => {
      println("get main start message")
      //获取的actor的路径
      val receiverActor = context.actorSelection("akka://actorSystem/user/receiveActor")
      receiverActor ! SubmitTaskMessage("i am senderActor, i send a message to you ")

    }
    case SuccessSubmitTaskMessage(msg) =>println(s"senderActor get a back message:${msg}")
  }
}
//入口程序
object Entrance {
  def main(args: Array[String]): Unit = {

    //创建ActorSystem加载自定义的Actor对象管理他们
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())
    //被ActorSystem管理的自定义actor
    val senderActor = actorSystem.actorOf(Props(SenderActor), "senderActor")
    val receiveActor = actorSystem.actorOf(Props(ReceiverActor), "receiveActor")

    //4. 由actorSystem发送一个start消息
    senderActor ! "start"
  }
}
```
Akka定时任务 Sheduler.schedule()
```scala
object MainActor {
  object ReceiverActor extends Actor{
    override def receive: Receive = {

      case x => println(x)
    }
  }

  def main(args: Array[String]): Unit = {
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())
    val receiverActor = actorSystem.actorOf(Props(ReceiverActor), "receiverActor")
    import actorSystem.dispatcher
    import scala.concurrent.duration._
    //采用发送消息实现定时任务,延迟3s，每两秒发送一次消息
//    actorSystem.scheduler.schedule(3 seconds,2 seconds,receiverActor,"hello scala")
    //采用发送自定义消息，结合函数实现定时任务
//    actorSystem.scheduler.schedule(3 seconds,2 seconds)(receiverActor !"hello scala")

    //实际开发中使用这种方式
    actorSystem.scheduler.schedule(0 seconds,2 seconds){
      receiverActor ! "hello scala"
    }
  }
}
```
akka实现两个进程之间的通信
master：
```conf
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }
  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2554
    }
    artery {
          enabled = on
          canonical.hostname = "127.0.0.1"
          canonical.port = 10001
        }
  }

}
akka.actor.allow-java-serialization=on
```
```scala

object MasterActor extends Actor {
  override def receive: Receive = {

    case "connect" => {
      println("master get a message :connect")
      sender ! "success"
    }
    case x => println(s"other message:${x}")
  }
}
object MasterEntrance {

  def main(args: Array[String]): Unit = {
    val masterActorSystem = ActorSystem("masterActorSystem", ConfigFactory.load())
    val masterActor = masterActorSystem.actorOf(Props(MasterActor), "masterActor")
    masterActor ! "测试数据"
  }

}
```
worker
```conf
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }
  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2553
    }
    artery {
          enabled = on
          canonical.hostname = "127.0.0.1"
          canonical.port = 10000
        }
  }

}
akka.actor.allow-java-serialization=on
```
```scala
//actor及程序入口
/**
 * @Description: akka进程之间的通信，spark通信框架的底层实现
 * @CreateDate: Created in 2023/4/8 10:08 
 * @Author: lijie3
 */
object Entrance {
  def main(args: Array[String]): Unit = {
    val actorSystem = ActorSystem("actorSystem", ConfigFactory.load())
    val workActor = actorSystem.actorOf(Props(WorkActor), "workActor")

    workActor ! "setup"
  }
}

/**
 * @Description:
 * @CreateDate: Created in 2023/4/8 10:07 
 * @Author: lijie3
 */
object WorkActor extends Actor{
  override def receive: Receive = {
    case "setup" => {
      println(s"worker Actor get message:setup")

      val masterActor = context.system.actorSelection("akka://masterActorSystem@127.0.0.1:10001/user/masterActor")
      masterActor ! "connect"
    }
    case "success" => println(s"worker get messsage :success")
  }
}
```

### spark中的通信框架实现--使用akka实现worker的心跳检测demo
原理简介：
1. worker向master发送注册信息RegisterMessage
2. 注册成功后，master向worker发送SuccesRegisterMessage
3. worker向master发送HeartBeatMesssage发送心跳消息
4. master更新worker的更新时间
5. master定时检测worker的心跳时间是否超时，如果超时，则将worker从worker列表中移除
代码示例
消息实体
```scala
/**
 * worker信息
 * @param workId
 * @param cpu
 * @param mem
 * @param lastHeartbeatTime
 */
case class WorkerInfo(workId:String,cpu:Int,mem:Int,lastHeartbeatTime:Long)
//worker注册消息实体
case class WorkerRegisterMessage(workerId:String,cpu:Int,mem:Int)

//注册成功后消息回执消息样例
case class RegisterSuccessMessage()

//注册完成之后的心跳消息实体
case class WorkerHeartBeatMessage(workId:String,cpu:Int,mem:Int)
```
master代码
```conf
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }
  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2554
    }
    artery {
          enabled = on
          canonical.hostname = "127.0.0.1"
          canonical.port = 10001
        }
  }

}
akka.actor.allow-java-serialization=on

## 心跳检查时间间隔
master.check.hearbeat.internal = 6
## 心跳超时时间
master.check.hearbeat.timeout= 15

```
```scala
/**
 * @Description: master入口
 * @CreateDate: Created in 2023/4/8 11:28 
 * @Author: lijie3
 */
object Master {
  //akka address akka://masterActorSystem@127.0.0.1:10001
  def main(args: Array[String]): Unit = {
    val masterActorSystem = ActorSystem("masterActorSystem", ConfigFactory.load())
    val masterActor = masterActorSystem.actorOf(Props(MasterActor), "masterActor")
    masterActor ! "hello spark"

  }
}
import com.typesafe.config.{Config, ConfigFactory}

/**
 * @Description: 读取master配置文件
 * @CreateDate: Created in 2023/4/8 12:37 
 * @Author: lijie3
 */
object ConfigUtils {

  private val config: Config = ConfigFactory.load()
   val `master.check.hearbeat.internal`: Int = config.getInt("master.check.hearbeat.internal")
   val `master.check.hearbeat.timeout`: Int = config.getInt("master.check.hearbeat.timeout")

}

object MasterActor extends Actor{

  //map集合存储注册好的worker信息
  private val registeredWokerMap = Map[String,WorkerInfo]()


  override def preStart(): Unit = {
    //使用定时器检查worker状态
    import scala.concurrent.duration._
    import context.dispatcher
    context.system.scheduler.schedule(0 seconds,ConfigUtils.`master.check.hearbeat.internal` seconds){
      val timoutWorkerMap = registeredWokerMap.filter{
            //计算worker心跳是否超时
        keval => {
          val time = keval._2.lastHeartbeatTime
            //判断是否超时
          if(new Date().getTime - time > ConfigUtils.`master.check.hearbeat.timeout` * 1000) true else false
        }
      }
      if(!timoutWorkerMap.isEmpty){
        //超时移除
        registeredWokerMap --= timoutWorkerMap.keys
      }
      //获取还存活的Worker列表
      val workerList = registeredWokerMap.values.toList
      val sortedWorkerList = workerList.sortBy(_.mem).reverse
      println(s"sorted workers: ${sortedWorkerList}")
    }

  }

  override def receive: Receive = {
    //接收worker的注册信息
    case WorkerRegisterMessage(workerId,cpu,mem) =>{
      println(s"Master receive register message:${workerId},${cpu},${mem}")
      //注册
      registeredWokerMap += workerId-> WorkerInfo(workerId,cpu,mem,new Date().getTime)
      sender ! RegisterSuccessMessage
    }
     //接收心跳消息
    case WorkerHeartBeatMessage(workerId,cpu,mem) =>{
      println(s"Master receive get hearbeat info :${workerId},${cpu},${mem}")
      //更新心跳时间
      registeredWokerMap += workerId-> WorkerInfo(workerId,cpu,mem,new Date().getTime)
      println(registeredWokerMap)
    }

  }
}
```
worker代码
```conf
akka {
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
  }
  remote {
    netty.tcp {
      hostname = "127.0.0.1"
      port = 2553
    }
    artery {
          enabled = on
          canonical.hostname = "127.0.0.1"
          canonical.port = 10000
        }
  }
}
akka.actor.allow-java-serialization=on
## 5s发送一次心跳
worker.heartbeat.internal = 5
```
```scala
object ConfigUtils {
  //获取配置信息对象
  private val config: Config = ConfigFactory.load()
  //获取心跳间隔时间
  val `worker.heartbeat.internal`: Int = config.getInt("worker.heartbeat.internal")
}
object WokerActor extends Actor{

  private var masterActorRef:ActorSelection = _
  private var workerId:String = _
  private var cpu:Int = _
  private var mem:Int = _
  //模拟cpu和内存
  private var cpu_list = List(1,2,4,6,8)
  private var mem_list = List(512,1024,2048,4096)

  //启动后立即发送注册信息
  override def preStart(): Unit = {
    //获取masterActor引用
    masterActorRef = context.system.actorSelection("akka://masterActorSystem@127.0.0.1:10001/user/masterActor")
    workerId = UUID.randomUUID().toString
    val r = new Random()
    cpu = cpu_list(r.nextInt(cpu_list.size))
    mem = mem_list(r.nextInt(mem_list.size))
    //发送注册信息
    val registerMessage = WorkerRegisterMessage(workerId, cpu, mem)
    //发送消息
    masterActorRef !registerMessage
  }

  override def receive: Receive = {
    case RegisterSuccessMessage => println("workeractor register success")
    //定时给masteractor发送心跳消息
    import scala.concurrent.duration._
    import context.dispatcher

      context.system.scheduler.schedule(0 seconds,ConfigUtils.`worker.heartbeat.internal` seconds)(
        //发送心跳
        masterActorRef ! WorkerHeartBeatMessage(workerId,cpu, mem)
      )
  }
}
object Worker {
  //akka address akka://masterActorSystem@127.0.0.1:10001
  def main(args: Array[String]): Unit = {
    val workerActorSystem = ActorSystem("workerActorSystem", ConfigFactory.load())
    val workerActor = workerActorSystem.actorOf(Props(WokerActor), "workerActor")
    workerActor ! "hello spark client"
  }
}
```






























































