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
        <value>hdfs://node1:8020</value>
    </property>
    <!--配置Hadoop的临时目录，默认/tem/hadoop-${user.name}-->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/export/data/hadoop-3.3.4</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property>
    <!--系统垃圾保存时间 -->
    <property>
        <name>fs.trash.interval</name>
        <value>1440</value>
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
        <value>node2:9868</value>
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
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>node1:10020</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>node1:19888</value>
    </property>
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=${HADOOP_HOME}</value>
    </property>
</configuration>
```
yarn-site.xml yarn模块配置文件
```xml
<configuration>
    <!--指定YARN集群的管理者（ResourceManager）的地址-->
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node1</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
      <!--关闭物理内存检测-->
    <property>
        <name>yarn.nodemanager.pmem-check-ebable</name>
        <value>false</value>
    </property>
     <!--关闭虚拟内存检测-->
    <property>
        <name>yarn.nodemanager.vmem-check-ebable</name>
        <value>false</value>
    </property>
    <property>

        <name>yarn.log-aggregation-enable</name>
        <value>true</value>
    </property>
    <property>
        <name>yarn.log.server.url</name>
        <value>http://node1:19888/jobhistory/logs</value>
    </property>
     <property>
        <name>yarn.log-aggregation.retain-seconds</name>
        <value>604800</value>
    </property>
</configuration>
```

workers 集群worker的配置
```
node1
node2
node3
```

格式化 namemode format (初始化,不能执行多次，会导致数据丢失)
```shell
hadoop namenode -format
```
启停hadoop
hdfs集群停动
```shell
start-dfs.sh
stop-dfs.sh
```
yar集群启停
```shell
start-yarn.sh
stop-yarn.sh
```
启停所有
```shell
start-all.sh
stop-all.sh
```
> 使用jps确认java进程是否启动正常
> 查看集群的日志

工具web网页
http://node1:9870/explorer.html#/  --hdfs集群总览
http://node1:8088/cluster ---查看yarn集群



上传文件问什么慢？

### mapreduce初体验
hadoop jar hadoop-mapreduce-examples-3.3.4.jar pi 2 2
对于少量数据，mapreduce的时间是很长的

### hdfs分布式文件系统基础 （hadoop distributed file system）
传统文件系统通常是单机文件系统，都带有抽象的目录结构
文件系统的数据和元数据
*数据* ：存储的内容本身
*元数据*：解释性数据，记录数据的数据，记录文件的大小，存储位置，所有者信息，权限信息

传统文件系统无法支撑海量数据存储，数据过大之后单节点的IO性能成为系统的瓶颈

分布式存储
* 分布式存储 ：多机横向扩展
* 元数据记录 ：记录文件及其存储位置信息，快速定位文件位置（类比mysql索引）
* 分块存储： 文件分块存储在不同的机器。针对块并行操作提高效率
* 副本机制：不同机器设置备份，冗余粗出，保障数据安全（raid）

hdfs提供统一的访问接口（nameNode），类似于普通的linux文件系统的访问
设计目标
* 硬件故障是常态，具备故障检测和快速恢复功能
* 注重大量数据访问的高吞吐量
* 能从一个平台轻松移植到另一个平台

应用场景： 大文件，数据流访问，一次写入多次读取，低成本部署，廉价pc高容错

特性
* 主从架构 一个主节点（namenode）多个从节点（datanode）
* 分块存储 物理上分块存储（block 默认128M，hdfs-site.xml 的dfs.blocksize属性）
* 副本机制 保证数据安全性，将副本放在不同的datanode上保证数据的安全
* 元数据记录 文件自身属性信息（文件名称、文件大小、复制因子、块大小信息），文件块位置映射信息
* 抽象统一的目录树结构9（namepsace） 与liunx目录类似

hdfs shell
基础命令： hadoop fs [option]
查看本地文件： hadoop fs -ls file:///
查看hdfs ： hadoop fs -ls hdfs://node1:8020/
默认查看hdfs根目录(读取环境变量中的fs.defaultFS)： hadoop fs -ls /

```shell
hadoop fs -ls -h(人性化显示) -R（递归查看） /   #查看
hadoop fs -mkdir /steven # 新建
hadoop fs -put anaconda-ks.cfg(客户端文件系统文件)  /steven #文件上传
hadoop fs -cat text.txt # linux cat
hadoop fs -tail text.txt # linuc tail
hadoop fs -get [-f] [-p] <dst> <localsrc> #下载文件到本地文件系统
hadoop fs -appendToFile <localsrc><localsrc2>... <dst> #将本地文件合并到hdfs上,用于小文件合并
hadoop fs -mv [src] [dst] #linux mv 重命名
```
hdfs 角色职责

* Namenode： hdfs的核心，维护管理文件系统元数据，包括名称空间目录结构，文件和块的位置信息，访问权限信息，不持久化存储文件中块的datanode的位置信息，是hdfs的唯一入口
             内部通过内存和磁盘文件保证数据安全
             存在单点故障，需要配置大量内存
* datanode： 负责具体的数据块存储，决定数据的存储能力，需要向namenode汇报块列表信息，需要配置大量的磁盘空间
* Secondarynamenode： 充当namenode的辅助节点，不能代替namenode，帮助namenode进行元数据文件的合并动作（秘书）

hdfs写数据流程


