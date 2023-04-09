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
* Simple --简单
* Fast --快
* Scalable -- 可扩展的
* Unified --统一的，能从任何地方读取数据 hdfs，csv，parquet，rdbms，es，redis，kafka等

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

* sc -- SparkContext用于加载数据，封装到RDD集合中，调度每个job执行

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
**standalone模式**

主从架构，类似与hadoop的yarn架构，管理整个集群资源，分配资源给spark Application使用
角色：
* Master
类似于ResourceManager，挂历所有资源状态，分配资源（内存，cpu核数）
* worker
类似于nodeManager，管理每个节点中的资源，启动进程，执行task任务
高可用，使用zk的强一致性实现自动切换故障主节点，实现故障转移
* HistoryServer 任务历史服务器
专门供用户查看运行完成的sparkApplication，方便监控操作

> standalone ---使用内部的资源调度器
* 在原来的local mode 方式部署的配置文件基础上修改spark-env.sh
修改conf/skark-env.sh
```shell
  #master主机及端口
  SPARK_MASTER_HOST=node1
 SPARK_MASTER_PORT=7077
 SPARK_MASTER_WEBUI_PORT=8080
 #worker常用配置
 SPARK_WORKER_CORES=1
 SPARK_WORKER_MEMORY=1g
  SPARK_WORKER_PORT=7078
 SPARK_WORKER_WEBUI_PORT=8081
 #配置历史服务器
 export SPARK_HISTORY_OPTS="
-Dspark.history.ui.port=18080
-Dspark.history.fs.logDirectory=hdfs://node1:8020/spark/history
-Dspark.history.retainedApplications=30"
```
* 修改workers
```
node1
node2
node3
```
* 修改spark-default.sh配置修改对event的日志记录配置
```shell
# 开启事件日志记录
spark.eventLog.enabled           true
# 日志记录的hdfs目录
spark.eventLog.dir               hdfs://node1:8020/spark/history
```
* 修改log4j.properties日志级别,根据实际需求进行修改
```shell
rootLogger.level = warn
```

* 启动进程：
```shell
## 启动master
./sbin/start-master.sh
## 启动所有的worker
./sbin/start-slaves.sh
## 启动历史服务器
./sbin/start-history-server.sh
## 尝试通过sparks-submit运行计算圆周率的应用 10 --标识执行1次个任务 --master指定master地址
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master 'spark://node1:7077' \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
# 正常情况下能在18080端口的页面看到运行的信息
# 停止master
./sbin/stop-master.sh
#停止worker
./sbin/stop-slaves.sh

#集群快捷启动
./bin/start-all.sh
# 集群停止
./bin/stop-all.sh
```
* 当spark application 运行到standalone集群上时，有两部分组成
    * Driver Program （AppMaster） ：jvm process，运行在master上，必须创建SparkContext上下文，且只能存在一个
    * Executors ：jvm process，java进程，运行在worker节点上运行task任务，以线程为单位来运行，Executors可以认为是一个线程池，需要获取cpu核数和内存资源
流程：
> driver 将用户程序划分成不同的执行阶段stage，执行每个阶段由一组完全相同的Task组成，task分别作用于带处理数据的不同分区，在阶段划分完成和task创建完成后，driver会向executor发送Task
一个spark application中包含多个job，每个job中有多个stage，每个job会按照DAG图进行执行

> spark application 程序运行的三个核心概念： job -> stage(一个job分为多个stage) -> Task（一个stage会分为多个Task，task处理的数据不同，处理不同分区的数据，处理数据的逻辑相同）


**高可用**

上面的集群部署存在单点故障的问题，可以使用zk的强一致性进行故障转移
* 在conf/spark-env.sh中添加高可用的配置
```shell
export SPARK_DAEMON_JAVA_OPTS="  
-Dspark.deploy.recoveryMode=ZOOKEEPER  
-Dspark.deploy.zookeeper.url=node1:2181,node2:2181,node3:2181 
-Dspark.deploy.zookeeper.dir=/spark-ha" 
#注释掉前面的master配置
#SPARK_MASTER_HOST=node1
# 由于zk会占用8080的端口这里将8080 端口改成10001
 SPARK_MASTER_WEBUI_PORT=10001
```

* 启动zk集群
```shell
zkServer.sh start
zkServer.sh status
```

* 启动集群
```shell
##node1，node2上的master
./sbin/start-master.sh
## 启动worker
./sbin/start-slaves.sh
```
HA模式下提交spark任务 --master 中指定主备master的地址
```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master 'spark://node1:7077,node2:7077' \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
```
停止node1上的master进程后，node2上的master切换成active状态需要1-2分钟
**使用yarn来做资源调度**
* 在spark-env.sh中添加yarn环境变量
```shell
export JAVA_HOME=/export/server/jdk1.8.0_131
HADOOP_CONF_DIR=/export/server/hadoop-3.3.4/etc/hadoop
YARN_CONF_DIR=/export/server/hadoop-3.3.4/etc/hadoop
```
* 配置MRHistoryServer地址，修改etc/hadoop/yarn-site.xml
```xml
<!--日志聚集开启-->
 <property>

        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
 </property>
 <!--日志保存时间-->
        <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
     <!--日志服务器地址-->
     <property>
        <name>yarn.log.server.url</name>
        <value>http://node1:19888/jobhistory/logs</value>
    </property>
    <!--禁用yarn的内存检查-->
    <property>
        <name>yarn.nodemanager.pmem-check-enabled</name>
        <value>false</value>
    </property>
    <property>
        <name>yarn.nodemanager.vmem-check-enabled</name>
        <value>false</value>
    </property>
```


* 修改spark-default.conf，添加mrHistoryServer配置
```shell
spark.yarn.historyServer.address node1:18080
# 设置sparkjar目录
spark.yarn.jars hdfs://node1:8020/spark/apps/jars
```

> 当sparkapplication应用提交运行在yarn上时，默认情况下，每次提交都需要将依赖spark相关jar包上传到yarn集群中，为了节省提交时间和存储空间，将spark相关jar包上传到hdfs目录中，设置属性告知sparkapplilcation应用
```shell
#创建目录
hdfs dfs -mkdir -p /spark/apps/jars/
#上传spark的jar包
hdfs dfs -put /export/server/spark-3.3.2-bin-hadoop3/jars/* /spark/apps/jars/
```

* 启动namenode和datanode
```shell
hadoop-daemon.sh start namenode
hadoop-daemon.sh start datanode
```
* 启动yarn服务
```shell
yarn-daemon.sh start resourcemanager
yarn-daemon.sh start nodemanager
```
* 启动mrhistoryserver服务
```shell
 mr-jobhistory-daemon.sh start historyserver
```
* 启动spark的historyserver
```shell
./sbin/start-history-server.sh
```

* 使用hadoop的yarn进行资源调度
```shell
./bin/spark-submit --class org.apache.spark.examples.SparkPi\
 --master yarn \
 --deploy-mode cluster \
./examples/jars/spark-examples_2.12-3.3.2.jar \
10
```

* 常见端口号
spark-shell 查看任务状况端口号 4040
spark master内部通信服务端口号 7077
standalone 模式下soark master web端口号 8080
spark 历史服务器端口号 18080
hadoop yarn任务运行情况查看端口号 8088

> k8s

> mesos



## 基本的wordCount实现
* 本地文件系统读取数据
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
* 从hdfs读取数据

拷贝hadoop etc/hadoop下的core-site.xml和hfds-site.xml，spark的conf下的log4j2.propeties到idea项目下的resource目录下
```scala
import org.apache.spark.{SparkConf, SparkContext}

/**
 * @description: spark实现wordcount,连接hdfs从hdfs上读取文本文件，统计词频，然后将数据写回hdfs
 * @author Administrator
 * @date 2023/4/9 11:40
 * @version
 */
object SparkWordCount {

  def main(args: Array[String]): Unit = {
    //创建sc
    val sparkConf = new SparkConf().setAppName("sparkWordCount").setMaster("local[2]")
    val sc:SparkContext = new SparkContext(sparkConf)
    //从hdfs读取文件
    val inputRDD = sc.textFile("/spark/data/wordcount.txt")
    //分析数据
    val resultRDD = inputRDD.flatMap(line => line.trim.split("\\s+")).map(word =>(word,1)).reduceByKey((tmp,item) => tmp + item)
    //简单遍历
    resultRDD.foreach(tuple =>println(tuple))
    //将结果保存到hdfs
    resultRDD.saveAsTextFile(s"/spark/output/wc-output-${System.currentTimeMillis()}")
    //关闭spark上下文
    sc.stop()
  }
}
  }
}
```
* 求topN
```scala
    //方式1 .将tuple中的两个元素交换,并按降序排列,获取前3
    val top3 = resultRDD.map(tuple => tuple.swap).sortByKey(ascending = false).take(3)
    top3.foreach(tuple => println(tuple))
    //方式2.
    val top3_2 = resultRDD.sortBy(tuple => tuple._2, ascending = false)
    top3_2.foreach(tuple => println(tuple))
    //方式3 ，当数据量比较小时使用，top方法会将RDD数据都刷到内存中
    val top3_3 = resultRDD.top(3)(Ordering.by(tuple => tuple._2))
    top3_3.foreach(tuple => println(tuple))
```

## spark应用的提交
将上面的scala开发的程序打成jar包，提交运行到standalone集群或本地模式
使用spark-submit提交jar程序到spark
命令格式：
```
 ./bin/spark-submit [options] <app jar | python file | R file> [app arguments]
 # options 包含三种参数： 基本参数，Driver Program相关参数，Executor相关参数
  --master MASTER_URL
  --class CLASS_NAME
  --driver-memory MEM 
  --driver-library-path
```
修改wordcount的代码,通过main参数列表传递输入输出信息
```scala
object SparkWordCount {

  def main(args: Array[String]): Unit = {
    if(args.length < 2){
      println("Usage :spark-submit <input> <ouput>")
      System.exit(1)
    }

    //创建sc
//    val sparkConf = new SparkConf().setAppName("sparkWordCount").setMaster("local[2]")
    val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sc:SparkContext = new SparkContext(sparkConf)
    //从hdfs读取文件
//    val inputRDD = sc.textFile("/spark/data/wordcount.txt")
//通过参数传入
    val inputRDD = sc.textFile(s"${args(0)}")
    //分析数据
    val resultRDD = inputRDD.flatMap(line => line.trim.split("\\s+")).map(word =>(word,1)).reduceByKey((tmp,item) => tmp + item)
    //简单遍历
    resultRDD.foreach(tuple =>println(tuple))
    //将结果保存到hdfs
//    resultRDD.saveAsTextFile(s"/spark/output/wc-output-${System.currentTimeMillis()}")
    //通过参数传递指定
    resultRDD.saveAsTextFile(s"${args(1)}-${System.currentTimeMillis()}")

    //关闭spark上下文
    sc.stop()
  }
}
```

将打好的jar包放入到hdfs的/spark/apps目录下
提交任务 可以通过master来指定集群
```shell
## local mode
./bin/spark-submit --master 'local[2]' \ 
--class com.steven.wordcount.SparkWordCount \
hdfs://node1:8020/spark/apps/ spark-demo.jar /spark/data/wordcount.txt /spark/output/wc-output-

## standalone HA，--queue default 任务默认队列，公平调度 --supervise 运行失败后会重启
./bin/spark-submit \
 --master 'spark://node1:7077,node2:7077' \
  --class com.steven.wordcount.SparkWordCount \
  --deploy-mode cluster \
  --supervise \
  --driver-memory 512m \
  --executor-memory 512m \
  --executor-cores 1 \
  --queue default \
  --total-executor-cores 2 \
   hdfs://node1:8020/spark/apps/spark-demo.jar  /spark/data/wordcount.txt /spark/output/wc-output-

## 使用yarn来运行词频统计 ,
./bin/spark-submit \
 --master yarn \
  --class com.steven.wordcount.SparkWordCount \
  --deploy-mode cluster \
  --supervise \
  --driver-memory 512m \
  --executor-memory 512m \
  --executor-cores 1 \
  --total-executor-cores 2 \
  --queue default \
   hdfs://node1:8020/spark/apps/spark-demo.jar  /spark/data/wordcount.txt /spark/output/wc-output-   
```

## 部署模式 DeployMode ,实际生产中使用cluster，测试开发时使用client
使用spark-submit 提交应用运行时，指定参数 --deploy-mode,有两个值可选
> client :driver program进程运行在提交应用的客户端Client，这种模式下：
  * driver进程名称为`SparkSubmit`（当在yarn上运行时会有一个ExecutorLanucher）
  * executor进程名称为`CoarseGrainedExecutorBackend` 或 `YarnCoarseGrainedExecutorBackend`

> cluster: driver Program 进程运行在集群从节点上(worker或nodemanager)
  * driver进程名称为`driverwrapper`
  * executor进程名称为`CoarseGrainedExecutorBackend`或 `YarnCoarseGrainedExecutorBackend`

## 应用程序运行在yarn上，yarn的组成
1. AppMaster,应用管理者，负责整个应用运行时资源申请（RM）以及Task运行和监控
2. 启动进程process，运行Task任务

## spark应用运行在spark集群上的组成
1. Driver program 应用管理者，负责整个应用运行时资源申请及Task运行和监控（申请运行Executor）
2. Executors，进程运行Task任务和缓存数据

* 将spark应用运行在yarn集群上时，上面的四个进程都要存在(AppMaster和Driver Program存在冲突)，会存在冲突，这时：
1. 当--deploy-mode 设置成cluster时，AppMaster与Driver合体进行资源申请，运行Executors，调度Job执行
2. 当--deploy-mode 设置成client时，由AppMaster向RM申请资源，运行Executor，由Driver 调度Job执行

## spark应用在yarn上运行的流程
* client模式：
1. driver进程启动，并向resourceManager申请资源
2. resourceManger分配container通知nodeManager启动ApplicationMaster，此时applicationMaster相当于一个ExecutorLanucher，只负责向ResourceManager申请Executor内存
3. resourceMnager收到ApplicationMaster的资源申请后分配container，ApplicationMaster在资源分配指定的nodeManager上启动Executor进程
4. executor进程启动后向Driver反向注册，Executor全部注册完成后，Driver开始执行main函数
5. 之后执行到Action算子时触发一个JOB（当执行RDD的api没有返回值，返回值不是RDD时，会触发一个新的job），并根据宽依赖开始划分Stage，每个stage生成对应的TaskSet，之后将Task分发到各个Executor上执行
缺点：


* cluster模式
1. 任务提交后会和ResourceManager通信并启动ApplicationMaster
2. resourceManger分配container，通知nodeManager启动ApplicationMaster，此时ApplicationMaster相当于Driver
3. driver启动后向ResourceManager申请executor内存，resourcemanager接收到applicationmaster的资源申请后会分配container，然后再合适的nodemanager上启动executor进程
4. executor进程启动后向Driver反向注册，
5. Executor全部注册完成后，Driver开始执行main函数，
6. 之后执行到Action算子时触发一个JOB（当执行RDD的api返回值不是RDD时，会触发一个新的job），并根据宽依赖开始划分Stage，每个stage生成对应的TaskSet，之后将Task分发到各个Executor上执行

* client模式driver运行在本地，导致本地机器负担过大，网卡流量激增
* 通常情况下dirver和yarn集群运行在同一个机房内性能上来说会更好

## wordcount中的main函数代码
1. sparkContext的构建与关闭都是在Driver中执行的
2. 其他的关于数据处理的代码（关于RDD的代码，返回值是RDD的RDD方法）都是在Executor上执行
3. 当RDD.take，RDD.collect等返回值非rdd代码执行时，会在driver上执行

spark应用运行在yarn上的四个JVM进程
1. Driver Program
2. ResourceManager
3. NodeManager
4. Executor

## RDD --Resilient Distributed Dataset --弹性分布式数据集
spark-core的核心抽象概念：
1. 数据集：RDD
2. 共享变量：
    * Accumulators --累加器
    * Broadcast variables --广播变量

### RDD的设计核心点
* 内存计算
* 适合集群
* 有容错

不可变immutable分区partition的集合collection，可以并行parallel进行计算
RDD将spark的自动容错，位置感知，任务调度，失败重试等实现细节都隐藏起来，程序员不需要再考虑这些问题，只需要专注于任务的开发

### RDD的特性
* 分区列表--是个分区的列表集合
* 计算函数--一个函数可以计算每个分片
* 依赖关系一个RDD依赖于其他RDD的计算结果
* 分区函数--（可选）键值对（如hash）的分区器，只有key value的RDD才有分区器
* 最佳位置（可选）计算每个分区的首选位置列表(例如，HDFS文件的块位置)，找到计算成本最小的位置（例如分区在hdfs的node1上，则分区的task运行在node1上）

### RDD的创建