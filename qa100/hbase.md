# hbase  

## hadoop的局限
* hadoop主要实现批量数据的处理，并通过顺序方式访问数据
* 要查找数据必须搜索整个数据集，如果要进行随机读取，效率较低

## hbase的概念
* nosql数据库
* 是bigdata的开源java版本，建立在hdfs上，提供高可靠性、高性能、列存储、可伸缩、实时读写NoSQL的数据库系统
* HBase仅能通过主键rowKey和主键的range来检索数据，仅支持单行数据
* 主要用来存储结构化和半结构化的松散数据
* 不支持join等复杂查询，不支持复杂的事务，缺少rdbms的许多特性，如带类型的列，二级索引（一般的索引）以及高级查询语言
* 支持的数据类型 byte[]
* 依靠横向扩展，提高存储能力和处理能力
* 表很大，一个表可以存储十亿以上的行，上百万列
* 面向列（族）的存储和权限控制，列族独立索引
* 稀疏：对为空的列不占用存储空间，表可以设计的非常稀疏

## 应用场景
* 对象存储  
各种新闻、网页、图片等
* 时序数据
有一个openTSDB模块，满足时许类场景需求
* 推荐画像
  用户画像，一个比较大的稀疏矩阵，蚂蚁金服的疯狂就是在hbase上构建
* 时空数据
  轨迹，气象，卫星类数据
* Cubedb OLAP
  kylin是一个cube分析工具，底层数据存储在hbase上
* 消息/订单
  电商，银行领域很多订单查询底层的存储，通信，消息同步应用构建在Hbase上
* Feeds流
  类似朋友圈的应用
* newSql
    phoneix（查询引擎）的插件，可以满足二级索引，sql的需求，对接产痛数据需要sql非事务的需求
* 海量数据备份，存储爬虫数据，短网址（视频网址）

## hbase的特点
* NoSQL
* 强一致性读、写
    * hbase不是最终一致性的数据存储
    * 非常适合如高速计算、聚合等任务 
* 自动分块
    * habse通过region分布在集群上，随着数据增长，区域被自动拆分和重新分布
* 自动RegionServer故障转移
* hadoop/hdfs集成
    * hbase支持hdfs开箱即用
* mapreduce
    * 通过mapreduce支持大规模并行处理，将hbase作为源和接收器
* thrift /REST api
* 块缓存和布隆过滤器
* 运行管理
* 不支持ACID，支持单行数据的事务操作
* 因为不支持join，不适合做数仓
* 支持数据更新

与rdbms 的比较
* 数据库以表形式存在，支持fat，ntfs，ext文件系统，使用主键，通过外部中间件可以支持分库分表，底层单机引擎，使用行列单元格存储数据，面向行查询，支持事务，具备acid，支持join，面向OLTP

* 联机事务处理OLTP（on-line transaction processing） 传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易
* 联机分析处理OLAP（On-Line Analytical Processing） 数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。
## hbase的结构
* 主要面向OLTP，不适合做大规模的OLAP
* 支持hdfs文件系统
* 以表方式存储
* 使用行健（rowkey）
* 原生支持分布式存储，计算引擎
* 使用行，列，列族和单元格
* 文件类型称为storeFiles

## 安装zookeeper
1. 修改配置文件
```shell
#指定数据目录
dataDir=/export/data/zk/data
#指定集群信息
server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888

```
创建 myid文件
```shell
#根据上面的server信息分别指定节点名称1，2，3
echo 3 > /export/data/zk/data/myid
```
配置环境变量
```shell
export ZK_HOME=/export/server/apache-zookeeper-3.8.1-bin
export PATH=$PATH:$ZK_HOME/bin
```
各个节点上分别启动zkserver
```shell
zkServer.sh start
#查看状态
zkServer.sh status
#停止集群
zkServer.sh stop
```

## hbase安装
修改conf/hbase-env.sh
配置javahome目录
配置HBASE_MANAGES_ZK=false
```shell
export JAVA_HOME=/export/server/jdk1.8.0_131
export HBASE_MANAGES_ZK=false
```
配置hbase-site.xml
1. 指定hdfs路径
2. 指定hbase为分布式模式
3. 指定hbase master
4. 指定zk集群
5. zk数据目录
6. 兼容性检查
```xml
<property>
<name>hbase.rootdir</name>
<value>hdfs://node1:8020/hbase</value>
</property>
<property>
<name>hbase.cluster.distributed</name>
<value>true</value>
</property>

<property>
<name>hbase.zookeeper.quorum</name>
<value>node1,node2,node3</value>
<description>Comma separated list of servers in the ZooKeeper Quorum. For example, "host1.mydomain.com,host2.mydomain.com,host3.mydomain.com". By default this is set to localhost for local and pseudo-distributed modes of operation. For a fully-distributed setup, this should be set to a full list of ZooKeeper quorum servers. If HBASE_MANAGES_ZK is set in hbase-env.sh this is the list of servers which we will start/stop ZooKeeper on. </description>
</property>
<property>
<name>hbase.zookeeper.property.dataDir</name>
<value>/export/data/zk/data</value>
<description>Property from ZooKeeper's config zoo.cfg. The directory where the snapshot is stored. </description>
</property>
<property>
<name>hbase.unsafe.stream.capability.enforce</name>
<value>false</value>
</property>
```
配置环境变量
```shell
export HBASE_HOME=/export/server/hbase-3.0.0-alpha-3
export PATH=$PATH:$HBASE_HOME/bin
```
拷贝jar包
```shell
 cp lib/client-facing-thirdparty/htrace-core4-4.1.0-incubating.jar lib
```
修改conf/regionservers文件，指定server列表