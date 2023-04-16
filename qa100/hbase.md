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
  用户画像，一个比较大的稀疏矩阵，蚂蚁金服的风控就是在hbase上构建
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
* thrift（rpc框架） /REST api
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
`describe_namespace`

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
pom基础包
```xml
    <dependency>
      <groupId>org.apache.hbase</groupId>
      <artifactId>hbase-client</artifactId>
      <version>2.5.3</version>
    </dependency>
```
配置文件，从服务端拷贝即可
```xml
<configuration>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1,node2,node3</value>
    </property>

</configuration>
```
创建连接
```java
@Slf4j
public class HBaseConnection {

    public static Connection connection = null;

    static{

        //默认使用同步连接
        try {
            //默认会加载hbase-site.xml
            connection = ConnectionFactory.createConnection();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) throws IOException {
        log.info("connection:{}",connection);
        closeConnection();
    }

    public static void closeConnection() throws IOException {
        if(connection != null){
            connection.close();
        }
    }
}
```

### ddl操作
```java
public static Connection connection = HBaseConnection.connection;

    /**
     * 创建命名空间，获取Admin对象来操作
     * @param namespace
     */
    public static void createNamespace(String namespace){
        try {
            //轻量级连接，非线程安全，不推荐池化或者缓存
            Admin admin = connection.getAdmin();
            //传入命名空间描述
            NamespaceDescriptor.Builder builder = NamespaceDescriptor.create(namespace);
            //设置命名空间
            builder.addConfiguration("user","steven-hbase");
            admin.createNamespace(builder.build());
            //关闭admin
            admin.close();
        } catch (IOException e) {
            throw new RuntimeException(e);
        }

    }

    public static void main(String[] args) throws IOException {
        //命名空间不能带'-'
        createNamespace("test_ns1");
        //关闭hbase连接
        HBaseConnection.closeConnection();
    }

```

判断表格是否存在
```java
    /**
     * 判断表格是否存在
     * @param namespace 命名空间名称
     * @param tableName 表名
     * @return
     */
    public static boolean isExistedTable(String namespace,String tableName) throws IOException {
        Admin admin = connection.getAdmin();

        //判断逻辑
        boolean exists = false;
        try {
            exists = admin.tableExists(TableName.valueOf(namespace, tableName));
            return exists;
        } catch (IOException e) {
            throw new RuntimeException(e);
        }finally {
            admin.close();
        }
        
    }

 //判断表格是否存在
    boolean exists = isExistedTable("bigdata","student1");
    log.info("is exists:{}",exists);
```
创建表格
```java
  /**
     * 创建表格
     *
     * @param namespace      命名空间
     * @param tableName      表明
     * @param columnFamilies 列族
     */
    public static void createTable(String namespace, String tableName, String... columnFamilies) throws IOException {
        Admin admin = connection.getAdmin();

        try {
            TableDescriptorBuilder tableDescriptorBuilder =
                    TableDescriptorBuilder.newBuilder(TableName.valueOf(namespace, tableName));
            //添加列族描述
            for (String columnFamily : columnFamilies) {
                ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder = ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(columnFamily));
                columnFamilyDescriptorBuilder.setMaxVersions(5);
                tableDescriptorBuilder.setColumnFamily(columnFamilyDescriptorBuilder.build());
            }
            admin.createTable(tableDescriptorBuilder.build());
        } catch (IOException e) {
            log.error("表格已经存在");
            throw new RuntimeException(e);
        }finally {
            admin.close();
        }
    }

    createTable("test_ns1","student","info","msg");

```
修改列族版本信息
```java
 /**
     * 修改列族的版本
     * @param namespace
     * @param tableName
     * @param columnFamily
     * @param version
     */
    public static void modifyTable(String namespace,String tableName,String columnFamily,int version) throws IOException {
        Admin admin = connection.getAdmin();

        try {
            //修改时必须要拿到原表的列族信息进行修改，使用newBuilder（）,要使用旧的表格描述
            final TableDescriptor descriptor = admin.getDescriptor(TableName.valueOf(namespace, tableName));
            TableDescriptorBuilder tableDescriptorBuilder = TableDescriptorBuilder.newBuilder(descriptor);
            //使用表格描述拿到列族描述
            ColumnFamilyDescriptor columnFamilyDescriptor = descriptor.getColumnFamily(Bytes.toBytes(columnFamily));
            ColumnFamilyDescriptorBuilder columnFamilyDescriptorBuilder = ColumnFamilyDescriptorBuilder.newBuilder(columnFamilyDescriptor);
            columnFamilyDescriptorBuilder.setMaxVersions(version);
            tableDescriptorBuilder.modifyColumnFamily(columnFamilyDescriptorBuilder.build());
            admin.modifyTable(tableDescriptorBuilder.build());
        } catch (IOException e) {
            e.printStackTrace();
        }
        admin.close();
    }

     modifyTable("test_ns1","student","info",6);
```

删除表格
```java
    /**
     * 删除表格
     * @param namespace
     * @param tableName
     * @return
     */
    public static boolean deleteTable(String namespace,String tableName) throws IOException {
        //判断表格是否存在
        if(!isExistedTable(namespace,tableName)){
            log.warn("要删除的表格不存在");
            return false;
        }
        final Admin admin = connection.getAdmin();
        boolean result = false;
        try {
            //删除操作,先disable，再删除
            admin.disableTable(TableName.valueOf(namespace,tableName));
            admin.deleteTable(TableName.valueOf(namespace,tableName));
            result = true;
        } catch (IOException e) {
            e.printStackTrace();
        }
        admin.close();
        return result;
    }
    final boolean b = deleteTable("test_ns1", "student");
    log.info("delete result:{}",b);
```

### DML操作
插入数据
```java
    /**
     * 插入数据
     * @param namespace 命名空间
     * @param tableName 表名
     * @param rowkey 主键
     * @param columnFamily 列族名
     * @param column 列名
     * @param value 值
     */
    public static void putCell(String namespace,String tableName,String rowkey,String columnFamily,String column,String value) throws IOException {
        //获取table
        Table table = connection.getTable(TableName.valueOf(namespace, tableName));

        try {
            //插入数据
            Put put = new Put(Bytes.toBytes(rowkey));
            put.addColumn(Bytes.toBytes(columnFamily),Bytes.toBytes(column),Bytes.toBytes(value));
            table.put(put);
        } catch (IOException e) {
            e.printStackTrace();
        }

        table.close();

    }
     //添加数据
        putCell("test_ns1","student","1001","info","name","zhangsan");
        putCell("test_ns1","student","1001","info","age","30");
```
扫描数据
```java

    /**
     * 扫描范围
     * @param namespace
     * @param tableName
     * @param startRow 开始的row ,包含
     * @param stopRow   结束的row，不包含
     */
    public static void scanRows(String namespace, String tableName,String startRow,String stopRow) throws IOException {
        Table table = connection.getTable(TableName.valueOf(namespace, tableName));

        try {
            //如果不对scan进行配置，则是扫描整张表
            Scan scan = new Scan();
            //默认包含
            scan.withStartRow(Bytes.toBytes(startRow));
            //默认不包含
            scan.withStopRow(Bytes.toBytes(stopRow));
            //返回多行数据，二维数组
            ResultScanner scanner = table.getScanner(scan);
            for (Result result : scanner) {
                final Cell[] cells = result.rawCells();
                for (Cell cell : cells) {
                    String resultValue = new String(CellUtil.cloneValue(cell));
                    String column  = new String(CellUtil.cloneQualifier(cell));
                    String rowkey  = new String(CellUtil.cloneRow(cell));
                    System.out.print(rowkey+":"+column+":"+resultValue+  "| ");
                }
                System.out.println();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        table.close();
    }
    scanRows("test_ns1", "student","1001","1005");
```
带过滤的扫描
```java
  /**
     *
     * @param namespace
     * @param tableName
     * @param startRow
     * @param stopRow
     * @param columnFamily
     * @param column
     * @param value
     * @throws IOException
     */
    public static void scanRowsWithCondition(String namespace, String tableName,String startRow,String stopRow,
                                             String columnFamily,String column,String value) throws IOException {
        Table table = connection.getTable(TableName.valueOf(namespace, tableName));

        try {
            //如果不对scan进行配置，则是扫描整张表
            Scan scan = new Scan();
            //默认包含
            scan.withStartRow(Bytes.toBytes(startRow));
            //默认不包含
            scan.withStopRow(Bytes.toBytes(stopRow));
            //添加过滤
            final FilterList filterList = new FilterList();
            //过滤结果只保留当前列的数据，只会取最新版本的数据，不满足条件的会删除，会保留没有当前列的数据
            final ColumnValueFilter columnValueFilter = new ColumnValueFilter(Bytes.toBytes(columnFamily),
                    Bytes.toBytes(column), CompareOperator.EQUAL, Bytes.toBytes(value));
            //过滤结果保留整行数据，只会取最新版本的数据
            final SingleColumnValueFilter singleColumnValueFilter = new SingleColumnValueFilter(Bytes.toBytes(columnFamily),
                    Bytes.toBytes(column), CompareOperator.EQUAL, Bytes.toBytes(value));
            filterList.addFilter(singleColumnValueFilter);
            scan.setFilter(filterList);

            //返回多行数据，二维数组
            ResultScanner scanner = table.getScanner(scan);
            for (Result result : scanner) {
                final Cell[] cells = result.rawCells();
                for (Cell cell : cells) {
                    String resultValue = new String(CellUtil.cloneValue(cell));
                    String columnName  = new String(CellUtil.cloneQualifier(cell));
                    String rowkey  = new String(CellUtil.cloneRow(cell));
                    System.out.print(rowkey+":"+columnName+":"+resultValue+  "| ");
                }
                System.out.println();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        table.close();
    }
    scanRowsWithCondition("test_ns1", "student","1001","1006","info","name","zhangsan");
    //结果
    /*
    1001:age:30| 1001:name:zhangsan| 
    1004:name:zhangsan| 
    1005:age:30|   (保留了不包含name的数据)
    */
```

```java

    /**
     * 删除一行中的一列数据
     * @param namespace
     * @param tableName
     * @param rowKey
     * @param columnFamily
     * @param columnName
     */
    public static void deleteColumn(String namespace, String tableName,String rowKey,String columnFamily,String columnName) throws IOException {
        final Table table = connection.getTable(TableName.valueOf(namespace, tableName));


        try {
            final Delete delete = new Delete(Bytes.toBytes(rowKey));
            //添加列信息,删除一个版本
//        delete.addColumn(Bytes.toBytes(columnFamily),Bytes.toBytes(columnName));
            //添加列信息，删除字段的所有版本
            delete.addColumns(Bytes.toBytes(columnFamily),Bytes.toBytes(columnName));
            table.delete(delete);
        } catch (IOException e) {
            e.printStackTrace();
        }

        table.close();
    }
    log.info("delete before");
    getCell("test_ns1", "student", "1003", "info", "name");
    log.info("delete......");
    deleteColumn("test_ns1", "student","1003","info","name");
    log.info("after delete");
    getCell("test_ns1", "student", "1003", "info", "name");
```

### master架构详情
> 具体实现类是HMaster，一般部署在hadoop的namenode上，管理regionserver
> hbase的表元数据信息存储在hdfs的/hbase/data/hbase/data上
> 负载均衡器：通过读取meta表了解region的分配，通过连接zk了解regionserver的活动情况，5分钟调控一次分配平衡
> 元数据表管理器： 管理hdfs的/hbase/data/hbase/data上，控制数据版本
> MasterProcWAL 预写日志管理器 ，管理maste的预写日志没如果发生宕机，让backUpMaster读取日志数据，预写日志存储在/hbase/MasterData/WALs上，设置32M或一小时滚动当前操作执行日志
> 默认情况下客户端不需要连接HMaster，直接连接zookeeper进行数据的读写操作

### RegionServer架构详情
> 实现类HRegionServer,通常部署在datanode上
> hbase的表数据先按region进行切分，然后按照列族进行store的切分，数据按照region分配给regionserver进行管理、
> WAL 预写日志，保证rowkey有序，当接受到put命令，首先会写入预写日志，然后写入到region
> block cache 读缓存，优化读效率，第一次读从文件读然后缓存在内存中，一个regionserver只有一个读缓存；写缓存，对数据按rowkey进行排序，region中的一个store对应一个写混存，regionserver存在多个写缓存
> region数据的拆分与合并

### habse写流程
从客户端创建连接开始，到最终刷写hdfs落盘数据结束
> client向zk发起连接
> 读取zk存储的meta表是由哪个regionserver管理
> 访问对应的regionserver的meta表（生产环境中的meta表数据量会较大，hbase客户端是重量级客户端）
> 将读取的meta表作为属性保存在客户端中
> 客户端发送put请求，有可能由于region管理的变更导致put失败，客户端需要在meta表发生变更时重新发送请求读取缓存
> 客户端的put数据首先放在WAL中
> 随后写入hdfs的WAL中
> 操作put请求写入到对应的Mem story（写缓存），等待触发刷写条件写入对应的store

### MemStore Flush 写缓存刷写
> 当某个Memstore的大小达到了hbase.hregion.memstore.flush.size(默认128M)，或hbase.hregion.memstore.block.multiplier(默认4，意思是memstore的数据达到了128M*4)， region所有的memstore都会进行刷写
> hregionserver中的属性memstoreflusher内部线程FlushHandler控制，当memstore的总大小达到低水位线，`java_heapsize*hbase.regionserver.global.memstore.size(默认是0.4)*hbase.regionserver.global.memstore.size.lower.limit(默认0.95) `

### HFile（store）结构
存储在HDFS上每个store文件夹下实际存储数据的文件
包括数据本身（key value键值对）、元数据记录、文件信息、数据索引、元数据索引、一个固定长度的尾部信息
使用以下命令查看文件内容信息

```shell
hbase hfile -m -f /hbase/data/test_ns1/student/4e3213fb7da295b19227f9cc17f0fb04/info/12c09af87ee042a98e850429d462aadd
```
### 读流程
> client向zk发起连接
> 读取zk存储的meta表是由哪个regionserver管理
> 访问对应的regionserver的meta表（生产环境中的meta表数据量会较大，hbase客户端是重量级客户端）
> 将读取的meta表作为属性保存在客户端中
> 客户端发送get请求
> 将请求写入wal并落盘
> 读取bolck cache，判断之前是否缓存过
> 读取对应写缓存和store文件同时缓存到bolck cache
> 合并数据并发挥

### 文件合并
由于memstore么次刷写都会生成一个新的hfile，文件过多不方便读取，需要进行文件合并，并清除冗余数据
分为小合并（相邻文件进行合并）和大合并

### region 切分
预分区（自定义分区）
创建表时直接操作,按照rowkey进行预分区
```shell
create 'staff1','info', SPLITS =>['1000','2000','4000']
```
生成16进制的预分区,切分成16个分区
```shell
create 'staff2','info',{NUMREGIONS => 15,SPLITALGO => 'HexStringSplit'}
```

使用java API也可以实现自定义分区region

### 系统拆分
region的拆分由regionserver完成，在操作之前需要汇报zk，修改对应的meta表信息添加两列info：splitA和info：splitB信息，之后操作hdfs上对应的文件，按照拆分之后的region范围进行标记分区

## rowkey的设计
> hbase表格可以按照mysql的表格的方式进行设计

> TSDB（timestamp db） ：时间序列数据库，将时间戳写入到rowkey里，记录值的每一次变化

一条数据的唯一标识就是rowkey，这条数据存储于哪个分区，取决于rowkey处于哪一个预分区的区间内，设计rowkey的主要目的，是让数据均匀扽分布在所有的region重，在一定程度上防止数据倾斜。
方案：
1. 生成随机数，hash，散列
2. 时间戳反转（99999999-时间戳），将新的时间戳放在前面
3. 字符串拼接
实际案例：
统计2021年12月zhangsan的消费总金额
使用scan扫描，设置startrow，stoprow
填充字段，保证所有的数据有相同的长度，拼接足够多的^A保证长度相同，
startrow： ^Azhangsan2021-12
结尾需要写和设计的rowkey对应位置的痣打的字符 -的后面时.
stoprow：  ^Azhangsan2021-12.

rowkey设计：
^：ascii表的第一个字符
`^A^A(user)date(yyyy-MM-dd hh:mm:ss ms)`

统计2021年12月所有人消费的总金额
上面的rowkey只能按用户进行扫描，无法满足统计所有人2021年12月的消费
设计思想：**可以穷举的写在前面**
将日期进行拆分，年月放在前面，中间放定长的用户名，后面放时间的后半部
`2021-12^A^Azhangsan-01 10:10:11`
对与只查zhangsan的
`scan startrow => 2021-12^A^Azhangsan`
     `stoprow  => 2021-12^A^Azhangsan.`
对与查所有人的
`scan startrow => 2021-12`
     `stoprow  => 2021-12.  `

添加预分区优化：在rowkey的前面添加分区号
`000-2021-12^A^Azhangsan-01 10:10:11` 
`001-2021-12^A^Azhangsan-01 10:10:11 `
  
分区号 => hash(user+ date(MM))%120


参数优化
hbase-site.xml
zookeeper.session.timout，zookeeper会话超时时间,默认是90000（90s）
hbase.client,重试间隔时间,默认100ms
hbase.client.retries,number 默认15次
hbase.regionserver.handler.count 指定rpc监听的数量，可以根据客户端的请求数进行调整
hbase.hregion.majorcompaction 大合并时间，默认值60480000（7天）
hbase.hregion.max.filesize region文件切分大小，默认10g，推荐20G
hbase.client.write.buffer 指定hbase客户端缓存，增大该值可以减少rpc调用次数，默认2M
hbase.client.scanner.caching 用于指定scan.next方法获取默认行数，值越大，消耗内存越大
hfile.block.cache.size blockCache（读缓存）占用regionserver堆内存的比例，默认0.4
hbase.regionserver.global.memstore.size memstore写缓存占用regionserver内存的比例

JVM调优
内存设置

垃圾回收器设置，考虑数据洪峰造成的OOM
设置使用CMS垃圾回收器
-XX：+UseConcMarkSweepGC
保持新生代尽量小，同时尽早开启GC
-XX：CMSInitiatingOccupancyFraction=70 内存占用到70%开启垃圾回收
并行垃圾货收
-XX:+UseParNewGC
 
HBase经验法则
1. region的大小默认控制在10-50G，建议20G
2. cell的大小不超过10M（性能对小于100K的值优化），如果使用Mob则不超过50M
3. 一张表的列族不要设计太多，1-3个 ，最好1个，保证尽量保证不会同时读取多个列族
4. 1-2个列族的表格，设计50-100个region
5. 列族名称尽量短
6. 如果rowkey设计时间在最前面，会导致有大量的就数据存储在不活跃的region中，使用时，仅会操作少数的活动region，此时建议增加更多的region的数据
7. 如果只有一个列族用于写入数据，分配内存资源的时候可以做出调整，写缓存不会占用太多的内存

## 整合phoenix
Phoenix是hbase开源的sql皮肤，可以使用标准的jdbcAPI代理hbase的API来创建表，插入数据和查询hbase数据
Phoenix对用户输入的sql同样有大量的优化搜短
### 安装
拷贝
phoenix-server-hbase-xxx.jar 放入到hbase的lib目录下
配置环境变量

```shell
export PHOENIX_HOME=/Users/lijie3/Documents/tool-package/phoenix-hbase-2.5-5.1.3-bin
export PATH=$PATH:$PHOENIX_HOME/bin
export PHOENIX_CLASSPATH=$PHOENIX_HOME/bin
```
重启habse

连接phoenix ：
/phoenix-hbase-2.5-5.1.3-bin/bin/sqlline.py localhost:2181

如果连接失败,需要删除/home/.sqlline

建表：
记得表名、列名加双引号避免变成大写
```sql
create table if not exists student1(id varchar primary key, name varchar, age bigint,addr varchar);
```

查询表
select * from student1;

+----+------+-----+------+
| ID | NAME | AGE | ADDR |
+----+------+-----+------+
+----+------+-----+------+

phoenix为了减少数据对磁盘空间的占用，默认会对hbase中的列名做了编码处理，如果不想对列名进行编码，可在建表语句末尾加上COLUMN_ENCODED_BYTE=0；避免列编码

退出
!quit

视图映射（不可写）
创建hbase的表
`create 'test2','info1','info2'`
`put 'test2','1001','info1:name','zhangsan'`
`put 'test2','1001','info2:address','hz'`

创建视图
`crete view "test2"(id varchar primary key,"info1"."name" varchar,"info2"."address" varchar);`
查询视图数据
`select * from "test2";`
+------+----------+---------+
|  ID  |   name   | address |
+------+----------+---------+
| 1001 | zhangsan | hz      |
+------+----------+---------+

表映射（必须将COLUMN_ENCODED_BYTE=0;加上避免列编码）,可读可写，可编辑
create table "test2"(id varchar primary key,"info1"."name" varchar,"info2"."address" varchar) COLUMN_ENCODED_BYTE=0 ;

使用无符号的数据避免数值数据乱码
unsigned_long

### phoenix jdbc连接
```java
public static void main(String[] args) throws SQLException {
        //标准jdbc连接
        String url = "jdbc:phoenix:localhost:2181";
        //配置对象。无用户名密码
        Properties properties = new Properties();

        //获取连接
        Connection connection = DriverManager.getConnection(url, properties);
        // 4.编译sql,sql不能加分号
        final PreparedStatement preparedStatement = connection.prepareStatement("select * from 'test1'");
        final ResultSet resultSet = preparedStatement.executeQuery();
        while (resultSet.next()) {
            System.out.println(resultSet.getString(1));
        }
        connection.close();
    }
```

### phoenix的二级索引
设置二级索引参数,hbase-site.xml加上如下的配置
```
 hbase.regionserver.wal.codec   => org.apache.hadoop.regionserver.wal.IndexedWALEditCodec
```
全局索引 默认索引格式，适用于多读少写的场景
包含索引 
本地索引  适合写操作频繁的场景

## 集成hive
如果大量数据已经存储在habse上，需要对已经粗在的数据进行数据分析处理，那么phoeinx不适合做特别复杂的sql处理，可以使用hive映射hbase的表格，然后使用hql进行分析处理







