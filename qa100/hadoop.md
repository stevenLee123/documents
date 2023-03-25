# 大数据入门
## 基本工具描述
> hadoop： 用于分布式存储和map-reduce计算
* yarn负责资源和任务管理
* hdfs负责分布式存储
* map-reduce负责分布式计算，程序变写比较麻烦
* hive实现在hadoop上进行结构化处理的组件，存储结构化信息，可以实现将sql转华为mapreduce，将执行结果进行加工返回给用户
* hive的sql的灵活性不如直接使用mapreduce
* spark是hadoop上的计算框架，spark是在内存中进行计算的，也提供spark-sql，可以直接编写sql


> spark: 为大量数据处理而设计的快速通用设计引擎
> hive: Hive 是基于 Hadoop 的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供 SQL 查询功能，可以将 SQL 语句转换为 MapReduce 任务进行运行。
> hbase : 分布式kv系统，强读写一致性，适合高速计算聚合，以hdfs作为底层文件系统

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

core-site.xml 核心配置文件模块
```xml
<configuration>
    <!--用于设置Hadoop的文件系统，由URI指定-->
    <property>
        <name>fs.defaultFS</name>
        <!--用于指定namenode地址在node1机器上-->
        <value>hdfs://node1:8020</value>
    </property>
    <!--配置Hadoop的临时目录，默认/temp/hadoop-${user.name}-->
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
需要做副本、保证数据传输的安全性等

### mapreduce初体验
hadoop jar hadoop-mapreduce-examples-3.3.4.jar pi 2 2
对于少量数据，mapreduce的时间是很长的


## hdfs分布式文件系统基础 （hadoop distributed file system）
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
**pipeline管道**
上传文件写数据过程中采用的数据传输方式
* 客户端将数据块写入第一个数据节点，第一个节点保存数据知乎再把数据块复制到第二个数据节点，以此类推。
* 线性传输沿着一个方向传输，充分利用带宽，避免遇到网络瓶颈和高延迟的连接，最小化推送所有数据的延迟

**ack应答响应**
pipeline管道传输过程中反方向进行ACK校验，确保数据安全
**默认3副本存储策略**
默认副本由BlockPlacementPolicyDefault类指定
* 第一块副本优先选择客户端本地
* 第二块副本放在不同于第一块副本的不同机架
* 第三块副本放在与第二块副本放在相同的机架上的不同机器

代码层面的实现
1. hdfs客户端创建对象实例DistributedFileSystem
2. 调用DistributedFileSystem的create（）方法，通过rpc请求Namenode创建文件
3. 检查判断：目标文件是否存在，父文件夹目录是否存在
4. namenode为本次请求记录下一条记录，返回FSDataOutputStream对象
5. 客户端通过FSDataOutputStream输出流开始写数据
6. 客户端写入数据是，将数据分成一个个数据包（packet 默认64K），内部组件DataStreamer请求namenode挑选出合适的存储数据副本的一组datanode地址，默认3个副本地址
7. DataStreamer将数据包流式传给pipeline的第一个datanode，datanode存储并发送给第二个，依此类推
8. 传递反方向进行ACK校验
9. 数据写入完成后，FSDataOutputStream关闭
10. DistributedFileSystem告知namenode其文件写入完成，等待namenode确认
11. 当pipline过程中出现文件传输失败时，namemode只需要确认有一个副本上传成功（由dfs.namenode.replication.min指定，默认1），就认为数据上传成功

## mapreduce （目前基本上不会直接使用mr）
**分而治之**
将复杂问题按照一定的方法分解成几个子问题，并行处理，然后再进行合并

两个阶段
> map 拆分问题，并行处理子任务
> reduce 合并map阶段的结果

**设计思想**
> 对不具有计算依赖关系的大数据计算任务，实现并行最自然的办法就是采用mapreduce的策略
> 借助函数式语言的思想，用map和reduce两个函数提供高层的并行编程抽象模型
> map：对一组数据元素进行某种重复式的处理
> reduce：对map的中间结果进行整理
使用key，value的形式定义编程接口
> map: `(k1，v1)—>(K2,V2)` 
> map: `(k2，[v2])—>(K3,V3) `

统一框架，隐藏底层细节，使用者只需要关心应用层的具体计算问题，编写少量处理应用本身计算问题的业务程序代码即可。
* 易于编程
* 良好的扩展性
* 高容错性
* 适合海量数据的离线处理
* 实时计算性能差，只能做离线计算
* 不能进行流式计算
完整的mr实例：
> MRAppMaster :负责mr过程调度及状态协调
> MapTask:负责map阶段的整个数据处理流程
> ReduceTask: 负责reduce阶段整个数据处理流程

阶段组成
> 一个mapreduce 只能有一个map阶段，有一个或0个reduce阶段
> 出现复杂的业务逻辑则需要使用多个mr来实现
> 数据都是以k,v键值对的形式进行流转

mr任务的提交
```shell
hadoop jar hadoop-mapreduce-examples-3.3.4.jar wordCount wordCount.txt
```
> input 数据读取
> splitting：对数据进行拆分，输出格式 `<word,1>`
> mapping：统计拆分的文件中的各个词的数量
> shuffling： 分组，排序，将各个词的统计分别放在一起
> reducing：将单词统计的所有数据进行累加
> final result 输出结果都是聚合之后的 kv形式

**map阶段执行**
> 1. 逻辑切片，将数据切片规划，默认split size = block size 128M，每个切片由MapTask处理
> 2. 按行读取数据 ，将切换中的数据按照一定的规则读取解析返回`<k,v>`，key是每一行的其实位置偏移量，value是本行的文本内容
> 3. 调用Mapper类中的map方法处理数据，每读取解析出来一个`<k,v>`,调用一次map方法
> 4. 按照一定的规则对map输出的键值对进行分区partition，默认不分区，因为只有一个reduceTask，分区的数据量就是reduceTask运行的数量
> 5. map输出数据写入内存缓冲区，达到比例溢出（spill）到磁盘上，spill时根据key进行排序sort，默认根据key字段顺序排序
> 6. 对所有移除文件进行最终的merge合并，成为一个文件，一个MapTask只会输出一个文件

**reduce阶段执行**
> 1. ReduceTask 主动从MapTask复制拉取数据自己要处理的数据
> 2. 把拉取来的数据全部进行合并merge，把分散的数据合并成一个大数据，再对合并的数据排序
> 3. 对排序后的键值对调用reduce方法，键相等的键值进行合并

**shuffle机制**
shuffle类似洗牌的相反过程，将map端的无规则输出按指定的规则进行排序，以便reduce端接收处理
*map端shuffle*
> collect阶段，将MapTask的结果收集输出到默认大小为100M的环形缓冲区，保存之前会对key进行分区计算，默认hash分区
> spill阶段：当内存中的数据量达到一定的阈值时，将数据溢出（spill）写入本地磁盘，在数据写入磁盘之前会进行一次排序操作，如果配置了combiner，还会将由相同分区号和key的数据进行排序
> merge阶段：把所有移除的临时文件进行一次合并操作，以确保一个maptask最终只产生一个中间数据文件

*reduce端shuffle*
> copu阶段： reduceTask启动Fetcher线程到已完成的MapTask的节点上复制一份属于自己的数据
> Merge阶段：在ReduceTask远程复制数据的同时，会在后台开启两个线程对内存到本地的数据文件进行合并操作
> Sort阶段： 在对数据进行合并的同时，会整体进行排序操作，在MapTask阶段已经对数据进行了局部的排序，ReduceTask只需要保证Copy的数据最终整体有效性即可


shuffle机制的弊端：
shuffle频繁设计到数据在内存、磁盘之间多次往复，导致mapreduce过程很慢

## yarn （yet another resource negotiator） 
**yarn是通用的资源管理系统和调度平台，可为上层应用提供同一个的资源管理和调度**
资源管理系统： 集群的硬件资源，和程序运行关系， 比如内存、cpu
调度平台：支持多个程序同时申请计算资源如何分配，调度的规则算法设置
通用：不仅支持mr，支持各种其他的各种程序，如spark、flink

**yarn架构**
集群物理层面划分
> ResourceManager 资源管理者，决定系统中所有应用程序之间资源分配的最终权限，最终仲裁者
> NodeManager 从角色，一台机器上一个，负责本台机器上的计算资源，根据RM的命令，启动Container容器，监视容器资源使用情况，并像主角色汇报
应用程序层面
> ApplicationMaster ，应用程序的老大，负责程序各个阶段资源申请，监督程序的执行情况
> client 客户端
> Container 容器，硬件资源抽象

**yarn程序提交的流程（以mapreduce为例）**
第一阶段客户端申请资源启动运行本次程序的ApplicationMaster
第二阶段ApplicationMaster根据本次程序内部具体情况，为它申请资源，并监视他的整个运行过程

1. MR作业提交 client -> ResourceManager  
2. 资源的申请 ApplicationMaster ->ResourceManager 
3. MR作业状态汇报 container(map|reduce task) -> 
4. 节点状态汇报 NodeManage -> ResourceManager  


> 客户端向yarn 的ResourceManager提交应用程序
> ResourceManager 为应用程序分配第一个Container，并于对应的NodeManager通行，要求它在这个Container中启动这个应用程序的ApplicationMaster
> container(ApplicationMaster) ApplicationMaster启动成功后，向ResourceManager注册并保持通信，用户可以通过ResourceManager查看程序运行状态
> ApplicationMaster 为本次程序内部的各个Task任务向ResourceManager申请资源，并监控它的运行状态 （使用yarn的scheduler组件进行申请）
> 一旦Application Master申请到资源后，与对应的NodeManager通信要求其启动任务
> NodeManager为任务设置好运行环境之后，将任务启动命令写入到一个脚本中，并通过运行该脚本之行任务
> 任务通过rpc向ApplicationMaster汇报自己的状态和进度
> 运行程序完成后，ApplicationMaster向ResourceManager注销并关闭自己

**yarn的资源调度器Scheduler**
调度没有最佳策略，要根据实际应用场景使用
三种调度策略
* FIFO scheduler
  先进先出，先提交应用先运行
  拥有控制全局的queue
  不适合共享集群，不需要配置
* Capacity scheduler
  容量调度
  允许多个组织共享整个集群资源  
  每个组织可以获得集群的一部分计算能力，通过为每个组织分配专门的队列，然后再为每个队列分配一定的集群资源
  层次话的队列设计
  容量保证
  安全
  弹性分配
* fair Scheduler
    公平调度策略
    动态调整资源，按作业来进行分配资源
    保证最小配额
    允许资源共享
    默认不限制每个队列和用户可以同时运行应用的数量

## 数据仓库
> 用于存储分析，报告的数据系统
> 面相分析的集成化数据环境
> 本身不生产数据
> 也不消费任何数据
> 主要用来分析数据

特征
> 面向主题性 -- subject-oriented 主题是一个抽象的概念，是较高层次数据综合，归类并进行分析利用的抽象
* 抽象层次上对数据进行完整、一致和准确的描述
> 集成性 -- integrated 主题相关的数据通常会分不在多个操作性系统中，彼此分散、独立、异构，需要继承到数据仓库主题下
* 统一源数据中所有矛盾之处，进行数据综合计算
> 非易失性 -- non-volatile  非易变性，数据仓库是分析数据的平台，而不是创造数据的平台
* 大量的查询操作，修改操作很少
> 时变性-- time-variant 数据仓库的数据需要随着时间更新，以适应决策的需要
* 包含各种粒度的历史数据，数据仓库的数据需要随着时间更新，以适应决策的需要

数据仓库开发语言 --sql（数据分析的主流语言）


## hive
hive是建立在hadoop上的开源数据仓库系统
> 使用sql语言，提高快速开发的能力
> 避免直接写mapreduce
> 支持自定义函数

和hadoop的关系
利用hdfs存储数据，利用mapreduce分析查询数据

### hive的思想
> 映射信息记录 --写的sql是针对表，不是针对文件，规定文件和表的对应关系（保存数据的元数据信息，通过元数据信息查找表与hdfs文件的对应关系）
> SQL语法解析、编译
> 解析sql后并拿到表与文件映射信息后，转换成mapreduce任务执行拿到结果

### hive的组件
* 用户接口 jdbc、odbc webGUI、CLI（shell命令行）
* 元数据存储 hive中的元数据包括表的名字，表的列和分区及其属性，表的属性，表的数据所在的目录 （文件数据存储在hdfs上）
* driver驱动程序 包括语法解析器，计划编译器，优化器，执行器
* 执行引擎 hive本身不处理数据，由执行引擎处理，当前支持mapreduce，tez，spark3 

### hive安装
元数据 metadata：描述数据的数据，描述数据的属性信息，如存储位置，历史数据，大小等，元数据存储在关系型数据库中，如hive内置的derby，或第三方mysql
元数据服务 metastore：允许多个客户端同时连接，客户端不需要知道mysql数据库的用户名和密码。只需要连接metastore服务即可，保证元数据的安全

安装模式： 内嵌模式，本地模式，远程模式 ---*使用远程模式部署*
metastore是否需要单独配置，单独启动   ---*单独启动*
metastore是使用内置derby，还是使用mysql  ---*使用mysql*

安装MySQL数据库 centos7上的安装
卸载mariadb
```shell
rpm -qa|grep mariadb
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
rpm -e --nodeps mariadb-libs-5.5.60-1.el7_5.x86_64
```
依赖安装包
yum -y install libaio
参考地址：
https://blog.csdn.net/Darlight/article/details/107787178

安装hive
修改配置文件：
hive-en.sh
```shell
export HADOOP_HOME=/export/server/hadoop-3.3.4
export HIVE_CONF_DIR=/export/server/apache-hive-4.0.0-alpha-2-bin/apache-hive-4.0.0-alpha-2-bin/conf
export HIVE_AUX_JARS_PATH=/export/server/apache-hive-4.0.0-alpha-2-bin/apache-hive-4.0.0-alpha-2-bin/lib
```
hive-site.xml
```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!--连接数据的用户名-->
  <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>steven</value>
  </property>
<!--连接数据的密码-->
  <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>steven</value>
  </property>
<!--mysql数据库的访问路径，没有路径则自动创建-->
  <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://node1:3306/hive?createDatabaseIfNotExist=true&amp;characterEncoding=utf8&amp;serverTimezone=UTC&amp;useSSL=&amp;allowPublicKeyRetrieval=true</value>
  </property>
<!--连接数据库的驱动-->
  <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.cj.jdbc.Driver</value>
  </property>
<!--元数据是否校验-->
  <property>
      <name>hive.metastore.schema.verification</name>
      <value>false</value>
  </property>
<!--是否自动创建核心文件-->
  <property>
    <name>datanucleus.schema.autoCreateAll</name>
    <value>true</value>
  </property>
<!--thrift服务器绑定的主机-->
  <property>
    <name>hive.server2.thrift.bind.host</name>
    <value>node1</value>
  </property>

<!--默认的存储地址-->
 <property>
  <name>hive.metastore.warehouse.dir</name>
  <value>hdfs://node1:9000/user/hive/warehouse</value>
  <description>location of default database for the warehouse</description>
</property>
<!--设置显示表头字段名-->
 <property>
   <name>hive.cli.print.header</name>
   <value>true</value>
 </property> 
 <property>
   <name>hive.cli.print.current.db</name>
   <value>true</value>
 </property>
 <!--远程模式部署metastore metasotre地址-->
  <property>
   <name>hive.metastore.uris</name>
   <value>thrift://node1:9083</value>
 </property>
 <!--关闭元数据存储授权-->
  <property>
   <name>hive.metastore.event.db.notification.api.auth</name>
   <value>false</value>
 </property>


</configuration>
```
启动hive
前台启动
```./bin/hive --service metastore ```
后台启动
``` 
    nohup ./bin/hive --service metastore > metastore.log 2>&1 &       #先启metastore
    nohup ./bin/hive --service hiveserver2  > hiveserver2.log 2>&1 &  #再启动hiveserver2
```

客户端使用
使用beeline客户端访问hive
```shell
$ ./bin/beeline --启动客户端
beeline> ! connect jdbc:hive2://node1:10000  --连接
#执行Hql
beeline> create database user; 创建库
```

第三方可视化客户端
datagrip dbeaver

### hql语法
DDL语法

### hive授权
hive中访问三种权限配置：
* Metastore Server 中基于存储的授权
* HiveServer2 中基于 SQL 标准的授权
* 使用 Apache Ranger 和 Sentry 的授权



