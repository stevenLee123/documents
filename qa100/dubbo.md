## RPC
remote procedure call 远程过程调用协议，通过网络从远程计算机上请求服务，不需要了解底层网络技术协议
http协议是rpc规范的一种以实现

dubbo admin dubbo监控管理

## dubbo开发流程：
1. 指定当前服务名称
2. 指定注册中心
3. 指定通信协议
4. 暴露服务

## RPC原理
1. client调用以本地调用的方式调用服务
2. client stub接收到请求后负责将方法、参数等组装成可以在网络上传输的消息体
3. client stub查找服务地址，将消息发送到服务端
4. server stub收到消息进行解码
5. server stub根据解码结果调用本地服务
6. server端本地服务执行并返回给server stub
7. client接收消息，并进行解码
8. client端得到最终调用结果

## dubbo 框架设计
service层：服务接口层
config层：包含referenceconfig和serviceconfig
proxy层：proxyFactory生成service层的代理包装在config层中
registry层：注册层
cluster层
monitor层：监控层
protocol层：远程调用层
exchange层：数据交换成
transport层：传输层
serialize层：序列/反序列化层

## Dubbo启动解析配置文件

## dubbo服务暴露流程

## 服务引用流程

## 服务调用流程



