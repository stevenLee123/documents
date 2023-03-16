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

## spark基础环境
单机模式：不需要AppManager的JVM进程，只有一个node用来执行mapreduce
集群模式：需要有一个AppManager的jvm进程管理mapreduce工作进程


## sparkcore核心RDD
Resilient Distributed Dataset --弹性分布式数据集
## spark sql

## spark 离线分析

## spark 实时分析 spark streaming/structredStreaming


