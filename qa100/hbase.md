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
* 稀疏的分布式的持久的多维排序map，key-value结果
* 映射由行键，列键和时间戳索引

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
<!-- 与web ui访问有关的配置 -->
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

启停hbase
```shell
./bin/start-hbase.sh
./bin/stop-hbase.sh
```
shell连接hbase
```shell
./bin/hbase shell
# 退出hbase shell
hbase:013:0> quit
```
**注意，以上部署由于各个hbase和hadoop的版本问题，可能出现某些不可预料的问题**
1. hdfs 集群启动正常，使用start-hbase.sh启动集群时，可能由于jar包的冲突导致hbase集群启动失败，解决方案：
    *  删除 hbase种的slf4j-log4j12-1.7.25.jar包 该文件存储在 hbase/lib/client-facing-thirdparty/路径下删除即可
    * 打开hbase-env.sh 的export HBASE_DISABLE_HADOOP_CLASSPATH_LOOKUP="true"选项，启动时告诉HBase是否应该包含Hadoop的lib
2. 到官网上查找hadoop和hdfs的版本兼容列表，选择兼容的版本进行安装
3. 16010端口的web ui打不开问题：hbase-site.xml加上下面的配置 (http://192.168.10.101:16010/master-status)
```xml
<property>
  <name>hbase.unsafe.stream.capability.enforce</name>
  <value>false</value>
</property>
```
集群统一启停命令
```shell
start-hbase.sh
#注意关闭时在master的节点上关闭
stop-hbase.sh
```



**高可用配置**
在conf下添加backup-masters,里面指定要设置为备用master的节点

```
node2
node3
```

habse shell的使用


dml ：操作数据
namespace：  
`create_namespace bigdata`
`update_namespace`
`list_namespace`

ddl ：操作表格
`list` 查看表格列表
`create 'student','info','msg' `创建表格
`create 'bigdata:person',{NAME => 'info', VERSIONS => 5},{NAME => 'msg', VERSIONS => 5}` 指定namespace和维护的版本号个数
` describe 'bigdata:person'` 查看表信息
`alter 'bigdata:person',  NAME => 'info', VERSIONS => 10` 修改列族
`alter 'student', 'delete' => 'info'` 删除列族
`disable 'student'` disable 表，删除前必须操作
`drop 'student'` 删除表


`put 'bigdata:student','1003','info:name','zhaosi'` 写入数据，允许在列族下增加列
`get 'bigdata:student','1001'` 查询数据
`get 'bigdata:student','1003' , {COLUMN => 'info:name' }` 查询具体字段
`scan 'bigdata:student'` scan扫描
` scan 'bigdata:student',{STARTROW => '1001',STOPROW => '1004'}` 左闭右开扫描
`get 'bigdata:student','1001',{COLUMN => 'info:name',VERSIONS=>6}` 获取最新的六个版本的数据
` delete 'bigdata:student','1001','info:name'` 删除数据，默认删除rowkey最新的一个版本

## 表的逻辑结构 与物理存储
逻辑上
> 列族
> 列（与关系型数据库类型）
> rowKey 行号
> region ，横向拆分按rowkey字典进行拆分
> store ，纵向拆分，以列族拆分

物理上 store
key： rowkey+column family + column qualifier + timestamp + type（put/delete）
value ：单元格的值
以时间戳 timesteamp作为版本，对数据进行修改

## 数据模型
* 命名空间 namespace 即常见的database的概念，默认有两个命名空间 ，hbase和default
* table 表，只需要声明列族即可，可按需动态追加列
* row 行，每一行都有一个rowkey和多个column组成，查询数据时只能根据rowkey进行检索，rowkey的设计十分重要
* column 每个列都有column family（列族） 和column qualifier（列限定符）进行限定，如info：name，info:age
* timestamp
* cell 概念上的单元格 ，rowkey+column family + column qualifier + timestamp + type唯一确定

## 基础架构
master：通过zk实现分布式管理，实现对region server的管理，当某个region server宕机，会将该region server管理的hdfs上的region转移给其他region server管理
region server ：实际存储的进行
使用zk实现高可用
在hdfs中region server管理的实际上是一个个region

## API操作