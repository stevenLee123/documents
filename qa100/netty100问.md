# netty
一个基于NIO客户端、服务器端的编程框架，基于netty可以快速开发网络应用

## nio 
nonblock io
使用selector监听socket连接，不需要为每个socket连接创建一个新的线程
### 三个组件
buffer -- ByteBuffer、CharBuffer、IntegerBuffer等，
flip（）切换为读模式，
clear（）切换为写模式，
compact（）进行压缩，将已读数据压缩掉
Channel -- FileChannel
Selector --选择器

## 文件复制的几种方式
1. 通过FileInputStream、FileOutputStream进行文件复制，这种方式速度较慢
2. 使用java.nio的Channel的transferTo/transferFrom实现
3. 使用Files.copy()实现拷贝

### 文件工具类
Path 路径类，可用来创建文件 
Paths 路径工具类
```java
Path path= Paths.get(/d1/d2);
//创建一个路径
Files.createDirectories(path);
Files.move(...)
Files.delete(path)
//遍历文件夹，访问者模式应用,可以用该方法删除多级文件目录
Files.walkFileTree(...)
//通过walk方法遍历目录文件，结合FileChannel、ByteBuffer，可以实现多级文件目录的拷贝
Files.walk(Path).forEach(path ->{});
```



### nio阻塞模式
单线程无法在阻塞模式下处理多个客户端的连接，一个客户端的操作会影响另一个客户端的操作
```java
//创建服务器
ServerSocketChannel ssc = ServerSocketChannel.open();
//监听端口
ssc.bind(new InetSocketAddress(8080));
//accept
List<SocketChannel> channels = new ArrayList<>();
ByteBuffer buffer = ByteBuffer = ByteBuffer.allocate(16);
while(true){
  //建立与客户端之间的连接，与客户端之间通信,阻塞模式，accept会一直等待客户端连接的到来
SocketChannel sc = ssc.accept();
channels.add(sc);
//接收客户端发送的数据
  for(SocketChannel channel: channels){
    //阻塞方法，线程停止运行等待数据读入，实际上是将数据写入Buffer
    channel.read(buffer);
    buffer.flip();
    // 从buffer中读数据...
    
    buffer.clear();
  }
}

//客户端代码
SocketChannel sc = SocketChannel.open();
//服务端在客户端connect之后accept方法才会有响应
sc.connect(new InetSocketAddress("localhost",8080));
//服务端会在客户端执行write之后才会执行channel.read
sc.write(Charset.defaultCharset().encode("hello"));
```

## 非阻塞模式
单线程可以建立连接并处理多个客户端的连接与数据
```java
//创建服务器
ServerSocketChannel ssc = ServerSocketChannel.open();
//设置非阻塞模式
ssc.configureBlocking(false);
//监听端口
ssc.bind(new InetSocketAddress(8080));
//accept
List<SocketChannel> channels = new ArrayList<>();
ByteBuffer buffer = ByteBuffer = ByteBuffer.allocate(16);
while(true){
  //建立与客户端之间的连接，与客户端之间通信，设置非阻塞模式后，如果没有连接建立，sc会返回null
SocketChannel sc = ssc.accept();

if(sc!= null){
  //设置sc非阻塞
  sc.configureBlocking(false);
  channels.add(sc);
}
//接收客户端发送的数据
    for(SocketChannel channel: channels){
      //非阻塞模式下，read会返回0,不会发生阻塞，
      int read = channel.read(buffer);
      if(read > 0){
        buffer.flip();
        // 从buffer中取数据...
        
        buffer.clear();
      }

    }
}

//客户端代码
SocketChannel sc = SocketChannel.open();
//服务端在客户端connect之后accept方法才会有响应
sc.connect(new InetSocketAddress("localhost",8080));
//服务端会在客户端执行write之后才会执行channel.read，客户端将数据写入到Buffer中
sc.write(Charset.defaultCharset().encode("hello"));
```
上面的非阻塞模式存在的问题： 
* 无连接请求时，线程依然会不断的运行，cpu的利用路也是100%

## selector改进非阻塞模式
selector.select()何时不阻塞：
* 事件发生时
* 调用Selector.wakeup(),唤醒Selector继续运行
* 调用Selector.close()
* selector所在的线程interrupt
```java
// 创建selector,管理多个channel
Selector selector = Selector.open();

//创建服务器
ServerSocketChannel ssc = ServerSocketChannel.open();
//设置非阻塞模式
ssc.configureBlocking(false);

//设置channel与selector之间的关联,注册
//SelectionKey 事件发生后可以通过它可以知道事件和事件发生的channel

SelectionKey sscKey = ssc.register(selector,0,null);
//只关注accpet事件
sscKey.interestOps(SelectionKey.OP_ACCEPT);

//监听端口
ssc.bind(new InetSocketAddress(8080));
//accept
List<SocketChannel> channels = new ArrayList<>();
ByteBuffer buffer = ByteBuffer = ByteBuffer.allocate(16);
while(true){
  //select方法，没有事件时阻塞，有事件发生恢复运行，在事件未处理时，select方法不会阻塞，会继续运行
  selector.select();

  //处理事件
  Iterator<SelectionKey> iter = selector.selectionKeys().iterator();
  while(iter.hasNext()){
    //区分事件类型
    //迭代key信息
    SelectionKey key = iter.next();
    //从迭代器中移除key,如果不移除，key还是在selector的集合中，下次运行时还是会遍历该key，而该key没有对应的事件需要处理，会出现问题
    iter.remove();
    if(key.isAcceptable()){
      ServerSocketChannel channel = (ServerSocketChannel)key.channel();
      //建立连接
      SocketChannel sc = channel.accept();
      //设置非阻塞模式
      sc.configureBlocking(false);
      //SocketChannel的key
      //第三个参数是附件，将一个bytebuffer座位附件关联到selecionkey上
      ByteBuffer buffer = ByteBuffer = ByteBuffer.allocate(16);
      SelectionKey scKey = sc.register(selector,0,buffer);
      scKey.interestOps(SelectionKey.OP_READ);
    }else if(key.isReadable()){
      //需要对客户端断开的read事件进行处理，分为客户端的正常断开和异常断开，正常断开看read方法的返回结果，返回-1，异常断开read时会报错
      try{
        SocketChannel channel = (SocketChannel)key.channel();
        ByteBuffer buf = (ByteBuffer) key.attachment();
        //如果客户端正常断开，read方法返回值时-1
        int read = channel.read(buffer);
        if(read == -1){
          key.cancel();
        }else{
          buffer.flip();
          //读取buffer中的数据.....
          //处理消息边界
          // 对buffer扩容
          ByteBufer newbuffer = ByteBuffer.allocate(buf.capacity() * 2);
          //将buffer设置会key的附件
          key.attachment(buffer);
        }
      }catch(IOException e){
        e.printStackTrace();
        //异常的事件进行反注册
        key.cancel();
      }
      
    }
    
  }
}
```
注意的问题：
* ByteBuffer 不是线程安全的
* 每个channel对应一个ByteBuffer
* 扩容时要考虑内存占用的问题

SelectionKey的 分类
* accept 有连接请求时出发
* connect 是客户端，，连接建立后出发
* read 可读事件
* write 可写事件

## 处理消息边界
由于bytebuffer与消息数据的大小不匹配，可能出现半包、粘包的问题
解决方案：
* 客户端使用固定长度，服务器端进行相应的解析
* 使用特定的分隔符来进行消息的边界确定，服务器端通过对比分隔符确定消息边界
* 将消息分为两个部分，前面一部分存储消息的长度，后面是消息实体，服务端通过读取前面的长度，确定分配多大的bytebuffer，再来读取数据大小


## 可写事件
```java
// 创建selector,管理多个channel
Selector selector = Selector.open();

//创建服务器
ServerSocketChannel ssc = ServerSocketChannel.open();
//设置非阻塞模式
ssc.configureBlocking(false);
//注册selector
SelectionKey sscKey = ssc.register(selector,SelectionKey.OP_ACCEPT);
ssc.bind(new InetSocketAddress(8080));

while(true){
  selector.select();
  Iterator<SelectionKey> iter = selector.selectKeys().iterator();
  while(iter.hasNext()){
    SelectionKey key = iter.next();
    iter.remove();
    if( key.isAcceptable()){
      //因为ServerSocketChannel只有一个可以直接执行
      SocketChannel = ssc.accept();
      sc.configureBlockiing(false);
      //注册
      SelectionKey sckey = sc.register(selector,0,null);
      //向客户端发送大量数据
      Stringbuilder sb = new StringBuilder();
      for(i = 0;i < 300000000; i++){
        sb.append("a");
      }
     
      ByteBuffer buffer = Charset.defaultCharset().encode(sb.toString());
       //由于操作系统缓冲区的限制，数据在发送过程中可能需要等待缓冲区的空闲,需要关注可写事件
      //返回值代表实际写入的字节数
      // while(buffer.hasRemaining()){
      //   int write = sc.write(buffer);
      // }
      int write = sc.write(buffer);
      System.out.println(write);
      if(buffer.hasRemaining()){
        //关注可读可写写事件
        sckey.interestOps(sckey.interestOps() + SelectionKey.OP_WEITE);
        //把未写完的数据挂在sckey上
        sckey.attach(buffer);
      }
    }else if(key.isWriteable()){
        BytBeuffer buf = (BytBeuffer)key.attachment();
        SocketChannel sc = key.channel();
        int write = sc.write(buffer);
        System.out.println(write);
        //清理操作
        if(buffer.hasRemaining()){
          key.attach(null);
          //不再关注可写事件
          key.interestOps(sckey.interestOps() - SelectionKey.OP_WEITE)
        }
    }
  }
}

//客户端
//建立连接
SocketChannel sc = SocketChannel.open();
sc.connect(new InetSocketAddress("localhost",8080));
//接收数据
while(true){
  ByteBuffer buf = ByteBuffer.allocate(1024 * 1024);
  count+ = sc.read(buf);
  System.out.println(count);
  buf.clear();
}

```
