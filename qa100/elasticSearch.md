# ES
* ElasticSearch 是一个基于lucene搜索服务器
* lucene是一套基于java的搜索api
* es是对lucene封装实现
* solr是与es都是lucene的实现
* 基于restful的web接口，通过http可实现es的操作
* 与关系型数据库查询的对比：
> es 通过分词提高了查询的效率，关系型数据库的查询性能比es差很多
> mysql 查询功能弱，模糊查询会导致全表扫描
> es不具有事务性，没有外键，不能向mysql一样保证数据的安全性
> 


* 倒排索引：将一段文本按照一定的规则拆分成不同的词条（term），记录词条和文本之间的关系
key（term） value
床          床前明月光
床前        床前明月光
光          床前明月光，
月          床前明月光 /明月几时有 ---><静夜思>/<水调歌头>  将文本文唯一标识放入
....
存储value时，将文本内容的唯一标识放在value中

使用场景：
> 适合海量数据的查询
> 日志数据的分析
> 数据实时分析


## 概念
* index 索引
es存储数据的地方
一个索引类似于mysql的库
生成的倒排索引中，词条会排序，形成一颗树形结构，提升词条查询速度

* mapping 映射
定义字段的类型，字段所使用的分词器等
规定数据的格式，类似于mysql的一张表

* document 文档
es中最小的数据单元，常以json格式展示，一个document相当于关系型数据库中的一行数据

* 倒排索引
一个倒排索引由文档中所有不重复的列表构成，对与其中的每个词，对应都包含她的文档id列表

* 类型 type
一种类型type就像一类表，入用户表，角色表
es7后逐渐淘汰了type

## elasticSearch操作
### 脚本操作es（restful）

**索引操作**
添加
``` curl --location --request PUT 'http://127.0.0.1:9200/goods_index' ```
```
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "index": "goods_index"
}
```
查找（多个用,分割）
``` curl --location --request GET http://127.0.0.1:9200/goods_index```
```
{
    "goods_index": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "goods_index",
                "creation_date": "1679579768250",
                "number_of_replicas": "1",
                "uuid": "e3Hdrn7dTPC3D-Rr0tUvNA",
                "version": {
                    "created": "8060299"
                }
            }
        }
    }
}
```
查询全部
``` curl --location --request GET http://127.0.0.1:9200/_all```
```
{
    "goods_index": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "goods_index",
                "creation_date": "1679579768250",
                "number_of_replicas": "1",
                "uuid": "e3Hdrn7dTPC3D-Rr0tUvNA",
                "version": {
                    "created": "8060299"
                }
            }
        }
    },
    "goods_index2": {
        "aliases": {},
        "mappings": {},
        "settings": {
            "index": {
                "routing": {
                    "allocation": {
                        "include": {
                            "_tier_preference": "data_content"
                        }
                    }
                },
                "number_of_shards": "1",
                "provided_name": "goods_index2",
                "creation_date": "1679579932715",
                "number_of_replicas": "1",
                "uuid": "rZXjBy2bRmaCkk2xlf77sA",
                "version": {
                    "created": "8060299"
                }
            }
        }
    }
}
```
删除
``` curl --location --request DELETE http://127.0.0.1:9200/goods_index2```
```
{
    "acknowledged": true
}
```
关闭(关闭后客户端可查询但不可添加数据)
``` curl --location --request POST  http://127.0.0.1:9200/goods_index2/_close```
```
{
    "acknowledged": true,
    "shards_acknowledged": true,
    "indices": {
        "goods_index2": {
            "closed": true
        }
    }
}
```
打开
``` curl --location --request POST http://127.0.0.1:9200/goods_index2/_open```
{
    "acknowledged": true,
    "shards_acknowledged": true
}

**mapping 操作**
#### 数据类型
**简单数据类型**
* 字符串 
    * text 会分词，不支持聚合
    * keyword 不会分词，将全部内容作为词条支持聚合
* 数值
    * long
    * integer
    * short
    * byte 
    * double
    * float
    * half_float
    * scaled_float
* 布尔 boolean
* 二进制 binary
* 范围类型
  * integer_range
  * float_range
  * long_range
  * double_range

* 日期 date

**复杂数据类型**
* 数组 []
* 对象 {}
*使用kibana操作*
添加映射
```
PUT person/_mapping
{
  "properties":{
    "name":{
      "type":"keyword"
    },
    "age":{
      "type":"integer"
    }
  }
}
--结果
{
  "acknowledged" : true
}
```

创建索引时添加映射
```
PUT person
{
  "mappings": {
     "properties":{
    "name":{
      "type":"keyword"
    },
    "age":{
      "type":"integer"
    }
  }
  }
}

--结果
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "person"
}
```

查询映射
```
GET person/_mapping

--结果
{
  "person" : {
    "mappings" : {
      "properties" : {
        "age" : {
          "type" : "integer"
        },
        "name" : {
          "type" : "keyword"
        }
      }
    }
  }
}
```

添加字段
```
PUT person/_mapping
{
   "properties":{
  "address":{
    "type":"text"
  }
   }
}

--结果
{
  "acknowledged" : true
}
```

查询
```
GET person
--结果
{
  "person" : {
    "aliases" : { },
    "mappings" : {
      "properties" : {
        "address" : {
          "type" : "text"
        },
        "age" : {
          "type" : "integer"
        },
        "name" : {
          "type" : "keyword"
        }
      }
    },
    "settings" : {
      "index" : {
        "routing" : {
          "allocation" : {
            "include" : {
              "_tier_preference" : "data_content"
            }
          }
        },
        "number_of_shards" : "1",
        "provided_name" : "person",
        "creation_date" : "1679581156802",
        "number_of_replicas" : "1",
        "uuid" : "qp_1no-ESMSFGpBvWi_iuw",
        "version" : {
          "created" : "8060299"
        }
      }
    }
  }
}
```
**操作文档**
添加文档
指定id方式
```
PUT/POST person/_doc/1
{
  "name":"zhangsan",
  "age":20,
  "address":"beijing"
}
--结果
{
  "_index" : "person",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```
不指定id 必须使用post
```
POST person/_doc
{
  "name":"lisi",
  "age":30,
  "address":"shanghai"
}

--结果，生成随机id
{
  "_index" : "person",
  "_id" : "rlrcDocB0iSM-C068bkJ",
  "_version" : 1,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "lisi",
    "age" : 30,
    "address" : "shanghai"
  }
}
```
查询文档
根据id查询
GET person/_doc/1
--结果
{
  "_index" : "person",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 0,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "zhangsan",
    "age" : 20,
    "address" : "beijing"
  }
}

查询所有文档
GET person/_search
--结果
{
  "took" : 197,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "person",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "name" : "zhangsan",
          "age" : 20,
          "address" : "beijing"
        }
      },
      {
        "_index" : "person",
        "_id" : "rlrcDocB0iSM-C068bkJ",
        "_score" : 1.0,
        "_source" : {
          "name" : "lisi",
          "age" : 30,
          "address" : "shanghai"
        }
      },
      {
        "_index" : "person",
        "_id" : "2",
        "_score" : 1.0,
        "_source" : {
          "name" : "wangwu",
          "age" : 20,
          "address" : "beijing"
        }
      }
    ]
  }
}


修改文档
PUT person/_doc/2
{
  "name":"wangwu",
  "age":20,
  "address":"beijing"
}
--结果
{
  "_index" : "person",
  "_id" : "2",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}

删除文档
DELETE person/_doc/rlrcDocB0iSM-C068bkJ
--结果
{
  "_index" : "person",
  "_id" : "rlrcDocB0iSM-C068bkJ",
  "_version" : 2,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}

## 分词器analyzer 
将一段文本，按照一定的逻辑，分析称多个词语的一种工具
standardAnalyzer 默认分词器，按词切分
Simple Analyzer
Stop Analyzer
language 提供30多种常见语言的分词器
····
 *es内置分词器对中文不够友好，按一个字一个字进行分词*

 GET _analyze
{
  "analyzer":"standard",
  "text":"我爱中国"
}

--结果
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<IDEOGRAPHIC>",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "<IDEOGRAPHIC>",
      "position" : 1
    },
    {
      "token" : "中",
      "start_offset" : 2,
      "end_offset" : 3,
      "type" : "<IDEOGRAPHIC>",
      "position" : 2
    },
    {
      "token" : "国",
      "start_offset" : 3,
      "end_offset" : 4,
      "type" : "<IDEOGRAPHIC>",
      "position" : 3
    }
  ]
}

GET _analyze
{
  "analyzer":"standard",
  "text":"i love china"
}

--结果
{
  "tokens" : [
    {
      "token" : "i",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "love",
      "start_offset" : 2,
      "end_offset" : 6,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "china",
      "start_offset" : 7,
      "end_offset" : 12,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
### 使用IK分词器
安装中文分词器插件
ikanalyzer是基于java开发的轻量级中文分词工具包
基于maven构建
支持60万字/s的高速处理能力

安装： 针对es版本在github上下载对应的ik分词器

ik的两种模式：
ik_max_word: 做最细粒度的拆分
ik_smart: 智能模式
GET _analyze
{
  "analyzer":"ik_max_word",
  "text":"我爱中国北京"
}
--结果
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国北京",
      "start_offset" : 2,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 2
    },
    {
      "token" : "中国",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 3
    },
    {
      "token" : "北京",
      "start_offset" : 4,
      "end_offset" : 6,
      "type" : "CN_WORD",
      "position" : 4
    }
  ]
}

GET _analyze
{
  "analyzer":"ik_smart",
  "text":"我爱中国"
}

--结果
{
  "tokens" : [
    {
      "token" : "我",
      "start_offset" : 0,
      "end_offset" : 1,
      "type" : "CN_CHAR",
      "position" : 0
    },
    {
      "token" : "爱",
      "start_offset" : 1,
      "end_offset" : 2,
      "type" : "CN_CHAR",
      "position" : 1
    },
    {
      "token" : "中国",
      "start_offset" : 2,
      "end_offset" : 4,
      "type" : "CN_WORD",
      "position" : 2
    }
  ]
}

查询文档

词条查询： term，词条查询不会拆分查询条件，只有当词条和查询字符串完全匹配是才会匹配搜索
PUT person/_doc/5
{
  
  "name":"zhaosi",
  "age":20,
  "address":"中国北京天安门"
}
PUT person/_doc/6
{
  
  "name":"sunzi",
  "age":20,
  "address":"北京"
}

GET person/_search
{
  "query": {
    "term": {
      "address": {
        "value": "北京"
      }
    }
  }
}
-普配不出任何结果，这是因为创建的索引使用的是默认的分词器standard，会对汉字一个字一个字进行分词
{
  "took" : 0,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 0,
      "relation" : "eq"
    },
    "max_score" : null,
    "hits" : [ ]
  }
}

--创建索引指定mapping和分词器
PUT person
{
  "mappings": 
  {
  "properties":{
    "name":{
      "type":"keyword"
    },
    "age":{
      "type":"integer"
    },
    "address":{
      "type":"text",
      # 为字段指定分词器，使用细粒度的ik分词
      "analyzer": "ik_max_word"
    }
  }
}
}

插入数据后，再次查询：
GET person/_search
{
  "query": {
    "term": {
      "address": {
        "value": "北京"
      }
    }
  }
}

--结果
{
  "took" : 37,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 1.1005893,
    "hits" : [
      {
        "_index" : "person",
        "_id" : "6",
        "_score" : 1.1005893,
        "_source" : {
          "name" : "sunzi",
          "age" : 20,
          "address" : "北京"
        }
      },
      {
        "_index" : "person",
        "_id" : "5",
        "_score" : 0.4815079,
        "_source" : {
          "name" : "zhaosi",
          "age" : 20,
          "address" : "中国北京天安门"
        }
      }
    ]
  }
}

全文查询： match，全文查询会分析查询条件，先将条件进行分词，然后查询，
先对查询条件字符串分词，然后再查询求交集

GET person/_search
{
  "query": {
    "match": {
        #注意这里没有value，使用value时报错
      "address": "北京海淀"
      
    }
  }
}
--结果
{
  "took" : 4,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 2,
      "relation" : "eq"
    },
    "max_score" : 0.93125117,
    "hits" : [
      {
        "_index" : "person",
        "_id" : "6",
        "_score" : 0.93125117,
        "_source" : {
          "name" : "sunzi",
          "age" : 20,
          "address" : "北京"
        }
      },
      {
        "_index" : "person",
        "_id" : "5",
        "_score" : 0.45862365,
        "_source" : {
          "name" : "zhaosi",
          "age" : 20,
          "address" : "中国北京天安门"
        }
      }
    ]
  }
}

## java api

### 整合springboot
```xml
  <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>

        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-client</artifactId>

        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
        </dependency>
```
```java
 @Configuration
@ConfigurationProperties(prefix = "elastic")
public class ElasticConfig {

    private String host;
    private Integer port;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public Integer getPort() {
        return port;
    }

    public void setPort(Integer port) {
        this.port = port;
    }

    @Bean
    public RestHighLevelClient client(){
        return
                new RestHighLevelClient(RestClient.builder(new HttpHost(host,port,"http")));

    }
}
```
```yml
elastic:
  host: 127.0.0.1
  port: 6379
  ```
### 简单操作
创建索引

```java
    @Test
    public void addindex() throws IOException {
        IndicesClient indices = client.indices();
        //add index
        CreateIndexRequest createIndex = new CreateIndexRequest("person2");
        // set mappings
        String mapping = "{\n" +
                "  \"properties\":{\n" +
                "    \"name\":{\n" +
                "      \"type\":\"keyword\"\n" +
                "    },\n" +
                "    \"age\":{\n" +
                "      \"type\":\"integer\"\n" +
                "    },\n" +
                "    \"address\":{\n" +
                "      \"type\":\"text\",\n" +
                "      \"analyzer\": \"ik_max_word\"\n" +
                "    }\n" +
                "  }\n" +
                "}";
        createIndex.mapping(mapping, XContentType.JSON);
        CreateIndexResponse createIndexResponse = indices.create(createIndex, RequestOptions.DEFAULT);

        System.out.println(createIndexResponse.isAcknowledged());
    }
```
查询索引
```java

    @Test
    public void queryIndex() throws IOException {
        IndicesClient indices = client.indices();
        GetIndexRequest getRequest = new GetIndexRequest("person2");
       GetIndexResponse getIndexResponse = indices.get(getRequest, RequestOptions.DEFAULT);
       //获取结果
        final Map<String, MappingMetadata> mappings = getIndexResponse.getMappings();
        mappings.forEach((key,value)->{
            log.info("key:{},value:{}",key,value.getSourceAsMap());
        });
    }
    //key:person2,value:{properties={address={analyzer=ik_max_word, type=text}, name={type=keyword}, age={type=integer}}}
```
删除索引
```java
    @Test
    public void deleteIndex() throws IOException {
        IndicesClient indices = client.indices();
        DeleteIndexRequest deleteIndexReuest = new DeleteIndexRequest("es-test");
        AcknowledgedResponse delete = indices.delete(deleteIndexReuest, RequestOptions.DEFAULT);
        log.info("result:{}",delete.isAcknowledged());
    }
```
判断索引是否存在
```java
  @Test
    public void existIndex() throws IOException {
        IndicesClient indices = client.indices();
        GetIndexRequest getIndexRequest = new GetIndexRequest("es-test");
        boolean exists = indices.exists(getIndexRequest, RequestOptions.DEFAULT);
        log.info("result:{}",exists);
    }
```
操作文档,注意一定要保证客户端api的版本和实际使用的es版本保持一致，不然可能解析response报错
```java
    @Test
    public void createDoc() throws IOException {
        //source指定数据,mapp数据
        Map dataMap =new HashMap<>();
        dataMap.put("address","北京昌平");
        dataMap.put("name","zhaosi");
        dataMap.put("age",25);
        IndexRequest indexRequest = new IndexRequest("person2").id("1").source(dataMap);
        IndexResponse response = client.index(indexRequest, RequestOptions.DEFAULT);
        log.info("result:id:{},seqno:{},status:{},shardId:{}",response.getId(),response.getSeqNo(),response.status(),response.getShardId());

        //使用pojo转json
         Person data = new Person(2L,"sunwukong",1000,"花果山水帘洞");
         //使用json串
        IndexRequest indexRequest = new IndexRequest("person2").id(data.getId().toString()).source(JSONObject.toJSONString(data),XContentType.JSON);
        IndexResponse response = client.index(indexRequest, RequestOptions.DEFAULT);
        log.info("result:id:{},seqno:{},status:{},shardId:{}",response.getId(),response.getSeqNo(),response.status(),response.getShardId());
    
    }
```
修改
```java
   @Test
    public void updateDoc() throws IOException {
        //修改，当id存在时是修改，当id不存在时是新增
        Person data = new Person(2L,"孙悟空",1000,"花果山水帘洞");
        IndexRequest indexRequest = new IndexRequest("person2").id(data.getId().toString()).source(JSONObject.toJSONString(data),XContentType.JSON);
        IndexResponse response = client.index(indexRequest, RequestOptions.DEFAULT);
        log.info("result:id:{},seqno:{},status:{},shardId:{}",response.getId(),response.getSeqNo(),response.status(),response.getShardId());
    }
```
查询
```java
    @Test
    public void getDoc() throws IOException {
        GetRequest getRequest = new GetRequest("person2","2");
        GetResponse response = client.get(getRequest, RequestOptions.DEFAULT);
        System.out.println(response.getSourceAsString());
    }
```
删除文档
```java
    @Test
    public void deleteDoc() throws IOException {
        DeleteRequest deleteRequest = new DeleteRequest("person2","2");
        DeleteResponse response = client.delete(deleteRequest, RequestOptions.DEFAULT);
        System.out.println(response.getId());
    }
```

### 高级操作
**批量操作**
bulk 将文档的增删改查的一系列操作通过一次请求完全做完，减少网络传输次数，bulk内部的各个操作相互之间不会干扰对方
```shell
    POST _bulk
    {"delete":{"_index":"person","_id":"VKUVD4cBMCkLBtUeHUOM"}}
    {"create":{"_index":"person","_id":"10"}}
    {"name":"孙悟空","age":10001,"address":"花果山水帘洞"}
    {"update":{"_index":"person","_id":"2"}}
    {"doc":{"name":"王武"}}

    #结果
    {
  "took" : 16,
  "errors" : false,
  "items" : [
    {
      "delete" : {
        "_index" : "person",
        "_id" : "VKUVD4cBMCkLBtUeHUOM",
        "_version" : 2,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 6,
        "_primary_term" : 1,
        "status" : 200
      }
    },
    {
      "create" : {
        "_index" : "person",
        "_id" : "10",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 7,
        "_primary_term" : 1,
        "status" : 201
      }
    },
    {
      "update" : {
        "_index" : "person",
        "_id" : "2",
        "_version" : 2,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 1,
          "failed" : 0
        },
        "_seq_no" : 8,
        "_primary_term" : 1,
        "status" : 200
      }
    }
  ]
}
```
java代码bulk
```java

    @Test
    public void bulkTest() throws IOException {
        BulkRequest bulkReuquest = new BulkRequest();
        //删除操作
        bulkReuquest.add(new DeleteRequest("person","1"));
        //添加
        Map dataMap =new HashMap<>();
        dataMap.put("address","中国北京昌平");
        dataMap.put("name","赵四");
        dataMap.put("age",25);
        bulkReuquest.add(new IndexRequest("person").id("20").source(dataMap));
        //修改
        Map dataMap2 =new HashMap<>();
        dataMap2.put("name","孙子");
        dataMap2.put("age",90);
        bulkReuquest.add(new UpdateRequest("person","6").doc(dataMap2));
        BulkResponse bulk = client.bulk(bulkReuquest, RequestOptions.DEFAULT);
        log.info("status:{}",bulk.status());
    }
```    

**导入数据**
将数据库中的数据导入es中
* 创建索引
* 将数据从mysql等存储源中读取出来
* 将数据写入es

```java
    //将数据写入es
    @Test
    public void importToEs() throws IOException {
        List<Word> list = wordService.list();

        BulkRequest bulkRequest = new BulkRequest();
        list.forEach(user ->{
            IndexRequest indexRequest  = new IndexRequest("word");
            indexRequest.id(user.getId().toString()).source(JSONObject.toJSONString(user), XContentType.JSON);
            bulkRequest.add(indexRequest);
        });
        BulkResponse bulk = client.bulk(bulkRequest, RequestOptions.DEFAULT);
        log.info("result:{}" ,bulk.status());
    }
```


**各种查询**

**match_all**
查询指定条数的数据，默认情况下es会自懂进行分页
```
# 从0开始查询300条数据，分页查询
get word/_search
{
  "query":{
    "match_all":{}
  },
  "from":0,
  "size":300
}

----结果

{
  "took" : 1,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 10000,
      "relation" : "gte"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "word",
        "_id" : "25",
        "_score" : 1.0,
        "_source" : {
          "createTime" : 1663945788000,
          "creator" : "yanghailong",
          "id" : 25,
          "name" : "实体词测试1",
          "reviewer" : "zhouyunjie",
          "sourceType" : 1,
          "status" : 1,
          "updateTime" : 1663954758000,
          "wordType" : 13
        }
      },
      {
        "_index" : "word",
        "_id" : "26",
        "_score" : 1.0,
        "_source" : {
          "createTime" : 1663947575000,
          "id" : 26,
          "name" : "类型测试2",
          "sourceType" : 2,
          "status" : 2,
          "updateTime" : 1663947575000,
          "wordType" : 13
        }
      },
      {
        "_index" : "word",
        "_id" : "27",
        "_score" : 1.0,
        "_source" : {
          "createTime" : 1663947575000,
          "id" : 27,
          "name" : "类型测试3",
          "reviewer" : "yinxing",
          "sourceType" : 2,
          "status" : 3,
          "updateTime" : 1663955823000,
          "wordType" : 13
        }
      }
    ]
  }
}

```
java api操作
```java
 @Test
    public void testMatchAll() throws IOException {
        SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //指定查询条件,match_all,不指定分页情况下默认查询10条
        sourceBuilder.query(QueryBuilders.matchAllQuery());
        //指定分页
        sourceBuilder.from(0).size(100);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
            log.info("获取到的word：{}",word);
        }
    }
```

**term 查询**
不会对查询词条进行分词
一般查询分类字段等枚举类型的数据
如果指定了ik_max_word分词，可能查询不到任何数据
```
get word/_search
{
  "query":{
    "term":{
      "modifier":{
        "value":"zhouyunjie"
      }
    }
  }
}
```
```java
//termquery
SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //指定查询条件,match_all,不指定分页情况下默认查询10条
        sourceBuilder.query(QueryBuilders.termQuery("wordType",16));
        //指定分页
        sourceBuilder.from(0).size(100);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
            log.info("获取到的word：{}",word);
        }
```
**match**
对查询条件进行分词查询（分词后的本值匹配）,默认取并集（or），可以指定交集（and）
```json
#并集
get word/_search
{
  "query":{
    "match":{
      "name":"鼾声"
    }
  }
}
#交集
get word/_search
{
  "query":{
    "match":{
      "name":{
        "query": "血压",
        "operator": "and"
      }
    }
  }
}
```

java api
```java
  @Test
    public void testMatch() throws IOException {
        SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //指定查询条件,match_all,不指定分页情况下默认查询10条
        MatchQueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("name", "血压");
        //指定交集
        matchQueryBuilder.operator(Operator.AND);
        sourceBuilder.query(matchQueryBuilder);
        //指定分页
        sourceBuilder.from(0).size(100);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
//            log.info("获取到的word：{}",word);
        }
    }
```
**模糊查询wildcard**
注意不要在查询字符串前面写通配符（*，?等），这样做会导致扫描全部的倒排索引
```
get word/_search
{
  "query":{
    "wildcard":{
      "name":{
        "value": "血*"
        
      }
    }
  }
}
```
**regexp 正则查询**
```
get word/_search
{
  "query":{
    "regexp":{
      "name":"血*"
    }
  }
}
```
**前缀查询**
```
get word/_search
{
  "query":{
    "prefix":{
      "name":{
        "value":"血"
      }
    }
  }
}
```

**范围查& 排序查询**
range
gte 大于等于
lte 小于等于
sort 排序
```
get word/_search
{
  "query":{
    "range": {
      "wordType": {
        "gte": 35,
        "lte": 40
      }
    }
  },
  "sort":[
    {
      "id":{
        "order":"desc"
      }
    }
    ]
}
```
```java
   @Test
    public void testRange() throws IOException {
        SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //range查询
        final RangeQueryBuilder queryBuilder = QueryBuilders.rangeQuery("wordType");
        queryBuilder.gte(30).lte(40);
        sourceBuilder.query(queryBuilder);
//        sourceBuilder.sort("id", SortOrder.DESC);
        sourceBuilder.sort("id", SortOrder.ASC);
        //指定分页
        sourceBuilder.from(0).size(10);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
            log.info("获取到的word：{}",word);
        }
    }
```

**queryString查询**
对查询条件进行分词
默认匹配查询取并集
多字段同时查询
query_string : 将query中的AND符号也作为分词后的查询条件
```json
GET word/_search
{
  "query": {
    "query_string": {
      "fields": ["name","creator"],
      "query": "lijie3 AND 实体"
    }
  }
}
```
simple_string_query: 将query 中的AND不作为分词后的条件
```json
GET word/_search
{
  "query": {
    "simple_query_string": {
      "fields": ["name","creator"],
      "query": "lijie3 AND 实体"
    }
  }
}
```

**布尔查询**
boolQuery: 对多个查询条件连接，连接方式：
> must (and):条件必须成立,计算得分
> must_not(not)：条件必须不成立
> should( or) ：条件可以成立 
> filter：条件必须成立，性能比must高，不会计算得分

```json
GET word/_search
{
  "query": {
    "bool": {
      "must": [
        {
          
          "match":{
            "creator": "zhouyunjie"
          }
        }
      ],
      "filter": [
        {
          "term": {
            "name": "啦"
          }
        }
      ]
    }
  }
}
```
**聚合查询**
与mysql类型
* 指标聚合，相当于 mysql的 max、min、avg等
* 桶聚合，相当于mysql的group by，注意不要使用text类型的进行桶聚合，因为text字段会分词，导致分组失败

指标聚合
```json
GET word/_search
{
  "query": {
    "match": {
      "name": "实体"
    }
  },
  "aggs": {
    "max_id": {
      "max": {
        "field": "id"
      }
      
    }
  }
}
--结果
{
    .....
    //聚合结果
  "aggregations" : {
    "max_id" : {
      "value" : 287919.0
    }
  }
}

```

桶聚合
```json
GET word/_search
{
  "query": {
    "match": {
      "name": "实体"
    }
  },
  "aggs": {
    "creator_name": {
      "terms": {
        "field": "creator",
        "size": 10
      }
    }
  }
  
}
---结果
"aggregations" : {
    "creator_name" : {
      "doc_count_error_upper_bound" : 0,
      "sum_other_doc_count" : 0,
      "buckets" : [
        {
          "key" : "lijie3",
          "doc_count" : 11
        }
      ]
    }
  }
```

```java
    @Test
    public void testAggrs() throws IOException {
        SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //range查询
        final MatchQueryBuilder queryBuilder = QueryBuilders.matchQuery("name","实体");
        sourceBuilder.query(queryBuilder);
        AggregationBuilder agg = AggregationBuilders.terms("agg_creator").field("creator").size(20);
        sourceBuilder.aggregation(agg);
        //指定分页
        sourceBuilder.from(0).size(10);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
            log.info("获取到的word：{}",word);
        }
        //对聚合查询的结果解析
        Aggregations aggregations = search.getAggregations();
        Map<String, Aggregation> stringAggregationMap = aggregations.asMap();
        Terms aggCreator = (Terms) stringAggregationMap.get("agg_creator");
        List<? extends Terms.Bucket> buckets = aggCreator.getBuckets();
        buckets.forEach(bucket -> {
            log.info("key:{},count:{}",bucket.getKey(),bucket.getDocCount());
        });
    }
```
**高亮查询**
```json
GET word/_search
{
  "query": {
    "match": {
      "name": "实体"
    }
  },
  "highlight": {
    "fields": {
      "name":{
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>"
      }
    }
  }
}

--结果
{
  "took" : 961,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 11,
      "relation" : "eq"
    },
    "max_score" : 11.840421,
    "hits" : [
      {
        "_index" : "word",
        "_id" : "199815",
        "_score" : 11.840421,
        "_source" : {
          "createTime" : 1679322522000,
          "creator" : "lijie3",
          "id" : 199815,
          "modifier" : "lijie3",
          "name" : "实体瘤",
          "sourceType" : 2,
          "status" : 1,
          "updateTime" : 1679322522000,
          "wordType" : 43
        },
        "highlight" : {
          "name" : [
            "<font color='red'>实体</font>瘤"
          ]
        }
      }
      }

```
java api
```java
    @Test
    public void testHighLight() throws IOException {
        SearchRequest searchRequest = new SearchRequest("word");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        //range查询
        final MatchQueryBuilder queryBuilder = QueryBuilders.matchQuery("name","实体");
        HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.field("name").preTags("<font color='red'>").postTags("</font>");
        sourceBuilder.highlighter(highlightBuilder);
        sourceBuilder.query(queryBuilder);
        //指定分页
        sourceBuilder.from(0).size(10);
        searchRequest.source(sourceBuilder);

        SearchResponse search = client.search(searchRequest, RequestOptions.DEFAULT);
        SearchHits hits = search.getHits();
        log.info("hits:{}",hits.getTotalHits().value);
        final SearchHit[] hits1 = hits.getHits();
        for (SearchHit hit : hits1) {
            final String sourceAsString = hit.getSourceAsString();
            Word word = JSONObject.parseObject(sourceAsString,Word.class);
            final Map<String, HighlightField> highlightFields = hit.getHighlightFields();
            final HighlightField highlightField = highlightFields.get("name");
            Text[] text = highlightField.fragments();
            word.setName(text[0].toString());
            log.info("获取到的word：{}",word);
        }

    }
```
### 索引重建,索引别名
**索引重建**
es的索引一旦创建，只允许添加字段，不允许改变字段，因为改变字段，需要重新创建倒排索引，影响内部缓存结构，性能太低

需要创建时，需要创建一个新的索引，将原有的索引数据导入新索引中，删除老板本索引，然后设置新版本索引别名

```json
POST _reindex
{
  "source": {
    "index": "word"
  },
  "dest": {
    "index": "word2"
  }
}
```
设置索引别名
```json
POST word2/_alias/word3
```

## es集群

