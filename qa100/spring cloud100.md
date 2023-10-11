springcloud的组件：
spring cloud是一个微服务框架规范

spring cloud netflix 的组件：
1. 服务注册于发现 eureka
2. Feign + Ribbon  实现服务调与负载均衡
3. Hystrix 服务降级、服务熔断
4. config 分布式配置管理
5. zuul 网关
6. Sleuth 服务追踪

spring cloud alibaba的组件
1. Nacos 实现动态服务发现、服务配置、服务元数据及流量管理（负载均衡）。
2. Sentinel
3. RocketMQ
4. Dubbo 服务之间的rpc调用
5. Seata  开源的分布式事务解决方案
6. gateway

dubbo
* 服务治理框架
* 服务监控
* 服务注册与发现
* 服务的通信
* 服务的负载均衡

## nacos
主要功能
* 配置中心
* 服务发现
* 服务治理
* 提供负载均衡能力

基本的api操作：
启动nacos

sh startup.sh -m standalone

服务注册

curl -X POST 'http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=nacos.naming.serviceName&ip=20.18.7.10&port=8080'

发现服务

curl -X GET 'http://127.0.0.1:8848/nacos/v1/ns/instance/list?serviceName=nacos.naming.serviceName'

发布配置
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test&content=HelloWorld"

获取配置
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=nacos.cfg.dataId&group=test"

关闭服务
sh shutdown.sh

默认情况下nacos 的数据信息存储在嵌入式的数据库中
可以将nacos的数据源切换成mysql

一些概念
配置集（data ID）：一个配置文件通常就是一个配置集

配置分组（group）：对配置集进行分组



