# 概述

启动
java -jar arthas-boot.jar

指定端口号启动

java -jar arthas-boot.jar --telnet-port 9999 --http-port -1

在浏览器中访问（默认端口3658）：
http://localhost:3658/

监控远程服务器：
java -jar arthas-boot.jar --target-ip 172.16.1.133
选择被监控的java进程
arthas-client connect 172.16.1.133 3658

常用命令：
## dashboard 仪表板


## thread 查看线程信息

## jad demo.MathGame 查看反编译后的代码

## watch 监视函数返回值
 watch demo.MathGame primeFactors returnObj

## quit/exit  退出，但是端口号还是被占用

## stop 彻底退出

## help 命令

## cat 打印文件内容

## grep  匹配查找

## pwd 打印当前所在目录

## cls  清除屏幕信息

## session 查看当前会话信息

 Name        Value                      
--------------------------------------------------                                                                                                       
 JAVA_PID    50176                                                        
 SESSION_ID  ba49678e-0d70-43b0-917d-942da0362168


 ## reset 重置增强类

 reset 还原所有类

 reset *List 还原匹配 *List的类

 reset Test 还原Test类

 ## version 版本信息

 ## history 历史命令

 quit 退出当前客户端

 stop 退出arthas服务器

 keymap 显示arthas快捷键

**jvm相关命令**

 ## dashboard 仪表板
 显示线程，内存，gc，系统环境等信息
 * 如果出现线程占用cpu过高，可以用thread tid来查看线程堆栈信息
 * 查看内存占用情况，查看使用的垃圾回收器，查看垃圾回收时间和垃圾回收次数
 * 查看当前系统配置及环境配置信息
 


 ## thread 线程相关
 thread -n 5 显示最繁忙的5个线程
 thread 显示所有线程
 thread -b 显示处于阻塞状态的线程
 thread --state WAITING 查看某一状态的线程

 ## jvm 虚拟机相关信息

 ## sysprop 显示与修改系统属性相关信息
 sysprop java.home 显示java.home
 sysprop test.java aaaa 设置系统属性

## sysenv 查看当前jvm的环境属性

## vmoption 查看、更新JVM相关的参数
vmoption PrintGCDetails 查看某个属性值
vmoption PrintGCDetails true 修改某个属性的值

## getstatic 在程序执行过程中实时查看类的静态属性
getstatic demo.MathGame random 查看demo.MathGame的random属性


## ognl 执行ognl表达式  （对象图表达语言）
ognl '@java.lang.System@out.println("hello")' 执行java命令，打印返回值（这里打印null）
ognl '@demo.MathGame@random' 打印静态值

ognl '#value1=@System@getProperty("java.home"),(#value1)' 取一个环境变量放在list中并返回
打印结果：@String[/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home/jre]

**类与类加载器命令**

## sc  （search class）查看jvm中已经加载的类信息，默认开启子类匹配信息
sc demo.* 显示包中的所有类
sc demo.* -d 显示包中类的详细信息

sm （search method） 查看已加载类的方法信息，当前类声明的方法
sm demo.MathGame 查看类下所有方法
sm demo.MathGame run 查看类下某一方法

反编译相关的
## jad 把字节码反编译成源代码

jad demo.MathGame 反编译并包含类加载器
jad demo.MathGame --source-only 反编译，不包含类加载器信息
jad demo.MathGame main 反编译指定的main方法
jad --source-only demo.MathGame > MathGame.java 反编译并输出到MathGame.java文件

```java
classLoader:                                                                                                                                             
+-sun.misc.Launcher$AppClassLoader@5c647e05                                                                                                              
  +-sun.misc.Launcher$ExtClassLoader@4554617c                                                                                                            

Location:                                                                                                                                                
/Users/lijie3/Documents/tool-package/arthas-bin/math-game.jar                                                                                            

       /*
        * Decompiled with CFR.
        */
       package demo;
       
       import java.util.ArrayList;
       import java.util.List;
       import java.util.Random;
       import java.util.concurrent.TimeUnit;
       
       public class MathGame {
           private static Random random = new Random();
           private int illegalArgumentCount = 0;
       
           public List<Integer> primeFactors(int number) {
/*44*/         if (number < 2) {
/*45*/             ++this.illegalArgumentCount;
                   throw new IllegalArgumentException("number is: " + number + ", need >= 2");
               }
               ArrayList<Integer> result = new ArrayList<Integer>();
/*50*/         int i = 2;
/*51*/         while (i <= number) {
/*52*/             if (number % i == 0) {
/*53*/                 result.add(i);
/*54*/                 number /= i;
/*55*/                 i = 2;
                       continue;
                   }
/*57*/             ++i;
               }
/*61*/         return result;
           }
       
           public static void main(String[] args) throws InterruptedException {
               MathGame game = new MathGame();
               while (true) {
/*16*/             game.run();
/*17*/             TimeUnit.SECONDS.sleep(1L);
               }
           }
       
           public void run() throws InterruptedException {
               try {
/*23*/             int number = random.nextInt() / 10000;
/*24*/             List<Integer> primeFactors = this.primeFactors(number);
/*25*/             MathGame.print(number, primeFactors);
               }
               catch (Exception e) {
/*28*/             System.out.println(String.format("illegalArgumentCount:%3d, ", this.illegalArgumentCount) + e.getMessage());
               }
           }
       
           public static void print(int number, List<Integer> primeFactors) {
               StringBuffer sb = new StringBuffer(number + "=");
/*34*/         for (int factor : primeFactors) {
/*35*/             sb.append(factor).append('*');
               }
/*37*/         if (sb.charAt(sb.length() - 1) == '*') {
/*38*/             sb.deleteCharAt(sb.length() - 1);
               }
/*40*/         System.out.println(sb);
           }
       }
```
## mc memory compiler 在内存中把源代码编译成字节码文件
mc Hello.java 生成class文件在当前目录
mc Hello.java -d /root 编译生成class文件到root目录下

## redefine 把新生成的字节码文件在内存中执行,可以变更正在执行的逻辑
redefine风险较高，有可能会失败
redefine要在jad/watch/trace/monitor/tt等命令之后执行，否则会导致redefine的代码会被重置

## 实际生产中为了快速解决线上bug可以使用下面的操作
redefine Hello.class
jad、mc、redefine串联操作：
使用jad命令将字节码反编译成源代码 -〉修改源代码文件-〉使用mc命令将源代码编译成class文件-〉使用redefine命令将字节码文件在内存中执行，从而增强原来的类

## dump 将已加载类的字节码文件保存到特定的目录中

dump demo.*  提取demo包下的所有类的字节码，保存在类加载器目录下

## classloader 获取所有类加载器信息
classloader -l 按类加载实例进行统计
classloader -a 列出类加载器加载的所有类
classloader -c 548cc2df 查找resource
classloader -c 548cc2df -r META-INF/MANIFEST.MF 查找资源在哪个jar包

**其他调试命令**

## monitor 监视类中方法的执行情况，默认120s执行一次
monitor -c 5 demo.MathGame primeFactors 5s监视一次demo.MathGame的primeFactors方法


## watch 观察指定方法的调用情况，查看入参、返回值、抛出异常
watch demo.MathGame primeFactors "{params,returnObj}" -x 2 查看primeFactors的入参和返回值，便利深度为2
watch demo.MathGame primeFactors "{params,returnObj}" -x 2 -b 方法执行前监控，无返回值
watch demo.MathGame primeFactors "target" -x 2 -b 查看方法执行前MathGame对象的所有属性
watch demo.MathGame primeFactors "target.illegalArgumentCount" -x 2 查看某一个属性
watch demo.MathGame primeFactors "{params,target,returnObj}" -x 2 -b -s -n 2 执行两次（-n 2） 查看方法执行前、方法执行后入参、出参、MathGame属性信息
watch demo.MathGame primeFactors "{params,target}" "params[0]<0" 使用ognl表达式，判断入参小于0的情况

## trace 对方法内部调用路径进行追踪
trace demo.MathGame run 追踪run方法
trace demo.MathGame run -n 2 追踪两次退出
trace --skipJDKMethod false demo.MathGame run -n 2 追踪jdk内部的方法
trace --skipJDKMethod false demo.MathGame run -n 2 '#cost > .5'  run方法调用时间大于0.5ms

## stack 输出方法被调用的路径
stack demo.MathGame primeFactors 追踪primeFactors的调用路径
stack demo.MathGame primeFactors 'params[0] < 0' -n 2 追踪两次第一个入参小于0的情况
stack demo.MathGame primeFactors '#cost > 0.5' -n 2 追踪耗时大于0.5ms的情况两次

## tt （time-tunel） 记录指定方法每次调用的入参和返回值信息，
就是记录下当前方法的每次调用环境现场
tt -t demo.MathGame primeFactors 记录某个方法在一个时间段中的调用
tt -l 显示所有已经记录的列表
tt -n 记录多少次
tt -s 'method.name=="primeFactors"' 搜索表达式,查找primeFactors方法的调用
tt -i 1011 查看索引号1011的详细调用信息
tt -i 1011 -p --replay-times 3 对方法调用索引号1011重新调用3次
tt -i 1011 -p --replay-times 3 --replay-interval 2 间隔2s调用一次
tt --delete-all  清除方法调用的所有记录

## options 全局开关
options unsafe true 对系统级别的类进行增强，可能导致JVM挂掉
options dump true 是否将增强的类保存到外部文件

## heapdump 堆转储
类似于 jmap -dump命令

## profiler 火焰图
profiler start 启动火焰图
profiler list 查看profiler支持的命令
profiler getSamples 获取当前的样本数
profiler status 查看profiler状态
profiler stop --format html 生成火焰图

y轴表示调用栈，调用栈越深
x轴表示抽样数量

火焰图就是看顶层哪个函数占用宽度最大，存在平顶时表示该函数可能存在性能问题


### 使用arthas排查jvm内存溢出问题
1. 首先使用dashboard命令查看系统的堆内存占用情况
2. 使用heapdump将系统中内存快照dump下来
3. 或使用classloader 查看系统中类加载器加载类的情况
4. 使用sc -d 查看类加载的详细信息

### 使用arthas排查线上服务器cpu占用过高问题
1. 使用dashboard 查看占用cpu时间多的线程
2. 或是使用thread -n 5 查看占用cpu时间最高的几个线程
3. 观察线程堆信息，使用jad com.steven.jvm.TestMemLeak main 反编译具体的方法，查看方法中的代码逻辑是否存在死循环或是方法是否存在被循环调用的情况

## 使用arthas不停机解决线上问题
当线上bug比较急时可以使用如下的方式进行问题修复
1. 在服务器上部署arthas
2. 使用jad --source-only 类全限定名 > /test/Test.java 反编译class文件为源码,使用其他编辑器修改源码
3. 使用mc -c 类加载器hashocde /test/Test.java -d out/ 重新编译修改好的源码
4. retransform out/Test.class 重新加载新的字节码
当服务重启后，retransform的字节码会失效












