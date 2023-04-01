# Spark

谷歌三驾马车： GFS(hdfs)、 MapReduce、BigTable(Hbase)
分布式文件系统 GFS  （存储引擎）   ---存储数据
> block,replication
> nosql数据库 hbase
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

spark不做数据存储，做海量的数据分析，提供从hdfs、hive、mysql、kafka等存储工具中读取数据

**基于内存的快速、通用、可扩展的大数据分析计算引擎**
hadoop中的mapreduce适合一次性计算，不适合复杂多流程的数据计算，一个map只能有一个reduce，多个mapreduce会进行多次磁盘io
spark可以基于内存进行计算，多个作业之间的通信是通过内存


## spark基础环境
单机模式：不需要AppManager的JVM进程，只有一个node用来执行mapreduce
集群模式：需要有一个AppManager的jvm进程管理mapreduce工作进程

## spark 架构
spark core ：spark最基础与最核心的功能，spark的其他功能如spark sql，spark streaming 都是在core基础上进行扩展的
spark sql： 用来操作结构化数据的组件，可以使用sql或hive的sql方言来查询数据
spark streaming： 针对实时数据进行流式计算的组件，提供丰富的数据流api
spark MLlib: 提供机器学习算法库
spark GraphX：面相图计算提供的框架与算法库

## sparkcore核心RDD
Resilient Distributed Dataset --弹性分布式数据集
是一种对数据集形态的抽象，基于此抽象，使用者可以在集群中执行一系列计算，而不用将中间结果落盘。而这正是之前 MR 抽象的一个重要痛点，每一个步骤都需要落盘，使得不必要的开销很高


## spark 部署
在国内中主要是用yarn
三种方式：
> 本地运行模式
> 独立运行模式
> yarn

## spark 离线分析

## spark 实时分析 spark streaming/structredStreaming


