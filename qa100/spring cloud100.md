SOA（Service-Oriented Architecture，面向服务的架构）: 面向服务的架构，是一种高层级的架构设计理念，可通过在网络上使用基于通用通信语言的服务接口，让软件组件可重复使用。
微服务（Microservices）是一种软件架构风格，它是以专注于单一责任与功能的小型功能区块 (Small Building Blocks) 为基础，利用模块化的方式组合出复杂的大型应用程序，各功能区块使用与语言无关 (Language-Independent/Language agnostic)的API集相互通信。
区别：
SOA粒度更粗，微服务粒度更细
SOA更多考虑兼容已有系统，微服务考虑快速交付，提供自动化测试、持续集成、自动化部署、自动化运维等最佳实践
SOA适合庞大、复杂、异构的企业级应用，微服务适用于快速、轻量级、基于web的互联网系统


最简单的微服架构：
消费者服务：接口调用方
生产者服务：提供接口放
服务注册中心：记录服务和服务地址之间的关系，当我们需要调用其他服务时，从注册中心找到服务地址进行调用

常见的服务注册中心：
zookeeper、eureka、consul、Nacos

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
2. Sentinel 实现限流、降级、服务保护等功能
3. RocketMQ
4. Dubbo 服务之间的rpc调用
5. Seata  开源的分布式事务解决方案，提供AT、TCC、SAGA和XA事物模式，为用户打造一站式的分布式解决方案
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

领域模型：
命名空间namespace：对不同的环境进行隔离，例如隔离开发环境、测试环境和生产环境
配置分组（group）：对配置集进行分组，将若干个服务、若干个配置归为一组，通常习惯一个系统归为一个组
配置集（data ID）：一个配置文件通常就是一个配置集
服务service： 微服务集群中的某一个服务


## Feign的实现原理
1. feign实际上整合了RestTemplate和Ribbon这两个工具
2. @FeignClient标记的接口通过jdk的动态代理实现了使用RestTemplate调用HTTP接口的过程


## Sentinel 熔断降级工具
以流量为切入点，从流量路由、流量控制、熔断降级、系统自适应过载保护、热点流量防护等多个维度保障微服务的稳定性
资源：被保护java代码称为资源、由应用程序调用其他应用提供服务，使用资源描述代码块
规则：保护资源的实施状态设定的规则，包括流量控制、熔断降级规则、系统保护规则
支持的规则种类：
流量控制规则、熔断降级规则、系统保护规则来源访问控制规则、热点参数规则

## gateway
网关的组成=路由转+过滤器
概念：
Route（路由）：网管的基础构建块，由ID、目标URI、一组断言和一组过滤器定义，断言为真，则匹配路由
Predicate(断言)：输入类型是一个serverwebExchange，可以使用它来匹配自HTTP请求的任何内容，例如headers或参数
Filter（过滤器）：对请求和响应进行修改处理，氛围GateWayFilter和GlobalFilter

对路由配置的支持：
* 通过时间匹配
* 通过Cookie匹配
* 通过Header属性匹配
* 通过Host匹配
* 通过请求方式匹配
* 通过请求路径匹配
* 通过请求参数匹配
* 通过请求ip地址进行匹配


## 服务降级熔断时怎么实现的
服务降级和熔断时一种常见的为服务治理机制，用于应对系统故障、资源不足活服务过载的情况，实现方式有多种：
* 超时设置：设置请求的最大等待时间，超过该时间认为服务不可用，进行降级处理
* 错误率限制：监控服务的错误率，当错误率超过设定域值时进行降级或熔断处理
* 限流策略：通过限制并发请求数量或请求速率来控制服务的负载，避免服务不可用
* 断路器模式：故障超时时，打开断路器，停止向服务发送请求
* 降级响应：服务不可用时方回预设的响应结果
