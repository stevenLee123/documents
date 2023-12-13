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
类似于ResourceManager，管理所有资源状态，分配资源（内存，cpu核数）
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
spark.yarn.jars hdfs://node1:8020/spark/apps/jars/*.jar
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

## 关于spark中的job，stage，task
spark中的数据抽象都是RDD，支持两种类型的算子：Transformation和Action
Transformation算子的代码只有在遇到一个action算子时才会真正被执行
Transformation算子主要包括：map、mapPartitions、flatMap、filter、union、groupByKey、repartition、cache等。
Action算子主要包括：reduce、collect、show、count、foreach、saveAsTextFile等。
* job：当程序运行遇到一个action算子时，就会提交一个job，执行前面的一系列操作，通常一个任务中会有多个job
* stage：一个job通常包含多个stage，stage之间按照顺序执行。一个job中会包含多个算子操作，这些算子都是将一个父RDD转换为子RDD，如果一个父RDD只能进入到一个子RDD，比如map，union等操作，称为窄依赖（narrow  dependency），否则会形成宽依赖（wide dependency），宽依赖又称为shuffle依赖，比如groupByKey，reduceByKey。stage的划分就是根据shuffle依赖进行的，shuffle依赖是两个stage的分界点。因为数据可能存放在HDFS不同的节点上，下一个stage的执行首先要去拉取上一个stage的数据（shuffle read操作），保存在自己的节点上，就会增加网络通信和IO。Shuffle操作其实是一个比较复杂的过程，这里暂且不表。
* task ：代表着spark中最细的执行单元，task的数量代表着stage的并行度。每个RDD分区都会起一个task，rdd的分区数据决定了task的数据，每个task执行的结果生成了目标rdd的一个partition。在map阶段partition数目保持不变，reduce阶段会触发shuffle操作，可能由于聚合操作导致分区数量变化



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
* 并行化本地集合，将本地集合数据存储到RDD中，测试开发
* 加载外部存储系统数据，如hdfs，hbase，elasticsearch，kafka等  

并行化本地集合
```scala
    val sc:SparkContext = {
      val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
      //有则获取context，没有则获取
      val context = SparkContext.getOrCreate(sparkConf)
      context
    }
        //创建本地集合
    val seq = (1 to 10) .toSeq
    //本地集合转化为rdd
    val inputRDD:RDD[Int] = sc.parallelize(seq, numSlices = 2)
    inputRDD.foreach(i => println(i))
//makeRDD方式创建
//    val inputRDD = sc.makeRDD(seq)
    inputRDD.foreach(i => println(i))
    //读取文件系统
    val inputRDD2 = sc.textFile(path = "datas/")
    inputRDD2.foreach(item => println(item))
    //关闭资源
    sc.stop()
```
读取小文件,设置分区数
```scala
    //读取小文件数据
    val inputRDD:RDD[(String,String)] = sc.wholeTextFiles("datas/", minPartitions = 2)
    println(s"partition:${inputRDD.getNumPartitions}")
    inputRDD.take(10).foreach(println(_))
```
### RDD分区数配置
* rdd的分区数尽量等于集群中cpu的核心数
* 实际中为了更加充分的压榨cpu的计算资源，会把并行度设置为cpu核心数的2-3倍
* rdd分区数和启动时指定的核心数，调用方法时指定的分区数，文件数量的配置具体说明：
    * 从hdfs上加载海量数据时，rdd分区数据为block数目
    * 从hbase表加载数据时，分区数为表的region数目


调度器在分配 tasks 的时候是采用延迟调度来达到数据本地性的目的（说白了，就是数据在哪里，计算就在哪里）

### RDD函数--算子
1. RDD转换函数transformation，调用后产生一个新的RDD
    * 所有的转换函数都是懒惰Lazy的
  函数列表：
  基本函数：
    flatMap
    map
    filter
    groupBy
  分区函数
    reduceByKey
  数学函数：
  Double函数：
  Async函数：
  其他： 

2. action动作函数，无返回值或返回值不是RDD，非懒惰加载
    * 每个action函数都触发一个新的Job
    * 当执行到Action时才会执行前面的一系列transformation算子
* RDD不存储真正计算的数据
* RDD中所有的转换函数都是懒惰执行

**基本函数（大部分与scala中的函数一致）**
* map
* flatMap
* filter
* foreach
* saveAsTextFile
```scala
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$"))
    val sparkContext = new SparkContext(sparkConf)

    val inputRDD = sparkContext.textFile("datas")
    val resultRDD = inputRDD
      .filter(line => line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .map(word => (word -> 1))
      .reduceByKey((tmp, item) => tmp + item)
    resultRDD.foreach(tuple =>println(tuple))
    sparkContext.stop()
  }
```

**分区操作函数**

map,foreach这些函数都是对单条数据进行操作
可以使用针对分区的操作函数：
mapPartitions
foreachPartitions
```shell
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sparkContext = new SparkContext(sparkConf)
    //默认情况下本地的一个文件对应了一个分区
    val inputRDD = sparkContext.textFile("datas")
    val resultRDD = inputRDD
      .filter(line => line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      //使用分区函数
      .mapPartitions{iter =>
        iter.map(word => word -> 1)
      }
      .reduceByKey((tmp, item) => tmp + item)
//    resultRDD.foreach(tuple =>println(tuple))
    //分区操作函数
    resultRDD.foreachPartition {iter =>
      //获取分区编号
      val partitionId:Int = TaskContext.getPartitionId()
      iter.foreach(tuple => println(s"partitionId:${partitionId},tuple:${tuple}"))
    }
    sparkContext.stop()
  }
```
**重分区函数**
调整RDD分区数目
* 保存结果RDD时，数据量很小，可能分区很多，需要降低分区数目
* 从外部存储系统读取数据时，如hbase，默认情况下一个region对应一个分区partition，如果region太大，则需要增加分区

增加分区数： repartition --常用于增加分区，会产生shuffle 
减少分区数： coalesce --不会产生shuffle
kv类型时使用：PairRDDFunction
```scala
 //默认情况下本地的一个文件对应了一个分区,设置最小分区数为2
    val inputRDD = sparkContext.textFile("datas",minPartitions = 2)
    println(s"partitions:${inputRDD.getNumPartitions}")
    //将分区数修改成3
    val inputRDD2 = inputRDD.repartition(3)
    println(s"partitions:${inputRDD2.getNumPartitions}")
    //降低分区书
    val inputRDD3 = inputRDD2.coalesce(2)
    println(s"partitions:${inputRDD3.getNumPartitions}")
```
* 当处理的数据集很多时，可以考虑增加分区数
* 当RDD数据进行过滤filter操作后，考虑降低RDD分区数；当结果RDD存储到外部系统时，考虑降低分区数

**聚合函数**
聚合函数时，需要中间临时变量，然后持续迭代聚合
refuceByKey((tmp ,item) => tmp+item)
fold(zero:T)((tmp,item) => ...)
aggregateByKey
combineBykey
```scala
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sparkContext = new SparkContext(sparkConf)

//    reduce ，fold
//设置两个分区
    val dataRDD = sparkContext.parallelize(1 to 10,numSlices = 2)
    //reduce方法详解
    val result = dataRDD.reduce { (tmp, item) =>
      val partitionId: Int = TaskContext.getPartitionId()
      println(s"partitionId:${partitionId},tmp = ${tmp},item = ${item},sum = ${tmp + item}")
      tmp + item
    }
    println(result)
     /**
     * 先分区聚合，然后全局聚合
     * partitionId:0,tmp = 1,item = 2,sum = 3
      partitionId:0,tmp = 3,item = 3,sum = 6
      partitionId:0,tmp = 6,item = 4,sum = 10
      partitionId:0,tmp = 10,item = 5,sum = 15
      partitionId:1,tmp = 6,item = 7,sum = 13
      partitionId:1,tmp = 13,item = 8,sum = 21
      partitionId:1,tmp = 21,item = 9,sum = 30
      partitionId:1,tmp = 30,item = 10,sum = 40
      partitionId:0,tmp = 40,item = 15,sum = 55
      55
     */
      val result2 = dataRDD.fold(0) { (tmp, item) =>
      val partitionId: Int = TaskContext.getPartitionId()
      println(s"partitionId:${partitionId},tmp = ${tmp},item = ${item},sum = ${tmp + item}")
      tmp + item
    }
    //与reduce同样的思想
    println(result2)
        val result2 = dataRDD.fold(0) { (tmp, item) =>
      val partitionId: Int = TaskContext.getPartitionId()
      println(s"partitionId:${partitionId},tmp = ${tmp},item = ${item},sum = ${tmp + item}")
      tmp + item
    }
    println(result2)
    /**
     * partitionId:1,tmp = 0,item = 6,sum = 6
      partitionId:0,tmp = 0,item = 1,sum = 1
      partitionId:0,tmp = 1,item = 2,sum = 3
      partitionId:0,tmp = 3,item = 3,sum = 6
      partitionId:0,tmp = 6,item = 4,sum = 10
      partitionId:0,tmp = 10,item = 5,sum = 15
      partitionId:1,tmp = 6,item = 7,sum = 13
      partitionId:1,tmp = 13,item = 8,sum = 21
      partitionId:1,tmp = 21,item = 9,sum = 30
      partitionId:1,tmp = 30,item = 10,sum = 40
      partitionId:0,tmp = 0,item = 40,sum = 40
      partitionId:0,tmp = 40,item = 15,sum = 55
      55
     */
      //使用aggregate聚合
    //zeroValue 中间临时变量初始值
    //seqop：分区内聚合
    //combop：分区间聚合
    //获取上面数据集中最大的两个元素,首先获取分区内的最大两个元素，然后再获取分区间最大两个元素
    val result3:ListBuffer[Int] = dataRDD.aggregate(new ListBuffer[Int]())(
      (tmp: ListBuffer[Int], item: Int) => {
        //将分区数据放入listbuffer，然后排序从右边取最大的两个数
        tmp += item
        tmp.sorted.takeRight(2)
      },
      //分区间的聚合，分区聚合后都是一个listbuffer，然后再将两个listbuffer合并，取其中的最大两个值
      (tmp: ListBuffer[Int], item: ListBuffer[Int]) => {
        tmp ++= item
        tmp.sorted.takeRight(2)
      }
    )
    println(s"top2: ${result3.toList.mkString(",")}")
    sparkContext.stop()
  }
```
PairRDDFunction kv类型数据的聚合
groupByKey --不建议使用，容易产生数据倾斜
reduceByKey --  聚合结果与value类型相同
foldByKey -- 聚合结果与value类型相同
aggregateByKey -- 聚合结果与value类型相同，几乎所有的
```scala
  def main(args: Array[String]): Unit = {
    val sparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sparkContext = new SparkContext(sparkConf)
    val lineSeq:Seq[String] = Seq("hadoop scala java hive hadoop hbase storm",
      "hadoop scala spark redis elastic", "hello world hello java hello redis hello spark"
    )
    val inputRDD:RDD[String] = sparkContext.parallelize(lineSeq, numSlices = 2)
    val wordsRDD:RDD[(String,Int)] = inputRDD.flatMap(line => line.split("\\s+")).map(word => (word, 1))
    //使用groupBykey聚合
    val wordsGroupRDD:RDD[(String,Iterable[Int])] = wordsRDD.groupByKey()
    val resultRDD = wordsGroupRDD.map{
      case (word,values) =>
        val sum = values.sum
        word -> sum
    }
    println(resultRDD.collectAsMap())
    //使用reducebykey聚合
    val wordsGroupRDD2:RDD[(String,Int)] = wordsRDD.reduceByKey((tmp,item) => tmp + item)
    println(wordsGroupRDD2.collectAsMap())

    //使用aggregateBykey
    val wordsGroupRDD3:RDD[(String,Int)] = wordsRDD.aggregateByKey(0)(
      (tmp,item)=>tmp + item,
      (tmp,item) => tmp + item)
    println(wordsGroupRDD3.collectAsMap())
    sparkContext.stop()
  }
```
**关联函数join--针对PairRDDFunction**
当对两个rdd数据进行关联时，数据类型必须时kv二元组
按照key关联
* 等值join
* 左外关联join
  
 ```scala
    val empRDD = sparkContext.parallelize(Seq((1001, "zhangsan"), (1001, "lisi"),
      (1001, "wangwu"), (1002, "zhaosi"), (1003, "tianqi"), (1002, "zhaowu")))
    val deptRDD = sparkContext.parallelize(Seq((1001,"sales"),(1002,"dev"),(1003,"res")))
    //等值join
    val joinRDD: RDD[(Int, (String, String))] = empRDD.join(deptRDD)
    joinRDD.foreach{case (deptNo,(empName,deptName)) =>
        println(s"deptNo:${deptNo},empName:${empName},deptname:${deptName}")
    }
     println("*"*15)
    val leftJoinRDD: RDD[(Int, (String, Option[String]))] = empRDD.leftOuterJoin(deptRDD)
    leftJoinRDD.foreach{case(deptNo,(empName,option))=>{
      println(s"deptNo:${deptNo},empName:${empName},deptname:${option.orNull}")
    }}

    /**
     * deptNo:1001,empName:zhangsan,deptname:sales
      deptNo:1002,empName:zhaosi,deptname:dev
      deptNo:1001,empName:lisi,deptname:sales
      deptNo:1001,empName:wangwu,deptname:sales
      deptNo:1002,empName:zhaowu,deptname:dev
      deptNo:1003,empName:tianqi,deptname:res
           ***************
      deptNo:1002,empName:zhaosi,deptname:dev
      deptNo:1002,empName:zhaowu,deptname:dev
      deptNo:1001,empName:zhangsan,deptname:sales
      deptNo:1001,empName:lisi,deptname:sales
      deptNo:1001,empName:wangwu,deptname:sales
      deptNo:1003,empName:tianqi,deptname:res
     */

 ``` 
 **其他**
 keys，values,collectAsMap等

 ### RDD持久化
缓存函数
```scala
 //放在executor的内存中
    resultRDD.cache()
    //触发缓存
    resultRDD.count()
    //设置缓存级别
    resultRDD.persist()
    //释放缓存
    resultRDD.unpersist()
```
缓存级别
```scala
 val NONE = new StorageLevel(false, false, false, false)
 //磁盘
  val DISK_ONLY = new StorageLevel(true, false, false, false)
  //磁盘且副本为2
  val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
  val DISK_ONLY_3 = new StorageLevel(true, false, false, false, 3)
  //内存
  val MEMORY_ONLY = new StorageLevel(false, true, false, true)
  //内存中副本为2
  val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
  val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
  val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
  //缓存内存和磁盘，当内存不足缓存到磁盘
  val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
  val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
   //缓存内存和磁盘，当内存不足缓存到磁盘，序列化存储
  val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
  val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
  //缓存到系统内存
  val OFF_HEAP = new StorageLevel(true, true, true, false, 1)
```
释放缓存
unpersist()
何时使用缓存：
1. 当某个RDD多次被使用时
2. 当某个RDD来之不易（如大量的数据计算的到），且RDD不止使用一次

### RDD checkpoint
将RDD数据持久化到可靠的地方，如hdfs
dataRDD.checkpoint()
checkpoint和cache的区别
cache是放在disk或内存中，缓存不可靠，需要记录RDD的依赖
checkpoint由于依赖于hdfs的可靠性，不会记录RDD的依赖，不提供清除的方法
```scala
//checkpoint
    sparkContext.setCheckpointDir("/spark/datas/checkpoint/")
    resultRDD.checkpoint()
```

### 实例
hanlp的分词
```xml
        <dependency>
            <groupId>com.hankcs</groupId>
            <artifactId>hanlp</artifactId>
            <version>portable-1.8.4</version>
        </dependency>
```
```scala
  def main(args: Array[String]): Unit = {
    //使用hanlp进行分词
    val terms: util.List[Term] = HanLP.segment("中国北京天安门")
    import scala.collection.JavaConverters._
    terms.asScala.foreach(term => println(term.word))
  }
```

## spark 接入外部数据源
在企业中sparkcore常常于hbase数据库和mysql数据库进行交互，从其中读写数据

### 从hbase读写数据 hbase sink /hbase source
spark从hbase读写数据底层采用的是TableInputFormat和TableOuputFormat的方式，与mapreduce读取hbase数据完全一样
读取的数据都是kv的形式，k是rowkey，v是result
写入数据时，k是rowkey，v是put
//将数据写入hbase
```xml
    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-client</artifactId>
        <version>2.5.3</version>
    </dependency>

    <dependency>
        <groupId>org.apache.hbase</groupId>
        <artifactId>hbase-mapreduce</artifactId>
        <version>2.5.3</version>
    </dependency>
```
```scala
import com.hankcs.hanlp.HanLP
import com.hankcs.hanlp.seg.common.Term
import org.apache.hadoop.conf.Configuration
import org.apache.hadoop.hbase.HBaseConfiguration
import org.apache.hadoop.hbase.client.Put
import org.apache.hadoop.hbase.io.ImmutableBytesWritable
import org.apache.hadoop.hbase.mapreduce.TableOutputFormat
import org.apache.hadoop.hbase.util.Bytes
import org.apache.spark.rdd.RDD
import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}

import java.util

object SparkHbaseSink {

  def main(args: Array[String]): Unit = {
    val sparkConf: SparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sc: SparkContext = SparkContext.getOrCreate(sparkConf)
    //从本地文件系统中解析测试数据
    val inputRDD = sc.textFile("datas/datademo1/test1.txt",minPartitions = 4)
    val elkRDD = inputRDD
      .filter(line => line != null && line.trim.nonEmpty)
      .map { line => line.replaceAll("^['\"{}\\(\\)\\[\\]\\*&.?!,…:;，。；：！《》【】“”]+$", "") }
      .flatMap { line => {
        val terms: util.List[Term] = HanLP.segment(line.trim)
        //转换为scala集合
        import scala.collection.JavaConverters._
        terms.asScala.map { iterm =>
          (iterm.word, 1)
        }
      }
      }
    elkRDD.persist(StorageLevel.MEMORY_AND_DISK)
    //按词进行聚合
    val elkRDD1: RDD[(String, Int)] = elkRDD.reduceByKey((tmp, item) => tmp + item)
    //将RDD转换为RDD[ImmutableByteWritable]

    val putRDD: RDD[(ImmutableBytesWritable, Put)] = elkRDD1.map {
      case (word, count) => {
        //创建rowkey和PUT
        val rowKey = new ImmutableBytesWritable(Bytes.toBytes(word))
        val put: Put = new Put(rowKey.get())
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("count"), Bytes.toBytes(count.toString))
        (rowKey, put)
      }
    }
    //连接hbase并连接
    val configuration: Configuration = HBaseConfiguration.create()
    //hbase连接信息
    configuration.set("hbase.zookeeper.quorum","localhost")
    configuration.set("hbase.zookeeper.property.clientPort","2181")
    configuration.set("hbase.znode.parent","/hbase")
    configuration.set(TableOutputFormat.OUTPUT_TABLE,"wordcount_test")
    //保存到hbase
    putRDD.saveAsNewAPIHadoopFile("/spark/datas/wc-0001",
      classOf[ImmutableBytesWritable],
      classOf[Put],
      classOf[TableOutputFormat[ImmutableBytesWritable]],
      configuration)
  }

}
```
```scala
object SparkHbaseSource {

  def main(args: Array[String]): Unit = {
    val sparkConf: SparkConf = new SparkConf()
      .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
      .setMaster("local[2]")
      //设置序列化
      .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    val sc: SparkContext = SparkContext.getOrCreate(sparkConf)

    //连接hbase并连接
    val configuration: Configuration = HBaseConfiguration.create()
    //hbase连接信息
    configuration.set("hbase.zookeeper.quorum", "localhost")
    configuration.set("hbase.zookeeper.property.clientPort", "2181")
    configuration.set("hbase.znode.parent", "/hbase")
    configuration.set(TableInputFormat.INPUT_TABLE, "wordcount_test")
    //从hbase的表汇总读取数据,使用TableInputFormat加载数据
    val hbaseRDD: RDD[(ImmutableBytesWritable, Result)] =
      sc.newAPIHadoopRDD[ImmutableBytesWritable, Result, TableInputFormat](configuration,
        classOf[TableInputFormat], classOf[ImmutableBytesWritable], classOf[Result])
    // 读取到的hbase数据ImmutableBytesWritable 不支持序列化
    hbaseRDD.take(500).foreach{case(rowkey,result) =>{
//      println(s"rowkey = ${Bytes.toString(rowkey.get())}")
      //从result中获取rowkey信息
      println(s"rowkey = ${Bytes.toString(result.getRow())}")
      result.rawCells().foreach(cell =>{
        import scala.collection.JavaConverters._

        val cf = Bytes.toString(CellUtil.cloneFamily(cell))
        val column = Bytes.toString(CellUtil.cloneQualifier(cell))
        val value = Bytes.toString(CellUtil.cloneValue(cell))
        println(s"columnFamily:${cf},column:${column},value:${value}")
      })
    }}
    println(hbaseRDD.count())
  }
}
```
### spark 读写mysql数据库
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.23</version>
</dependency>
```
创建mysql表
```sql
CREATE TABLE `wordcount` (
  `id` int NOT NULL AUTO_INCREMENT,
  `word` varchar(1024) CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci DEFAULT NULL,
  `count` int DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci
```

使用jdbc的方式写入mysql数据
```scala
import com.hankcs.hanlp.HanLP
import com.hankcs.hanlp.seg.common.Term
import com.mysql.cj.jdbc.Driver
import org.apache.spark.rdd.RDD
import org.apache.spark.storage.StorageLevel
import org.apache.spark.{SparkConf, SparkContext}

import java.sql.{Connection, DriverManager, PreparedStatement}
import java.util

/**
 * @Description: RDD数据保存到mysql
 * @CreateDate: Created in 2023/4/11 12:53 
 * @Author: lijie3
 */
object SparkMysqlSink {

  def main(args: Array[String]): Unit = {
    val sparkConf: SparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sc: SparkContext = SparkContext.getOrCreate(sparkConf)
    val inputRDD = sc.textFile("datas/datademo1/test1.txt", minPartitions = 4)
    val elkRDD = inputRDD
      .filter(line => line != null && line.trim.nonEmpty)
      .map { line => line.replaceAll("^['\"{}\\(\\)\\[\\]\\*&.?!,…:;，。；：！《》【】“”]+$", "") }
      .flatMap { line => {
        val terms: util.List[Term] = HanLP.segment(line.trim)
        //转换为scala集合
        import scala.collection.JavaConverters._
        terms.asScala.map { iterm =>
          (iterm.word, 1)
        }
      }
      }
    elkRDD.persist(StorageLevel.MEMORY_AND_DISK)

    //按词进行聚合
    val elkRDD1: RDD[(String, Int)] = elkRDD.reduceByKey((tmp, item) => tmp + item)

    elkRDD1
      //降低分区数
      .coalesce(1)
      //保存到mysql
      .foreachPartition { iter => saveToMysql(iter)
      }
  }

  /**
   * 存储数据到mysql
   * @param datas
   */
  def saveToMysql(datas: Iterator[(String,Int)]): Unit = {
    Class.forName("com.mysql.cj.jdbc.Driver")
    var conn:Connection = null
    var pstmt:PreparedStatement =  null
    try {
      conn = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=UTF-8",
        "root","dxy123456")
      val insertSql = "insert into wordcount(`word`,`count`) values(?,?)"

      pstmt = conn.prepareStatement(insertSql)
      //手动提交事务
      val commit = conn.getAutoCommit
      conn.setAutoCommit(false)
      datas.foreach(wc => {
        pstmt.setString(1,wc._1)
        pstmt.setString(2,wc._2.toString)
        //加入批次
        pstmt.addBatch()
      })
      //批量插入
      pstmt.executeBatch()
      conn.commit()
      conn.setAutoCommit(commit)

    }catch {
      case e:Exception => e.printStackTrace()
    }finally {
      if(null != pstmt) pstmt.close()
      if(null != conn) conn.close()
    }
  }
}
```

### 共享变量 shared variables
分类：
广播变量： broadcase variables，类似于mapreduce中的map distributedCache 
   * 将某个变量的数据发送给所有的executor，让所有Task共享这个变量的值，变量值不可变
累加器： Accumulators 类似于mapreduce中的计数器counters
   * 计算功能 

不使用广播变量，每个task会接收一份driver的字典数据（如下面的过滤标点符号的list）
```scala
import java.util
import scala.collection.immutable

/**
 * @Description: 不使用广播变量
 * @CreateDate: Created in 2023/4/11 19:04 
 * @Author: lijie3
 */
object BroadcastVarTest {

  def main(args: Array[String]): Unit = {
    val sparkConf: SparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sc: SparkContext = SparkContext.getOrCreate(sparkConf)
    //过滤非单词数据 ['"{}\(\)\[\]\*&.?!,…:;，。；：！《》【】“”] ,不实用广播变量
    val list: List[String] = List(",","\"","{","}",";",".","<",">","+","-","'","，","：","？","！","【","】","[","]","#","$","%","'")
    val inputRDD = sc.textFile("datas/datademo1/test1.txt",minPartitions = 4)
    val elkRDD = inputRDD
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap { line => {
        val terms: util.List[Term] = HanLP.segment(line.trim)
        //转换为scala集合
        import scala.collection.JavaConverters._
        terms.asScala.map { iterm =>
          (iterm.word, 1)
          //不使用广播变量进行过滤
        }.filter(tuple => !list.contains(tuple._1))
      }
      }
    elkRDD.persist(StorageLevel.MEMORY_AND_DISK)
    //按词进行聚合
    val elkRDD1: RDD[(String, Int)] = elkRDD.reduceByKey((tmp, item) => tmp + item)
    elkRDD1.take(200).foreach(println)
  }
}
```
使用广播变量,发送给每个executor一份，按executor进行分发字典数据
```scala
import com.hankcs.hanlp.HanLP
import com.hankcs.hanlp.seg.common.Term
import org.apache.spark.broadcast.Broadcast
import org.apache.spark.rdd.RDD
import org.apache.spark.storage.StorageLevel
import org.apache.spark.util.LongAccumulator
import org.apache.spark.{SparkConf, SparkContext}

import java.util
import scala.collection.immutable

/**
 * @Description: 使用累加器实现词频统计累加
 * @CreateDate: Created in 2023/4/11 19:04 
 * @Author: lijie3
 */
object BroadcastVarTest {

  def main(args: Array[String]): Unit = {
    val sparkConf: SparkConf = new SparkConf().setAppName(this.getClass.getSimpleName.stripSuffix("$")).setMaster("local[2]")
    val sc: SparkContext = SparkContext.getOrCreate(sparkConf)
    //过滤非单词数据 ['"{}\(\)\[\]\*&.?!,…:;，。；：！《》【】“”] ,
    val list: List[String] = List(",","\"","{","}",";",".","<",">","+","-","'","，","：","？","！","【","】","[","]","#","$","%","'")
    //进行广播，使用广播变量
    val broadcastList: Broadcast[List[String]] = sc.broadcast(list)
    //定义累加器，记录单词为符号的数据的个数
    val numberAccum: LongAccumulator = sc.longAccumulator("number_accum")

    val inputRDD = sc.textFile("datas/datademo1/test1.txt",minPartitions = 4)
    val elkRDD = inputRDD
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap { line => {
        val terms: util.List[Term] = HanLP.segment(line.trim)
        //转换为scala集合
        import scala.collection.JavaConverters._
        terms.asScala.map { iterm =>
          (iterm.word, 1)
          //不使用广播变量进行过滤
//        }.filter(tuple => !list.contains(tuple._1))
          //使用广播变量
        }.filter(tuple => {
          //接收广播变量
          val blist = broadcastList.value
          val isFlag = blist.contains(tuple._1)
          if(isFlag){
            numberAccum.add(1L)
          }
          !isFlag
        })
      }
      }

    elkRDD.persist(StorageLevel.MEMORY_AND_DISK)
    //按词进行聚合
    val elkRDD1: RDD[(String, Int)] = elkRDD.reduceByKey((tmp, item) => tmp + item)
    elkRDD1.take(200).foreach(println)
    //获取累加器的值,需要action函数触发
    println(s"number_accum:${numberAccum.value}")
  }
}
```

## spark SQL
* 使用最多的模块，类似于hive框架，功能远大于hive框架
* 前身是shark（spark on hive），shark 主要的功能是将SQL转换为RDD（hive是将SQL转化为mapreduce）
* 2015年开发catalyst，移除了原来对hive的依赖代码
* spark1.3 提出DataFrame = RDD[Row] + schema
* 概念： Row：一行数据,schema:约束，字段名称fieldName和字段类型FieldType
* 数据结构 DataFrame，底层基于RDD，来源于Python中的数据分析库Pandas 
* spark1.6出现了DataSet，从flink借鉴
* spark2.0之后，将Dataset和DataFrame合并
* Dataset= RDD+Schama, DataFrame[Row] = DataSet
* 交互式处理
* 如何将各种数据源的数据封装到DataFrame/DataSet

**特点**
* 针对结构化数据的处理：存在schema信息
* 数据结构Dataset
* 分布式引擎，将SQL转化为RDD
* sparkSQL既可以处理离线数据（spark.read），还可以处理实时流式数据(spark.readStream)

### 基本用法
* spark sql中使用SparkSession（与SparkContext类似）来加载不同的数据源

简单加载文本数据
```scala
object SparkSessionPoint {

  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    //使用SparkSession加载文本
    // DataFrame = RDD[row]+Schema
    //Dataset = RDD +schema
    val inputDS: Dataset[String] = spark.read.textFile("datas")
    //获取schema信息
    println("schema: ")
    inputDS.printSchema()
    println("show:")
    inputDS.show(10)
    //关闭资源
    spark.stop()
  }
}
```
使用DSL分析统计 （domain specific language） 特定领域语言
```scala
object SparkDSLWordCount {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    //使用SparkSession加载文本
    // DataFrame = RDD[row]+Schema
    //Dataset = RDD +schema
    val inputDS: Dataset[String] = spark.read.textFile("datas/")
    // 过滤数据,进行切分
    //类似于sql中的select .. where
    val dataSet: Dataset[String] = inputDS
      .filter(line => null != line && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
    //类似于sql中的 group by
    val resultDF: DataFrame = dataSet.groupBy("value").count()
    // |-- value: string (nullable = true)
    // |-- count: long (nullable = false)
    resultDF.printSchema()
    resultDF.show(100, truncate = true)
    //关闭资源
    spark.stop()
  }
}
```

使用sql进行分析
```scala
//将dataset注册为临时视图 tmp view ，编写sql语句并进行分析
object SparkSQLWordCount {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    //使用SparkSession加载文本
    // DataFrame = RDD[row]+Schema
    //Dataset = RDD +schema
    val inputDS: Dataset[String] = spark.read.textFile("datas/")
    // 过滤数据,进行切分
    //类似于sql中的select .. where
    val dataSet: Dataset[String] = inputDS
      .filter(line => null != line && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
   //注册dataset为临时视图
    dataSet.createOrReplaceTempView("tmp_view_words")
    //编写sql并执行
    val resultDf = spark.sql(
      """
        |select value, count(1) as total from tmp_view_words group by value
        |""".stripMargin)
    resultDf.printSchema()
    resultDf.show(100,truncate = true)
    //关闭资源
    spark.stop()
  }
}
```
DSL与SQL两种方式的DAG图是一样的，性能是一样的

### DataFrame
在RDD的基础上赋予schema信息
RDD相当于知道数据的外部定义，类似于一行数据
DataFrame相当于知道一行数据的每一列
特点：
1. 分布式数据集，以列的方式组合，相当于schema的RDD
   
启动spark shell,通过scala的命令行操作dataset，使用的是hdfs中的json数据
```scala
./bin/spark-shell --master 'local[2]'
val empDf = spark.read.json("/spark/datas/examples/employees.json")
# 能解析到schema类型
# empDf: org.apache.spark.sql.DataFrame = [name: string, salary: bigint]
empDf.show()
# +-------+------+
|   name|salary|
+-------+------+
|Michael|  3000|
|   Andy|  4500|
| Justin|  3500|
|  Berta|  4000|
+-------+------+
empDf.schema  #schema是StructType 类型
// res2: org.apache.spark.sql.types.StructType = StructType(StructField(name,StringType,true),StructField(salary,LongType,true))
empDf.rdd
// res3: org.apache.spark.rdd.RDD[org.apache.spark.sql.Row] = MapPartitionsRDD[12] at rdd at <console>:24
val first = empDf.first() //取第一行数据，拿到的是Row，是一个弱类型，获取不到row中数据的详情，需要结合schema才能拿到具体的字段信息
first.getAs[String]("name") //根据字段名获取
```
**创建schema**
```scala
import org.apache.spark.sql.types._

val schema = StructType(
  Array(
    StructField("name", StringType, nullable = false),
    StructField("age", StringType, nullable = false)
  )
)
```
### RDD转换DataFrame
DataFrame = RDD[Row] + Schema
转换方式：
1. 通过反射方式 RDD[caseClass] 是样例类
```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{Dataset, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}

/**
 * @Description: 反射转换RDD到dataframe
 * @CreateDate: Created in 2023/4/11 23:11 
 * @Author: lijie3
 */
object SparkRDDInferring {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    val sc: SparkContext = spark.sparkContext;
    val rawRDD: RDD[String] = sc.textFile("datas/6.txt", minPartitions = 2)
    //通过反射转换
    val personRDD: RDD[Person] = rawRDD.mapPartitions { iter =>
      iter.map {
        line => {
          val Array(name, salary) = line.trim.split("\\s+")
          Person(name, salary.toInt)
        }
      }
    }
    //直接将RDD转换为dataframe
    val personDF = personRDD.toDF()
    personDF.printSchema()
    personDF.show()
    //转换为DS
    val personDS = personRDD.toDS()
    personDS.printSchema()
    personDS.show()
    spark.stop()
  }
}
```

2. 自定义Schema
```scala
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{Dataset, SparkSession}
import org.apache.spark.{SparkConf, SparkContext}

/**
 * @Description: 反射转换RDD到dataframe
 * @CreateDate: Created in 2023/4/11 23:11 
 * @Author: lijie3
 */
object SparkRDDInferring {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    val sc: SparkContext = spark.sparkContext;
    val rawRDD: RDD[String] = sc.textFile("datas/6.txt", minPartitions = 2)
    //通过反射转换
    val personRDD: RDD[Person] = rawRDD.mapPartitions { iter =>
      iter.map {
        line => {
          val Array(name, salary) = line.trim.split("\\s+")
          Person(name, salary.toInt)
        }
      }
    }
    //直接将RDD转换为dataframe
    val personDF = personRDD.toDF()
    personDF.printSchema()
    personDF.show()
    //转换为DS
    val personDS = personRDD.toDS()
    personDS.printSchema()
    personDS.show()
    spark.stop()
  }
}
```
3. toDF函数隐式转换
当RDD中的数据为元组时，可以直接调用toDF函数，指定列名称，将RDD转换为DataFrame
```scala
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{DataFrame, SparkSession}

/**
 * @Description: toDF函数的使用
 * @CreateDate: Created in 2023/4/11 23:22 
 * @Author: lijie3
 */
object SparkRDDInferring3 {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    val sc: SparkContext = spark.sparkContext;

    val rawRDD = sc.textFile("datas/6.txt", minPartitions = 2)
    //解析每行数据封装到ROW对象中
    val rowRDD: RDD[(String, Int)] = rawRDD.mapPartitions { iter =>
      iter.map {
        line => {
          val Array(name, salary) = line.trim.split("\\s+")
          (name, salary.toInt)
        }
      }
    }
    //直接在toDF中指定具体的字段名，隐式转换
    val resultDF: DataFrame = rowRDD.toDF("name", "salary")
    resultDF.printSchema()
    resultDF.show()
    spark.stop()
  }
}
```

**完整案例，数据加载-》数据分析-》结果存储**
```scala
import com.hankcs.hanlp.HanLP
import com.hankcs.hanlp.seg.common.Term
import org.apache.spark.sql.{Dataset, SaveMode, SparkSession}

import java.util
import java.util.Properties


/**
 * @Description: 使用SQL分析数据
 * @CreateDate: Created in 2023/4/11 21:48 
 * @Author: lijie3
 */
//将dataset注册为临时视图 tmp view ，编写sql语句并进行分析
object SparkSQLWordCount {
  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    //使用SparkSession加载文本
    // DataFrame = RDD[row]+Schema
    //Dataset = RDD +schema
    val inputDS: Dataset[String] = spark.read.textFile("datas/test1.txt")
    // 过滤数据,进行切分
    //过滤中英文标点符号
    val regex = "[\\p{P}\\p{S}]".r
    //类似于sql中的select .. where
    val dataSet: Dataset[String] = inputDS
      .filter(line => null != line && line.trim.nonEmpty)
      //过滤标点符号
      .map { line => regex.replaceAllIn(line, "") }
      .flatMap(line => {
        val terms: util.List[Term] = HanLP.segment(line.trim)
        //转换为scala集合
        import scala.collection.JavaConverters._
        terms.asScala.map(term => term.word)
      })
    //注册dataset为临时视图
    dataSet.createOrReplaceTempView("tmp_view_words")
    //编写sql并执行
    val resultDf = spark.sql(
      """
        |select value, count(1) as total from tmp_view_words group by value
        |""".stripMargin)
    resultDf.printSchema()
    resultDf.show(100, truncate = true)
    //缓存数据
    resultDf.cache()
    //存储数据到mysql
    resultDf
      .write
      .mode(SaveMode.Overwrite)
      .option("driver", "com.mysql.cj.jdbc.Driver")
      .option("user", "root")
      .option("password", "dxy123456")
      .jdbc("jdbc:mysql://localhost:3306/bigdata?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=UTF-8",
        table = "wordcount", new Properties())
    //存储数据到csv,数行时列名称
    resultDf.write.mode(SaveMode.Overwrite).option("header", "true").csv("datas/output/")
    //关闭资源
    spark.stop()
  }
}
```
### shuffle 设置
spark3.0之前，shuffle默认200个分区，会开启200个任务
spark3.0之后，shuffle分区会自动调整
配置项： `spark.sql.shuffle.partitions`

**加载load和保存save数据**
spark sql 可以将各种外部数据源加载（load）到spark，也可以将spark中的dataframe数据存储（save）到外部数据源

读取数据的配置：
* 数据源
* 连接信息
* 触发动作
```scala
//读取json
spark.read.format("json").option("","").load()
//读取mysql
val jdbcUrl = "jdbc:mysql://localhost:3306/mydatabase"
val tableName = "mytable"
val connectionProperties = new java.util.Properties()
connectionProperties.setProperty("user", "myuser")
connectionProperties.setProperty("password", "mypassword")

val df = spark.read.jdbc(jdbcUrl, tableName, connectionProperties)
//写入数据到mysql
def writeToMySQL(df: DataFrame, url: String, table: String, user: String, password: String): Unit = {
  val properties = new Properties()
  properties.put("user", user)
  properties.put("password", password)
  df.write.mode(SaveMode.Append).jdbc(url, table, properties)
}
```
**保存时的模式SaveMode**
Append 追加
Ingore 如果数据存在，则忽略
Overwrite 如果数据存在则会覆盖
ErrorIfExists ，默认情况，如果数据存在，则报错，抛出异常

读数据示例
```scala
import org.apache.spark.sql.functions._
import org.apache.spark.sql.{DataFrame, Dataset, SparkSession}

import java.util.Properties

/**
 * @Description:
 * @CreateDate: Created in 2023/4/12 21:52 
 * @Author: lijie3
 */
object SparkSqlSourceDemo {

  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    //加载parquet数据
    val dataFrame: DataFrame = spark.read.parquet("datas/Test.parquet")
    //默认数据源就是parquet
    //    spark.read.load("datas/Test.parquet")
    dataFrame.show(10)
    //以文本加载json数据
    val jsonDF: Dataset[String] = spark.read.textFile("datas/people.json")
    //使用sql的api进行操作
    jsonDF.select(
      get_json_object($"value", "$.name").as("name"),
      get_json_object($"value", "$.age").as("age")
    ).show(10)
    //以json加载json数据
    val jsonDF2: DataFrame = spark.read
      .json("datas/people.json")
    jsonDF2.show(10)
    //读取csv数据
    val csvDF = spark.read
      .option("header", "true")
      //制表符
      .option("sep", "\\t")
      //字段类型自动推断
      .option("inferSchema", "true")
      .csv("datas/Italy.csv")
    csvDF.show(10, truncate = false)
    //读取jdbc数据
    val jdbcDF: DataFrame = spark.read.format("jdbc")
      .option("driver", "com.mysql.cj.jdbc.Driver")
      .option("user", "root")
      .option("password", "dxy123456")
      .option("url", "jdbc:mysql://localhost:3306/bigdata?useUnicode=true&zeroDateTimeBehavior=convertToNull&autoReconnect=true&characterEncoding=UTF-8")
      .option("dbtable", "wordcount")
      .load()
    jdbcDF.show(10,truncate = false)
  }
}
```

### Dataset
分布式数据集，集成RDD和DataFrame的所有优势
线程安全
dataset = RDD（外部数据类型） + Schema（内部数据类型）
rdd,dataframe,dataset之间的转换(必须导入隐式转换)
rdd -》 toDF  -》 dataframe
dataframe -> .rdd -> rdd
dataset-> .rdd -> rdd
rdd -> toDS -> dataset
dataframe -> dataframe.as[caseclass] -> dataset
dataset -> toDF -> dataframe

### 集成hive
集成方式：让sparkSQL能读取到hive的metastore服务即可
方式1:
配置hive-default.xml
```xml
<property>
  <name>hive.metastore.uris</name>
  <value>thrift://node1:9083</value>
</property>
```
启动thrift
`$HIVE_HOME/bin/hive --service metastore`
直接读取
```scala
spark.read.table(db_hive.emp)
```

方式2:
代码中读取数据
依赖
```xml
      <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-hive_2.12</artifactId>
            <version>3.3.2</version>
      </dependency>
```
```scala
val spark = SparkSession.builder()
  .appName("Spark Hive Thrift Integration")
  .config("hive.metastore.uris", "thrift://localhost:9083")
  .enableHiveSupport()
  .getOrCreate()
val df = spark.sql("SELECT * FROM my_database.my_table")
```

### 自定义UDF （user defined function）
udf ：一对一关系
```scala
import org.apache.spark.sql.{DataFrame, SparkSession}
import org.apache.spark.sql.functions._

/**
 * @Description: udf函数示例
 * @CreateDate: Created in 2023/4/12 22:34 
 * @Author: lijie3
 */
object SparkUDFDemo01 {

  def main(args: Array[String]): Unit = {
    //构建sparksession
    val spark: SparkSession = SparkSession.builder()
      .appName(this.getClass.getSimpleName.stripSuffix("$"))
      .master("local[2]")
      .getOrCreate()
    //导入隐式转换函数库
    import spark.implicits._
    //以文本加载json数据
    val jsonDF2: DataFrame = spark.read
      .json("datas/people.json")
    //自定义udf,将name字段进行大写转化
    val c_to_upper = udf((name:String) => {name.toUpperCase})
    //DSL中使用
    jsonDF2.select(c_to_upper($"name").as("name"),$"age").show()
    //注册dataframe为临时视图
    jsonDF2.createOrReplaceTempView("person_view")
    //sql中使用,注册
    spark.udf.register("c_to_lower",(name:String) => {name.toLowerCase})
    spark.sql(
      """
        |select c_to_lower(name) ,age from person_view
        |""".stripMargin)
      .show()
  }
}
```

### 分布式sql引擎
使用spark-shell,直接在命令行中执行sql
```shell
./bin/spark-sql --master 'local[2]' --conf spark.sql.shuffle.partitions=4
```
集成sparkThriftServer，通过jdbc cli在代码中连接
### 离线分析数据的步骤

### 调度框架hue和Oozie


### catalyst优化器

## spark 内核调度

### 基本概念
* 在spark中当RDD调用action函数时（返回值非rdd），触发一个job

* 宽依赖与窄依赖
  * 窄依赖：表示父亲 RDDs 的一个分区最多被子 RDDs 一个分区所依赖。 map操作是一个窄依赖 （narrow dependency）
  * 宽依赖：表示父亲 RDDs 的一个分区可以被子 RDDs 的多个子分区所依赖。join，reduceByKey等操作是一个宽依赖 （shuffle dependency）
  * 对与窄依赖来说数据可以并行计算，窄依赖的流程类似于pipeline（管道模式），当子rdd的分区丢失时，只需要查找对应父rdd的一个分区重新计算即可

* DAG stage的划分
  * 对与窄依赖，rdd数据不需要shuffle，多个数据处理可以在同一个机器的内存中完成，所以窄依赖在saprk中被划分为同一个stage
  * 对与宽依赖，由于shuffle的存在，必须等到父rdd的shuffle处理完成后才能进行接下来的计算，所以会在shuffle处进行stage的切分

* spark中的DAG生成流程关键在与回溯，调度器将RDD堪称一个stage，遇到shuffle就断开，遇到窄依赖就归并到同一个stage，所有任务完成后就生成了一个DAG图
 
### spark shuffle
* mapreduce的shuffle
mapreduce中的shuffle分为read shuffle和write shuffle。
map的结果首先写入一个buffer，当buffer使用达到80%时，会对数据进行分区，排序，然后合并成大文件放到磁盘中存储。
reduce阶段从磁盘中拉取数据，然后进行reduce聚合输出
* spark的shuffle与mapreduce的shuffle的过程类似
* spark的DAG
spark中上游stage类似于map，下游stage类似于reduce
spark的shuffle分为write和read两个阶段，属于两个不同的stage，write是parent stege的最后一步，read是child stage的第一步
spark1.1之前是用hash shuffle，之后都是用sort shuffle，现在都是**sort shuffle**

**spark调度两个层面**
RDD的数据如果不显示的缓存（cache、persist）是不直接放在内存中的，是靠分区的迭代器来对数据进行操作的，
创建sparkcontext对象时，就会创建DAGScheduler和TaskScheduler对象，先创建taskScheduler再创建DAGScheduler
* DAGshceduler：将job对应的DAG图划分为stage，划分依据：RDD之间是否产生shuffle，将DAG切分成若干个stage，并将stage打包成TaskSet讲给Taskscheduler
* TASKScheduler：对每个stage中的task调度执行，运行在Executor，每个task以线程方式运行，需要一核cpu，采用pipeline管道计算模式
  * TASKScheduler将TaskSet按照指定的调度策略分发到Executor执行，调度过程中SchedulerBackend负责提供可用资源，SchedulerBackend对应的就是资源管理系统
  * 调度默认策略： FIFO（先进先出）
  * task是stage的子集，以并行度（分区数量来衡量），分区数量是多少就有多少task任务
**task并行度**
saprk中各个stage的并行度是以job的最大并行度为准
* 资源并行度：由节点数和cpu决定
* 数据并行度：即task的数量，或partition数量

**task数量的设置**
将task数量设置成Application总CPU core数量相同（理想情况）
官方推荐task数量设置成总CPU core数量的2～3倍
例如30G的数据，128M一个分区，则存在240个分区，则需要分配240/3=80个核心或240/2 = 120核心

并行度设置参数： spark.default.parallel = 10

## spark流式处理
流式计算的两种思想：
* 原生流处理： 实时的数据来一条就处理一条，每条数据进入流处理系统，依次处理，实施性高，saprk structured streaming支持
* 微批处理方式： 每次处理一个时间窗口的数据，按时间间隔划分流为批次，对每批次数据进行处理，
### spark流处理的数据结构 
sparkStreaming 的数据结构DStreaming
structuredStreaming 的数据结构DataFrame，属于sparksql的一部分

## saprk streaming
* 数据结构：Dstreaming
* 流处理入口： StreamingContext
* Lambda架构：
    * 集成离线分析和实时计算
    * 实时计算：kafka ->sparkstreaming,flink -> redis
    * 离线分析分析：hbase -> sparksql,hbase -> rdbms
    * lambda架构的三层架构：batch layer ：批处理，离线分析；speed layer：速度层，实时分析；serving layer，服务层，存储批处理层和速度层
* 按照时间间隔batchInterval划分流式数据，每批次数据当作RDD，然后对RDD数据进行分析处理，流是分散的，在sparkstreaming中将这种流称为DStreaming = Seq[RDD] 
* 基于spark core的实时计算,介于批处理和实时数据处理的一种流处理

#### 接收socket信息流
```shell
# 发送单词数据
nc -lk 9999
# 启动saprk示例代码
./bin/run-example streaming.NetworkWordCount localhost 9999
```
wordcount代码示例
```scala
import org.apache.spark.SparkConf
import org.apache.spark.sql.catalyst.expressions.Second
import org.apache.spark.streaming.dstream.{DStream, ReceiverInputDStream}
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * @Description:
 * @CreateDate: Created in 2023/4/14 09:07 
 * @Author: lijie3
 */
object StreamingWordCount01 {

  def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc:StreamingContext = {
      //本地运行时local[3]
      val sparkConf =  new SparkConf().setMaster("local[3]").setAppName(this.getClass.getSimpleName.stripSuffix("$"))
    //设置时间间隔为5s
      new StreamingContext(sparkConf,Seconds(5))
    }
    //从tcp socket读取数据
    val inputDStream: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 9999)
    //每批次进行词频统计
    val resultDStramming: DStream[(String, Int)] = inputDStream
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .map(word => word -> 1)
      .reduceByKey((tmp, item) => tmp + item)
    //打印结果
    resultDStramming.print(10)

    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true,stopGracefully = true)
  }
}
```
流聚合

```scala
 //从tcp socket读取数据,按时间blockInterval（默认200ms）间隔划分block，设置存储级别
    val inputDStream1: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 9999,StorageLevel.MEMORY_AND_DISK)
    //监听多个端口
    val inputDStream2: ReceiverInputDStream[String] = ssc.socketTextStream("localhost", 8888)
    //聚合多个流
    val inputDStream = ssc.union(Seq(inputDStream1,inputDStream2))
```
**本地运行时local[3]**
  * 从socket读取数据每个端口启动一个Recevier接收器
  * 每个接收器以线程Thread方式运行Task，实时接收数据
  * 运行Receiver，使用1核cpu
  * 剩余的2核cpu用来流计算
  * 所以本地模式至少需要2核cpu的设置，一个用来接收数据，一个用来计算流数据

监控中的几个概念
scheduling delay：延迟调度
processing time：处理时间
total delay = scheduling delay + processing time 总延迟
**流的性能衡量：**
total delay 总延迟 <= batchInterval批次时间间隔，此时是实时处理每批次处理数据
**工作原理分析**
1. StreamContext创建底层使用的是SparkContext
2. ssc.start() 启动一个或多个Receiver来接收流数据
3. receiver接收到数据后将流数据按时间间隔划分成block，时间间隔为BlockInterval，默认时间间隔200ms，并设置存储级别
4. receiver将流数据的block存储位置报告给StreamContext，再上报给SparkContext
5. sparkContext启动executors进行RDD的处理流程

时间间隔：
1. 批处理时间间隔：BatchInterval
2. Block时间间隔BlockInterval 默认200ms 

将BatchInterval 时间内的流数据划分为多个BlockInterval，对应RDD的多个分区，统一处理
  
#### DStream   
离散数据流，本质上是一系列时间上连续的RDD（`Seq（RDD)`),按时间间隔累积的流数据
* DStreamde 函数使用
  * Transformations:将一个DStream转换为另一个DStream
      * map、flatMap、filter
      * count、reduce、countBykey，countByValue
      * union、join、cogroup
      * transform函数（很重要）
  * Ouput Operations：将DStream中每批次RDD处理结果resultRDD输出
      * print（）打印、saveAsTextFiles、saveAsObjectFiles、saveAsHadoopFiles、foreachRDD（很重要）
**在实际开发中，能对RDD操作就针对RDD操作，不要针对DStream进行操作，使用transform转化为RDD**  
```scala
    val resultRDD = inputDStream.transform { rdd =>
      val batchRDD: RDD[(String, Int)] = rdd
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
//    resultRDD.print(10)
    //使用foreachRDD来进行操作

    //自定义打印输出
    inputDStream.foreachRDD {
      (resultRDD,batchTime) =>{
        //将batchTIme进行转换
        val formatTime = FastDateFormat.getInstance("yyyy-MM-dd HH:mm:ss").format(batchTime.milliseconds)
        println("-"*30)
        println(s"time: ${formatTime}")
        if(!resultRDD.isEmpty()){
          //降低分区数，并输出
          resultRDD.coalesce(1).foreachPartition(iter => iter.foreach(println))
        }
      }
    }
```

###  典型应用场景
电商实时大屏统计、商品推荐数据、工业数据的实时监控、集群监控等
  * 预警
  * 实时报告
  * 实时增量ETL
  * 实时决策
  * 在线机器学习

###  针对的不同的使用场景，使用不同的函数
  * 无状态stateless业务，大多数业务都是无状态的--实时增量ETL，使用transform和forechRDD
  * 有状态state，数据分析，实时报告-- 使用updateStateBykey，mapWithState
  * 窗口统计--每隔段时间对流数据进行统计，类似于日志的按时间滚动后进行切分，使用window相关函数

### 集成kafka
国内对mysql数据binlog的日志变更是通过cancal（国外使用maxwell）进行监控然后发送到kafka
spark消费kafka的两种客户端
0.80版本的kafka，使用老的消费方式，存在两种模式：
  * 1.Direct方式，spark主动pull消息，消费的速率由spark自身控制
使用案例：
创建kakfa topic
```shell
./bin/kafka-topics.sh --create --topic wc-topic --partitions 3 --replication-factor 1  --bootstrap-server localhost:9092
```
发送消息
```shell
./bin/kafka-console-producer.sh --topic wc-topic --bootstrap-server localhost:9092
> hello world
```
依赖添加

```xml
<dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-streaming-kafka-0-8_2.11</artifactId>
            <version>2.4.8</version>
        </dependency>
```
```scala
 def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]").setAppName(this.getClass.getSimpleName.stripSuffix("$"))
       //设置每批次RDD中各个分区数据的最大值，每个分区每秒的最大数据量
        .set("spark.streaming,kafka.maxRatePerPartition","1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(5))
      //使用old consumer api direct消费topic

    }
    val kafkaParams: Map[String, String] = Map(
      "bootstrap.servers" -> "localhost:9092",
      "auto.offset.reset" -> "largest")
    val topics: Set[String] = Set("wc-topic")
    val kafkaStream: InputDStream[(String, String)] = KafkaUtils.createDirectStream[String, String, StringDecoder, StringDecoder](
      ssc, kafkaParams, topics
    )
    //只需要获取消息value值
    val inputDStream: DStream[String] = kafkaStream.map(tuple => tuple._2)
    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val resultRDD = inputDStream.transform { rdd =>
      val batchRDD: RDD[(String, Int)] = rdd
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
    resultRDD.print(10)
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```

  * 2.reciver方式，kafka主动推送消息的方式，可能由于kafka推送消息速率和客户端消费速率不一致导致数据丢失的问题
  
0.10版本之后的kakfa

0.10版本使用心得kakfa api来获取数据 （使用Direct方式）
pom依赖
```xml
    <dependency>
        <groupId>org.apache.spark</groupId>
        <artifactId>spark-streaming-kafka-0-10_2.11</artifactId>
        <version>${spark.version}</version>
    </dependency>
```
```scala
  def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition","1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(5))


    }
    //消费位置策略
    val locationStrategy:LocationStrategy = LocationStrategies.PreferConsistent
    val topics:Iterable[String] = Set("wc-topic")
    val kafkaParams:collection.Map[String,Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy:ConsumerStrategy[String,String] = ConsumerStrategies.Subscribe(topics,kafkaParams)
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)


    //只需要获取消息value值
    val inputDStream: DStream[String] = kafkaDStream.map(record => record.value())
    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val resultRDD = inputDStream.transform { rdd =>
      val batchRDD: RDD[(String, Int)] = rdd
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
    resultRDD.print(10)
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```

* 获取kafka偏移量

```scala
  def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition","1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(5))


    }
    //消费位置策略
    val locationStrategy:LocationStrategy = LocationStrategies.PreferConsistent
    val topics:Iterable[String] = Set("wc-topic")
    val kafkaParams:collection.Map[String,Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy:ConsumerStrategy[String,String] = ConsumerStrategies.Subscribe(topics,kafkaParams)
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)


    var offsetRanges: Array[OffsetRange]  = Array.empty
    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val resultRDD = kafkaDStream.transform { rdd =>
      //获取偏移量
      offsetRanges= rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      //打印偏移量
      offsetRanges.foreach(offsetRange =>
        println(s"topic:${offsetRange.topic},partition:${offsetRange.partition},offsets:${offsetRange.fromOffset}-${offsetRange.untilOffset}"))

      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
     resultRDD.print(10)
     //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```

### updateStateByKey 按状态统计
按批次统计，并对指定的批次数据进行合并，updateStateByKey默认情况下是对开始的数据不断进行累计
```scala
  def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition","1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(5))


    }
    //消费位置策略
    val locationStrategy:LocationStrategy = LocationStrategies.PreferConsistent
    val topics:Iterable[String] = Set("wc-topic")
    val kafkaParams:collection.Map[String,Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy:ConsumerStrategy[String,String] = ConsumerStrategies.Subscribe(topics,kafkaParams)
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)
    //设置检查点目录，保存之前批次的数据
    ssc.checkpoint(s"datas/searchlogs-1001-${System.nanoTime()}")

    var offsetRanges: Array[OffsetRange]  = Array.empty
    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val reduceDStream = kafkaDStream.transform { rdd =>
      //获取偏移量
      offsetRanges= rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      //打印偏移量
      offsetRanges.foreach(offsetRange =>
        println(s"topic:${offsetRange.topic},partition:${offsetRange.partition},offsets:${offsetRange.fromOffset}-${offsetRange.untilOffset}"))

      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
    //使用Dstream中的updateStateByKey将当前批次状态与以前的状态合并更新
    val stateDStream: DStream[(String, Int)] = reduceDStream.updateStateByKey((values: Seq[Int], state: Option[Int]) => {
      var currentState: Int = values.sum
      //取以前的状态值
      val previousState: Int = state.getOrElse(0)
      val lastestState: Int = currentState + previousState
      Some(lastestState)
    })
    stateDStream.print()
     //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```
### mapWithState函数
如果没有数据输入，则不会返回之前key的状态，只关心哪些已经发生变更的key
```scala
  def main(args: Array[String]): Unit = {
    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition", "1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(5))


    }
    //消费位置策略
    val locationStrategy: LocationStrategy = LocationStrategies.PreferConsistent
    val topics: Iterable[String] = Set("wc-topic")
    val kafkaParams: collection.Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy: ConsumerStrategy[String, String] = ConsumerStrategies.Subscribe(topics, kafkaParams)
    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)
    //设置检查点目录，保存之前批次的数据
    ssc.checkpoint(s"datas/searchlogs-1001-${System.nanoTime()}")

    var offsetRanges: Array[OffsetRange] = Array.empty
    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val reduceDStream = kafkaDStream.transform { rdd =>
      //获取偏移量
      offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      //打印偏移量
      offsetRanges.foreach(offsetRange =>
        println(s"topic:${offsetRange.topic},partition:${offsetRange.partition},offsets:${offsetRange.fromOffset}-${offsetRange.untilOffset}"))

      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
    //使用Dstream中的updateStateByKey将当前批次状态与以前的状态合并更新
    //    val stateDStream: DStream[(String, Int)] = reduceDStream.updateStateByKey((values: Seq[Int], state: Option[Int]) => {
    //      var currentState: Int = values.sum
    //      //取以前的状态值
    //      val previousState: Int = state.getOrElse(0)
    //      val lastestState: Int = currentState + previousState
    //      Some(lastestState)
    //    })
    //对状态进行处理

    val spec: StateSpec[String, Int, Int, (String,Int)] = StateSpec.function((keyword: String, countOption: Option[Int], state: State[Int]) => {
      val currentState: Int = countOption.getOrElse(0)
      val previousState = state.getOption().getOrElse(0)
      val lastestState = currentState + previousState
      state.update(lastestState)
//      //打印更新后的状态数据
      (keyword,lastestState)
    })
    val stateDStream = reduceDStream.mapWithState(spec = spec)
    println("------map state with key")
    stateDStream.print()
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```

### 实时窗口统计
每隔一段时间，统计最近的一个时间窗口的数据
例如：每5分钟统计最近15秒的数据
窗口大小：windowInterval 15分钟
滑动大小：sliderInterval  5分钟
当滑动大小 == 窗口大小时，称为滚动窗口，不会出现数据的重复计算
* 窗口大小和滑动大小必须时batchInterval的整数倍

示例：每隔3秒计算最近9秒的数据
```scala
  def main(args: Array[String]): Unit = {
    //DStream数据批次大小
    val batchInterval = 2
    //窗口大小
    val windowInterval = batchInterval * 2
    //滑动大小
    val sliderInterval = batchInterval *1

    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition", "1000")
        .set("spark.serializer",classOf[KryoSerializer].getName)
      //设置时间间隔为2s
      new StreamingContext(sparkConf, Seconds(batchInterval))
    }
    //消费位置策略
    val locationStrategy: LocationStrategy = LocationStrategies.PreferConsistent
    val topics: Iterable[String] = Set("wc-topic")
    val kafkaParams: collection.Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy: ConsumerStrategy[String, String] = ConsumerStrategies.Subscribe(topics, kafkaParams)

    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)

    //这里需要进行一次转换，不转化会报ConsumerRecord序列化的错误
    val inputDStream2: DStream[ConsumerRecord[String, String]] = kafkaDStream
      .map(record => record)

    //设置窗口大小和滑动大小
    val windowDStream: DStream[ConsumerRecord[String, String]] =
      inputDStream2.window(Seconds(windowInterval), Seconds(sliderInterval))

    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val reduceDStream = windowDStream.transform { rdd =>
      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
        .reduceByKey((tmp, item) => tmp + item)
      batchRDD
    }
    println("------map state with key")
    reduceDStream.print()
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```
实时窗口统计reduceByKeyAndWindow
将窗口和聚合函数写在一起
```scala
  def main(args: Array[String]): Unit = {
    //DStream数据批次大小
    val batchInterval = 2
    //窗口大小
    val windowInterval = batchInterval * 2
    //滑动大小
    val sliderInterval = batchInterval *1

    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition", "1000")
        .set("spark.serializer",classOf[KryoSerializer].getName)
      //设置时间间隔为2s
      new StreamingContext(sparkConf, Seconds(batchInterval))
    }
    //消费位置策略
    val locationStrategy: LocationStrategy = LocationStrategies.PreferConsistent
    val topics: Iterable[String] = Set("wc-topic")
    val kafkaParams: collection.Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    val consumerStrategy: ConsumerStrategy[String, String] = ConsumerStrategies.Subscribe(topics, kafkaParams)

    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] =
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)

    //这里需要进行一次转换，不转化会报ConsumerRecord序列化的错误
    val inputDStream2: DStream[ConsumerRecord[String, String]] = kafkaDStream
      .map(record => record)



    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val etlDStream = inputDStream2.transform { rdd =>
      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
      batchRDD
    }
    //将聚合与窗口设置放在一起
    val resultDStream: DStream[(String, Int)] = etlDStream.reduceByKeyAndWindow((tmp: Int, item: Int) => tmp + item,
      Seconds(windowInterval), Seconds(sliderInterval))
    println("------map state with key")
    resultDStream.print()
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }
```
使用优化后的reduceByKeyAndWindow重载函数


### 偏移量管理
在对kafka数据进行读取时，需要对消费的偏移量进行管理，来保证因为停机导致的消费信息偏移量丢失从而导致的窗口消费数据出现不准的情况
有三种方式实现偏移量的管理：
1. 设置检查点checkpoint，将消费偏移量写入hdfs文件系统中当出现停机恢复时从hdfs文件系统中恢复消费偏移量
检查点存储的数据信息： 
  * 元数据，用户恢复driver
  * 具体的数据，容错流状态的恢复
使用检查点恢复偏移量，构建StreamingContext
```scala
package com.steven.stream.kafka.offset

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.serializer.KryoSerializer
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * @Description: 使用新的kafka api来读取消息数据 ，每3秒统计最近9秒内的数据
 * @CreateDate: Created in 2023/4/16 09:36 
 * @Author: lijie3
 */
object StreamingKakfaOffset1 {

  def main(args: Array[String]): Unit = {
    //DStream数据批次大小
    val batchInterval = 2
    //窗口大小
    val windowInterval = batchInterval * 2
    //滑动大小
    val sliderInterval = batchInterval * 1
    val checkDir = "datas/searchlogs-1002"

    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition", "1000")
        .set("spark.serializer", classOf[KryoSerializer].getName)
      //使用检查点目录恢复数据,第一次运行时由于没有检查点目录，需要自己创建ssc，
      //再次重起流应用，会从检查点去恢复ssc
      val context = StreamingContext.getActiveOrCreate(checkDir, () => new StreamingContext(sparkConf, Seconds(batchInterval)))
      //从数据源端消费数据
      processData(context,windowInterval,sliderInterval)
      context
    }
    ssc.checkpoint(checkDir)
    //从数据源端消费数据
    processData(ssc,windowInterval,sliderInterval)
    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }

  def processData(ssc: StreamingContext,windowInterval:Int,sliderInterval:Int): Unit = {
    //消费位置策略
    val locationStrategy: LocationStrategy = LocationStrategies.PreferConsistent
    val topics: Iterable[String] = Set("wc-topic")
    val kafkaParams: collection.Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> "use_a_separate_group_id_for_each_stream",
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )

    val consumerStrategy: ConsumerStrategy[String, String] = ConsumerStrategies.Subscribe(topics, kafkaParams)

    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = {
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)
    }


    //这里需要进行一次转换，不转化会报ConsumerRecord序列化的错误
    val inputDStream2: DStream[ConsumerRecord[String, String]] = kafkaDStream
      .map(record => record)


    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val etlDStream = inputDStream2.transform { rdd =>
      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
      batchRDD
    }
    //将聚合与窗口设置放在一起
    val resultDStream: DStream[(String, Int)] =
      etlDStream.reduceByKeyAndWindow((tmp: Int, item: Int) => tmp + item,
        //反聚合函数,当要计算的时间窗口存在重叠时，可以通过这种优化来减少数据聚合的计算，需要在前面设置检查点checkponit
        (tmp: Int, item: Int) => tmp - item,
        Seconds(windowInterval), Seconds(sliderInterval),
        //指定过滤条件，过滤掉为0的统计
        filterFunc = (tuple: (String, Int)) => tuple._2 != 0)
    println("------map state with key")
    resultDStream.print()
  }
}
```

  问题：使用checkponit管理偏移量，当程序代码修改时从checkpoint恢复偏移量可能会出现异常，一般在企业中不会使用checkpoint来管理偏移量

2. 手动管理偏移量
每批次运行后保存状态和偏移量
> 将kafka的消费者groupId、topic、partition offset信息保存到mysql中
> 将流的状态保存到redis中
> 加载偏移量，仅在流应用启动运行时才调用
> 保存偏移量，每批次运行后保存偏移量

```scala
package com.steven.stream.kafka.offset

import org.apache.kafka.clients.consumer.ConsumerRecord
import org.apache.kafka.common.serialization.StringDeserializer
import org.apache.spark.SparkConf
import org.apache.spark.rdd.RDD
import org.apache.spark.streaming.dstream.{DStream, InputDStream}
import org.apache.spark.streaming.kafka010._
import org.apache.spark.streaming.{Seconds, StreamingContext}

/**
 * @Description: 使用新的kafka api来读取消息数据 ，每3秒统计最近9秒内的数据
 *               将偏移量写入mysql中
 * @CreateDate: Created in 2023/4/16 09:36 
 * @Author: lijie3
 */
object StreamingKakfaOffset2 {

  def main(args: Array[String]): Unit = {
    //DStream数据批次大小
    val batchInterval = 2
    //窗口大小
    val windowInterval = batchInterval * 2
    //滑动大小
    val sliderInterval = batchInterval * 1
    val checkDir = "datas/searchlogs-1003"

    //创建StreamingContext
    val ssc: StreamingContext = {
      val sparkConf = new SparkConf().setMaster("local[3]")
        .setAppName(this.getClass.getSimpleName.stripSuffix("$"))
        //设置每批次RDD中各个分区数据的最大值
        .set("spark.streaming,kafka.maxRatePerPartition", "1000")
      //设置时间间隔为5s
      new StreamingContext(sparkConf, Seconds(batchInterval))
    }
    //检查点保存流状态
    ssc.checkpoint(checkDir)
    //从数据源端消费数据
    processData(ssc, windowInterval, sliderInterval)

    //启动流
    ssc.start()
    ssc.awaitTermination()
    //设置优雅关闭，如果正在处理某一批次数据，则等待数据处理完成
    ssc.stop(stopSparkContext = true, stopGracefully = true)
  }

  def processData(ssc: StreamingContext, windowInterval: Int, sliderInterval: Int): Unit = {
    //消费位置策略
    val locationStrategy: LocationStrategy = LocationStrategies.PreferConsistent
    val topics: Iterable[String] = Set("wc-topic")
    val groupId = "wc-topic-group"
    val kafkaParams: collection.Map[String, Object] = Map[String, Object](
      "bootstrap.servers" -> "localhost:9092",
      "key.deserializer" -> classOf[StringDeserializer],
      "value.deserializer" -> classOf[StringDeserializer],
      "group.id" -> groupId,
      "auto.offset.reset" -> "latest",
      "enable.auto.commit" -> (false: java.lang.Boolean)
    )
    //消费策略：当mysql表中没有偏移量时，表示第一次消费，则使用不带偏移量的方法，存在值，则从指定的偏移量消费
    val offsets = Offsetutils.getOffsetsToMap(topics, groupId)
    val consumerStrategy: ConsumerStrategy[String, String] =
      if (offsets.isEmpty) {
        ConsumerStrategies.Subscribe(topics, kafkaParams)
      } else {
        ConsumerStrategies.Subscribe(topics, kafkaParams, offsets)
      }

    val kafkaDStream: InputDStream[ConsumerRecord[String, String]] = {
      KafkaUtils.createDirectStream[String, String](ssc, locationStrategy, consumerStrategy)
    }
    //定义偏移量
    var offsetRanges: Array[OffsetRange] = Array.empty

    //使用transform将stream转化为RDD,能对RDD操作就不要对DStream进行操作
    val etlDStream = kafkaDStream.transform { rdd =>
      //获取偏移量
      offsetRanges = rdd.asInstanceOf[HasOffsetRanges].offsetRanges
      //todo    当结果rdd已经保存或处理完成后，保存偏移量
      //todo 存在问题，当偏移量出现变更时才要保存
      Offsetutils.saveOffsetsToTable(offsetRanges, groupId)
      //转化为rdd.只需要获取消息value值
      val batchRDD: RDD[(String, Int)] = rdd
        .map(record => record.value())
        .filter(line => line != null && line.trim.nonEmpty)
        .flatMap(line => line.trim.split("\\s+"))
        .map(word => word -> 1)
      batchRDD
    }

    //将聚合与窗口设置放在一起
    val resultDStream: DStream[(String, Int)] =
      etlDStream.reduceByKeyAndWindow((tmp: Int, item: Int) => tmp + item,
        //反聚合函数,当要计算的时间窗口存在重叠时，可以通过这种优化来减少数据聚合的计算，需要在前面设置检查点checkponit
        (tmp: Int, item: Int) => tmp - item,
        Seconds(windowInterval), Seconds(sliderInterval),
        //指定过滤条件，过滤掉为0的统计
        filterFunc = (tuple: (String, Int)) => tuple._2 != 0)
    println("------map state with key")

    resultDStream.print()

  }
}

```
偏移量读写工具类
```scala
package com.steven.stream.kafka.offset

import org.apache.kafka.common.TopicPartition
import org.apache.spark.streaming.kafka010.{HasOffsetRanges, OffsetRange}

import java.sql.DriverManager

/**
 * @Description: 偏移量读写工具类，从mysql中读写
 * @CreateDate: Created in 2023/4/16 23:00 
 * @Author: lijie3
 */
object Offsetutils {
  /**
   * 加载偏移量
   * @param topicNames
   * @param groupId
   * @return
   */
  def getOffsetsToMap(topicNames:Iterable[String],groupId:String):Map[TopicPartition,Long] = {
    val url = "jdbc:mysql://localhost:3306/bigdata"
    val driver = "com.mysql.jdbc.Driver"
    val username = "root"
    val password = "dxy123456"

    Class.forName(driver)
    val connection = DriverManager.getConnection(url, username, password)

    val statement = connection.prepareStatement("SELECT * FROM kafka_offset  where topic in (?) and  groupid = ?")
    val sqlIN:String = topicNames.map(name => s"\'${name}\'")mkString(",")
    statement.setString(1, sqlIN)
    statement.setString(2,groupId)
    val resultSet = statement.executeQuery()
    var offsets = Map[TopicPartition, Long]()
    while (resultSet.next()) {
      val topic = resultSet.getString("topic")
      val partition = resultSet.getInt("partition")
      val offset = resultSet.getLong("offset")
      offsets += (new TopicPartition(topic, partition) -> offset)
    }

    resultSet.close()
    statement.close()
    connection.close()
    offsets
  }

  def saveOffsetsToTable(offsetRanges: Array[OffsetRange],groupId:String):Unit = {
    val connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/bigdata", "root", "dxy123456")
    val driver = "com.mysql.jdbc.Driver"
    Class.forName(driver)
    val statement = connection.prepareStatement("INSERT INTO kafka_offset (topic,groupid, `partition`, offset) VALUES (?, ?, ?,?) ")
    offsetRanges.foreach { offsetRange =>
      statement.setString(1, offsetRange.topic)
      statement.setString(2, groupId)
      statement.setLong(3, offsetRange.partition)
      statement.setLong(4, offsetRange.untilOffset)
      statement.execute()
    }
    statement.close()
    connection.close()
  }
}
```
3. 让kafka自己管理偏移量
每批次rdd数据处理完成后，异步提交偏移量到kafka


## StructedStreaming
structedStreaming是spark sql的一部分
spark2.4版本structedStreaming添加continues processing，类似于storm和flink的计算引擎，对流数据的处理是一条一条的处理
默认情况下StructedStreaming与spark streaming一样都是微批处理
structed Streaming默认就有状态的保存
将批处理和流处理统一
数据结构：DataFrame/DataSet，流式表
如果设置了检查点，下次重启时会自动加载偏移量和状态信息
基于事件时间的窗口分析
流数据去重
相关论文： the dataflow model

* 增量查询模型，在新增的流数据不断执行增量查询
* 支持端到端的输出
* 复用spark sql的执行引擎

### 编程模型
结构化流将流数据看作一个不断增长的表（unbouned table），进行增量的查询，将结果放入结果表（result table）中，输出时设置输出（output）模式（结果表的数据是全部输出，还是有更新输出还是追加输出）

* 官方示例
```shell
> nc -lk 9999
## 结构化流不需要receiver，这里只需要两个线程，一个用来接收数据，一个用户来进行数据处理
> ./bin/run-example --master 'local[2]' --conf spark.sql.shuffle.partitions=2 \
org.apache.spark.examples.sql.streaming.StructuredNetworkWordCount \
localhost 9999
```

* 编程实现wordcount,使用三种模式进行输出
```scala
import org.apache.spark.sql.streaming.{OutputMode, StreamingQuery}
import org.apache.spark.sql.{DataFrame, SparkSession}

/**
 * @Description: 使用结构化流处理wordcount
 * @CreateDate: Created in 2023/4/17 12:57 
 * @Author: lijie3
 */
object StructedStreamWc01 {

  def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()
    import sparkSession.implicits._

    //从tcp socket读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream.format("socket")
      .option("host", "localhost").option("port", 9999)
      .load()
    //处理数据

    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .groupBy($"value").count()

    //将结果进行输出
    val query: StreamingQuery = frame

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      .format("console").option("numRows", "20")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
}
```

### structedStreaming支持的数据源
file source 文件数据源，支持容错
socket source socket数据源
rate source
kafka source kafka数据源

file数据源
```scala
object StructedStreamFile02 {

  def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.sources.default", "org.apache.spark.sql.execution.datasources.csv.CSVFileFormat")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()
    import sparkSession.implicits._

    val csvSchema = new StructType()
      .add("Date",StringType,nullable = false)
      .add("Country",StringType,nullable = false)
      .add("Confirmed",IntegerType,nullable = false)
      .add("Cases",IntegerType,nullable = false)
      .add("Deaths",IntegerType,nullable = false)
      .add("New Cases",IntegerType,nullable = false)
      .add("Country Population",IntegerType,nullable = false)

    //从文件读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream
      .format("org.apache.spark.sql.execution.datasources.csv.CSVFileFormat")
      .option("sep",",")
      .option("header","true")
      .schema(csvSchema)
      .csv("datas/csv")
    //处理数据
    //..... 
    //将结果进行输出
    val query: StreamingQuery = inputStreamDF

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      .format("console").option("numRows", "20")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
}
```
rate source 时间戳和数字
```scala
object StructedStreamRate03 {

  def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()



    //从文件读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream
      .format("rate")
      .option("rowsPerSecond","10")
      .option("rampUpTime","0s")
      .option("numPartitions","2")
      .load()
    //处理数据


    //将结果进行输出
    val query: StreamingQuery = inputStreamDF

      .writeStream
      //追加模式不支持聚合
      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
//      .outputMode(OutputMode.Update())
      .format("console").option("numRows", "20")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
}
```

**query的设置**
* writeStream：返回DataStreamWriter对象
* ouputMode：输出模式设置 (有聚合不能使用append模式，没有聚合不能用complete模式)
* queryName：查询名称
* triggerInternal：触发时间间隔，有三种模式：固定时间间隔触发一次Trigger.processing，一次性微批，连续处理 Trigger.Continuing
* checkpointLocation：检查点,使用检查点和预写日志wal进行故障恢复，保证流的继续运行时数据的正确性
* outputSink：输出的目的地
示例1
```scala
//将结果进行输出
    val query: StreamingQuery = frame

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      //设置查询名称
      .queryName("query-wordcount")
      //设置触发器
      .trigger(Trigger.ProcessingTime("5 seconds"))
      .format("console").option("numRows", "20")
      //设置检查点目录
      .option("checkpointLocation","datas/structed/ckpt-wordcount")
      .option("truncate", "false").start()
```
输出终端
支持文件、控制台、
memory sink --将流数据输出到内存中作为表存储，查询时可以直接使用sql查询指定的queryName

foreachBatch sink --批次保存

```scala
  def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()
    import sparkSession.implicits._

    //从tcp socket读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream.format("socket")
      .option("host", "localhost").option("port", 9999)
      .load()
    //处理数据

    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .groupBy($"value").count()

    //将结果进行输出
    val query: StreamingQuery = frame

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      //设置查询名称
      .queryName("query-wordcount")
      //设置触发器
      .trigger(Trigger.ProcessingTime("5 seconds"))
//      使用foreach保存数据到mysql中
      .foreach(new MySQLForeachWriter)
      //设置检查点目录
      .option("checkpointLocation","datas/structed/ckpt-wordcount5")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
```

foreach sink --每一条进行保存
```scala
  def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()
    import sparkSession.implicits._

    //从tcp socket读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream.format("socket")
      .option("host", "localhost").option("port", 9999)
      .load()
    //处理数据

    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .groupBy($"value").count()

    //将结果进行输出
    val query: StreamingQuery = frame

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
//      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      //设置查询名称
      .queryName("query-wordcount")
      //设置触发器
      .trigger(Trigger.ProcessingTime("5 seconds"))
//      使用foreach保存数据到mysql中
      .foreachBatch((batchDF:DataFrame,batchId:Long) =>{
        println(s"batchid :${batchId}")
        if(!batchDF.isEmpty){
          batchDF
            //降低分区数
            .coalesce(1)
            .write
            //使用overwrite会重新创建一个张表
            .mode(SaveMode.Overwrite)
            .format("jdbc")
            .option("driver","com.mysql.cj.jdbc.Driver")
            .option("url","jdbc:mysql://localhost:3306/bigdata")
            .option("user","root")
            .option("password","dxy123456")
            .option("dbtable","wordcount2")
            .save()
        }
      })
      //设置检查点目录
      .option("checkpointLocation","datas/structed/ckpt-wordcount6")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
```

**容错语义**
在streaming 的处理的三个阶段，从数据源接收数据（source），经过数据处理分析（streaming execution），到最终数据输出（sink）仅被处理一次，是最好的状态
* at most once 最多一次，可能出现数据丢失
* at least once 至少一次，数据至少消费一次，可能出现多次消费
* exactly once 精确消费一次

structed streaming 支持offset，让spark追踪读取数据源的位置
基于checkpoint和wal来持久化保存每个trigger interval内处理的offset的范围
sink被设计成支持多次计算处理时保持幂等性

### 集成kafka
* 从kafka消费数据
* 向kafka写入数据

创建topic
```shell
./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 3 --topic wordTopic
```
pom文件依赖
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql-kafka-0-10_2.12</artifactId>
    <version>3.3.2</version>
</dependency>
```

读取topic数据输出到控制台
```scala
object KafkaSource01 {
  def main(args: Array[String]): Unit = {
    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions", "2")
        .getOrCreate()
    import sparkSession.implicits._

    //从kafka读取数据
    val kafkaStreamDF: DataFrame = sparkSession.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "wordTopic")
      .load()
    // 转换消息格式，取value值
    val inputStreamDF: Dataset[String] = kafkaStreamDF
      .selectExpr("CAST(value as STRING)")
      .as[String]

    //处理数据
    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .groupBy($"value").count()

    //将结果进行输出
    val query: StreamingQuery = frame
      .writeStream
      //追加模式不支持聚合
      //      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
      //      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      .format("console").option("numRows", "20")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
}
```

kafka作为接收器
处理流式数据时，先消费kafka 的数据，进行etl之后再写回kafka中
```scala
 def main(args: Array[String]): Unit = {
    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions", "2")
        .getOrCreate()


    import sparkSession.implicits._

    //从kafka读取数据
    val kafkaStreamDF: DataFrame = sparkSession.readStream
      .format("kafka")
      .option("kafka.bootstrap.servers", "localhost:9092")
      .option("subscribe", "wordTopic")
      .load()
    // 转换消息格式，取value值
    val inputStreamDF: Dataset[String] = kafkaStreamDF
      .selectExpr("CAST(value as STRING)")
      .as[String]

    //处理数据
    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.nonEmpty)
      .flatMap(line => line.trim.split("\\s+"))
      .groupBy($"value").count()

    //将结果进行输出,将数据写回到kafak中
    val query: StreamingQuery = frame
      .selectExpr("CAST(value AS STRING) AS key", "CAST(count AS STRING) AS value")
      .writeStream
      //追加模式不支持聚合
      //      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
      //      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
      .outputMode(OutputMode.Update())
      .format("kafka")
      .option("kafka.bootstrap.servers","localhost:9092")
      .option("topic","etl-word-count")
      .option("checkpointLocation","datas/structed/kafka-etl-wc-0001")
      .start()
    query.awaitTermination()
    query.stop()
  }
```
结构化流中可以使用sql和DSL的方式实现数据elt

**对流数据进行去重**
一个用户一天的最近一个小时或一天的访问统称为一次访问（UV），这时需要进行去重处理
structed streaming 使用水位线进行去重

**连续处理**
默认情况下structed streaming还是使用的微批次进行处理，每次处理多条数据，实施性较低
spark2.3引入的新功能，真正实现来一条数据处理一条数据，实实时性很高
缺点：task的数量无法进行扩展，只支持从kafka读从kafka写数据
启动连续流处理
```scala
spark
  .readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("subscribe", "topic1")
  .load()
  .selectExpr("CAST(key AS STRING)", "CAST(value AS STRING)")
  .writeStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host1:port1,host2:port2")
  .option("topic", "topic1")
  .trigger(Trigger.Continuous("1 second"))  // only change in query
  .start()
```
**时间窗口**
spark streaming中时间窗口属于processingTime（基于处理时间的窗口）
spark structedstreaming中时间窗口是eventTime（基于事件的时间窗口）

三个时间的概念：
* 事件时间eventTime，如订单下单时间
* 注入时间 IngestionTime，流式应用从kafka消息队列中获取到的订单数据的时间
* 处理时间 ProcessingTime，流式应用处理订单数据的时间

实际项目中是将注入时间和处理时间进行合并，统称为处理时间

使用时间时间生成窗口
```scala
def main(args: Array[String]): Unit = {

    //构建sparkSession
    val sparkSession: SparkSession =
      SparkSession.builder()
        .appName(this.getClass.getSimpleName.stripSuffix("$"))
        .master("local[2]")
        .config("spark.sql.shuffle.partitions","2")
        .getOrCreate()
    import sparkSession.implicits._

    //从tcp socket读取流数据
    val inputStreamDF: DataFrame = sparkSession
      .readStream.format("socket")
      .option("host", "localhost").option("port", 9999)
      .load()
    //处理数据
    //数据格式 2021-10-10 12:10:11,dog cat
    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.split(",").length ==2)
      .flatMap{line =>
        val arr = line.trim.split(",")
        arr(1).trim.split("\\s+").map(word => (Timestamp.valueOf(arr(0)),word))
      }
      .toDF("insert_timstamp","word")
      .groupBy(
        //设置基于事件事件的窗口
        window($"insert_timstamp","10 seconds","5 seconds"),$"word"
      ).count()
      .orderBy($"window")

    //将结果进行输出
    val query: StreamingQuery = frame

      .writeStream
      //追加模式不支持聚合
//      .outputMode(OutputMode.Append())
      //完全模式,将resulttable中的更新数据进行输出
      .outputMode(OutputMode.Complete())
      //update 更新模式，当resulttable中更新数据时进行输出
//      .outputMode(OutputMode.Update())
      .trigger(Trigger.ProcessingTime("5 seconds"))
      .format("console").option("numRows", "20")
      .option("truncate", "false").start()
    query.awaitTermination()
    query.stop()
  }
```
窗口生成规则：通过滑动时间和窗口时间进行计算

**延迟数据处理**
* 由于统计的数据是放在内存中的，如果每个时间窗口都保存会导致大量的内存消耗 --- 抛弃时间久远的状态
* 由于数据延迟可能导致流的性能问题 --- 抛弃超过时间阈值的延迟数据

设置水位线（watermarking），当延迟的数据的时间超出水位线，就不再计算，对数据进行丢弃
水位线：让spark sql引擎自动追踪数据中当前时间时间，依据规则清除
通过上一次批最大的事件时间，向前推算出需要计算的最小的事件时间，小于最小事件时间的数据直接丢弃
指定watermark
```scala
    //处理数据
    //数据格式 2021-10-01 12:10:11,dog cat
    val frame: DataFrame = inputStreamDF
      .as[String]
      .filter(line => line != null && line.trim.split(",").length ==2)
      .flatMap{line =>
        val arr = line.trim.split(",")
        arr(1).trim.split("\\s+").map(word => (Timestamp.valueOf(arr(0)),word))
      }
      .toDF("insert_timstamp","word")
      .groupBy(
        //设置基于事件事件的窗口
        window($"insert_timstamp","10 seconds","5 seconds"),$"word"
      ).count()
      //设置水位线，超过最小事件事件的数据将会丢弃
      .withWatermark("insert_timstamp","30 second")
      .orderBy($"window")
```
**天猫双十一的实时大屏统计**
* 订单数据首先存入RDMBS，通过canal监控mysql的binlog日志发送到kafka中
* 使用spark/flink 对kafka数据读取并进行实时的etl
* 将etl的结果数据以json的格式放入kafka中
* 从kafka中读取etl的json数据，将数据写入es和hbase中，进行增量存储
* 从kafka中读取etl中json数据并实现实时的报表


**流数据的停止**
使用yarn停止
yarn application -kill applicationid
优雅停止：
扫描hdfs中的某个文件是否存在，如果文件存在则表示需要将应用停止，使用streaming/structed streaming 的优雅停机

### spark集成es




















