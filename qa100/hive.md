# hive

## 数据仓库
* data warehouse，用于存储，分析，报告的数据系统
* 目的是构建面向分析的集成化的数据环境，分析结果为企业提供决策支持
* 不生产任何数据，数据来自外部系统
* 不需要消费任何数据，其分析结果开放给外部应用使用
* 是数据仓库，不是数据工厂
  
### 场景
* 操作型记录的保存，分析性决策的制定，数据驱动决策
* 在不影响OLTP 联机事务处理系统的情况下，将OLTP中的数据进行统一etl，实现对数据的分析
* 面向分析支持分析，OLAP，联机分析处理系统

### 数据仓库的特征
> 面向主题性 -- subject-oriented 主题是一个抽象的概念，是较高层次数据综合，归类并进行分析利用的抽象
* 抽象层次上对数据进行完整、一致和准确的描述
> 集成性 -- integrated 主题相关的数据通常会分不在多个操作性系统中，彼此分散、独立、异构，需要继承到数据仓库主题下
* 统一源数据中所有矛盾之处，进行数据综合计算
> 非易失性 -- non-volatile  非易变性，数据仓库是分析数据的平台，而不是创造数据的平台
* 大量的查询操作，修改操作很少
> 时变性-- time-variant 数据仓库的数据需要随着时间更新，以适应决策的需要
* 包含各种粒度的历史数据，数据仓库的数据需要随着时间更新，以适应决策的需要




### 一些概念
* 联机事务处理系统（OLTP）-- 关系型数据库时OLTP的典型应用，对数据进行增删改查，保证对事物的支持，对实时性要求高，数据量小  
* 联机分析处理系统 （OLAP）-- 做数据分析，赠对某些主题的历史数据进行复杂的多位分析，支持管理决策，对实施性要求小，数据量大
* ETL ：抽取extra、转换trasfer、加载load ，从数据源获取数据及在数据仓库内的数据转换和流动都可以认为是ETL
* ELT：抽取，先加载再转换
* 数据仓库不是用来取代数据库，是在数据库数据的基础上开展数据分析
* 数据库是为了捕获数据而设计，而数据仓库是为了分析数据而设计
* 数据集市data mart：面向单个部门使用，数据仓库面向整个企业 

### 数据仓库的分层
分层能清晰数据结构、追踪数据血缘、减少重复开发
* ODS 操作型数据层，源数据层，数据结构上源系统保持一致，实现与数据源系统进行解耦
* DW 数据仓库层，完成数据加工与整合
* DA 数据应用层

## hive的定义
建立在hadoop智商的开源数据仓库系统，将存储在hadoop的结构化，半结构化数据文件映射为一张数据库表，使用HQL，将HQL转换为MapReduce程序，将程序提交到Hadoop集群执行
让使用者专注于写HQL，hive将HQL转换为mapreduce完成数据分析

* hive能将数据文件映射成一张表
* hive将sql语法解析编译成为mapreduce代码

## hive的组织架构
* 用户接口，编写sql的方式进行交互，包括CLI、JDBC/ODBC、webGUI
* 元数据存储，表和文件之间的映射关系，存储在关系型数据库中（msyql/derby）
* Driver驱动程序，包括语法解析器，计划编译器，优化器，执行器
* 执行引擎，通过执行引擎处理（mapreduce、tez、spark）进行数据处理

## hive数据模型
分为几个个类别：
* database，类似于rdbms中的数据库的概念
* table 表 ，表对应的数据存储在hdfs上，表相关的元数据存储在mysql中
* partition 分区，一个文件夹表示一个分区，分区列= 分区值
* bucket 桶，优化join查询和方便抽样查询

## 安装

### 元数据metastore
元数据 -- 描述数据的数据
部署模式：内嵌模式（derby，不需要单独启动），本地模式（mysql，不需要单独启动），远程模式（mysql单独启动）

### 服务器基础环境
集群时间同步、防火墙关闭、主机host映射、免密登陆、jdk安装
* 从hadoop中复制guava包到hive的lib目录下解决guava的版本冲突
* 配置hadoop的core-site.xml
```xml
    <!-- 整合hive-->
    <property>
        <name>hadoop.proxyuser.root.hosts</name>
        <value>*</value>
    </property>
    <property>
        <name>hadoop.proxyuser.root.groups</name>
        <value>*</value>
    </property> 
```
* 配置hive-env.sh
```shell
export HADOOP_HOME=/Users/lijie3/Documents/tool-package/hadoop-3.3.5
export HIVE_CONF_DIR=/Users/lijie3/Documents/tool-package/apache-hive-3.1.3-bin/conf
export HIVE_AUX_JARS_PATH=/Users/lijie3/Documents/tool-package/apache-hive-3.1.3-bin/lib
```
**内嵌模式只适合测试使用**
* 内嵌模式启动metastore，初始化metadata,使用内置的derby
```shell
./bin/schematool -dbType derby -initSchema
```
* 启动hive
```shell
./bin/hive
```



**本地模式**
安装mysql数据库
配置hive-site.xml
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
<!--本地模式-->
        <property>
                <name>hive.metastore.local</name>
                <value>true</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>	
                <!-- 这里自己搓自己的mysql的ip地址 -->
        </property>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.cj.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>dxy123456</value>
                <!-- 这里搓自己的mysql账号和密码 -->
        </property>
        <property>
                <name>hive.metastore.event.db.notification.api.auth</name>
                <value>false</value>
        </property>
        <property>
                <name>hive.metastore.schema.verification</name>
                <value>false</value>
        </property>

</configuration>
```
* 初始化元数据
`./bin/schematool -initSchema -dbType mysql -verbos`
* 启动hive
`./bin/hive`  

**远程模式**
* 修改本地模式下的hive-site.xml文件
```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
     
        <property>
                <name>javax.jdo.option.ConnectionURL</name>
                <value>jdbc:mysql://localhost:3306/hive?createDatabaseIfNotExist=true</value>	
                <!-- 这里自己搓自己的mysql的ip地址 -->
        </property>
        <property>
                <name>javax.jdo.option.ConnectionDriverName</name>
                <value>com.mysql.cj.jdbc.Driver</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionUserName</name>
                <value>root</value>
        </property>
        <property>
                <name>javax.jdo.option.ConnectionPassword</name>
                <value>dxy123456</value>
                <!-- 这里搓自己的mysql账号和密码 -->
        </property>
        <property>
                <name>hive.metastore.event.db.notification.api.auth</name>
                <value>false</value>
        </property>
        <property>
                <name>hive.metastore.schema.verification</name>
                <value>false</value>
        </property>
         <property>
                <name>hive.server2.thrift.bind.host</name>
                <value>locahost</value>
        </property>

        <property>
                <name>hive.metastore.uris</name>
                <value>thrift://localhost:9083</value>
        </property>
</configuration>
```
* 启动metastore
`./bin/hive --service metastore`
`nohup ./bin/hive --service metastore &`
开启debug日志
`nohup ./bin/hive --service metastore --hiveconf hive.root.logger=DEBUG,console &`

**hive第一代客户端使用**
远程访问：
在本地配置metastore的地址
```xml
        <property>
                <name>hive.metastore.uris</name>
                <value>thrift://localhost:9083</value>
        </property>
```
执行一下命令
`./bin/hive`


**使用beeline**
不能直接访问metastore，需要先启动hiveserver2
`nohup ./bin/hive --service hiveserver2 &`
使用beeline
```shell
./bin/beeline
# 连接hiveserver2
beeline> !connect jdbc:hive2://localhost:10000
```
## 数据类型
### 原生数据类型


### 复杂数据类型
array、map、struct、union

读写文件机制：SerDe

内部表：默认情况下使用的是内部表，drop内部表会直接删除内部表相关的元数据及数据信息
外部表：数据文件放在指定的目录下，使用external的建表语句，drop外部表时不会直接删除外部表对应的数据文件，只会删除元数据

## HQL语法
### DDL语言
基本语法
create table
### 分区
避免扫描全部文件（全表）
* 静态分区
用法
使用partition指定分区字段
使用load字段指定分区的值
-- 手动指定分区字段及分区值,在hdfs中分区使用文件夹来体现
-- 实际中可以使用双分区表，使用两个字段进行两种级别的分区
* 动态分区
开启动态分区
set hive.exec.dynamic.partition=true;
set hive.exec.dynamic.partition.mode=nonstrict;
```sql
insert into table tablename partition(role) 
select tmp.* ,tmp.role from tablename
```

### 分桶表 bucket tables
将数据文件分成若干个部分
好处：
* 基于分桶字段扫描时，可以避免全表扫描
* join时可以提高MR程序效率，减少笛卡尔积数量（根据join字段进行分桶）
* 分桶表数据进行抽样效率会比较高
开启分桶功能
hive.enforce.bucketing=true
hive.optimize.bucketmapjoin=true
hive.optimize.bucketmapjoin.sortedmerge=true

### 事物表
开启事务：
```xml
<property>
  <name>hive.support.concurrency</name>
  <value>true</value>
</property>
<property>
  <name>hive.enforce.bucketing</name>
  <value>true</value>
</property>
<property>
  <name>hive.txn.manager</name>
  <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
```
创建事务表
```sql
CREATE TABLE my_table (
  id INT,
  name STRING
)
PARTITIONED BY (date STRING)
STORED AS ORC
TBLPROPERTIES ('transactional'='true');
```
### 视图
开启视图功能：
```sql
set hive.enforce.bucketing=false;
set hive.enforce.sorting=false;
```
创建视图：
```sql
CREATE VIEW my_view AS
SELECT col1, col2
FROM my_table
WHERE col3 = 'some_value';
```

### 物化视图 meterialized views
一个包含查询结果的数据库对象，预先计算并保存表连接或聚集等耗时较多的操作
通过与计算，提高查询性能，需要占用一定的存储空间
创建物化视图
```sql
CREATE MATERIALIZED VIEW my_materialized_view
STORED AS PARQUET
LOCATION '/path/to/my/materialized/view'
AS
SELECT *
FROM my_table
WHERE my_column = 'some_value'
```

### 其他
* 创建数据库
create database ... location ...

* 描述表信息
describe formatter tablename

* 删除表
drop table tablename [purge] --purge 使用垃圾桶机制

* 清空数据
truncate table tablename

* 修改表
alter table

* 添加分区

alter table name add partition(clomunname= "") location "...";

* 检查/修复分区，快捷创建分区的方式

MSCK partition



### DML/DQL
**load加载数据**
-- 从hiveserver2的那台服务器上加载文件到hive表中
load data local inpath '/root/data/test.data' into table tablename;
--从hdfs上移动文件mv的操作
load data inpath '/student.txt' into table student_hdfs;
--加载分区表
load data inpath '/student.txt' into table student_p partition(country="china")

* 3.0版本hive 的load加载数据时会将操作转化为insert as select，同时还允许指定分区字段值


**insert的使用**
使用insert插入数据时底层使用的是mr程序来执行，效率很低
实际很多场景都需要对数据进行抽取、转换，可以使用中使用insert+select来加载数据

insert into/overwrite table table_name select cloumn1,column2 from table_name2;

多重插入：一次扫描多次插入
from student
insert overwrite table student_2
select num --将num字段插入student_2
insert overwrite table student_3
select name; --将name字段插入student_3

**dynamic partition insert 动态分区插入**
通常使用非严格模式 nostrict
insert into table student_1 partition(dept)
select num,name,sex,age,dept from student

严格模式 strict下，必须存在一个静态分区的指定:
from tablename t1
insert overwrite table tablename2 partition(dt = "2023-02004",country)
select t1.id,t1.name ,t1.age null,t1.country

**导出数据 insert directory**
将select 查询的结果导出成文件存放在文件系统中
--导入到hdfs文件中
insert overwrite directory '/tmp/hive/_export/f1' select num,name,age from student limit 200;
--导入到本地文件系统，hiveserver2的本地文件系统
insert  overwrite local directory '/tmp/hive/_export/f1' select num,name,age from student limit 200;


### hive 事务表
* 使用事务的原因
流式传输数据（kafka数据）可能会导致脏读的问题
变化缓慢数据更新
数据修正 --局部对数据进行变更
* 实现原理
用hdfs文件作为原始文件，用delta保存食物操作的记录增量数据
使用合并器定期的合并delta增量文件
* 设置事务参数，开启事务，
* 创建事务表（文件是orc存储）
  
  create table emp (id int, name string)
  stored as orc tblproperties ( 'transactional' = 'true')


### update、delete
hive中如果一张表不支持事务表，则这张表不支持事务

### select 查询数据
* 基础语法与mysql的语法一致
* 查询语句的执行顺序：
from > join >  where > group、聚合 > having > order > select

* 分组、排序
> order by
在hive中的group by中如果需要对数据进行排序，由于底层使用的是MR程序来执行，只会有一个reduceTask执行，如果输出的行数太大，会导致需要很长时间完成
在实际使用中尽量少用排序
> cluster by
只能根据一个字段进行分组排序
根据指定的字段将数据进行分组（hash分组），每组内再根据该字段正序排序

多字段分组排序
cluster by 是一个字段的分组排序
cluster by c1 = distribue by c1 sort by c1

distribue by cloumn1 sort by cloumn2
按column1 分组  按照column2 进行排序

* union 
默认union会删除重复行 ，等效于 union distinct

* 子查询

* cte -- common table expressions
临时结果集

with q1 as (select * from student where num = 95002)
     q2 as (select * from student where num = 95004)
select * from q1 union all select * from q2;    

create table t2 as 
with q1 as (select * from student where num = 95002)
select * from q1;

* join 

支持隐式连接和不等值的连接
内连接 inner join = join
外连接 full outer join = full join
左连接 left join 
右连接 right join
只返回左边表满足join的on条件的记录，相当于inner join，但是只返回左表的部分 left semi join
交叉连接，两个表的笛卡尔积 cross join 

当join条件使用相同的列时，hive将使用同一个mr作业来完成数据处理，有助于性能优化

join时将数据量最大的表放在最后，reducer将会使用流式传输，其中缓冲之前的其他表

join条件是在where条件之前执行

### hive参数配置
* 使用set方式设置是会话级别的参数修改
* 使用hiveconf的方式进行配置（命令行上的配置）
* 使用配置文件（hive-site.xml,metastore-site.xml,hiveserver2-site.xml）

推荐使用set方式进行配置

### 操作符
查看所有的运算符
show functions;
查看某个运算符或函数
describe function +;
查看详情
describe function extended count;
创建一张空表
create table dual（id string）;

### hive函数
分类
内置函数：数值类型操作函数、时间函数
用户自定义函数 UDF user defined function
按输入的行数分为
UDF 普通函数， 一进一出
UDAF：聚合函数，多进一出函数
UDTF：表生成函数，输入一行，输出多行，一进多出
**内置函数**
条件函数：
if(1=2,100,200) -- 条件为真返回100，为假返回200

空值转换函数：
select nvl("allen","bob") ,第一个为空返回第二个值

select case 100 when 50 then 'tom' when 60 then 'bob' else 'jeff' end;

coalesce(val1,0) ,返回第一个非空的字段，如果val1为空，则返回0

**UDF**
实现步骤：
* 写一个java类，继承UDF，并重载evaluate方法，方法中实现函数的业务逻辑；
* 重载意味着可以在java类中实现多个函数功能
* 程序打成jar报，上传hs2服务器本地或hdfs
* 客户端命令行添加jar到hive的classpath：hive>add jar /jar/udf.jar
* 注册成为临时函数（给UDF命名）：create temporary function 函数名 as ‘udf类path’
* hql中使用函数

**explode函数 UDTF函数 一行变多行**
select explode(`array`(11,22,33,445,555)) array转多行 
select explode(`map`("id:,11,"name":"tom")) map kv转多行

**侧视图 lateral view**
lateral view，结合explode UDTF函数，解决udtf函数的一些查询限制问题，实现多字段的表生成 --类似于join的语法
select a.team,b.year 
from table1 a lateral view  explode(cloumn1) b as year
order by b.year desc;
也可以用于排序分组排序分组
select a.team,count(*)  as cnt
from table1 a lateral view  explode(cloumn1) b as year
 group by a.team
order by cnt desc;

**聚合函数 aggregation  udaf 多行变一行**
count 对非空字段进行计数统计
max、sum、min、avg

增强聚合
* grouping sets 多个分组聚合的简写
将多个group by 写在一个sql语句中的便利写法
select month,day,count(distinct coolieid) as nums,
grouping_id
group by month,day
grouping set(month,day)
order by grouping_id

* cube 数据立方
select 
        month,
        day
        count(distinct cookied) as nums,
        grouping_id
 from cookie_info 
 group by month,day
 with cube  --对数据进行多个维度的聚合统计 group by month、day、（month、day）、（）
 order by group_id       

* rollup
以左侧维度为主，从该维度进行层级聚合
abc3个维度，所有的组合情况是（a,b,c） (a,b) (a),()

select 
        month,
        day
        count(distinct cookied) as nums,
        grouping_id
 from cookie_info 
 group by month,day
 with rollup
 order by group_id 


**窗口函数 OLAP 与mysql的窗口函数几乎完全一致**
输入值是从select语句的结果集中的一行或多行的窗口中获取的
格式：
function（arg1....） over([partition by ....] [order by .....][window_expression])  --注意order by 是累加聚合
group by的聚合
select dept ，sum(salary) as total from emplyee group by dept
sum窗口聚合,按照dept进行窗口分组
select id ,name,deg,salary,dept,sum(salary) over(partition by dept) as total from emplyee

window expression
控制行范围的能力

序号函数
rank() over... --重复的数据标号被挤占，标号会重复，会出现不连续的情况
dense_rank() over ... --重复的数据标号被挤占，标号会重复，标号会连续
row_number() over ... --重复的数据标号不被挤占，标号连续切不重复
适合topN业务分析

ntile函数
将组内的数据氛围指定的若干个桶，并为每个桶分配一个桶编号
ntile(3) over ... 将分组后的数据分到3个桶里

窗口分析函数
lag 向上取一行或取第几行的数据

lead 向下取一行数据

first_value  分组后截止到当前行的第一行数据

last_value 分组后截止到当前行的最后一行数据

**抽样函数**
数据量过大时，使用抽样函数反喜数据中的子集
随机抽样 
rand（）函数，随机，缺点是抽样数据慢
select * from studen distribute by rand() sort by rand() limit 2;

block 基于数据块的抽样,不够随机
随机取n行数据，百分比数据或指定大小的数据
采样粒度时hdfs的大小
select * from student tablesample(1 rows);

基于分桶表的抽样 bucket table
tablesample( bucket x out of y [on column1]) 从第x个桶开始抽样，y时table的桶的总数的因子或这倍数 
select * from student tablesample(bucket 1 out of 2 on age) --基于分桶字段来抽样，效率更高


### 多字节分隔符处理
LazySimpleSerDe 只能实现对文件的单字节分隔符进行字段分隔，无法处理多字节的分隔符
解决方案：
* 对数据进行清洗，将多字节分隔符换成单字节分隔符,无法解决字段中存在分隔符的情况
* 使用正则分隔处理类 RegExSerDe分隔数据字段 
```sql
CREATE TABLE my_table (
  col1 STRING,
  col2 INT,
  col3 DOUBLE
)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES (
  "input.regex" = "^(\\S+)\\s+(\\d+)\\s+(\\d+\\.\\d+)$"
)
STORED AS TEXTFILE;
```
* 自定义InputFormat实现对分隔符的解析，成本较高，需要重写组件清洗数据
```sql

CREATE TABLE my_table (
  col1 STRING,
  col2 INT
)
STORED AS INPUTFORMAT 'org.apache.hadoop.mapred.TextInputFormat'
OUTPUTFORMAT 'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION '/path/to/my_table';
```

### URL解析函数
使用parse_url和parse_url_tuple （UDTF函数）对url进行解析
```sql
SELECT parse_url(url, 'QUERY') as query_param
FROM table_name
--取id请求字段
SELECT parse_url(url, 'QUERY', 'id') as id_param
FROM my_table
```
```sql
SELECT parse_url('https://www.example.com/path/to/page?param1=value1&param2=value2','HOSt', 'QUERY');
```
* 配合侧视图使用parse_url_tuple
```sql
SELECT a.id as id,b.host as host,b.path as path,b.query as query

FROM my_table a 
LATERAL VIEW parse_url_tuple(url,"HOST","PATH","QUERY") b as  host,path,query;
```
外连接技巧(out join 相同的功能)
LATERAL VIEW out explode（array()）


### 行列转换
* 使用case  when实现转换

* 多行转单列
不去重：select collect_list(col1) from table1;
去重：select collect_set(col1) from table1;

--指定拼接符进行拼接
concat_ws(splitchar,e1,e2)

```sql
select 
        col1,
        col2 ,
        concat_ws(',',collect_list(col3 as string)) as val 
from table1 
group by col1,col2;
```

* 多列转多行
使用union all/union实现
```sql
select col1, 'c' as col2 ,col2 as col3 from table1
union all
select col1, 'd' as col2 ,col3 as col3 from table1
union all
select col1, 'e' as col2 ,col4 as col3 from table1
```

* 单行转多列
使用explode()+侧视图转换实现
```sql
select 
col1,col2
lv.col3 as col3
from table1 t1 lateral view
explode(split(col3,',')) lv as col3 
```

### json数据处理
* 提取json字段数据函数
get_json_object --一次只能提取一个字段
json_tuple  --UDTF，一次可以解析多个值，一般搭配lateral view使用

* 建表时使用JSONSerde加载json文件到表中，会自动解析json数据
```sql
CREATE TABLE my_table (
  id INT,
  name STRING,
  age INT
)
ROW FORMAT SERDE 'org.apache.hive.hcatalog.data.JsonSerDe'
STORED AS TEXTFILE;
```

### lead函数
连续登陆的窗口函数
lead(logintime,0) over(partition by userid order by logintime) --取下一行的对应数据

### 拉链表
用于解决数据仓库中数据发生变化如何实现数据存储的问题


### hive数据分析的优化
表优化设计：
* 分区表设计，使用表时使用分区字段进行查询，避免全表扫描
* 分桶表设计，在join操作时，为避免全表的笛卡尔积，避免shuffle，将表的文件将表文件根据hash划分成不同的hash桶
* 索引设计，hive不支持主键和外键索引，在hive3.0之后索引功能被完全移除，因为索引创建的过程过于复杂，是通过mr程序创建索引，效率较低，当数据表中的数据发生变更时，索引不会自动变更，需要手动执行alter来进行索引的更新

文件格式： 通过sorted as file_format_name来指定文件格式
* TextFile --文本格式
* SequenceFile --hadoop用户来存储序列化的键值对二进制文件，可以被mr直接读取
* ORC --列式存储，hive自带的文件格式
* Parquet --列式存储，提供磁盘IO率

数据压缩：
* 减小文件存储空间
* 加快文件传输效率，提供系统的处理速度
* 降低IO读写次数
* 使用数据时需要先对文件解压，加重CPU的负载

hive支持的压缩算法：
开启hive中间传输的压缩功能
Bzip2
snappy
Gzip

存储优化

explain 查询执行计划

本地模式： 让hive的计算处理都在本地运行，不依赖于hadoop的mapreduce
* 开启本地模式
set hive.exec.mode.local.auto = true

查看任务是否是本地模式：如果任务编号带local ： job_local1032541728_0001 则是本地模式
如果任务编号不带local，则是yarn模式

* jvm 重用（hadoop3不再支持该配置），使得JVM实例在同一个job中重新使用N次，当一个task运行结束后，JVM不会进行释放，而是继续提供下一个task运行，直到运行了N个Task之后释放
在mapred-site.xml中配置参数
mapreduce.job.jvm.numtask=10

  


开启并行模式，让不存在依赖关系的stage并行运行
set hive.exec.parallel = true
set hive.exec.parallel.thread.number = 16


join优化
hive的join通过mapreduce实现，hive提供的优化方案：
> map join 只有map的jion适合小表join大表，或小表join小表， 参数配置 hive.auto,convert.join.noconditionaltask.size=51200000
> reduce join 适合大表join，hive会自动判断是否需要进行reduce端join
> bucket join 适合大表join大表，基于桶之间join， 参数配置：set hive.optimize,bucketmapjoin=true;
> 使用sort merge bucket join（SMB），需要对数据分桶并排序 参数配置set hive.optimize.bucketmapjoin=true;
set hive.optimize.bucketmapjoin.sortedmerge=true;

优化器--关联优化
开启关联操作：
set hive。optimize.correlation=true

RBO优化器：基于规则的优化器，根据设定好的规则来对程序进行优化，hive默认使用RBO
CBO优化器：基于代价的优化器，根据不同的场景所需的，根据不同的场合付出的代价（使用analyze分析器来进行评估，也是使用mapreduce来对评估）来选择最合适的方案
CBO开启：
set hive.cbo.enable=true;

当一个程序中如果有一些操作存在关联性，可以在一个mapreduce中实现，在hive中有两种方式实现：
> 方式1: 

第一个mapreduce 做group by ，经过shuffle阶段对id做分组
第二个mapreduce 对第一个mapreduce的结果做order by 经过shuffle阶段对id进行排序   

> 方式2:

使用一个mapreduce的shuffle既做分组也做排序 


谓词下推（PPD） --将过滤的表达式尽可能移动到靠近数据源的位置
开启ppd(默认开启)
hive.optimize.ppd=true


避免数据倾斜
开启map端聚合
hive.map.aggr=true;
实现随机函数分区
select * from table distrubute by rand();
数据倾斜时自动负载均衡(空间换时间)
hive.groupby,skewindata=true;
针对join：
* 提前过滤
* 使用分桶表join
* 使用skew join，将map join和reducejoin合并


hive3之后的特性

使用tez引擎，DAG作业，tez最终也是将任务提交到yarn上

LLAP更新 --实时长期处理，降低系统IO和hdfs 的datanode的交互，以提高程序的性能呢，目前只支持TEZ

metastore独立部署模式







