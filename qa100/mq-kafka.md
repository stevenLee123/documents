# Kafka
* 消息队列
* 临时存储
* 分布式流平台（发布订阅流数据流、以容错的持久化方式存储数据流、处理流数据（kafka stream））
* 低延迟

## 消息队列的应用场景
* 异步处理，将耗时的操作放入到其他系统中，通过消息队列将需要进行处理的消息进行存储，（如短信验证码的发送）实现快速响应
* 系统解耦，避免两个系统之间的直接调用（迪米特法则），一个微服务将消息放入到消息队列中，另一个微服务可以从队列中把消息取出来进行处理，实现解除耦合
* 流量削峰，避免过高的并发导致的系统瘫痪
* 日志处理（大数据领域常见，将消息队列作为临时存储或通信管道）

## 生产者消费者模型
producer负责将消息放入队列
consumer负责将消息取出进行处理

## 两种模式
点对点模式：
* 每个消息只有一个接收者，一旦被消费，消息就会从消息队列中移除
* 生产者消费者没有依赖性
* 接收着在成功接收消息之后需向队列应答成功，以便队列删除当前消息
发布订阅模式：
* 一条消息可以被对多个订阅者接受
* 发布者和订阅者之间有时间上的依赖性，针对某个主题的订阅者，它必须创建一个订阅者之后，才能消费发布订阅者
* 为了消费消息，订阅者需要提前订阅该角色主题，保持在线运行


## kafka基准测试工具
kafka内部提供了性能测试工具
生产者：测试生产者每秒传输的数据量
消费者：测试消费每秒拉取的数据量

## javaAPI
生产者
```java
//同步模式
 Properties prop = new Properties();
        prop.put("bootstrap.servers","localhost:9092");
        //应答种类
        prop.put("acks","all");
        prop.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        prop.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String,String> kafkaProducer = new KafkaProducer<String, String>(prop);
        for (int i = 0; i < 10; i++) {
             Future<RecordMetadata> future = kafkaProducer.send(new ProducerRecord<>("quickstart-events", null, i + ""));
            future.get();
            log.info("第"+i+"条消息");
        }
```
```java
//异步回调
 public static void main(String[] args) throws ExecutionException, InterruptedException {
        Properties prop = new Properties();
        prop.put("bootstrap.servers","localhost:9092");
        //应答种类
        prop.put("acks","all");
        prop.put("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
        prop.put("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
        KafkaProducer<String,String> kafkaProducer = new KafkaProducer<String, String>(prop);
        for (int i = 0; i < 10; i++) {
            Future<RecordMetadata> future = kafkaProducer.send(new ProducerRecord<>("quickstart-events", null, i + ""),((recordMetadata, e) -> {
                    //判断发送是否成功
                if(e == null){
                    String topic = recordMetadata.topic();
                    int partition = recordMetadata.partition();
                    long offset = recordMetadata.offset();
                    log.info("topic：{},partition:{},offset:{}",topic,partition,offset);
                }else{
                    e.printStackTrace();
                }
            }));
            future.get();
            log.info("第"+i+"条消息");
        }
    }
```

```java
//消费者开发
 public static void main(String[] args) {
        Properties prop = new Properties();
        prop.put("bootstrap.servers","localhost:9092");
        //设置消费者组，组名一样的消费者消费的消息是一样的
        prop.put("group.id","quickstart-events-group");
        //自动提交offeset
        prop.put("enable.auto.commit","true");
        //自动提交时间间隔
        prop.put("auto.commit.interval.ms","1000");
        prop.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        prop.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(prop);
        //订阅主题
        consumer.subscribe(Arrays.asList("quickstart-events"));
        while(true){
            //一次拉取一批数据
            ConsumerRecords<String, String> consumerRecords = consumer.poll(Duration.ofSeconds(5));
            for (ConsumerRecord<String, String> consumerRecord : consumerRecords) {
                String topic = consumerRecord.topic();
                long offset = consumerRecord.offset();
                String key = consumerRecord.key();
                String value = consumerRecord.value();
                log.info("topic:{},offset:{},key:{},value:{}",topic,offset,key,value);
            }
        }
    }
```
消费者组的概念：
一个消费者组中包含多个消费者，一个组中的消费者共同消费kafka中的topic的数据，当组中的消费者挂掉后，kafka会记住消费者消费消息的offset，当消费者再次连接会从offset处再次消费

## kafka概念
* zookeeper集群：保存kafka相关元数据，管理协调kafka集群
* broker：由多个broker组成，无状态，通过zk来维护集群状态
* producer：
* consumer：
* consumer group: 可扩展且具有容错性的消费者机制，一个消费者组具有唯一的id group.id
* partition： 一个topic对应多个分区，将分区分布在不同的服务器上
* replicas：分区的副本，分副本也是放在不同的服务器上，用来容错
* topic： 主题是一个逻辑概念，用于生产者发布数据，消费者拉取数据，一个主题中的消息是有结构的，一般一个主题包含一类消息，一旦一个消息发送到主题中，这些消息不能被更新
* offset：偏移量，记录吓一跳将要发送给comsumer的消息的序号，默认kafka将offset存储在zk中，在一个分区中，消息是有顺序存储着，每个分区的消费都是有一个递增的id，这就是偏移量offset，偏移量在分区中有意义，在分区之间没有意义

## 消费者组
* 一个消费者组中能包含多个消费者，共同消费topic中的数据
* 一个topic中如果只有一个分区，那么这个分区只能被消费者组中的一个消费者消费
* 有多少个分区，嘛么久可以被同一个组内的多少个消费者消费

## kafka幂等性（解决生产者消息重复性问题）
> http请求中，一次请求或多次请求拿到的响应是一致的
> 执行多次操作与执行一次操作的影响时一样的
> 如果kafka生产者在提交数据到broker后数据写入分区中，而broker响应给producer的ack应答失败，这时，producer会再次尝试发送相同的消息到broker，直到收到正常的ACK应答，而broker能保证producer retry的多条数据只有一条写入分区中

开启kafka的幂等性： 发送消息时，会连着pid(生产者唯一编号)和squence number一起发送，kafka接受到消息，会将消息和pid、sequence number一起保存下来，当生产者发送过来的sequence number小于等于partition消息中的sequence number，kafka会忽略到这条消息
```java
 prop.put("enable.idempotence",true);
```
## 分区与副本机制
### 生产者分区写入策略
1. 轮询分区策略（默认策略）
    * 最大限度保证消息平均分配到一个分区
    * 如果在生产key为null的数据时，使用轮询算法均衡的分配分区
    * 使用轮询算法没有办法保证消息的有序性（只能保证在分区中的局部有序）

2. 随机分区策略（目前没有使用）

3. 按key分区分配策略
    * 当key不为null时，按key的hash来进行分区，可能出现数据倾斜，某个key中可能包含大量的数据。
    * 按key存储可以实现一定程度上的有序存储（局部有效存储），实际生产环境中需要结合实际情况来做取舍

4. 自定义分区策略
    * 实现partitoner接口自定义分区

## consumer group rebalance机制（再平衡机制）
> 确保consumer group 下所有的consumer达成一致，分配订阅的topic的每个分区的机制
rebalance 触发时机：
* consumer group中的消费者数量发生变化
* 订阅主题数据量发生变化
* 订阅分区发生变化

> rebalance的不良影响：
* 发生时，消费者组中的所有消费者都要协同参与，使用分配策略尽可能达到最公平的分配
* 发生时，所有的消费者都将停止工作，直到rebalance完成

## 消费者的分区分配策略
* range范围分配策略（默认分配策略），确保每个消费者消费的分区数量是均衡的。range范围分配是针对每个topic的。
公式：
n = 分区数量/消费者数量
m = 分区数量%消费者数量
前m个消费者分别消费n+1个分区，
剩余的消费者分别消费n个
* RoundRobin轮训策略
    将消费组内所有的消费者以及消费者所订阅的所有topic的partition按照字典顺序排序（topic和分区的hashcode进行排序）
    通过轮询方式逐个将分区以这种分配分给每个消费者
* stricky黏性分配
    分区分配尽可能均匀
    在发生rebalance时，走一遍轮询策略，分区的分配尽可能与上一次保持相同，仅将出现变化的分区进行重新分配
    没有发生rebalance时，与轮询分配策略保持一致

## 副本机制
冗余副本，当某个broker上分区丢失时，依然可以保证数据可用性，其在其他broker上的副本是可用的

### producer的ACKs参数
```java
 prop.put("acks","all");
```
ACKs对副本的影响较大
> -1/all 所有的副本都写入成功才算写入成功，性能最差
> 0 不等待broker确认，直接发送下一条数据，性能最高，可能存在数据丢失
> 1 等待leader副本确认接受后，才会发送下一条数据，性能中等
根据具体的业务场景选择不同的应答机制

为确保小粉着消费的数据是一致的，只能从分区leader读写消息，follower分区只负责同步数据，做热备份

## 高级API（Higher API）和低级API（lower API）
> 以上用的代码都是高级api
 * 不需要去执行offset，直接通过zk管理，不需要管理分区，副本，由kafka统一管理
 * 消费者会自动根据上一次在zk中保存的offset去接着获取数据
 * 在zk中，不同的消费者组同一个topic记录着不同的offset，这样不同的程序读取同一个topic不回收到offset的影响
高级api不能控制offset，无法从指定位置读取
低级api会在各种框架中进行使用，有编写的程序自己控制逻辑，自己管理offset，将offset存储在zk、mysql、redis、hbase、flink的状态存储

手动指定分区消费，无法进行rebalance
```java
 KafkaConsumer<String, String> consumer = new KafkaConsumer<>(prop);
        TopicPartition partition0 = new TopicPartition("quickstart-events",0);
        TopicPartition partition1 = new TopicPartition("quickstart-events",0);
        consumer.assign(Arrays.asList(partition0,partition1));
```
## kafka eagle 监控工具

## 实现原理
### leader、follower
> leader follower是针对分区,不是针对broker
> 每个分区必须有一个leader，有0个或多个follower
> 创建topic是kafka会尽量均匀的将topic的分区分配到broker上，尽量让leader分配均匀
> 当leader发生故障时，其他follower会被重新选举为leader
> follower只会从leader同步数据，不对客户端提供读写功能
> 从配置文件指定的路径`log.dirs`能看到具体的kafka数据信息，可以看到索引、topic、分区等信息

### AR\ISR\OSR --follower的状态
leader出现故障后，通过投票进行选举，根据follower的状态进行选举
* AR （Assigned Replicas） 已分配副本，分区的所有副本称为AR
* ISR (in sync replicas) 所有与leader副本保持一定程度同步的副本，在同步中的副本
* OSR （out sync replicas） 由于follower副本同步滞后过多的副本组成OSR

正常情况下，所有副本都处于同步状态，即AR = ISR，OSR为空
可以使用kafka eagle 查看所有ISR副本

### Controller与leader选举
kafka如何确定某个partition为leader，哪个时follower？
> 在开始启动时时随机选择partition为leader
某个leader崩溃了，如何快速确定另一个leader？

**controller**
kafka在启动时会在所有的broker中选择controller,contoller是针对controller
创建topic、添加分区、修改副本数量都是由controller来实现的
controller选举
> 集群启动时，每个broker都会尝试去zk上注册成为controller
> 只要有一个注册成功，则其他的broker会注册该节点的监视器
> 一旦该节点发生变化，就可以进行相应的处理
> controller是高可用的，一旦broker崩溃，其他的broker就会重新注册为controller
> controller是通过zk来进行选举的
**controller选举leader**
> 所有的partition的leader选举都是由controller来决定的
> controller会将leader的改变直接通过rpc方式通知需为此做出相应的broker
> controller 读取到当前分区的ISR，只要有一个replica还幸存，就选择其中一个作为leader，否则任意选择一个replica作为leader
> 如果该partition的所有replica都已经宕机，则新的leader为-1（标识当前分区挂掉）

为什么不用zk的方式选取leader？
kafka集群如果业务很多的情况下，会有很多个partition
假设某个broker宕机，就会出现很多partition需要重新选举leader
如果使用zk选举leader，回给zk带来很大的压力，所以leader选举不能用zk来实现

### leader 的负载均衡 preferred replica
在ISR列表中第一个replica就是preferred replica
第一个分区存放的borker，就肯定时preferred replica
> 如果某个broker宕机滞后，就可能导致partition的leader分布不均匀，broker上存在一个topic下不同的partition的leader
> leader不是均匀分布在某台broker上（一个broker上有多个leader），则这台broker就不是preferred replica
通过执行`bin/kafka-leader-election.sh`命令对leader进行重新选举，确保leader是均匀分配的

### kafka的读写数据流程
**写入**
> 从zk上的‘/brokers/topics/主题名/partitions/分区名/state’节点找到partition的leader
> 生产者在zk中找到Id对应的broker
> broker进程上的leader将消息写入本地的log中（顺序写，速度快）
> follower从leader上拉取消息，写入本地log，并leader发送ACK
> leader接受到所有的ISR中的replica的ACK后，并向生产者返回ACK

**读取（消费）**
> kafka采取的是拉模式
> 由消费者自己记录消费状态，每个消费者互相独立的顺序拉取每个分区的消息
> 消费者可以按照人一的顺序消费消息，比如可以重置偏移量
> 每个consumer都可以根据分配策略（默认range），获得要消费的分区，通过zk找到分区对应的leader（leader负责读）
> 获取到consumer对应的offset，默认从zk中获取上一次消费的offset
> 找到改分区的leader，拉取数据
> 消费者提交offset

### kafka的数据存储
> 一个topic有多个分区组成
> 一个分区由多个segment（段）组层（默认lG进行滚动）
> 一个segment（段由多个文件组成（log、index、timeindex））

**存储日志**
写日志
> .log 日志数据文件
> .index 索引文件（使用稀疏索引，避免索引数据量过大）
> .timindex  时间索引文件
> leader-epoch-chekpoint 持久化每个partition leader对应的leo(log end offset)，leo日志文件中下一条待写入消息的offset
> 文件名是起始偏移量



读取数据
> 消费者使用offset（针对partition全局的offset）找到对应的segment
> 将offfset转化为segment段文件的offset
> 在根据segment段文件的局部offset查找segment段中的数据
> 为了提高查询效率，每个文件都会维护好对应的范围内存，找到的时候使用简单的二分法查找

删除消息
在kafka中，消息是会被定期清理，一次删除一个segment段的日志文件
kafka的日志管理器会根据kafka的配置，来决定哪些文件可以被删除

## 如何保证消息不丢失
> broker数据不丢失
生产者通过分区的leader写入数据后，所有在ISR中follower都糊从leader复制数据，这样可以确保即使leader崩溃了，其他的follower数据仍然不会丢失
> 生产者数据不丢失
生产者连接leader写入数据时，可以通过ACK机制来确保数据已经成功写入，ACK机制有三个选项可以配置
* -1/all 所有follower节点都要收到数据才发送ack
* 1 leader收到数据后响应
* 0 生产者只负责发送数据，不关心数据是否丢失
> 生产者采用同步和异步两种方式发送数据
* 同步：发送一批数据给kafka，等待kafka返回结果
* 异步：发送一批数据给kafka，只提供一个回调
如果broker不给ack，而buffer又满了，开发者可以设置是否直接清空buffer中的数据
> 消费者数据不丢失
* 消费者在消费数据是，只要每个消费者记录好offset值即可，就能保证数据不丢失
* 消费者在拿到消息，要保证消息的正常处理完成之后，才将offset写回zk


> 消息重复消费
* 消费者从zk拿到offset从分区leader中消费消息
* 消费者处理完消息再将offset存储到zk
* offset写回zk的过程中失败导致offset没有更新，可能出现重复消费的情况


## 消息传递语义
at most once 最多一次消费
at least once 最少一次消费（可能出现重复消费）
exactly once 仅有一次消费 （事务性保证消息仅被处理一次） 通过lowerlevel API自己维护offset，将offset写入mysql（使用mysql事务保证整个操作的原子性，不能使用kafka的事务保证），不使用zk维护 


## 数据积压
kafka消费者数据时由于外部IO、或者产生网络拥堵，就会造成kafka中的数据积压，如果数据一致积压，会导致数据的实时性受到影响
解决方案：
正常情况下kafka的消费者消费数据的速度是很快的，产生数据积压往往是因为消费者下游程序处理逻辑太慢导致的数据积压（不考虑网络的问题），处理好下游程序的逻辑提升效率就能解决数据积压的问题

## 日志清理
kafka消息存储在磁盘中，为了控制磁盘空间，Kafka每个分区都有很多日志文件，方便了清理
日志删除： 按照自定的策略直接删除不符合条件的日志
日志压缩：按照消息的key进行整合，相同的key具有不同的value值
设置保留策略
> 基于时间的保留策略(默认七天)
* log.retention.hours
* log.retention.minutes
* log.retention.ms

> 设置topic多少秒删除一次
* retention.ms 


## 如何保证消息的顺序

## kafka stream