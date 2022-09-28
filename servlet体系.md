# Servlet 体系

## 1.HTTP协议基础
### 1.1概念
web浏览器与web服务器之间交互通信的协议,无状态的协议

*通信过程* 
* 建立连接
* 发送请求信息
* 回送响应信息
* 关闭连接

> HTTP1.1支持持久连接，在同一个TCP连接上支持多个HTTP请求和响应的发送，减少建立和关闭连接的消耗和延迟。

#### 1.1.1请求消息格式

> 请求行
> 若干消息头（可选）
> 空行（\r\n）,标识消息头部已经结束
> 实体内容

例子：

```
GET /api/list HTTP1.1
Accept: */*
Accept-Language: en-us
Connection: Keep-Alive
Host: localhost
Referer: http://localhost/api/list
Accept-Encoding: gzip,deflate

a=1&b=2
```
#### 1.1.2响应消息格式

> 状态行
> 若干消息头
> 空行（\r\n）
> 消息实体

例子：
```
HTTP/1.1 200 OK
Server: Miscrosoft-IIS/5.0
Date: Thu,13 Jul 2000 
Content-Length: 2291

{
    a:1,
    b:2
}
```

### 请求消息格式
* 对于get请求（一般不包含消息实体），以query string的方式发送请求消息，将消息附在请求URL后面例如
    `http://localhost/api/list?a=1&b=2`
* 对于POST请求，如果是以form形式发送消息实体，需要将请求头的Content-Type消息头设置为“application/x-www-form-urlencoded"

### 响应状态码

./build/build.sh dev true http://apollo-server-dev.k8s-test.uc.host.dxy







