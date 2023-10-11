## jdk8的新特性



在 JDK 8（Java Development Kit 8）中引入了许多新特性和改进，以下是其中一些主要的新特性：

### Lambda 表达式： 
JDK 8 引入了函数式编程的概念，通过 Lambda 表达式可以更方便地传递匿名函数。这极大地简化了代码，并提供了更好的代码可读性。
lambda可分为四种：
可选类型声明：不需要声明参数类型，编译器可以统一识别参数值
可选的参数圆括号：一个参数无需定义圆括号，但多个参数需要定义圆括号
可选的大括号：如果主体包含了一个语句，就不需要使用大括号
可选的返回关键字：如果主体只有一个表达式返回值则编译器会自动返回值，大括号需要指定明表达式返回了一个数值
```java
 new Thread(()->{
            //主线程的修改对t1线程不可见
            while (run){
            }
        }).start();
```
Lambda 表达式可以引用类成员和局部变量，但是会将这些变量隐式得转换成final
```java
String separator = ",";
Arrays.asList( "a", "b", "c" ).forEach(
    ( String e ) -> System.out.print( e + separator ) );
```
Lambda 表达式的局部变量可以不用声明为final，但是必须不可被后面的代码修改
```java
int num = 1;
Arrays.asList(1,2,3,4).forEach(e -> System.out.println(num + e));
num =2; //报错
```
### Stream API：
 引入了 Stream API，使得处理集合数据更加简单和灵活。通过使用 Stream，可以进行各种数据操作，如过滤、映射、归约等。
stream包含中间操作结束操作
* 中间操作
  * 无状态： map、filter、mapToInt、mapToLong、mapToDouble、flatMap、flatMapToInt、flatMapToLong、flatMapToDouble、peek、unordered
  * 有状态：distinct、sorted、limit、skip
* 结束操作
  * 短路操作：anyMatch、allMatch、findFirst、findAny
  * 非短路操作：forEach、foreachOrdered、toArray、reduce、collect、max、min、count

#### 流的创建
使用Collection创建
```java
List<String> list = new ArrayList<>();
Stream<String> stream = list.stream(); //获取一个顺序流
Stream<String> parallelStream = list.parallelStream(); //获取一个并行流
```
使用Arrays.stream()
```java
Integer[] nums = new Integer[10];
Stream<Integer> stream = Arrays.stream(nums);
```
使用Stream中的静态方法：of()、iterate()、generate()
```java
Stream<Integer> stream = Stream.of(1,2,3,4,5,6);
// 第一个参数seed作为起始入参，x+2 相当于 从 0+2 到2+4 到4+2
Stream<Integer> stream2 = Stream.iterate(0, (x) -> x + 2).limit(6);
stream2.forEach(System.out::println); // 0 2 4 6 8 10
  
Stream<Double> stream3 = Stream.generate(Math::random).limit(2);
stream3.forEach(System.out::println);
```
使用BufferedReader.lines()
```java
BufferedReader reader = new BufferedReader(new FileReader("F:\\test_stream.txt"));
Stream<String> lineStream = reader.lines();
lineStream.forEach(System.out::println);
```
使用Pattern.splitAsStream()
```java
Pattern pattern = Pattern.compile(",");
Stream<String> stringStream = pattern.splitAsStream("a,b,c,d");
stringStream.forEach(System.out::println);
```

#### 流的中间操作
　　filter：过滤流中的某些元素
　　limit(n)：获取n个元素
　　skip(n)：跳过n元素，配合limit(n)可实现分页
　　distinct：通过流中元素的 hashCode() 和 equals() 去除重复元素
```java
Stream<Integer> stream = Stream.of(6, 4, 6, 7, 3, 9, 8, 10, 12, 14, 14);
  
Stream<Integer> newStream = stream.filter(s -> s > 5) //6 6 7 9 8 10 12 14 14
.distinct() //6 7 9 8 10 12 14
.skip(2) //9 8 10 12 14
.limit(2); //9 8
newStream.forEach(System.out::println);
```
　　map：接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
　　flatMap：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流。
```java
Stream<String> s3 = list.stream().flatMap(s -> {
//将每个元素转换成一个stream
String[] split = s.split(",");
Stream<String> s2 = Arrays.stream(split);
return s2;
});
```
sorted()：自然排序，流中元素需实现Comparable接口
sorted(Comparator com)：定制排序，自定义Comparator排序器  
```java
List<String> list = Arrays.asList("aa", "ff", "dd");
//String 类自身已实现Compareable接口
list.stream().sorted().forEach(System.out::println);// aa dd ff
  
Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
Student s3 = new Student("aa", 30);
Student s4 = new Student("dd", 40);
List<Student> studentList = Arrays.asList(s1, s2, s3, s4);
//自定义排序：先按姓名升序，姓名相同则按年龄升序
studentList.stream().sorted(
(o1, o2) -> {
if (o1.getName().equals(o2.getName())) {
return o1.getAge() - o2.getAge();
} else {
return o1.getName().compareTo(o2.getName());
}
}
).forEach(System.out::println);　
```
peek消费：如同于map，能得到流中的每一个元素。但map接收的是一个Function表达式，有返回值；而peek接收的是Consumer表达式，没有返回值。
```java
Student s1 = new Student("aa", 10);
Student s2 = new Student("bb", 20);
List<Student> studentList = Arrays.asList(s1, s2);
  
studentList.stream()
.peek(o -> o.setAge(100))
.forEach(System.out::println);
```
#### 终止操作符
匹配、聚合操作：
　　allMatch：接收一个 Predicate 函数，当流中每个元素都符合该断言时才返回true，否则返回false
　　noneMatch：接收一个 Predicate 函数，当流中每个元素都不符合该断言时才返回true，否则返回false
　　anyMatch：接收一个 Predicate 函数，只要流中有一个元素满足该断言则返回true，否则返回false
　　findFirst：返回流中第一个元素
　　findAny：返回流中的任意元素
　　count：返回流中元素的总个数
　　max：返回流中元素最大值
　　min：返回流中元素最小值
```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);
  
boolean allMatch = list.stream().allMatch(e -> e > 10); //false
boolean noneMatch = list.stream().noneMatch(e -> e > 10); //true
boolean anyMatch = list.stream().anyMatch(e -> e > 4); //true
  
Integer findFirst = list.stream().findFirst().get(); //1
Integer findAny = list.stream().findAny().get(); //1
  
long count = list.stream().count(); //5
Integer max = list.stream().max(Integer::compareTo).get(); //5
Integer min = list.stream().min(Integer::compareTo).get(); //1　
```
规约操作
　　Optional reduce(BinaryOperator accumulator)：第一次执行时，accumulator函数的第一个参数为流中的第一个元素，第二个参数为流中元素的第二个元素；第二次执行时，第一个参数为第一次函数执行的结果，第二个参数为流中的第三个元素；依次类推。
　　T reduce(T identity, BinaryOperator accumulator)：流程跟上面一样，只是第一次执行时，accumulator函数的第一个参数为identity，而第二个参数为流中的第一个元素。
　　U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator combiner)：在串行流(stream)中，该方法跟第二个方法一样，即第三个参数combiner不会起作用。在并行流(parallelStream)中,我们知道流被fork join出多个线程进行执行，此时每个线程的执行流程就跟第二个方法reduce(identity,accumulator)一样，而第三个参数combiner函数，则是将每个线程的执行结果当成一个新的流，然后使用第一个方法reduce(accumulator)流程进行规约。
```java
//经过测试，当元素个数小于24时，并行时线程数等于元素个数，当大于等于24时，并行时线程数为16
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24);
  
Integer v = list.stream().reduce((x1, x2) -> x1 + x2).get();
System.out.println(v); // 300
  
Integer v1 = list.stream().reduce(10, (x1, x2) -> x1 + x2);
System.out.println(v1); //310
  
Integer v2 = list.stream().reduce(0,
(x1, x2) -> {
System.out.println("stream accumulator: x1:" + x1 + " x2:" + x2);
return x1 - x2;
},
(x1, x2) -> {
System.out.println("stream combiner: x1:" + x1 + " x2:" + x2);
return x1 * x2;
});
System.out.println(v2); // -300
  
Integer v3 = list.parallelStream().reduce(0,
(x1, x2) -> {
System.out.println("parallelStream accumulator: x1:" + x1 + " x2:" + x2);
return x1 - x2;
},
(x1, x2) -> {
System.out.println("parallelStream combiner: x1:" + x1 + " x2:" + x2);
return x1 * x2;
});
System.out.println(v3); //197474048
```
收集操作
　　collect：接收一个Collector实例，将流中元素收集成另外一个数据结构。
　　Collector<T, A, R> 是一个接口，有以下5个抽象方法：
　　Supplier<A> supplier()：创建一个结果容器A
　　BiConsumer<A, T> accumulator()：消费型接口，第一个参数为容器A，第二个参数为流中元素T。
　　BinaryOperator<A> combiner()：函数接口，该参数的作用跟上一个方法(reduce)中的combiner参数一样，将并行流中各 个子进程的运行结果(accumulator函数操作后的容器A)进行合并。
　　Function<A, R> finisher()：函数式接口，参数为：容器A，返回类型为：collect方法最终想要的结果R。
　　Set<Characteristics> characteristics()：返回一个不可变的Set集合，用来表明该Collector的特征。有以下三个特征：
　　CONCURRENT：表示此收集器支持并发。（官方文档还有其他描述，暂时没去探索，故不作过多翻译）
　　UNORDERED：表示该收集操作不会保留流中元素原有的顺序。
　　IDENTITY_FINISH：表示finisher参数只是标识而已，可忽略。
```java
        Student s1 = new Student("aa", 10, 1);
        Student s2 = new Student("bb", 20, 2);
        Student s3 = new Student("cc", 10, 3);
        List<Student> list = Arrays.asList(s1, s2, s3);

//装成list
        List<Integer> ageList = list.stream().map(Student::getAge).collect(Collectors.toList()); // [10, 20, 10]

//转成set
        Set<Integer> ageSet = list.stream().map(Student::getAge).collect(Collectors.toSet()); // [20, 10]

//转成map,注:key不能相同，否则报错
        Map<String, Integer> studentMap = list.stream().collect(Collectors.toMap(Student::getName, Student::getAge)); // {cc=10, bb=20, aa=10}

//字符串分隔符连接
        String joinName = list.stream().map(Student::getName).collect(Collectors.joining(",", "(", ")")); // (aa,bb,cc)

//聚合操作
//1.学生总数
        Long count = list.stream().collect(Collectors.counting()); // 3
//2.最大年龄 (最小的minBy同理)
        Integer maxAge = list.stream().map(Student::getAge).collect(Collectors.maxBy(Integer::compare)).get(); // 20
//3.所有人的年龄
        Integer sumAge = list.stream().collect(Collectors.summingInt(Student::getAge)); // 40
//4.平均年龄
        Double averageAge = list.stream().collect(Collectors.averagingDouble(Student::getAge)); // 13.333333333333334
// 带上以上所有方法
        DoubleSummaryStatistics statistics = list.stream().collect(Collectors.summarizingDouble(Student::getAge));
        System.out.println("count:" + statistics.getCount() + ",max:" + statistics.getMax() + ",sum:" + statistics.getSum() + ",average:" + statistics.getAverage());
        //分组
        Map<Integer, List<Student>> ageMap = list.stream().collect(Collectors.groupingBy(Student::getAge));
        //多重分组,先根据类型分再根据年龄分
        Map<Integer, Map<Integer, List<Student>>> typeAgeMap = list.stream().collect(Collectors.groupingBy(Student::getType, Collectors.groupingBy(Student::getAge)));

//分区
//分成两部分，一部分大于10岁，一部分小于等于10岁
        Map<Boolean, List<Student>> partMap = list.stream().collect(Collectors.partitioningBy(v -> v.getAge() > 10));

//规约
        Integer allAge = list.stream().map(Student::getAge).collect(Collectors.reducing(Integer::sum)).get(); //40　
```

### 新的日期和时间
 API： JDK 8 替换了旧的日期和时间类，引入了新的日期和时间 API。这个 API 提供了更好的日期和时间处理能力，使得处理日期、时间和时间间隔更加方便。
 在旧版的 Java 中，日期时间 API 存在诸多问题，例如：

* 非线程安全：java.util.Date 是非线程安全的，所有的日期类都是可变的，这是Java日期类最大的问题之一。
* 设计很差：Java的日期/时间类的定义并不一致，在java.util和java.sql的包中都有日期类，此外用于格式化和解析的类被定义在java.text包中。java.util.* Date同时包含日期和时间，而java.sql.Date仅包含日期，将其纳入java.sql包并不合理。另外这两个类都有相同的名字，这本身就是一个非常糟糕的设计。
* 时区处理麻烦：日期类并不提供国际化，没有时区支持，因此 Java 引入了java.util.Calendar和java.util.TimeZone类，但他们同样存在上述所有的问题。

新版本API
Clock类
```java
  //代替System.currentTimeMillis()和TimeZone.getDefault()
        final Clock clock = Clock.systemUTC();
        System.out.println( clock.instant() );
        System.out.println( clock.millis() );
```
//日期、时间、日期时间、时区、duration工具类
```java
    public static void test1(){
        //代替System.currentTimeMillis()和TimeZone.getDefault()
        final Clock clock = Clock.systemUTC();
        System.out.println( clock.instant() );
        System.out.println( clock.millis() );
        //2021-02-24T12:24:54.678Z
        //1614169494678
    }
    //LocalDate、LocalTime 和 LocalDateTime类
    public static void test2(){
//        Clock clock = Clock.systemUTC();
        Clock clock = Clock.systemDefaultZone();
        //获取当前日期
        final LocalDate date = LocalDate.now();
//获取指定时钟的日期
        final LocalDate dateFromClock = LocalDate.now( clock );

        System.out.println( date );
        System.out.println( dateFromClock );
        //2023-08-20
        //2023-08-20
        //获取当前时间
        final LocalTime time = LocalTime.now();
//获取指定时钟的时间
        final LocalTime timeFromClock = LocalTime.now( clock );

        System.out.println( time );
        System.out.println( timeFromClock );
        //17:09:47.472
        //09:09:47.472

        //获取当前日期时间
        final LocalDateTime datetime = LocalDateTime.now();
//获取指定时钟的日期时间
        final LocalDateTime datetimeFromClock = LocalDateTime.now( clock );

        System.out.println( datetime );
        System.out.println( datetimeFromClock );
        //2023-08-20T17:09:47.472
        //2023-08-20T09:09:47.472
        //时区相关信息
        // 获取当前时间日期
        final ZonedDateTime zonedDatetime = ZonedDateTime.now();
//获取指定时钟的日期时间
        final ZonedDateTime zonedDatetimeFromClock = ZonedDateTime.now( clock );
//获取纽约时区的当前时间日期
        final ZonedDateTime zonedDatetimeFromZone = ZonedDateTime.now( ZoneId.of("America/New_York") );

        System.out.println( zonedDatetime );
        System.out.println( zonedDatetimeFromClock );
        System.out.println( zonedDatetimeFromZone );
        //2023-08-20T17:16:45.806+08:00[Asia/Shanghai]
        //2023-08-20T17:16:45.806+08:00[Asia/Shanghai]
        //2023-08-20T05:16:45.810-04:00[America/New_York]
        //Duration类，它持有的时间精确到秒和纳秒。利用它我们可以很容易得计算两个日期之间的不同，实例如下：
        final LocalDateTime from = LocalDateTime.of( 2020, Month.APRIL, 16, 0, 0, 0 );
        final LocalDateTime to = LocalDateTime.of( 2021, Month.APRIL, 16, 23, 59, 59 );

//获取时间差
        final Duration duration = Duration.between( from, to );
        System.out.println( "Duration in days: " + duration.toDays() );
        System.out.println( "Duration in hours: " + duration.toHours() );
        //Duration in days: 365
        //Duration in hours: 8783
    }
```


### 默认方法（Default Methods）
 接口现在可以包含默认方法的实现。这使得在已经存在的接口中可以添加新的方法，而不会破坏实现这个接口的类。
 默认方法使得开发者可以在不破坏二进制兼容性的前提下，往现存接口中添加新的方法，即不强制那些实现了该接口的类也同时实现这个新加的方法。
 ```java
public class Tester {
   public static void main(String args[]){
      Vehicle vehicle = new Car();
      vehicle.print();
   }
}

interface Vehicle {
   default void print(){
      System.out.println("我是一辆车!");
   }

   static void blowHorn(){
      System.out.println("按喇叭!!!");
   }
}

interface FourWheeler {
   default void print(){
      System.out.println("我是一辆四轮车!");
   }
}

class Car implements Vehicle, FourWheeler {
   public void print(){
      Vehicle.super.print();
      FourWheeler.super.print();
      Vehicle.blowHorn();
      System.out.println("我是一辆汽车!");
   }
}
 ```


### 方法引用 
方法引用允许你通过方法的名字来引用它，而不是调用它。这在与 Lambda 表达式一起使用时特别有用。
方法引用使用一对冒号::，通过方法的名字来指向一个方法。
```java
Map<Integer, List<Student>> ageMap = list.stream().collect(Collectors.groupingBy(Student::getAge));
//构造器应用
final Car car = Car.create( Car::new );
final List< Car > cars = Arrays.asList( car );
```
### 函数式接口
函数接口指的是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口，这样的接口可以隐式转换为 Lambda 表达式。
```java
@FunctionalInterface
public interface GreetingService {

    void sayMessage(String message);
}
//与lambda表达式进行结合
GreetingService greetService = message -> System.out.println("Hello " + message);
greetService.sayMessage("world");
```

### Optional 类
避免空指针异常的工具类
```java
public class OptionalTester {

    public static void main(String[] args) {
        OptionalTester tester = new OptionalTester();
        Integer value1 = null;
        Integer value2 = new Integer(10);

        // Optional.ofNullable - 允许传递为 null 参数
        Optional<Integer> a = Optional.ofNullable(value1);

        // Optional.of - 如果传递的参数是 null，抛出异常 NullPointerException
        Optional<Integer> b = Optional.of(value2);
        System.out.println(tester.sum(a,b));
    }

    public Integer sum(Optional<Integer> a, Optional<Integer> b){

        // Optional.isPresent - 判断值是否存在

        System.out.println("第一个参数值存在: " + a.isPresent());
        System.out.println("第二个参数值存在: " + b.isPresent());

        // Optional.orElse - 如果值存在，返回它，否则返回默认值
        Integer value1 = a.orElse(new Integer(0));

        //Optional.get - 获取值，值需要存在
        Integer value2 = b.get();
        return value1 + value2;
    }
}
```
### Base64 编码
Java 8中，Base64 编码已经成为 Java 类库的标准
```java
        final String text = "Base64 finally in Java 8!";
        final String encoded = Base64.getEncoder().encodeToString( text.getBytes( StandardCharsets.UTF_8 ) );
        System.out.println( encoded );
        final String decoded = new String(Base64.getDecoder().decode( encoded ), StandardCharsets.UTF_8 );
        System.out.println( decoded );
```

重复注解（Repeatable Annotations）： 允许同一个注解在同一地方重复使用，而不需要特殊的容器注解。

### Nashorn JavaScript 引擎
替代了旧的 Rhino 引擎，提供了更好的性能和支持。
```java
        ScriptEngineManager scriptEngineManager = new ScriptEngineManager();
        ScriptEngine nashorn = scriptEngineManager.getEngineByName("nashorn");

        String name = "Hello World";
        try {
            nashorn.eval("print('" + name + "')");
        }catch(ScriptException e){
            System.out.println("执行脚本错误: "+ e.getMessage());
        }
```


新的编译工具： JDK 8 引入了 javac 编译器的新版本，支持新的语言特性。


## java9 新特性
### jshell
有的时候我们只是想写一段简单的代码，例如HelloWorld，按照以前的方式，还需要自己创建java文件，创建class，编写main方法，但实际上里面的代码其实就是一个打印语句，此时还是比较麻烦的。在jdk9中新增了jshell工具，可以帮助我们快速的运行一些简单的代码。
System.out.println("HelloWorld");
此时可以直接看到命令提示符中打印了HelloWorld。

### 接口私有方法
新增了接口私有方法，我们可以在接口中声明private修饰的方法了，这样的话，接口越来越像抽象类了
```java
public interface MyInterface {
    //定义私有方法
    private void m1() {
        System.out.println("123");
    }

    //default中调用
    default void m2() {
        m1();
    }
}
```
### 改进的try with resource
jdk9中我们可以将这些资源对象的创建放到外面，然后将需要关闭的对象放到try后面的小括号中即可
```java
        //jdk8以前
        try (FileInputStream fileInputStream = new FileInputStream("");
             FileOutputStream fileOutputStream = new FileOutputStream("")) {

        } catch (IOException e) {
            e.printStackTrace();
        }

        //jdk9
        FileInputStream fis = new FileInputStream("abc.txt");
        FileOutputStream fos = new FileOutputStream("def.txt");
        //多资源用分号隔开
        try (fis; fos) {
                } catch (IOException e) {
            e.printStackTrace();
        }
```

### 不能使用下划线命名变量
下面语句在jdk9之前可以正常编译通过,但是在jdk9（含）之后编译报错，在后面的版本中会将下划线作为关键字来使用
```java
String _ = "monkey1024";
```

### String字符串的变化
对string优化的目的：减少string对内存的消耗
写程序的时候会经常用到String字符串，在以前的版本中String内部使用了char数组存储，对于使用英语的人来说，字符用一个字节就能存储，因此在jdk9中将String内部的char数组改成了byte数组，这样就节省了一半的内存占用。String中增加了一个判断，倘若字符超过1个字节的话，会把byte数组的长度改为两倍char数组的长度，用两个字节存放一个char。在获取String长度的时候，其源码中有向右移动1位的操作（即除以2），这样就解决了上面扩容2倍之后长度不正确的问题

### 模块化
Java 9引入了模块系统（Project Jigsaw），允许开发者将代码划分为模块，以提高可维护性和性能。

## jdk10的新变化

局部变量类型推断
在jdk10中前面的类型都可以使用var来代替，JVM会自动推断该变量是什么类型的，例如可以这样写：
```java
    var newName = "jack";
    var newAge = 10;
    var newMoney = 88888888L;
    var newObj = new Object();
```
var的使用是有限制的，仅适用于局部变量，增强for循环的索引，以及普通for循环的本地变量；它不能使用于方法形参，构造方法形参，方法返回类型等。

### jdk11新特性
### 直接运行
在java 11中，我们可以这样直接运行，省略掉javac的编译过程
```shell
java HelloWorld.java
```

### String新增方法
strip方法
可以去除首尾空格，与之前的trim的区别是还可以去除unicode编码的空白字符
```java
char c = '\u2000';//Unicdoe空白字符
String str = c + "abc" + c;
System.out.println(str.strip());
System.out.println(str.trim());

System.out.println(str.stripLeading());//去除前面的空格
System.out.println(str.stripTrailing());//去除后面的空格
```
isBlank方法
判断字符串长度是否为0，或者是否是空格，制表符等其他空白字符
```java
String str = " ";
System.out.println(str.isBlank());
```
repeat方法
字符串重复的次数
```java
 String str = "monkey ";
 System.out.println(str.repeat(4));
 //monkey monkey monkey monkey 
```
lambda表达式中的变量类型推断
```java
    @FunctionalInterface
    public interface MyInterface {
        void m1(String a, int b);
    }
      MyInterface mi = (var a,var b)->{
       System.out.println(a);
       System.out.println(b);
   };

   mi.m1("monkey",1024);
```

## jdk12的新特性
可以省略全部的break和部分case，不会出现case穿透
```java
int month = 3;
    switch (month) {
        case 3,4,5 -> System.out.println("spring");
        case 6,7,8 -> System.out.println("summer");
        case 9,10,11 -> System.out.println("autumn");
        case 12, 1,2 -> System.out.println("winter");
        default -> System.out.println("wrong");
    }
```

### jdk13的变化
对switch语句又进行了升级，可以switch的获取返回值
```java
int month = 3;
   String result = switch (month) {
        case 3,4,5 -> "spring";
        case 6,7,8 -> "summer";
        case 9,10,11 -> "autumn";
        case 12, 1,2 -> "winter";
        default -> "wrong";
    };

    System.out.println(result);
```
文本块的变化
```java
String s = """
            Hello
            World
            Learn
            Java
           """;
  System.out.println(s);
```

### jdk14的变化
instanceof模式匹配
```java
 //jdk14新特性  不用再强制转换了
        //这里相当于是将obj强制为Integer之后赋值给i了
        if(obj instanceof Integer i){
            int result = i + 10;
            System.out.println(i);
        }else{
            //作用域问题，这里是无法访问i的
        }
```
