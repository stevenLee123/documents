1. RocketMQ 一台机器消费特别快一台特别慢如何解决这个问题
出现原因：
机器性能差异导致，消费者之间的消费逻辑存在差异导致消费者消费能力的不同
实际负载均衡的问题，采取策略：
* 优化消费者消费逻辑，提高消费者的消费效率
* 采用异步批量处理，使用异步处理、批量消费的方式实现提高消费速度
* 升级硬件策略，增加CPU、内存，提高消费者处理速度
* 调整消费策略，将更多的消息分配给消费能力强的消费者
* 扩展消费者数量，来分摊消费者的压力

### java的特点
* 一次编译，处处执行
* 面向对象
* 语言特性包含了泛型、lambda、等特性，基础类库中包含了集合、IO/NIO、网络、兵法、安全等基础类库
* java的类加载过程包括了类加载、链接（验证、准备、解析）、初始化三个过程
* java无需像C++那样需要程序员手动回收垃圾，jvm内部提供了应对不同场景的垃圾回收器，大大简化了内存管理的难度
* jvm是一个强大的平台，不仅只有java语言可以运行在jvm上，只要是符合规范的字节码都可以运行在jvm平台上，如scala等
* java提供了相对完善的异常捕获机制，一定程度上保证了程序的可靠性

### java是解释执行吗
java源代码首先被javac编译成字节码，然后在运行时jvm内嵌的解释器将字节码转换成最终的机器码，从这个层面上看java是解释执行的语言
但是在hotspot jvm中提供了JIT（just in time）编译器，能在运行时将热点代码编译成机器码，这种情况属于编译执行

### java中Exception 和Error的区别
Exception 和Error都继承自Throwable类，java中只有Throwable类型的实例才能被捕获（catch）或抛出（throw），catch/throw是java的异常处理机制
* Exception 是程序正常运行中可以预料的意外情况，可以被捕获或抛出处理
  * Exception 又分为检查异常（checked） 和非检查异常（unchecked）
  * 检查异常必须在源代码中进行显式的捕获处理，这是编译期进行检查的
  * 非检查异常都是RunTimeException的子类，可以不进行捕获或抛出，在编译期不进行强制要求
* Error是指在正常情况下不大可能出现的情况，比如jvm自身处于非正常、不可恢复的情况，出现Error时不需要进行捕获，常见的如OutOfMemoryError之类，都是Error的子类

### 什么情况下finally语句块不会被执行
* 当try块中发生了System.exit()或者jvm崩溃时，finally语句快不会被执行
* 当try语句块中出现了死循环，finally语句块也不会被执行
* 当try中发生了线程死锁时，finally语句块也不会被执行

### final、finally、finalize的区别
* final是java中的变量、方法、类的限定词，final修饰的类不可继承扩展，final修饰的方法不可以被重写，final修饰的变量不可被改变
* finally时java中保证重点代码一定被执行的一种机制，如用finally块关闭IO、释放锁、关闭数据库连接等
* finalize是Object的一个方法，用来保证对象被垃圾回收前完成特定的资源回收，finalize方法的重写在垃圾回收阶段回导致垃圾回收效率低下，finalize在jdk9中已经被标记为deprecated，在实际代码中应该尽量避免使用
  
### java中的四种引用的区别及使用场景
强引用： 普通对象引用，只要还有强引用指向对象，表示对象还存活，垃圾回收器不会回收该对象
软引用（SoftReference）：  相对于强引用弱化一些的引用，只有在jvm认为内存不足时才会尝试回收软引用对象，jvm会保证在出现OOM之前去回收弱引用对象
举例：
```java
public class SoftReferenceExample {
    public static void main(String[] args) {
        // 创建一个字符串对象
        String data = new String("Hello, World!");
        
        // 创建一个软引用对象
        SoftReference<String> softRef = new SoftReference<>(data);
        
        // 可以通过get()方法获取软引用指向的对象
        System.out.println("Soft Reference: " + softRef.get());
        
        // 释放强引用
        data = null;
        
        // 手动触发垃圾回收
        System.gc();
        
        // 再次尝试获取软引用指向的对象,返回hello world，软引用对象只有在jvm回收强引用后认为内存不足才会被回收
        System.out.println("Soft Reference after GC: " + softRef.get());
    }
}
```
软引用在java中适合用于需要缓存数据的场景，能在一定程度上避免OOM
弱引用（WeakReference）：一种比软引用更弱的引用类型，弱引用在垃圾回收时，无论内存是否充足，都会被回收。弱引用适合用于一些临时性的引用场景，用于存储内存敏感的缓存数据
举例
```
public class WeakReferenceTest {
    public static void main(String[] args) throws InterruptedException {
        Map<String, WeakReference<String>> cache = new HashMap<>();

        // 添加数据到缓存
        String key = "key1";
        String value = "value1";
        cache.put(key, new WeakReference<>(value));

        // 从缓存中获取数据
        WeakReference<String> reference = cache.get(key);
        if (reference != null) {
            String cachedValue = reference.get();
            System.out.println("Cached Value: " + cachedValue);
        } else {
            System.out.println("Cache miss");
        }

        // 清除强引用
        value = null;

        // 手动触发垃圾回收
        System.gc();
        Thread.sleep(20000);

        // 再次尝试获取数据,当垃圾回收真正执行时，弱引用一定会被回收
        WeakReference<String> referenceAfterGC = cache.get(key);
        if (referenceAfterGC != null) {
            String cachedValueAfterGC = referenceAfterGC.get();
            System.out.println("Cached Value after GC: " + cachedValueAfterGC);
        } else {
            System.out.println("Cache miss after GC");
        }
    }
}
```
弱引用使用场景：
* 缓存数据：缓存对象不再被强引用持有时，可以被垃圾回收
* 监听器：在事件监听器中，有时需要持有一个对象的引用，但不希望该引用影响对象的生命周期。使用弱引用可以避免内存泄漏，当监听器不再被使用时，可以被垃圾回收器回收。
* 解决循环引用问题：当对象之间存在循环引用时，通过使用弱引用，可以打破循环引用，帮助垃圾回收器更好地回收对象。
幻象引用：最弱的一种引用关系，它不会决定对象的生存周期，也无法通过它来获取对象实例。幻象引用的主要作用是能通知你某个对象已经被收集器回收了。用于在垃圾回收过程中跟踪对象的可达性。


1. 亿级别的数据查询uid=400 的数据库快速查询
* 尝试分表，按照uid进行分表
* 使用ES搜索引擎，
* 数据冷热分离，将热数据存储在SSD上
* 使用数据仓库和OLAP系统，如clickhouse
  
1. 线程池线程出现异常时如何处理
需要了解：   
* 线程的生命周期
* 异常的传播方式
* 如何优雅的终止一个线程
处理方式：
* 在传递的任务中处理异常
* 使用future来获取异常结果
* 自定义ThreadFactory设置UncaughtExceptionHendler来处理线程异常，为每个线程创建一个异常处理Handler


4. 短链接的实现
* 实现原理：将短链接链接成长链接，通过hash生成映射长链的hash码

5. 实现序列化和反序列化为什么要实现Serializable接口
java中实现序列化的操作：
* 实现Serialiazable接口
* 添加static final long serialVersionUID属性
Serialiazable是java的一个标记接口，当一个类实现了这个接口时，可以被序列化为字节流，也可以从字节流反序列化成对象
* 确保可序列化对象才能被序列化
* 规范类的行为，表示该类可以被序列化

6. mysql数据字段为什么要设计成not null
数据完整性：将字段设计成not null可以确保数据完整性
提高查询性能：设计成not null的字段不需要额外处理空值的判断条件
开发的友好性：代码开发中不需要考虑null值的情况
数据一致性的约束： 在数据库层面强制实施数据一致性的约束，避免应用程序忽略处理null值导致的问题

7. redis的key过期了内存没有被释放
redis的key清除方式有定期删除和过期删除两种方式

8. spring 加载bean的方式
* 使用@Component及其衍生注解标记一个bean，通过@ComponentScan来扫描家在被标记的bean
* 使用xml配置文件配置一个<Bean>
* 使用@Configuration在配置类中的方法上通过@Bean来指定方法的返回值为一个bean
* 使用@Import 加载Configuration、普通bean以及ImportSelector、BeanDefinitionRegistrar(这两个类可以通过SPI的方式实现bean的注册、加载)

9. limit 50，10 和limit 5000000,10有区别吗
分页查询起始值影响了查询效率，当起始查询很大且没有使用索引时，需要扫描很多条数据才能定位到需要返回的数据

10. 高度为3的B+树能存储多少数据
B+树是一颗多路平衡树，通过减少非叶子结点的存储数据量以及增加树的分支数量，降低树的高度从而减少磁盘IO的次数来提高数据检索性能
数据页默认大小时16K
非叶子节点存储索引值和页的偏移量
叶子节点存储完整的每一行记录
一行数据1k，一页存储16条，一个主键id8个字节，指针大小6个字节，每个数据页存储16384/（8+6） = 1170个指针
高度为2的树可以存储1170 * 16 = 18720条数据
高度为3的树可以存储1170 * 16 * 16 = 2000万

11. ArrayList和LinkedList的区别
* 内部实现：
  * ArrayList使用数组实现，支持快速访问，插入数据涉及到数据移动，可能会比较低效
  * LinkedList使用双向链表实现，插入和删除操作比较高效
* 数据访问时间复杂度，ArrayList访问效率高，O（1）,LinkedList时间复杂度为O（n）
* ArrayList占用空间时连续的，LinkedList通过链表连接元素每个元素包含前后节点的引用

12. redis集群的最大槽数为16384个
hash slot 数量最为16384，这是综合以下几个因素考虑的：
* 对网络通信开销的平衡，CRC16算法的hash值是16位，可得到2的16次方的值，会导致每个节点维护的配置信息占用8Kb，心跳包数据量过大导致网络开销过大
* 集群规模限制，redis cluster不太可能扩展到1000个主节点
* 保证每个master都有足够的插槽数量

13. 什么是微服务
* 结构风格
* 使用轻量级的方式（http、rpc）进行通信

14. JVM的三色标记法
CMS、G1都使用三色标记法，尽量保证STW的时间短一点
* 白色：还没被垃圾回收器扫描的对象
* 黑色：已经被垃圾回收器扫描且对象和应用的其他对象都是存活的
* 灰色： 已经被垃圾回收器扫描过，但是对象引用的其他对象还没被扫描
扫描流程：
从根节点遍历与根节点直接相连的对象，把直接引用的对象标记为灰色
判断灰色对象是否存在子引用，不存在直接标记为黑色
存在则把子引用对象标记为灰色，按照这个步骤推导，直到灰色对象全部变成黑色
处于白色的对象可以直接回收


15. skipList

16. redis的哨兵选举算法是如何实现的
master节点出现故障时，哨兵会检测到master节点的心跳断开

17. 内存泄漏时有什么解决方案
内存泄漏是在程序运行由于某些原因导致不需要的对象没有被垃圾回收占用JVM内存空间，导致程序内存占用越来越大导致OOM的错误，或是频繁的FUll GC，内存占用量过大无法释放等问题
排查：
老年代是否一直增长
FullGC卡顿，FullGC 频繁
年轻代的内存一直无法释放等问题都可能是内存泄漏导致的
使用jstat命令查看虚拟机中各个内存的使用情况，使用dump工具将当前内存dump下来，使用mat（memory analysis tools）工具分析内存，dump文件太大时，使用轻量级的在线分析工具jmap进行分析，然后定位到有问题的类，根据分析结果找到代码优化
一般情况下的问题：
循环引用
内存泄漏对象没有被销毁
动态分配内存后没有被释放
长期持有对象引用
IO等资源未被关闭

18. GC年龄为什么要设置成15次
尽可能减少移动到老年代的对象数量，最大值是15，不能超过15（因为只使用4个bit来存储）

19. mysql索引什么情况下会失效
* 没有使用where条件进行索引过滤
* 对索引列进行函数操作，字符串操作
* 对索引列进行类型转化，类型不匹配会导致索引失效
* 查询的like条件以通配符'%'开头
* or条件查询，or条件中的条件都不涉及到索引列，mysql无法使用索引
* 查询条件涉及到大量数据，如in查询后面有很多过滤数据

20. 重写equals方法为什么要重写hashcode
在散列集合中会大量使用hashTable、HashMap，当向集合中添加元素时，需要根据hashcode进行&运算来计算元素的存储位置
当集合集合中的存储位置存在了元素，需要根据equals比较元素是否相等，相同就需要直接覆盖

当只重写equals方法时可能出现equals方法相等而hashcode不相等，则会造成存储的混乱
```java
package com.steven.basic;

import lombok.Data;

import java.util.HashMap;
import java.util.Objects;

/**
 * @Description:
 * @CreateDate: Created in 2023/8/28 00:27
 * @Author: lijie3
 */
public class HashCodeTest {
    public static void main(String[] args) {
        HashMap<User,Integer> map = new HashMap<>();
        map.put(new User("steven"),23);
        //这里通过get方法使用hashcode来计算key的存储位置，hashcode方法没有被重写，由于hashcode不一样，则导致无法获取到想要的值23，拿到的是null
        System.out.println(map.get(new User("steven")));
    }


    @Data
    static class User{
        String username;

        public User(String username) {
            this.username = username;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (!(o instanceof User)) return false;
            User user = (User) o;
            return Objects.equals(username, user.username);
        }

//        @Override
//        public int hashCode() {
//            return Objects.hash(username);
//        }
    }
}
```

21. 在2g的大文件中找到top100的频率的词汇
思考方向：分而治之，使用堆统计
文件如何加载到内存中--将文件分割为512kb大小的小文件，避免一次性载入过多啊的文件造成内存溢出的问题
文件单词采取什么策略来统计，使用hash（ConcurrentHashMap）来存储每个小文件中单词出现的频率
使用多线程遍历各个小文件，加快统计效率
将统计的hash结果数据存放到一个大顶堆里，通过堆的弹出操作来统计最终top100

22. mysql二阶段提交
在开启binlog日志的情况下Mysql要同时完成binlog和redolog的日志写入
为了保证日志的成功写入，需要用到两阶段提交机制
二阶段提交发生在binlog和redolog的日志写阶段，将日志写入和日志提交分为两个阶段
prepare 和commit两个阶段

23. 集群环境下的分布时单例模式
思考方向：
如何实现跨进程级别的单例实例
如何保证在任何时刻保证只有一个线程访问单例

将单例对象序列化保存到文件中，再将文件存储到外部共享存储组件
程序读取外部共享文件中序列化对象，进行反序列化使用
使用完成后再将对象序列化后存储回外部共享的存储组件中，将对象从本地内存中删除
需要使用分布式锁，可以使用redis分布式锁、zookeeper、etcd等分布式锁

24. InnoDB和MyISAM的区别
* 事务支持不同，InnoDB支持事务而MyISAM不支持
* InooDB 支持行级锁，容易发生死锁，MyISAM支持表级锁 ，不容易出现死锁
* InnoDB支持外键约束，MyISAM不支持
* MyISAM使用索引文件和数据文件存储数据，InnoDB使用idb文件存储数据（索引和数据存储在同一个文件中）
* 性能差异，并发不高的情况下MyISAM的读速度比InnoDB快，而在高并发的环境下InnoDB的性能更好，因为InnoDB支持行级锁，支持事务处理，读多写少可以使用MyISAM更合适
* 数据安全，InnoDB支持崩溃恢复和数据恢复（通过恢复日志来实现数据恢复），MyISAM不支持

25. 介绍观察者模式和策略模式
都是行为型模式
策略模式：根据上下文动态控制类的行为的场景使用，解决if else判断，将类的行为进行封装，程序可以动态替换这些类，如支付场景选择支付宝、微信支付可以使用策略模式
观察者模式：一对多的依赖关系中，实现对某一个对象状态变更之后的感知场景，降低对象关系的耦合度，通过状态的通知机制保证依赖对象堆状态的统一协同

26. 雪花算法原理
生成分布式全局唯一id的算法，通过64位来标识一个全局id
1bit不用，符号位，id不会为负数，一般为0
41bit 时间戳
10bit 工作机器id
12bit 序列号
保证多个服务器上id的唯一性

27. java spi怎么用
service provider interface ，基于接口的动态扩展机制，如数据库驱动的加载就是通过spi方式实现
将装配的控制权转化到程序之外
标准定义和接口实现分离，在模块化开发中实现解耦
实现功能扩展，更好实现定制化需求
spring的SpringFactoriesLoader实现了spi机制

28. 单例模式的实现
私有化构造方法，提供静态方法作为全局访问点
* 饿汉模式（使用类加载实现对象的实例化，利用累加载的安全性）
* 懒汉模式 （使用DCL方式）
* 使用一个静态的holer内部类提供一个单例的实例化引用并进行初始化，只有当用到该单例时才会去加载内部类才会去实例化，
* 使用枚举
* 避免反射破坏（在构造方法内设置标志位，只允许构造方法调用一次），
* 避免反序列化，实现readResolve方法，将已经存在的单例返回

29. ConcurrentHashMap的key为什么不能为null
ConcurrentHashmap的key和value都不能为空
避免多线程环境下的判断键值对是否存在的问题可能会出现歧义，当一个线程从ConcurrentHashMap中获取一个值时，如果返回的值时null，可能是因为map中不存在而返回null或因为value就是null，这样就会产生歧义，同样，判断key是否存在时也会出现这样的歧义，这样的歧义会导致线程安全的问题

### ConcurrentHashMap 是如何保证并发安全性的
ConcurrentHashMap 的线程安全是不断演进的
在jdk7中，ConcurrentHashMap 是通过分段锁+CAS+使用volitale的value字段来保证可见性
jdk7中的ConcurrentHashMap将数据存储在一个Segement数组中，每个Segement继承了ReetrantLock，每个Segment中又存储着一个HashEntry数组，当对数据进行访问时，只需要对Segement进行加锁

jdk8中ConcurrentHashMap使用了synchronized+cas来实现锁，jdk8中的ConcurrentHashMap只有一个Node数组（与HashMap一样），由于jdk8的synchronized性能已经大幅提升，通过控制synchronized的锁粒度能实现高性能的并发效果

### jdk8 中的ConcurrentHashMap的扩容机制
jdk8中的ConcurrentHashMap支持多线程扩容机制，与HashMap类似，默认情况下当容器内的负载达到0.75时，会触发扩容，扩容过程中，会生成一个双倍大小的素组，生成数组后，线程开始转移元素，在扩容过程中如果其他线程在put，这个put的线程会帮助元素进行转移（实际上是复制），扩容过程中原来数组上的NOde不会消失，而Node的value值使用volatile保证了对线程的可见性，不会有多线程问题

### HashMap为什么能允许null的键值对，ConcurrentHashMap为什么不能允许null的键值对
HashMap中键值对以Entry对象存储，Entry对象的key和value都可以是null值，HashMap的设计本身就是非线程安全的，这里不会考虑多线程造成的null值判断错误。在HashMap中应该谨慎使用null的键值对，避免出现空指针异常
ConcurrentHashMap中不允许null的键值对，是因为ConcurrentHashMap要考虑多线程环境下的语意一致性，当put一个null的键时，容器无法分辨到底是key为null还是容器中根本没有为null的key，这在多线程中时含糊不清的，所以不允许put null。
如果允许put null，假设线程A正在进行put（null，null）操作，线程B正在执行get(null)，这是线程B 拿到的null到底是指代拿到的值为null还是说容器没有null值，这一点没有办法分辨

### java中有哪些文件拷贝方式
* 可以利用java的流处理工具FileInputStream/FileOutputStream实现文件的拷贝
* 可以使用channel.transferTo()方法在流上获取channel并拷贝文件
* 可以直接使用java的Files.copy进行文件的拷贝
总体上来说，利用nio的transferTo的效率可能会更高一点

###  描述下java中的死锁，如何避免死锁
 
多个线程按照不同的顺序获取锁，不同线程之间持有对方需要的锁，从而导致所有线程持续处于阻塞等待的状态
死锁的产生条件：
互斥条件，共享资源只能被一个线程占有
请求和保持条件：线程t1已经获取共享资源并等待其他共享资源不释放已持有的资源
不可抢占条件：其他线程不能强行抢占线程已占用资源
循环等待条件：多个线程之间相互等待对方释放资源

代码中如何避免死锁：
* 尽量避免使用多把锁，使用单一的状态表示锁
* 尽量让参与获取锁的线程安相同的顺序获取锁，避免循环依赖问题
* 使用带超时时间的锁获取方法，为程序带来更多可控性

在出现死锁时一般只能重启程序来解决，通过jstack将线程堆栈打出来，定位死锁代码

### 谈谈jdk中的并发工具包
jdk的并发包提供更加高级的线程操作工具以及线程安全的集合工具，具体如下：
* 提供了Lock，比起内建锁synchronized更加灵活
* 提供了CountDownLatch（允许一个或多个线程等待某些操作的完成）、CyclicBarrier和Semaphore，方便对线程间的协作进行控制
CountDownLatch用法（允许一个或多个线程等待某些操作的完成）：
```java
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch latch = new CountDownLatch(3);

        new Thread(()->{
            log.debug("begin....");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.debug("end....");
            latch.countDown();
        },"t1").start();
        new Thread(()->{
            log.debug("begin....");
            try {
                Thread.sleep(1500);

            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.debug("end....");
            latch.countDown();
        },"t2").start();
        new Thread(()->{
            log.debug("begin....");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.debug("end....");
            latch.countDown();
        },"t3").start();
        log.debug("waiting");
        //等待线程计数结束后才会执行
        latch.await();
        log.debug("main running");
    }
```
CyclicBarrier用法（与CountDownLatch类似，允许多个线程等待到达屏障，适用于一组线程相互等待，直到所有线程都准备好后再同时执行的场景）：
```java
 public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(2);
        //当所有任务完成之后，barrier的计数会被重置成初始值2，可重复执行
        CyclicBarrier barrier = new CyclicBarrier(2,()->{
            //其他两个任务执行完成之后会执行这个任务
            log.debug("task1,task2 has finished....");
        });
        for (int i = 0; i < 3; i++) {
            service.submit(()->{
                log.debug("task1 begin....");
                try {
                    Thread.sleep(1000);
                    //线程执行完之后等待，计数barrier减一
                    barrier.await();
                    log.debug("task1 end....");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
            service.submit(()->{
                log.debug("task2 begin....");
                try {
                    Thread.sleep(2000);
                    //线程执行完之后等待
                    barrier.await();
                    log.debug("task2 end....");
                } catch (InterruptedException | BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }
        service.shutdown();
    }
```
Semaphore用法(同一时刻只允许指定数量线程运行)：
```java
    public static void main(String[] args) {
        //同一时刻只能有三个线程访问,非公平
        Semaphore semaphore = new Semaphore(5,false);

        for (int i = 0; i < 10; i++) {
            new Thread(()->{
                //获取许可
                try {
                    semaphore.acquire();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                log.debug(Thread.currentThread().getName() + " running");
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    //释放许可
                    semaphore.release();
                }
            }).start();
        }
    }
```

* 提供了ConcurrentHashMap、CopyOnWriteArrayList等线程安全的容器
* 提供了并发队列BlockingQueue的各种实现，例如ArrayBlockingQueue、SynchronizedQueue、PriorityBlockingQueue的各种实现等
* 提供了强大的Executor框架，管理并调度线程


### Concurrent、CopyOnWrite、Blocking三个类型的并发工具类有什么区别
* Concurrent类型没有类似CopyOnWrite的相对较重的修改开销，提供了较低的遍历一致性，使用迭代器遍历容器时，迭代器仍然可以继续进行遍历（同步容器在发现容器被修改时会“fail-fast”，抛出ConcurrentModificationException），size存在不准确的情况
* CopyOnWrite容器也是线程安全的一种容器，在数据写入时对容器进行复制，将新的元素插入到副本中，实现写操作的安全性，读操作是读取原来容器中的数据，不需要加锁，适合读多写少的场景使用
* Blocking 一般是代指线程安全的Queue或Deque，往往配合线程池使用

1.  rabbit MQ如何实现高可用的

2.  为什么加索引能提高查询效率
命中了索引的查询效率会比较高，InnoDB使用了B+tree来实现索引，可以在控制层高很低的情况下实现索引的构建，这样在查找数据时可以减少查询时的IO次数

1.  什么时链路追踪
链路追踪时分布式架构下的一种实现请求可视化监控的一种追踪方式，用户请求涉及到多个系统间的流转，链路追踪能实现对请求调用链路的清晰定位，并能协助分析系统的性能瓶颈，定位故障位置，梳理服务依赖关系进行优化,还能实现数据分析的功能
市面上流行的链路追踪 有 zipkin、skywalking、cat等

1.  JVM为什么使用元空间替换了永久代
jdk1.7中的永久代存放了类的元信息、运行时常量池、方法元信息等信息，永久带属于JVM运行时数据区的一块内存空间
JDK8 使用了元空间来存储方法区信息，使用的是系统的直接内存，不属于jvm垃圾回收管理的区域
元空间理论无上限，内存上限较大
老年代需要full gc来进行垃圾回收
而元空间可以不stop the world的情况下进行垃圾回收，简化了垃圾回收的过程
jrocket没有永久代

1.  常见的限流方法
滑动窗口、令牌桶、漏桶
令牌桶可以处理系统突发流量的情况

1.  finally语句块一定会执行吗
两种情况下不会执行：
* 未进行try语句块时程序已经异常终止了
* 在try catch中执行了System.exit(0) 语句时，finally语句不会执行

1.  空的Object占用多少内存空间
开启压缩指针的情况下，Object默认占用12个字节，jvm按照8个字节的的整数倍来进行填充，所以会占用16个字节
关闭压缩指针的情况下，Object默认占用16个字节
内存布局
对象头 -- markword（8个字节）、对象元数据指针（开启压缩4个字节，不开启8个字节）、数组长度（是数组时才会有，4个字节）
实例数据
对齐数据

1.  http协议和rpc协议的区别
功能上，http协议时应用层的超文本传输协议，web传输的基础协议，rpc时远程过程调用协议，实现不同计算机应用之间的通信，屏蔽通信的底层复杂度，实现远程调用
RPC协议是一种规范，不是一种实现，协议的框架才是具体的实现，如Dubbo、GRPC等框架，HTTP是一种已经实现的协议
底层都是用tcp协议进行通信
RPC协议只是规定不同服务之间数据通信的规范，也可以通过基于http协议的方式实现，如Grpc、feign等

1.  SimpleFormatDate的线程安全性
SimpleFormatDate不是线程安全的，因为他里面维护了一个Calendar的实例，多个线程操作SimpleFormatDate时会造成线程不安全的问题
JDK8引入了新的线程安全日期时间工具类，如LocalDate,LocalTime、LocalDateTime、DateTimeFormatter等

1.  redis的缓存淘汰策略
redis使用内存达到了maxmemory参数阈值时，redis会根据访问频率的高低，将访问低的数据从内存中移除掉，maxmemory指的是服务器的最大内存
redis默认提供八种缓存淘汰策略：
 * LRU策略
 * 采用LFU策略
 * 随机策略
 * ttl策略，挑选过期的key淘汰
 * 直接报错，内存不够直接报错

1.  任务数超过线程的核心数时，如何让任务不进入队列
使用SynchronousQueue，它不能存储任何任务，它的工作策略是没生产一个任务就必须要有一个线程来消费这个任务，当没有线程来消费这个任务时，可以避免新的任务进入阻塞队列，从而使用急救线程去执行新的任务

1.  阻塞队列被异步消费如何保证消费的顺序
阻塞队列是符合先进先出的规则
阻塞队列中使用ReentrantLock的两个condition分别来控制入队和出队操作
当有线程来消费任务时，任务首先要获取排它锁才能从队列中取出任务，当队列中没有任务时，线程会进入condition中等待生产者线程来生产任务放入队列中，当有任务时，消费者线程会被唤醒消费任务

1.  线程池如何实现线程的复用
线程池中采用生产者消费者的模式实现线程复用，生产者消费者模式通过中间容器实现生产者消费者的解耦
生产者生产任务保存到容器中，消费线程从容器消费任务
线程池中需要保证线程的复用，所以线程是在有任务时执行任务，当阻塞队列中没有任务时，线程等待执行（一般是使用wait、notify实现等待与唤醒），并释放cpu资源

1.  分布式ID的设计方案
实现方案：
* 使用mysql的全局表
* 使用zookeeper的有序节点
* 使用MongoDb的object id
* redis的自增id
* uuid
需要考虑
id的有序性，提高B+tree范围查找的效率，提升B+tree的维护效率
数据的安全性，避免恶意爬取数据造成数据泄漏的问题
可用性，id生成系统的可用性要求要高
性能，要求全局id生成系统性能高
目前成熟的方案是通过雪花算法实现全局唯一id的生成，雪花算法的好处：
实现简单、不存在太多外部依赖，可生成有意义的序列编号，基于位运算，性能比较好



规定对共享变量的写操作对其他线程读操作可见，是可见性与有序性的一套规则

* 线程解锁m之前对变量的写，对于接下来对m加锁的其他线程对变量的读可见
* 线程对volatile变量的写对接下来其他线程对变量的读可见
* 线程start之前对变量的写对该线程开始后对该变量的读可见
* 线程结束前对变量的写对其他线程得知它结束之后的读可见
* 线程1打断（interrupt）线程2前对变量的写对于其他线程得知线程2被打断后的变量的读可见
* 对变量默认值的写对其他线线程对该变量的读可见
* 传递性，volatile变量写入，写屏障会将写屏障之间的所有操作都同步到主存（即使写屏障之前的某个变量不是volatile变量）

55. Java官方提供了那几种线程池
java官方提供了一个Executors的工具类来生成各种线程池
* newCachedThreadPool 可以缓存的线程池，可以用来处理大量短期的突发流量，核心线程数为0，最大线程数是无限的，线程存活周期是60，阻塞队列时SynchronousQueue，不能存储仁和任务，需要一直分配工作线程来处理任务，可能出现OOM，栈内存溢出
* newFixedThreadPool 核心线程数和最大线程数最大线程数一样，没有急救线程，使用的无界阻塞队列，当出现大量任务时，可能会OOM
* newSingleThreadExecutor 只有一个线程的线程池，线程数量无法更改，保证提交到线程池中的任务有序执行，使用了无界队列，可能出现OOM
* newScheduledThreadPool 具有延迟功能的线程池，可以实现定时调度任务，使用DelayedWorkQueue队列，队列元素需要实现Delayed接口
* newWordStealingPool java8提供的线程池，内部使用forkJoinPool，利用工作窃取算法并行执行请求

56. 线程两次调用start会出现什么问题
调用两次start会报线程状态错误，java中的线程状态，线程运行时，会首先判断线程的状态，如果线程已经处于运行状态，则会报错
new 线程对象刚刚被创建
runnable 线程已经准备好运行正在等待cpu资源或者正在运行
blocked 线程处于锁等待状态 
waitting 线程处于条件等待状态，当触发唤醒后比如wait/notify，线程会继续执行
timed_wait,超时条件等待
terminated 线程执行结束

57. java 中文件拷贝的方式
* 使用文件IO流 FileInputStream/FileOutputStream
* 使用java.nio包下的库，使用transferTo transferFrom方法实现，零拷贝，避免拷贝和上下文切换
* 使用文件拷贝工具Files.copy实现

58. ArrayList的扩容实现
默认情况下ArrayList的数组长度是10个，可以使用构造方法指定数组长度
当元素不断插入导致数组容不下心的元素时会触发扩容
扩容首先是创建一个新的数组，长度为原来的1.5倍
然后使用Arrays.copyof（）将老数组copy到新数组，然后再将新加的元素加入到新的数组中

### String、StringBuffer、StringBuilder的区别
String 是不可变Immutable类，是线程安全的，类似于字符串的拼接、裁剪、等动作都会产生新的String对象，所以相关操作的效率对性能存在一定的影响
可变性：String不可变、StringBuffer、StringBuilder可变，String变更会产生一个新的对象
线程安全性：String、StringBuffer线程安全，Stringbuilder线程不安全
性能：String性能最差，每次变更都要创建对象，StringBuffer其次，因为其可变，但是由于使用synchronize加锁，效率会较低，StringBuilder最高，但线程不安全

### String为什么被设计成final类型
* 安全性：不可变对象天生就是线程安全的
* 在HashMap缓存中可以安全的作为缓存键，避免因为键的可变导致缓存失败
* 提高性能：由于不可变，可以进行字符串常量池的优化，避免重复创建相同值的字符串对象，提高性能
* 简化内存模型：不可变对象的值在创建后不会改变，简化了内存模型的设计和实现

### jdk9之后对String类的优化
* 降低内存消耗：jdk9之后使用byte数组存储数据，而在之前使用的是char数组，一个char类型需要占用两个字节的内存
* 提供repeat方法：指定字符串重复的次数生成新的字符串
* jdk11提供strip方法：去除字符串首尾的空白字符，包括空格、制表符、换行符等，而trim方法无法去除制表符、换行符等
* jdk11提供isBlank方法：检测字符串是否是空白（包括空格、制表符、换行符等空字符）
* jdk11提供lines方法：将字符串按行切分为Stream，方便处理文本内容

### 动态代理是基于什么实现的
动态代理的实现有多种实现方式：
动态代理是在运行时动态构建代理、动态处理代理方法调用的机制
* jdk的动态代理，主要是使用反射机制来实现的，通过InvocationHandler接口来实现对目标接口的代理
* 可以使用字节码操作机制，例如ASM、cglib等，在运行时动态修改类字节码达到动态代理的目的

### int 和Integer的区别
int 是java的原始数据类型，占用4个字节
Integer是java的类，是int对应的包装类，使用int类型的字段存储数据，提供基本的操作，如数学运算、int和字符串之间的转换等，在java5中引入了自动装箱和拆箱的工呢个，Java可根据上下文自动装箱、拆箱
Integer利用了缓存机制，默认缓存了-128到127的数值，在使用valueOf方法时利用这个缓存机制会带来明显的性能提升

### Vector、ArrayList、LinkedList的区别
Vector是基于动态数组的线程安全的集合，采用同步实现线程安全，当数组满时会创建新的数组，并拷贝原数组。如果不考虑线程安全的问题，不建议使用Vector
ArrayList基于动态数组的非线程安全的集合，性能会比Vector好很多。在数组容量扩充时，Vector会扩充一倍，而ArrayList会扩充50%。ArrayList由于使用数组，在查找时效率高，但插入和删除时，效率不太高
LinkedList是基于双向链表实现的集合，不需要像ArrayList那样调整容量，插入、删除速度较快，查找效率较低，LinkedList也不是线程安全的
通过Collections.synchronizedList(List) 可以将一个非线程安全的list转化为线程安全的list


1.  什么是缓存击穿，缓存击穿该如何解决
缓存击穿是指在使用缓存系统（比如内存缓存、分布式缓存等）的时候，当一个请求查询一个不存在于缓存中的数据，并且这个数据又是频繁被访问的热点数据时，大量的请求会直接穿透缓存，访问数据库或后端服务，导致后端系统负载急剧增加，甚至可能引起系统崩溃。
缓存击穿的典型场景是：一个热点数据的缓存过期，然后大量的并发请求尝试获取这个数据，由于数据不在缓存中，每个请求都会直接访问后端数据库或服务，导致数据库压力激增。
为了避免缓存击穿，可以考虑以下几种方法：
设置合适的缓存过期时间： 在设置缓存过期时间时，可以根据业务的访问模式和数据的更新频率来调整。对于热点数据，可以设置较长的过期时间，避免频繁的缓存失效导致击穿问题。
使用互斥锁（Mutex Lock）或分布式锁： 在缓存失效的情况下，使用互斥锁或分布式锁来保护对后端数据的访问，只允许一个线程去加载数据到缓存，其他线程等待，避免了大量并发请求同时穿透缓存。
使用永不过期的缓存： 对于热点数据，可以考虑使用永不过期的缓存策略，然后定期刷新缓存数据。这样可以确保即使缓存过期，也能够提供一个可用的旧数据，避免击穿。
使用预加载： 在系统启动时，预先加载热点数据到缓存中，避免在请求到来时才进行数据加载，减少缓存失效时的冲击。
限制并发请求： 使用限流策略，限制并发请求的数量，避免突发的大量请求同时访问后端数据。
使用布隆过滤器，应用程序启动时，先把数据缓存到布隆过滤器中

1.  一致性hash算法的理解
解决分布式情况下hash表可能存在的动态扩容或缩容的问题
一般使用hash表以k,v的方式存储数据
一致性hash算法通过hash环的方式实现数据存储，当数据因为扩容或缩容需要迁移时，只需要迁移少部分的数据


62. spring bean的作用域
非web应用中有两种
singleton：单例bean，单例bean在容器中只会有一个实例
prototype：原型bean，同一个bean会有多个实例存在，当实例创建之后不会由spring来管理
web应用中有三种：
request，针对每一次http请求都糊创建一个新的bean
session 同一个session共享同一个bean实例，不同的session产生不同的bean实例
globalSession 全局共享session，在全局http会话中创建一个新的bean实例
    
63. mybatis 的缓存机制
mybatis设置了两级缓存，避免每次查询都去查询数据库
一级缓存是sqlSession缓存，也叫本地缓存，将查询的数据缓存在sqlSession中，存放在local cache，后续sql如果在命中缓存的情况下，就可以从本地缓存中获取，
如果想实现跨sqlSession获取缓存，需要使用mybaits的二级缓存，
当多个用户在查询数据时，只要有一个sqlsession拿到了数据，则其他的sqlsession就可以从二级缓存中拿到数据

如果同时开启二级缓存，查询流程是先查二级缓存，再查一级缓存，最后查询数据

64. spring的事务传播行为
Spring框架中的事务传播行为是用来定义一个事务方法与现有事务之间的交互方式的规则。在Spring中，你可以使用@Transactional注解或者编程式事务管理来配置事务传播行为。以下是Spring中常见的事务传播行为：

REQUIRED（默认值）：如果当前存在事务，则加入该事务，如果没有事务则新建一个事务。这是最常用的传播行为，确保当前方法始终在一个事务内执行。
SUPPORTS：如果当前存在事务，则加入该事务，如果没有事务则以非事务方式执行。该选项适用于不需要强制事务的情况，如果存在事务，则在事务内执行，否则以非事务方式执行。
MANDATORY：要求当前存在事务，如果没有事务则抛出异常。该选项适用于需要在事务内执行的方法，如果没有事务则抛出异常。
REQUIRES_NEW：无论当前是否存在事务，都会创建一个新的事务，如果当前存在事务则将其挂起。这允许方法在一个新事务内独立执行。
NOT_SUPPORTED：以非事务方式执行方法，如果当前存在事务则将其挂起。这适用于不需要事务支持的方法。
NEVER：以非事务方式执行方法，如果当前存在事务则抛出异常。用于确保方法不在事务内执行。
NESTED：如果当前存在事务，则创建一个嵌套事务，它是当前事务的子事务。如果没有事务，则行为类似于REQUIRED。嵌套事务可以独立提交或回滚，但它们依赖于外部事务的最终提交或回滚。
NESTED_READ_COMMITTED：类似于NESTED，但是嵌套事务内部的读取操作使用读已提交（READ_COMMITTED）隔离级别，而不是外部事务的隔离级别。
这些事务传播行为允许你根据需求管理事务的行为，确保事务在不同的方法之间正确地传播和交互。你可以将@Transactional注解或者编程式事务管理与这些传播行为一起使用，以实现精确的事务控制。


65. 说一说对AQS的理解
AQS多线程同步器，JUC包中的多个组建的层实现，如Lock、CountDownLatch、Semaphore都用到了AQS
AQS提供两种锁机制，排他锁和共享锁
排他锁：同一时间只允许一个线程获取到锁资源，如ReentrantLock
共享锁：同一时克允许多个线程同时获得锁资源

66. 说一说对分布式事务的理解
微服务架构下由于数据库和应用服务细化拆分，导致原本一个事务单元中多个DML操作变成了跨进程跨数个数据库的多事务单元的多个DML操作，传统的数据库事务无法解决这类问题，需要使用分布式事务来解决。
分布式事务就是解决事务一致性问题，有两种说法：
*  强一致性，所有参与的事务要么全部成功，要么全部失败，  ---适用于对数据一致性要求高的场景
*  最终一致性，弱一致性，允许数据出现不一致的情况，但最终的某个时间会达成数据一致
seata（一站式的分布式事务解决方案）中的四种分布式事务的模式：
* AT 模式，基于本地事务+二阶段协议来实现最终的数据一致性方案，seata的默认方案
* TCC模式，try、conforn、cancel三个阶段，通过事务管理器在业务逻辑层面根据每个事务分支的执行情况分别调用该业务的confirm或者cancel方法
* saga模式，长事务解决方案，
* XA模式，强一致性事务的解决方案，利用事务资源对XA协议执行，以XA协议的机制来管理分支事务的一种事务模式

67. cpu飙高应该如何排查
cpu飙高的问题一般有两个原因：
* CPU上下文切换过多（各种阻塞操作过多或是存在大量线程导致线程上下文切换）
* 在程序中创建了大量线程，或是有线程进入了死循环一直占用cpu导致CPU资源无法释放
排查cpu飙高的步骤
通过top命令定位到占用cpu高的进程
使用jstack pid dump出线程详情，定位到执行时间过长的线程，并定位到指定的代码行数
分许代码问题

68. Synchronized和lock性能对比
Synchronized 和 Lock 在性能上有一些区别，但具体的性能对比会受到多个因素的影响，包括应用程序的实际使用情况、硬件配置、JVM 实现等。一般来说，以下是它们的性能对比的一些因素：

性能开销：

Synchronized：由于它是Java的内置语言特性，因此通常来说，使用 synchronized 的开销较低。JVM 对 synchronized 进行了优化，可以实现较高的性能。
Lock：Lock 通常需要更多的底层操作，因此在某些情况下，会产生更高的性能开销。
竞争情况：

在低竞争情况下，两者性能差异可能不明显，因为它们都可以有效地管理同步。
在高度竞争的情况下，Lock 可能具有更好的性能，因为它允许更多的控制，如公平锁和非公平锁，可以更好地处理线程竞争。
可重入性：

两者都支持可重入性，但在某些情况下，Lock 的可重入性可能需要更多的开销，因为需要维护计数器等信息。
异常处理：

Synchronized 在发生异常时会自动释放锁，这可以避免锁泄漏。
Lock 需要手动处理异常，这可能会导致在异常发生时不正确地释放锁，从而导致潜在的问题。

等待可中断：

Lock 具有等待可中断的能力，这意味着可以在等待锁的过程中中断线程，而 Synchronized 不具备这种特性。
综合考虑，性能对比通常不是决定使用 Synchronized 还是 Lock 的唯一因素。您应该根据应用程序的需求和设计来选择合适的同步机制。在许多情况下，性能差异可能不明显，而代码的可读性和维护性更为重要。在高度竞争和复杂同步需求的情况下，Lock 可能提供更多的控制，但需要谨慎使用以避免潜在的问题。最好的方法是根据具体情况进行性能测试和分析，以确定哪种同步机制最适合您的应用程序。

条件变量：

Lock：支持条件变量，通过 Condition 接口实现，可以实现更复杂的线程通信和等待/通知模式。
Synchronized：没有直接支持条件变量的概念，通常要使用 wait() 和 notify() 或 notifyAll() 方法来实现简单的线程通信。



69. 什么是sidecar
"Sidecar" 是一种软件架构模式，通常用于微服务架构中，用于扩展应用程序的功能和管理附加任务。Sidecar模式通过将辅助性功能模块部署为独立的容器（通常是一个独立的进程）来实现。这个辅助模块被称为"sidecar"，因为它附加到主应用程序容器旁边，就像自行车上的侧车一样。

Sidecar模式的主要目标是将应用程序的核心功能与辅助功能（如日志记录、监控、安全性、负载均衡、服务发现等）分离开来，以便更容易管理和维护。这种分离的架构具有以下特点：

模块化：每个功能模块都可以作为一个独立的容器运行，这使得它们可以根据需要独立部署、更新和扩展。

独立性：Sidecar模块不需要对主应用程序进行修改，因此可以轻松地添加或删除不同的辅助模块，而不影响主应用程序的开发。

分布式功能：辅助功能可以在多个主应用程序之间共享，从而避免了重复部署和维护。

隔离性：每个Sidecar模块都可以有自己的资源和隔离环境，这有助于确保它们不会干扰主应用程序的正常运行。

监控和管理：Sidecar模块通常用于监控主应用程序的性能、日志和异常情况，并向外部监控系统报告信息。它们还可以实施负载均衡、故障恢复和服务发现等任务。

安全性：一些Sidecar模块可以提供额外的安全层，例如身份验证、授权和加密，以确保应用程序的安全性。

70. redis和mysql如何保证数据的一致性
由于同时更新数据库和redis，他们更新的先后顺序会影响数据的一致性
一般来说有两种方式来：
1. 先更新数据库，再更新缓存
2. 先删除缓存，再更新数据库，用户请求数据时还是先走redis，如果redis没有找到，则从数据库中获取，同时更新redis数据    
两种情况都存在并发请求数据的原子性问题
第一种如果redis更新失败，则会导致数据不一致
第二种在极端情况下如果再删除redis数据和更新数据库时，如果有其他线程来访问，还是会导致数据不一致问题
可以在写入redis的流程中架设一层消息中间件，rocketMQ来保证消息的可靠投递，实现数据的最终一致性
可以使用canal从binlog中家在数据同步到redis中

71. 什么情况下索引会失效
 * 复合索引未用左列字段;
 * like 以% 开头
 * where条件索引列需要进行类型转换
 * where条件索引列中使用了函数
 * 如果mysql判定全表扫描更快时
 * 如果条件中有or，即使其中有部分条件带索引也不会使用(这也是为什么尽量少用or的原因)，要是用or时使用索引必须将or中的列全部设置为索引列

72. 什么样的列不适合设置索引
  * 唯一性差的字段不适合设置索引
  * 频繁更新的字段不适合作为索引列
  * where条件中不用的字段
  * 索引使用<> 判断时，效果一般

73. springboot的启动流程
  * 首先从main找到run()方法，在执行run()方法之前new一个SpringApplication对象
  * 进入run()方法，创建应用监听器SpringApplicationRunListeners开始监听
  * 然后加载SpringBoot配置环境(ConfigurableEnvironment)，然后把配置环境(Environment)加入监听对象中
  * 然后加载应用上下文(ConfigurableApplicationContext)，当做run方法的返回对象
  * 最后创建Spring容器，refreshContext(context)，实现starter自动化配置和bean的实例化等工作。

74. springboot自动装配原理
  通过@EnableAutoConfiguration注解，使用SPI机制在类路径的META-INF/spring.factories文件中找到所有的对应配置类，然后将这些自动配置类加载到spring容器中。

75. java中的final、finlize和finally
final：修饰符，被final修饰的类不能再派生出子类，被final修饰的方法不能被重写，被final修饰的成员变量不可变
finally：异常处理模块中用来执行清理操作用
finalize：方法名，允许使用finalize在垃圾回收前进行一些必要的清理工作

76. String 为什么要被设计成final类型的
final类型的类意味着类的不可以变，当重新给string赋值时，String指向的变量地址会变更，而不是在原来的内存地址中进行修改
String 类是final类型的，避免了子类继承导致的导致的行为的变化，String中包含一个final类型的char[],字符串中的数据实际上是存储在char[]中的，因为java开发中涉及到大量的字符串，如果每个字符串都去开辟内存去存储必然会造成大量的内存消耗，不利于性能，java将字符串存储在元空间中的常量池里共享，这样一方面保证保证了字符串共享时的安全性，另一方面保证了性能。
不可变可以保证线程安全，不可变行支持字符串常量池，避免在堆中大量创建对象影响性能
GPT的回答：
String 被设计为 final 类型的主要原因是确保 String 对象的不可变性。在 Java 中，String 对象是不可变的，这意味着一旦创建了 String 对象，就无法更改其内容。这种设计决策有助于确保字符串操作的安全性和线程安全性，因为它们可以被多个线程同时访问而不会发生意外的变化。此外，不可变性还有助于提高字符串操作的效率，因为可以对字符串进行缓存和共享，而不必担心其内容被修改。


77. 说一说java同步IO、同步非阻塞IO、异步IO

异步io采用“订阅-通知“模式，应用程序向操作系统注册io监听，然后继续做自己的事情，当操作系统发生io事件，饼做好数据准备后，主动通知应用程序，触发相应的函数
异步io由操作系统进行相应的支持：
在windows系统中使用IOCP（IO Completion port）
在linux下使用epoll对异步IO进行模拟

nio：同步非阻塞io，现将数据存储到缓存区，如果线程需要，从缓存区拿。
Buffer类座位缓冲区，Channel相当于IO的strean抽象，Selector是nio提供的管理多个channel的工具
aio：异步IO
先让io处理，线程去做其他事情，处理完成之后io通知一下即可
aio提供的时间处理接口是CompletionHandler，定义了回调函数，这些函数在IO完成之后会自动被调用

三种IO方式的举例：
海底捞很好吃，但是经常要排队。我们就以生活中的这个例子进行讲解。
A顾客去吃海底捞，就这样干坐着等了一小时，然后才开始吃火锅。(BIO)
B顾客去吃海底捞，他一看要等挺久，于是去逛商场，每次逛一会就跑回来看有没有排到他。于是他最后既购了物，又吃上海底捞了。（NIO）
C顾客去吃海底捞，由于他是高级会员，所以店长说，你去商场随便玩吧，等下有位置，我立马打电话给你。于是C顾客不用干坐着等，也不用每过一会儿就跑回来看有没有等到，最后也吃上了海底捞（AIO）
三种IO的使用场景：
BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4之前唯一的选择
NIO适用于链接数目较多且链接比较短的架构，比如聊天服务器，并发局限于应用中，编程复杂，JDK1.4开始支持
AIO使用链接数较多且连接较长的架构，如响彻服务器，充分调用OS饼参与并发操作，编程比较复杂，JDK7开始支持


78. java NIO的用法：
1. 组成
  Channel：读取数据的通道，可理解成BIO中的stream
  Buffer：缓冲区，读取到的数据缓冲的区域
  Selector：允许单线程处理多个Channel。如果应用中打开了多个Channel，但每个连接的流量都很低，使用Selector会很方便，如聊天服务器
  

79. git fetch/git pull的用法
git在本地保存两个版本的仓库，分为本地仓库和远程仓库
fetch 只更新远程仓库的代码为最新代码，本地仓库代码未被更新
pull 操作是将本地仓库和本地的远程仓库更新到远程的最新版本

所以pull = fetch + merge

80. httpclient 出现Timeout waiting for connection from pool是什么原因导致的
出现这个错误是因为httpclient的连接池耗尽导致的，与连接池相关的属性有以下几个配置
* connectionRequestTimeout 每个请求等待获取连接的最大时间
* connectTimeout 连接超时时间，获取到连接之后的操作
* defaultMaxPerRoute 每个路由的最大连接数
* maxTotal client的最大连接数
* socketTimeout 设置socket请求超时时间，当超过超时时间后，请求失败

获取连接发起请求流程
创建httpclient时会设置defaultMaxPerRoute、maxTotal参数，当并发量较大时，应适当提高这两个参数的值
httpclient在获取连接时会使用connectionRequestTimeout来判断是否获取超时
httpclient获取到连接后会使用connectTimeout、socketTimeout两个参数来判断是否请求超时

81. java是一个怎么样的平台，是解释执行的语言吗
* java是一个一次编译，到处执行的语言平台，java的另一个特性是使用垃圾收集器对系统内存进行回收
* java严格上说不是一个解释执行的语言，jvm将java源码编译成字节码后通过jvm内嵌的解释器进行执行，但是hotspot中包含JIT（即时编译）模块，可以将程中的部分热点代码编译成本地机器相关的机器码，进行优化，然后缓存下来以备下次再用，所以从这个角度看，java是一个半编译半解释的语言
平台













