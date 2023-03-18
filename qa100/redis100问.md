# redis100问
## 1. redis的特性
高性能、高可用的NOSQL数据库，以键值对的方式存储数据
> 键值对类型数据库
> 单线程，每个命令具有原子性
> 低延迟，速度快（基于内存，IO多路复用）
> 支持数据持久化（快照全量备份和AOF连续增量备份）
> 支持主从集群、分片集群（数据拆分，水平扩展）

## redis 配置文件
redis.conf:
```conf
    ##是否开启保护模式
    protected-mode yes
    ## 是否开启后台守护进程，开启守护进程后redis的启动会在后台运行
    daemonize yes
    #允许连接的客户端ip
    bind 0.0.0.0
    # redis数据目录
    dir /tmp/redis
    ##redis日志目录
    logfile /tmp/redis
    ## redis dump文件名称，和dir组成完整的数据存储路径
    dbfilename dump.rdb
    ## redis连接密码设置
    requirepass dsdfs
    ## 设置数据库的数量
    databases 16
```
## redis启动命令
```shell
    ## 启动
    redis-server redis.conf
    ## 客户端连接
    redis-cli -h 127.0.0.1 -p 6379  -a dsdfs(密码)
    auth [username] password
```
## 2.redis的数据类型
### String
> 字符串、整数或浮点
> 对字符串或字符串的一部分执行操作（GET、SET、DEL）
> 对整数或浮点数执行自增或自减操作

**命令**
> `SET key value [EX seconds|PX milliseconds|EXAT timestamp|PXAT milliseconds-timestamp|KEEPTTL] [NX|XX] [GET] ` 添加或修改
> `GET key` 获取
>  `MSET key value [key value ...]` 批量设置修改
> `MGET key [key ...]` 批量获取
> `INCR key` 自增
> `INCRBY key increment` 指定步长自增
> `INCRBYFLOAT key increment` 对浮点数指定步长自增
> `SETNX key value` 存在则新增
> `SETEX key seconds value` 新增并设置有效期

### list
> 链表--双向链表
> 有序
> 允许元素重复
> 插入删除快
> 查询速度一般
> 从链的两端推入或弹出（RPUSH、LRANGE、LINDEX、LPOP），根据偏移量进行修剪（trim）
> 读取单个或多个元素，根据值查找或移除元素

**命令**

### set
> 包含无重复字符串的无需收集器
> 添加、获取、移除单个元素，检查元素是否在集合中（SADD、SMEMBERS、SREM、SISMEMBER），计算交、并、差集，从集合中随机获取元素
### hash --无序字典
> 包含键值对的无序散列
> 添加、获取、移除单个键值对，获取所有键值对

**命令**
> `HSET key field value [field value ...]` 存储、修改hash
> `HGET key field` 获取hash中的值
> `HMSET key field value [field value ...]` 与hset类似
> `HMGET key field [field ...]`获取多个hash字段
> `HGETALL key` 返回key下的所有键值
> `HKEYS key` hash中的所有键
> `HVALS key` hash中的所有值
> `HINCRBY key field increment` 自增hash中指定的字段值
> `HSETNX key field value` hash中的字段不存在时添加
### zset
> 字符串成员与浮点数分值之间的有序映射，元素的排序顺序由分值大小决定
> 添加、获取、删除耽搁元素，根据分值范围或成员来获取元素

## 其他三种数据类型
### geo 地理作保

### bitmap （位图）

### hyperLog

## 通用命令
> `help keys` -- 查看keys名
> `help @Generic` 查看通用命令
> `keys pattern`  查看key列表
> `del key` 删除键值对,返回删除的键值对数量
> `exists key` 判断一个key是否存在
> `EXPIRE key seconds` 设置key的存活时间
> `TTL key` 查看key的有效期

## redis key的设置规则，以冒号隔开，形成层级结构
项目名：业务名：业务类型：id


## redis集群安装
三种模式
> 主从模式
> 哨兵模式
> 集群模式

单节点安装
```shell
yum -y install gcc
make distclean
make && make install PREFIX=/export/server/redis
```

**主从模式**
启动主从复制模式后，从服务器只提供给读的功能，不提供写共功能
主服务提供读写功能
修改配置从服务器配置文件
```
sloveof 192.168.10.102 6379 
masterauth steven  #主服务器密码
requeirepass steven #主服务器密码
```
主服务宕机后从服务器不会提升成为主服务器，导致整个系统数据不可写入


**哨兵模式**
哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。
哨兵的作用
* 通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
* 当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。


> redis.conf 从服务器配置
``` 
sloveof 192.168.10.102 6379 
masterauth steven
requeirepass steven
```
> sentinel.conf 配置 
```
sentinel monitor mymaster 192.168.10.102 6379 2
sentinel auth-pass mymaster steven
```
> 先启动 redis-server，再启动redis-sentinel
```
./bin/redis-server redis.conf
./bin/redis-sentinel sentinel.conf
```
启动后可以通过redis log查看数据同步信息
也可以通过主从服务上运行下面的命令查看主从信息
```
info replication
```
关闭主服务器进程，这是会重新选举新的主服务。
如果主服务器重新上线，此时并不会重新进行主服务器的选举。
此时观察redis.conf文件，发现文件内的slaveof 主从配置被修改掉（主服务器被配置了一个slaveof属性）

**集群模式**

依据 Redis Cluster 内部故障转移实现原理，Redis 集群至少需要 3 个主节点，而每个主节点至少有 1 从节点，因此搭建一个集群至少包含 6 个节点，三主三从，并且分别部署在不同机器上。
这里采用在三台centos7虚拟机上使用不同的端口号进行部署
每台机器部署两个redis进程，

参考地址：https://zhuanlan.zhihu.com/p/320510950




