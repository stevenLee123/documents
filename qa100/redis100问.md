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
> 链表--双向链表,支持正向检索和反向检索
> 有序
> 允许元素重复
> 插入删除快
> 查询速度一般
> 从链的两端推入或弹出（RPUSH、LRANGE、LINDEX、LPOP），根据偏移量进行修剪（trim）
> 读取单个或多个元素，根据值查找或移除元素

**命令**
> `LPUSH key element [element ...]` 从左侧插入
> `LPOP key [count]` 从左边取元素，移除元素并返回移除的元素
> `RPUSH key element [element ...]` 从右侧插入元素
> `RPOP key [count]` 从右边取元素，移除元素并返回移除的元素，没有则返回nil
> `LRANGE key start stop` 返回一段角标范围内的所有元素,`lrange dxy:user:test 0 -1` 取所有元素,这个命令不会移除key中的数据
> `BLPOP key [key ...] timeout` 阻塞一定时间取数据,没有元素时等待指定时间，超时后返回nil
> `RLPOP key [key ...] timeout` 阻塞一定时间取数据,没有元素时等待指定时间，超时后返回nil,timeout 是second

使用list的命令可以模拟栈（先进后出（lpop/lpush）），队列（先进先出（lpush/rpop）），阻塞队列（使用BLPOP/BRPOP）

### set
> 包含无重复字符串的无需收集器
> 与java的HashSet蕾丝
> 无序
> 元素不可重复
> 查找快
> 支持交集、并集、差集等功能（实现好友列表，共同好友等功能）
> 添加、获取、移除单个元素，检查元素是否在集合中（SADD、SMEMBERS、SREM、SISMEMBER），计算交、并、差集，从集合中随机获取元素

**命令**
> `SADD key member [member ...]` 数据插入
> `SREM key member [member ...]` 数据移除
> `SCARD key` 返回集合中的元素个数
> `SISMEMBER key member` 如果是集合中的元素，返回1，否则返回0
> `SMEMBERS key` 返回集合中的所有元素 (无序返回)
> `SINTER key [key ...]` 多个集合的交集
> `SDIFF key [key ...]` 差集，第一个key中有而其他key中没有的元素
> `SUNION key [key ...]`  求多个元素的并集

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
> 与java中的TreeSet(实现方式不同，TreeSet用红黑树实现)类似
> 通过score进行排序
> 底层实现的事一个跳表加hash
> 能方便的实现数据统计功能
> 字符串成员与浮点数分值之间的有序映射，元素的排序顺序由分值大小决定
> 添加、获取、删除耽搁元素，根据分值范围或成员来获取元素

**命令**
> `ZADD key [NX|XX] [GT|LT] [CH] [INCR] score member [score member ...]` 插入数据到集合中，并设置排序score 
> `ZREM key member [member ...]` 移除元素
> `ZSCORE key member` 返回集合中指定元素的分值
> `ZRANK key member`  返回集合中
> `ZRANK key member` 返回集合中指定元素的排名(index)
> `ZCARD key` 返回集合中元素个数
> `ZINCRBY key increment member` 让集合中指定元素分值自增，返回自增后的分值 zincrby dxy:user:zset1 2 wangwu
> `ZCOUNT key min max` 返回集合中指定分值范围内的元素个数 zcount dxy:user:zset1 70 100
> `ZRANGE key min max [BYSCORE|BYLEX] [REV] [LIMIT offset count] [WITHSCORES]` 按分值返回index在min和max之间的元素 zrange dxy:user:zset1 0 1   --从分值最小端返回最小的两个元素
> `ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]` 按score排序后获取score范围内的元素,zrangebyscore dxy:user:zset1 70 80 --返回分值在70-80之间的元素
> `ZDIFF numkeys key [key ...] [WITHSCORES]`
> `ZUNION numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX] [WITHSCORES]` 查询集合的交集，并将集合中的相同元素分值进行相加后排序：`zunion 2 dxy:user:zset1 dxy:user:zset2 withscores`
> `ZDIFF numkeys key [key ...] [WITHSCORES]` 在第一个key中不在其他key中的元素
> `ZINTER numkeys key [key ...] [WEIGHTS weight] [AGGREGATE SUM|MIN|MAX] [WITHSCORES` 返回集合中的交集，并将相同的元素分值相加
*排序命令如果需要反转排序则需要在命令前面加上rev ，例如`zrevrank dxy:user:zset1 zhaoliu`*

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


## java客户端

### jedis java redis 
> 以命令作为方法名称
> jedis实例线程不安全，在多线程环境下必须使用线程池进行管理保证线程安全

示例
```java
    private static volatile Jedis jedis;

    public static void main(String[] args) {
        String result = getJedis().set("test-java","java hello redis");
    }

    public static Jedis getJedis(){
        if(jedis == null){
            synchronized (Jedis.class){
                if(jedis == null){
                    jedis = new Jedis("localhost",6379);
                    jedis.auth("dxy123456");
                }
            }
        }
        return jedis;
    }
```

```java
//使用连接池
    private static final JedisPool jedisPool;
    static {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxIdle(8);
        jedisPoolConfig.setMaxTotal(8);
        jedisPoolConfig.setMinIdle(1);
        //获取时最多等待1000ms
        jedisPoolConfig.setMaxWaitMillis(1000);
        jedisPool  = new JedisPool(jedisPoolConfig,"localhost",6379,1000,"dxy123456");
    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
```

### letuce
> 基于netty实现，支持同步、异步和响应式编程，支持redis的哨兵模式，集群模式和管道模式

### redisson
基于redis实现的分布式的可伸缩的java数据结构集合，包含了如map、queue、lock、semaphore、atomicLong等强大的原子类和锁

### 使用springDataRedis操作
使用RedistTemplate简化输入对象的序列化操作
配置（注意springboot默认使用的是lettuce，）
```yml
spring:
  redis:
    host: localhost
    port: 6379
    database: 0
    lettuce:
      pool:
        enabled: true
        max-idle: 8
        min-idle: 1
        max-wait: 1000ms
        max-active: 8
    password: dxy123456
```
使用
```java
   public static void main(String[] args) {
        ConfigurableApplicationContext applicationContext = SpringApplication.run(Application.class, args);
        RedisTemplate redisTemplate = (RedisTemplate) applicationContext.getBean("stringRedisTemplate");
        redisTemplate.opsForValue().set("spring-redis","spring    hello world");
        log.info(redisTemplate.opsForValue().get("spring-redis").toString());
    }
```
RedisTemplate 接收的参数是对象，而不是字符串，默认使用的是jdk的序列化器，默认使用的ObjectOuptutStream，会导致存入redis中的键值对不是预期的键值对

```java
//自定义序列化和反序列化的工具
 @Bean
    public  RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory factory){
        //设置redisTempalate对象
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //设置连接工厂
        redisTemplate.setConnectionFactory(factory);
        //设置json序列化工具
        GenericJackson2JsonRedisSerializer jsonRedisSerializer = new GenericJackson2JsonRedisSerializer();
        //设置key序列化
        redisTemplate.setKeySerializer(RedisSerializer.string());
        redisTemplate.setHashKeySerializer(RedisSerializer.string());
        //设置value序列化
        redisTemplate.setValueSerializer(jsonRedisSerializer);
        redisTemplate.setHashValueSerializer(jsonRedisSerializer);

        return redisTemplate;
    }

    RedisTemplate redisTemplate = (RedisTemplate) applicationContext.getBean("redisTemplate");
     User user = new User(1,"steven");
          User user2  = new User(2,"lld");
          redisTemplate.opsForValue().set("dxy:user:1",user);
         User o = (User) redisTemplate.opsForValue().get("dxy:user:1");
        System.out.println(o);
        redisTemplate.opsForHash().put("dxy:user",user.getId().toString(),user);
          redisTemplate.opsForHash().put("dxy:user",user2.getId().toString(),user2);
          User user3 = (User) redisTemplate.opsForHash().get("dxy:user","2");
          log.info(user3.toString());

          //存入的数据，存入了字节码信息，在反序列化的时候拿到类的信息
          {
            "@class": "com.steven.pojo.User",
            "id": 1,
            "username": "steven"
            }
```
*在开发中为了节省redis的内存空间，统一使用String序列化器的来存储redis的数据*
```java
    User user = new User(1,"steven");   
    StringRedisTemplate redisTemplate1 = applicationContext.getBean(StringRedisTemplate.class);
    redisTemplate1.opsForValue().set("spring-redis-3", new ObjectMapper().writeValueAsString(user));
    String jsonuser = redisTemplate1.opsForValue().get("spring-redis-3");
    User user4 = new ObjectMapper().readValue(jsonuser,User.class);
    log.info("user4:{}",user4);
     //存入的数据
     {
    "id": 1,
    "username": "steven"
    }     
```



## 分布式缓存
### 单节点redis存在的问题
* 数据丢失问题      --利用redis持久化
* 并发压力问题      --利用主从集群，实现读写分离
* 单点故障的问题    -- 利用哨兵机制实现故障恢复，解决单点故障
* 存储能力上限问题   --利用插槽 slot实现动态扩容

### redis的持久化
**RDB redis database backup file**
使用数据快照方式存储在磁盘文件
使用save命令执行备份操作  -- save命令会阻塞所有的命令，不推荐，在redis停机时使用
使用bgsave ，开启额外的进行执行rdb，避免主进程受到影响
