# 大数据入门
## 基本工具描述
> hadoop： 用于分布式存储和map-reduce计算
* yarn负责资源和任务管理
* hdfs负责分布式存储
* map-reduce负责分布式计算

> spark: 为大量数据处理而设计的快速通用设计引擎
> hive: Hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供 SQL 查询功能，可以将 SQL 语句转换为 MapReduce 任务进行运行。
> hbase : 分布式kv系统，强读写译执行，适合高速计算聚合，以hdfs作为底层文件系统

## zookeeper
开源的分布式的程序协调服务，提供命名、配置管理、同步和组服务，实现高吞吐量和低延迟，保证高性能、高可用性、严格有序的访问

## 概念
**离线分析（批处理）**
面向过去，分析已有数据，在时间维度明显成批次性变化，也叫批处理
**实时分析（streaming）**
面向当下，分析实时产生的数据，分秒级别、毫秒级别
**机器学习**
基于历史预测未来的走势

**数据分析流程**
明确分析的目的和思路 -> 数据搜集 -> 数据处理（对数据进行结构化处理） ->数据分析 -> 数据展现（数据展现） -> 数据分析报告

**大数据的特征**
> volume 数据体量大
> variety 数据种类多
> value 低价值密度（需要对价值进行挖掘）
> velocity 增长速度快、获取速度快、处理速度快
> veracity 数据的质量 数据准确性、数据可信赖度

要解决的问题
> 数据的存储 --多台机器分布式存储
> 数据的计算 --多台机器分布式计算

## hadoop的核心组件
> hdfs 分布式文件存储系统
> yarn 集群资源管理和任务调度框架
> mapreduce 解决海量数据的计算问题,由于其自身的模型弊端，企业几乎不回再使用mapreduce进行编程处理，很多软件的底层还是使用mapreduce引擎来处理数据

**特点**
> 扩容能力强 scalability
> 成本低 economical
> 效率高 efficiency
> 可靠性 reliability
> 通用性

**hadoop包含的集群**
逻辑是分离、物理上通常在一起部署
hdfs集群
> nameNode 管理节点
> DataNode 数据节点
yarn集群

## 集群环境搭建
3台机器 
1. 配置静态网络（使用nat桥接模式）
2. 配置主机名称并写入/etc/hosts
           node1      node2       node3
hostname   node1      node2       node3

## 配置文件

hadoop-env.sh 
配置java home
> HDFS_NAMENODE_OPTS   -> NameNode
> HDFS_DATANODE_OPTS  -> Secondary NameNode
> HDFS_DATANODE_OPTS -> 

core-site.xml 核心配置文件模块
```xml
<configuration>
    <!--用于设置Hadoop的文件系统，由URI指定-->
    <property>
        <name>fs.defaultFS</name>
        <!--用于指定namenode地址在node1机器上-->
        <value>hdfs://node1:9000</value>
    </property>
    <!--配置Hadoop的临时目录，默认/tem/hadoop-${user.name}-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/export/servers/hadoop-2.7.4/tmp</value>
    </property>
</configuration>

```
hdfs-site.xml hdfs文件系统模块
```xml
<configuration>
    <!--指定HDFS的数量-->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    <!--secondary namenode 所在主机的IP和端口-->
    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>node2:50090</value>
    </property>
</configuration>
```
mapred-site.xml mr模块配置文件
```xml
<configuration>
    <!--指定MapReduce运行时的框架，这里指定在YARN上，默认在local-->
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```
yarn-site.xml yarn模块配置文件
```xml
<configuration>
    <!--指定YARN集群的管理者（ResourceManager）的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>hadoop01</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

workers 集群worker的配置
```
node1
node2
node3
```



