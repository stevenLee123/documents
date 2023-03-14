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
发布订阅模式：每个消息都有一个
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
* topic： 主题是一个逻辑的概念，用于生产者发布数据，消费者拉取数据，一个主题中的消息是有结构的，一般一个主题包含一类消息，一旦一个消息发送到主题中，这些消息不能被更新
* offset：偏移量，

## 如何保证消息不丢失

## 如何保证消息的顺序

## kafka stream