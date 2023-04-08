# Spark

谷歌三驾马车： GFS(hdfs)、 MapReduce、BigTable(Hbase)
分布式文件系统 GFS  （存储引擎）   ---存储数据
> block,replication
> nosql数据库 hbase
> kudu存储（集成hdfs和hbase）
> 分布式消息队列kafka
> 分布式搜索引擎 elasticSearch
> kv内存数据库 redis
并行计算引擎 MapReduce （分而治之）   ---分析数据
> 数据仓库工具 hive，提供sql分析数据，转换为MR，读取hdfs上的数据，运行yarn上
> 内存分析引擎 Impala
> 分布式OLAP kylin、druid
> 统一分析引擎 spark  批处理、离线分析
> 实时流式分析引擎 flink 流式处理、实时分析 --阿里使用的较多
辅助框架  
> 分布式集群管理 YARN 
> 数据转换（关系数据到hive表） SQOOP、Kettle
> 日志辅助搜集 logStach、Flum、FileBeats
> 调度框架 Mesos、Oozie（HUE）

## 基本概念

**基于内存的快速、通用、可扩展的大数据分析计算引擎**
spark不做数据存储，做海量的数据分析，提供从hdfs、hive、mysql、kafka等存储工具中读取数据并分析，spark更擅长于批处理，离线分析
spark中核心的数据结构： RDD 弹性分布式数据集 --Resilient Distributed Dataset
spark处理数据是，将数据封装到集合RDD中，RDD中有很多partition，每个partition被一个task处理
spark中每个task任务以线程Thread运行

spark特点
Simple --简单
Fast --快
Scalable -- 可扩展的
Unified --统一的，能从任何地方读取数据 hdfs，csv，parquet，rdbms，es，redis，kafka等
spark与hadoop处理数据的区别：
* spark处理数据时，可以将中间处理结果数据存储到内存中
* spark job调度以DGA方式，每个任务task以线程的方式运行，而mapreduce以进程方式运行

**spark 架构**
spark core ：spark最基础与最核心的功能，spark的其他功能如spark sql，spark streaming 都是在core基础上进行扩展的 数据结构：RDD
spark sql： 用来操作结构化数据的组件，可以使用sql或hive的sql方言来查询数据 ，sql，数据结构： DataFrame/Dataset = RDD+ Schema
spark streaming： 针对实时数据进行流式计算的组件，提供丰富的数据流api
spark MLlib: 提供机器学习算法库 
spark GraphX：面相图计算提供的框架与算法库
spark structuredStreaming 使用结构化方式处理流式数据


hadoop中的mapreduce适合一次性计算，不适合复杂多流程的数据计算，一个map只能有一个reduce，多个mapreduce会进行多次磁盘io
spark可以基于内存进行计算，多个作业之间的通信是通过内存
当内存资源受限时使用hadoop的mapreduce是一个比较好的选择


**spark基础环境**
单机模式：不需要AppManager的JVM进程，只有一个node用来执行mapreduce
集群模式：需要有一个AppManager的jvm进程管理mapreduce工作进程



**sparkcore核心RDD**
Resilient Distributed Dataset --弹性分布式数据集
是一种对数据集形态的抽象，基于此抽象，使用者可以在集群中执行一系列计算，而不用将中间结果落盘。而这正是之前 MR 抽象的一个重要痛点，每一个步骤都需要落盘，使得不必要的开销很高


## spark 部署
1. jdk安装
2. scala安装
3. 修改spark配置文件
4. spark安装包分发

两种模式：
**本地模式**
> 本地运行模式 local mode，所有的任务都运行在本地的一个JVM process进程中
* conf/spark-env.sh配置
```shell
# jdk路径
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home
# hadoop配置文件路径
HADOOP_CONF_DIR=/Users/lijie3/Documents/tool-package/hadoop-3.3.5/etc/hadoop
```
启动hadoop 
```shell
#只启动namenode和datanode
hadoop-daemon.sh start namenode
hadoop-daemon.sh start datanode
```

* 使用spark-shell
在spark的目录下直接运行`./bin/spark-shell` 出现scala的命令行，直接在命令行中执行代码即可
./bin/spark-shell --master 'local[2]'  指定两个线程

Spark context Web UI available at http://192.168.201.148:4040
Spark context available as 'sc' (master = local[2], app id = local-1680944700007).
Spark session available as 'spark'.

* 从hdfs的/spark/datas/README.md中读取文本数据并进行单词统计
简单词频统计功能：
1. 读取数据，封装到RDD集合汇总 ，从hdfs读取数据
2. 分析数据，调用RDD中的函数（RDD提供的高阶函数），flatMap,map,reduce
3. 保存数据，将最终的RDD结果数据保存到外部

```scala
# 一些api的使用
#获取rdd
var rdd = sc.textFile("/spark/datas/README.md")
# 获取第一条数据
rdd.first
# 统计总的数据条数
rdd.count
# 获取前几条数据
rdd.take(2)
数据切分
val wordsRDD  = rdd.flatMap(line => line.trim.split("\\s+"))
# 转化成tuples
val tuplesRdd = wordsRDD.map(word =>(word,1))
# 使用rdd的高阶函数
val resultRDD = tuplesRdd.reduceByKey((tmp,item) => tmp + item)
# 保存数据到hdfs
resultRDD.saveAsTextFile("/spark/datas/wc-ouput1")

# 以上词频统计可以直接使用下面的代码简化实现
sc.textFile("/spark/datas/README.md").flatMap(_.split("\\s+")).map((_,1)).reduceByKey(_ + _).saveAsTextFile("/spark/datas/wc-ouput1")
```

sc -- SparkContext用于加载数据，封装到RDD集合中，调度每个job执行

* 使用spark-submit 提交圆周率计算程序
使用`./bin/spark-submit`来通过jar包提交任务到本地任务
```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master 'local[1]' \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
```

--master spark://node1:7077 指定使用哪个spark运行

**集群模式**
管理者： AppMaster（MapReduce）、Driver Program（Spark） JobManager（Flink）
任务实施者： Nodemanager（hadoop）、executor（Spark）
> standalone ---使用内部的资源调度器
修改conf/skark-env.sh
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_291.jdk/Contents/Home
SPARK_MASTER_HOST=localhost
SPARK_MASTER_PORT=7077
```
修改workers
```
node1
node2
node3
```
* 启动：
```
./sbin/start-all.sh
```
提交任务
```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master 'spark://node1:7077' \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
```
10 ---标识执行10个任务

* 配置历史任务
./sbin/start-history-star.sh

* 集群个停止
./bin/stop-all.sh

高可用

> yarn
使用hadoop的yarn进行资源调度
```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master yarn \
 --deploy-mode cluster \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
# 使用yarn调度，使用集群模式
```
端口号
spark-shell 查看任务状况端口号 4040
spark master内部通信服务端口号 7077
standalone 模式下soark master web端口号 8080
spark 历史服务器端口号 18080
hadoop yarn任务运行情况查看端口号 8088

> k8s

> mesos


基本的wordCount实现
```scala
//使用scala的集合处理方法来实现
object SparkWordCount2 {
  def main(args: Array[String]): Unit = {

    //spark 框架

    //建立和spark框架的连接
    val sparkConf = new SparkConf().setMaster("local").setAppName("SparkWordCount")
    val sparkContext = new SparkContext(sparkConf)

    //执行业务操作
    val lines: RDD[String] = sparkContext.textFile("datas")
    //读取文件，一行一行的读取,将一行数据进行拆分，进行分词,扁平化
    val words: RDD[String] = lines.flatMap(_.split("\\s+"))
//    ('a'->1,'a'->1,'b'->1,'c'->1) 这种格式
    val wordToOne = words.map(word => (word, 1))
    //将数据根据单词进行分组便于统计
    //分组('a' ->[1,1],'b'->[1] 'c'->[1])
    val wordGroup: RDD[(String, Iterable[(String, Int)])] = wordToOne.groupBy(t => t._1)
    //对分组后的数据进行聚合，转换
    val wordToCount = wordGroup.map {
      case (word, list) => {
        list.reduce(
          (t1, t2) =>{
            //将第二个元素聚合成一个值
            (t1._1,t1._2+ t2._2)
          })
      }
    }
    //将转换结果输出
    val array: Array[(String, Int)] = wordToCount.collect()
    array.foreach(println)
    //关闭连接
    sparkContext.stop()
  }
}
```
```scala
//使用spark提供的api简化scala的集合聚合操作
object SparkWordCount3 {
  def main(args: Array[String]): Unit = {

    //spark 框架

    //建立和spark框架的连接
    val sparkConf = new SparkConf().setMaster("local").setAppName("SparkWordCount")
    val sparkContext = new SparkContext(sparkConf)

    //执行业务操作
    val lines: RDD[String] = sparkContext.textFile("datas")
    //读取文件，一行一行的读取,将一行数据进行拆分，进行分词,扁平化
    val words: RDD[String] = lines.flatMap(_.split("\\s+"))
//    ('a'->1,'a'->1,'b'->1,'c'->1) 这种格式
    val wordToOne = words.map(word => (word, 1))
    //spark可以将分组和聚合使用一个方法实现
    //相同的key的数据，可以对value reduce聚合
    val wordToCount = wordToOne.reduceByKey(_ + _)
//    //将转换结果输出
    val array: Array[(String, Int)] = wordToCount.collect()
    array.foreach(println)
    //关闭连接
    sparkContext.stop()
  }
}
```


## spark 离线分析

## spark 实时分析 spark streaming/structredStreaming


