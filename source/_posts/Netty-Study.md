---
title: Netty Study
date: 2019-12-26 15:59:12
tags: Netty
categories: Java
---

## 业界哪些流行的开源框架用Netty作为底层通信框架

- Dubbo： 阿里开源的高性能RPC框架
- RocketMQ： 阿里出品的高性能消息队列
- Spark： 炙手可热的大数据处理引擎，底层使用Netty
- Elasticsearch： 分布式多用户的全文搜索引擎
- Apache Cassandra：开源分布式搜索数据库
- Flink:分布式高性能高可用的流处理框架
- Netty-SocketIO的java服务端实现
- Spring5：使用Netty作为http协议框架
- Play：简单易用的http服务器
- Grpc:google开源的高性能rpc通信框架
- Infinispan：针对缓存的高并发键值对数据存储
- HornetQ：支持集群和多种协议，可嵌入、高性能的异步消息系统
- Vert.x轻量级高性能能JVM应用平台
  -<br>[完整的参考列表](https://netty.io/wiki/adopters.html)

## 业界有哪些公司在使用Netty

**在大型企业中**有：Apple、Twitter的Finagle、Facebook的Nifty、Google、Square、Instagram
<br>**在初创企业中**有：做http长连接的**Firebase**、支持各种各样消息推送通知的**Urban Airship**
<br><br> 当然，Netty也从这些项目中**受益**。通过实现 FTP、 SMTP、 HTTP 和 WebSocket 以及其他的基于二进制和基于文本的协议， Netty 扩展了它的应用范围及灵活性。

## Netty是什么

1. **异步**和**事件驱动**的高性能网络通信框架。
2. **特点:**它可以以任意的顺序响应在任意的时间点产生的事件，可以实现最高级别的可伸缩性。
3. **目的:**用于快速开发高性能服务端和客户端
4. **封装:**JDK底层BIO和NIO模型，提供高度可用的API,满足各类业务场景，其中ChannelHandler的热插拔机制解放了业务逻辑之外的细节问题，让业务逻辑的添加和删除非常容易
5. 自带编解码器解决拆包粘包问题，用户只关心业务逻辑
6. 精心设计的reactor线程模型支持高并发海量连接
7. 自带各种协议栈如http、websocket，处理任何一种通信协议都几乎不用亲自动手
8. **架构方法和设计原则：**每个小点都和它的技术性内容一样重要，穷其精妙。如关注点分离--业务和网络逻辑解耦，模块化和可复用性，可测试性。

## Netty的特性总结

![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2khv8xo9uj30xv0hnjwl.jpg)

<br>

## Socket & Netty

![Socket & Netty](http://ww1.sinaimg.cn/large/005Vjva3gy1g2jjx06me9j30wi0jj0z8.jpg)

## Netty基本组件
![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2kf4qvkn2j30wm0m4ti7.jpg)

- NioEventLoop ->Thread
- Channel ->Socket  	
  NioSocketChannel implements Channel  
  Chennel is a nexus to a network socket or a component which is capable operations such as read,write,connect,and bind
- ByteBuf ->IO Bytes  
  readBytes()、writeBytes() and so on
- Pipline ->Logic Chain 逻辑链 
- ChannelHandler ->Logic处理块

## Netty核心组件
### 1. Channel-Socket  
Channel是通讯的载体，其基本构造是Socket  
是对网络底层读写和连接原语言的抽象  

### 2. EventLoop-控制流、多线程处理、并发
定义了 Netty 的核心抽象， 用于处理连接的生命周期中所发生的事件

###注: Channel、 EventLoop、 Thread 以及EventLoopGroup 之间的关系
![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2q5th3hz7j30fu0cyq43.jpg)
##### A. 一个 EventLoopGroup 包含一个或者多个 EventLoop；  
##### B. 一个 EventLoop 在它的生命周期内只和一个 Thread 绑定
##### C. 所有由 EventLoop 处理的 I/O 事件都将在它专有的 Thread 上被处理；
##### D. 一个 Channel 在它的生命周期内只注册于一个 EventLoop；  
##### E. 一个 EventLoop 可能会被分配给一个或多个 Channel
##### F. 一个给定 Channel 的 I/O 操作都是由相同的 Thread 执行的， 实际上消除了对于同步的需要。
![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2q87lnpb3j30om0dm42s.jpg)

### 3. ChannelFuture-异步通知
Netty 中所有的 I/O 操作都是异步的,用于在之后的某个时间点确定其结果的方法
### 4. ChannelHandler和ChannelPipeline
ChannelHandler负责Channel中的逻辑处理  
其旨在简化应用程序处理逻辑的开发过程    
充当了所有处理入站和出站数据的应用程序逻辑的容器  
ChannelHandler子接口：  
ChannelInboundHandler——处理入站数据以及各种状态变化  
ChannelOutboundHandler——处理出站数据并且允许拦截所有的操作  
ChannelInboundHandler的方法:  
![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qektcmgnj30w70gyjxt.jpg)
ChannelOutboundHandler的方法:    
![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qems6khbj30w70glaex.jpg)

ChannelPipeline 提供了 ChannelHandler链的容器  
定义了用于在该链上传播入站和出站事件流的API

### 5. ByteBuf-Netty的数据容器	
Java NIO提供了ByteBuffer作为它的字节容器    
Netty的ByteBuffer替代品是ByteBuf  

##### A. 它可以被用户自定义的缓冲区类型扩展，通过内置的复合缓冲区类型实现了透明的零拷贝；  

##### B.容量可以按需增长（类似于 JDK 的 StringBuilder）

##### C.读和写使用了不同的索引

##### D.支持方法的链式调用

##### E.支持引用计数

##### F.支持池化

### 6. Bootstap-引导客户端和无连接协议
Bootstrap类负责为客户端和使用无连接协议的应用程序创建 Channel
![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2q8bxb9z6j30h109mgn0.jpg)

## 单元测试

使用EmbeddedChannel 测试 ChannelHandler  

1. 测试入站消息  
2. 测试出站消息  
3. 测试异常处理  

## 编解码器

- 解码器   
  将字节解码为消息  
  将一种消息类型解码为另一种
- 编码器  
  将消息编码为字节  
  将消息编码为消息

## Netty服务端启动

1. 创建服务端Channel
2. 初始化服务端Channel
3. 注册Selector
4. 端口绑定，实现对本地端口的接听

## 预置的ChannelHandler和编解码器

1. 通过 SSL/TLS 保护 Netty 应用程序
2. ChannelHandler处理 HTTP 和 HTTPS协议
3. 支持WebSocket
4. ChannelHandler检测空闲连接以及超时
5. FileRegion,通过支持零拷贝的文件传输的Channel来发送的文件区域
6. 使用JDK、JBOSS Marshalling、Protocol Buffers序列化数据
7. 使用UDP广播事件

### 创建服务端Channel
**bind()[用户代码入口] ->initAndRegister()[初始化并注册] ->newChannel()[创建服务端channel]**
![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2kg7oiieaj30tm0ds43k.jpg)

![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2kgfjm05gj314u0qg1kx.jpg)

![](http://ww1.sinaimg.cn/large/005Vjva3gy1g2kgxvgph7j31880o7nf4.jpg)

## 如何使用Netty进行RPC服务器的开发?  

1. 定义RPC请求消息、应答消息结构，里面要包括RPC的接口定义模块、包括远程调用的类名、方法名称、参数结构、参数值等信息。    
2. 服务端初始化的时候通过容器加载RPC接口定义和RPC接口实现类对象的映射关系，然后等待客户端发起调用请求。
3. 客户端发起的RPC消息里面包含，远程调用的类名、方法名称、参数结构、参数值等信息，通过网络，以字节流的方式送给RPC服务端，RPC服务端接收到字节流的请求之后，去对应的容器里面，查找客户端接口映射的具体实现对象。
4. RPC服务端找到实现对象的参数信息，通过反射机制创建该对象的实例，并返回调用处理结果，最后封装成RPC应答消息通知到客户端。
5. 客户端通过网络，收到字节流形式的RPC应答消息，进行拆包、解析之后，显示远程调用结果。
   ![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qilnmzwij30se0htwqp.jpg)   
   **客户端并发发起RPC调用请求，然后RPC服务端使用Netty连接器，分派出N个NIO连接线程，这个时候Netty连接器的任务结束。然后NIO连接线程是统一放到Netty NIO处理线程池进行管理，这个线程池里面会对具体的RPC请求连接进行消息编码、消息解码、消息处理等等一系列操作。最后进行消息处理（Handler）的时候，处于性能考虑，这里的设计是，直接把复杂的消息处理过程，丢给专门的RPC业务处理线程池集中处理，然后Handler对应的NIO线程就立即返回、不会阻塞。这个时候RPC调用结束，客户端会异步等待服务端消息的处理结果，通过消息回调机制实现。**  
   Netty对于RPC消息的解码、编码、处理对应的模块和流程，具体如下图所示：  
   ![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qipchj94j30s20chth5.jpg)  
   **客户端、服务端对RPC消息编码、解码、处理调用的模块以及调用顺序。    Netty把这样一个一个的处理器串在一起，形成一个责任链，统一进行调用。**



## 附录
### Netty疑问

1. Netty是什么？  
   Netty是一个基于JAVA NIO类库的异步通信框架，它的架构特点是：异步非阻塞、基于事件驱动、高性能、高可靠性和高可定制性。
2. 使用Netty能够做什么？  
   ①开发异步、非阻塞的TCP网络应用程序；  
   ②开发异步、非阻塞的UDP网络应用程序；  
   ③开发异步文件传输应用程序；  
   ④开发异步HTTP服务端和客户端应用程序；  
   ⑤提供对多种编解码框架的集成；  
   ⑥提供形式多样的编解码基础类库；  
   ⑦基于职责链模式的Pipeline-Handler机制；
   ⑧所有的IO操作都是异步的；
   ⑨IP黑白名单控制，性能统计；
   ⑩基于链路空闲事件检测的心跳检测；
3. Netty在哪些行业得到了应用  
   **①互联网行业：**随着网站规模的不断扩大，系统并发访问量也越来越高，传统基于Tomcat等Web容器的垂直架构已经无法满足需求，需要拆分应用进行服务化，以提高开发和维护效率。从组网情况看，垂直的架构拆分之后，系统采用分布式部署，各个节点之间需要远程服务调用，高性能的RPC框架必不可少，Netty作为异步高性能的通信框架，往往作为基础通信组件被这些RPC框架使用。  
   典型的应用有：阿里分布式服务框架Dubbo的RPC框架使用Dubbo协议进行节点间通信，Dubbo协议默认使用Netty作为基础通信组件，用于实现各进程节点之间的内部通信。其中，服务提供者和服务消费者之间，服务提供者、服务消费者和性能统计节点之间使用Netty进行异步/同步通信。除了Dubbo之外，淘宝的消息中间件RocketMQ的消息生产者和消息消费者之间，也采用Netty进行高性能、异步通信。  
   除了阿里系和淘宝系之外，很多其它的大型互联网公司或者电商内部也已经大量使用Netty构建高性能、分布式的网络服务器。  
   **②大数据领域：**经典的Hadoop的高性能通信和序列化组件Avro的RPC框架，默认采用Netty进行跨节点通信，它的Netty Service基于Netty框架二次封装实现。大数据计算往往采用多个计算节点和一个/N个汇总节点进行分布式部署，各节点之间存在海量的数据交换。由于Netty的综合性能是目前各个成熟NIO框架中最高的，因此，往往会被选中用作大数据各节点间的通信。   
   **③企业软件：**企业和IT集成需要ESB，Netty对多协议支持、私有协议定制的简洁性和高性能是ESB RPC框架的首选通信组件。事实上，很多企业总线厂商会选择Netty作为基础通信组件，用于企业的IT集成。  
   **④通信行业：**Netty的异步高性能、高可靠性和高成熟度的优点，使它在通信行业得到了大量的应用。  
   **⑤游戏行业：**无论是手游服务端、还是大型的网络游戏，Java语言得到了越来越广泛的应用。Netty作为高性能的基础通信组件，它本身提供了TCP/UDP和HTTP协议栈，非常方便定制和开发私有协议栈。账号登陆服务器、地图服务器之间可以方便的通过Netty进行高性能的通信。  
4. 使用传统的Socket开发挺简单的，我为什么要切换到NIO进行编程呢？  
   传统的同步阻塞IO通信存在如下几个问题：  
   **①线程模型存在致命缺陷：**一连接一线程的模型导致服务端无法承受大量客户端的并发连接；  
   **②性能差：**频繁的线程上下文切换导致CPU利用效率不高；  
   **③可靠性差：**由于所有的IO操作都是同步的，所以业务线程只要进行IO操作，也会存在被同步阻塞的风险，这会导致系统的可靠性差，依赖外部组件的处理能力和网络的情况。  
   **采用非阻塞IO（NIO）之后，同步阻塞IO的三个缺陷都将迎刃而解：**  
   ①Nio采用Reactor模式*，一个Reactor线程聚合一个多路复用器Selector，它可以同时注册、监听和轮询成百上千个Channel，一个IO线程可以同时并发处理N个客户端连接，线程模型优化为1：N（N < 进程可用的最大句柄数）或者 M : N (M通常为CPU核数 + 1， N < 进程可用的最大句柄数)；   
   ②由于IO线程总数有限，不会存在频繁的IO线程之间上下文切换和竞争，CPU利用率高；  
   ③所有的IO操作都是异步的，即使业务线程直接进行IO操作，也不会被同步阻塞，系统不再依赖外部的网络环境和外部应用程序的处理性能。  
   **由于切换到NIO编程之后可以为系统带来巨大的可靠性、性能提升，所以，目前采用NIO进行通信已经逐渐成为主流。**
5. 为什么不直接基于JDK的NIO类库编程呢？  
   即便抛开代码和NIO类库复杂性不谈，一个高性能、高可靠性的NIO服务端开发和维护成本都是非常高的，开发者需要具有丰富的NIO编程经验和网络维护经验，很多时候甚至需要通过抓包来定位问题。也许开发出一套NIO程序需要1个月，但是它的稳定很可能需要1年甚至更长的时间，这也就是为什么我不建议直接使用JDK NIO类库进行通信开发的一个重要原因。
6. 为什么要选择Netty框架？  
   Netty是业界最流行的NIO框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的，它已经得到成百上千的商用项目验证，例如Hadoop的RPC框架Avro使用Netty作为通信框架。很多其它业界主流的RPC和分布式服务框架，也使用Netty来构建高性能的异步通信能力。
   Netty的优点总结如下：  
   ①API使用简单，开发门槛低；  
   ②功能强大，预置了多种编解码功能，支持多种主流协议；  
   ③定制能力强，可以通过ChannelHandler对通信框架进行灵活的扩展；  
   ④性能高，通过与其它业界主流的NIO框架对比，Netty的综合性能最优；  
   ⑤成熟、稳定，Netty修复了已经发现的所有JDK NIO BUG，业务开发人员不需要再为NIO的BUG而烦恼；  
   ⑥社区活跃，版本迭代周期短，发现的BUG可以被及时修复，同时，更多的新功能会被加入；  
   ⑦经历了大规模的商业应用考验，质量得到验证。在互联网、大数据、网络游戏、企业应用、电信软件等众多行业得到成功商用，证明了它完全满足不同行业的商用标准。

## 代码
### 1. 基于Netty的客户端和服务端的简单通信
##### 要点：
①为初始化客户端， 创建了一个 Bootstrap 实例  
②为进行事件处理分配了一个 NioEventLoopGroup 实例， 其中事件处理包括创建新的连接以及处理入站和出站数据；  
③为服务器连接创建了一个 InetSocketAddress 实例；    
④当连接被建立时，一个 EchoClientHandler 实例会被安装到（该 Channel 的）ChannelPipeline 中；  
⑤在一切都设置完成后，调用 Bootstrap.connect()方法连接到远程节点；  
![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qkgzymnyj30gq0bht9k.jpg)

### EchoServerHandler
##### channelRead()—对于每个传入的消息都要调用；
##### channelReadComplete()—通知ChannelInboundHandler最后一次对channelRead()的调用是当前批量读取中的最后一条消息；
##### exceptionCaught()—在读取操作期间，有异常抛出时会调用。

```java
//标示一个ChannelHandler可以被多个 Channel 安全地共享
		@Sharable
		public class EchoServerHandler extends ChannelInboundHandlerAdapter {
		    @Override
		    public void channelRead(ChannelHandlerContext ctx, Object msg) {
		        ByteBuf in = (ByteBuf) msg;
		        //将消息记录到控制台
		        System.out.println(
		                "Server received: " + in.toString(CharsetUtil.UTF_8));
		        //将接收到的消息写给发送者，而不冲刷出站消息
		        ctx.write(in);
		    }
```

```
	    @Override
	    public void channelReadComplete(ChannelHandlerContext ctx)
	            throws Exception {
	        //将未决消息冲刷到远程节点，并且关闭该 Channel
	        ctx.writeAndFlush(Unpooled.EMPTY_BUFFER)
	                .addListener(ChannelFutureListener.CLOSE);
	    }
	
	    @Override
	    public void exceptionCaught(ChannelHandlerContext ctx,
	        Throwable cause) {
	        //打印异常栈跟踪
	        cause.printStackTrace();
	        //关闭该Channel
	        ctx.close();
	    }
	}
```

### EchoServer
##### 绑定到服务器将在其上监听并接受传入连接请求的端口；
##### 配置 Channel，以将有关的入站消息通知给 EchoServerHandler 实例

```
	public class EchoServer {
		private final int port;
	    public EchoServer(int port) {
	        this.port = port;
	    }
	
	    public static void main(String[] args)
	        throws Exception {
	        if (args.length != 1) {
	            System.err.println("Usage: " + EchoServer.class.getSimpleName() +
	                " <port>"
	            );
	            return;
	        }
	        //设置端口值（如果端口参数的格式不正确，则抛出一个NumberFormatException）
	        int port = Integer.parseInt(args[0]);
	        //调用服务器的 start()方法
	        new EchoServer(port).start();
	    }
	
	    public void start() throws Exception {
	        final EchoServerHandler serverHandler = new EchoServerHandler();
	        //(1) 创建EventLoopGroup
	        EventLoopGroup group = new NioEventLoopGroup();
	        try {
	            //(2) 创建ServerBootstrap
	            ServerBootstrap b = new ServerBootstrap();
	            b.group(group)
	                //(3) 指定所使用的 NIO 传输 Channel
	                .channel(NioServerSocketChannel.class)
	                //(4) 使用指定的端口设置套接字地址
	                .localAddress(new InetSocketAddress(port))
	                //(5) 添加一个EchoServerHandler到于Channel的 ChannelPipeline
	                .childHandler(new ChannelInitializer<SocketChannel>() {
	                    @Override
	                    public void initChannel(SocketChannel ch) throws Exception {
	                        //EchoServerHandler 被标注为@Shareable，所以我们可以总是使用同样的实例
	                        //这里对于所有的客户端连接来说，都会使用同一个 EchoServerHandler，因为其被标注为									 @Sharable，
	                        //这将在后面的章节中讲到。
	                        ch.pipeline().addLast(serverHandler);
	                    }
	                });
	            //(6) 异步地绑定服务器；调用 sync()方法阻塞等待直到绑定完成
	            ChannelFuture f = b.bind().sync();
	            System.out.println(EchoServer.class.getName() +
	                " started and listening for connections on " + f.channel().localAddress());
	            //(7) 获取 Channel 的CloseFuture，并且阻塞当前线程直到它完成
	            f.channel().closeFuture().sync();
	        } finally {
	            //(8) 关闭 EventLoopGroup，释放所有的资源
	            group.shutdownGracefully().sync();
	        }
	    }
	}
```

### 通过 ChannelHandler 实现客户端逻辑
##### channelActive()——在到服务器的连接已经建立之后将被调用；
##### channelRead0()——当服务器接收到一条消息时被调用
##### exceptionCaught()——在处理过程中引发异常时被调用。

```java
	@Sharable
		//标记该类的实例可以被多个 Channel 共享
		public class EchoClientHandler
		    extends SimpleChannelInboundHandler<ByteBuf> {
		    @Override
		    public void channelActive(ChannelHandlerContext ctx) {
		        //当被通知 Channel是活跃的时候，发送一条消息
		        ctx.writeAndFlush(Unpooled.copiedBuffer("Netty rocks!",
		                CharsetUtil.UTF_8));
		    }
        @Override
	    public void channelRead0(ChannelHandlerContext ctx, ByteBuf in) {
	        //记录已接收消息的转储
	        System.out.println(
	                "Client received: " + in.toString(CharsetUtil.UTF_8));
	    }
	
	    @Override
	    //在发生异常时，记录错误并关闭Channel
	    public void exceptionCaught(ChannelHandlerContext ctx,
	        Throwable cause) {
	        cause.printStackTrace();
	        ctx.close();
	    }
	}
```

### 引导客户端
##### 客户端是使用主机和端口参数来连接远程地址，也就是Echo 服务器的地址，而不是绑定到一个一直被监听的端口
​		public class EchoClient {
​		    private final String host;
​		    private final int port;
​		    public EchoClient(String host, int port) {
​		        this.host = host;
​		        this.port = port;
​		

```
	    }
	
	    public void start()
	        throws Exception {
	        EventLoopGroup group = new NioEventLoopGroup();
	        try {
	            //创建 Bootstrap
	            Bootstrap b = new Bootstrap();
	            //指定 EventLoopGroup 以处理客户端事件；需要适用于 NIO 的实现
	            b.group(group)
	                //适用于 NIO 传输的Channel 类型
	                .channel(NioSocketChannel.class)
	                //设置服务器的InetSocketAddress
	                .remoteAddress(new InetSocketAddress(host, port))
	                //在创建Channel时，向 ChannelPipeline中添加一个 EchoClientHandler实例
	                .handler(new ChannelInitializer<SocketChannel>()    {
	                    @Override
	                    public void initChannel(SocketChannel ch)
	                        throws Exception {
	                        ch.pipeline().addLast(
	                             new EchoClientHandler());
	                    }
	                });
	            //连接到远程节点，阻塞等待直到连接完成
	            ChannelFuture f = b.connect().sync();
	            //阻塞，直到Channel 关闭
	            f.channel().closeFuture().sync();
	        } finally {
	            //关闭线程池并且释放所有的资源
	            group.shutdownGracefully().sync();
	        }
	    }
	
	    public static void main(String[] args)
	            throws Exception {
	        if (args.length != 2) {
	            System.err.println("Usage: " + EchoClient.class.getSimpleName() +
	                    " <host> <port>"
	            );
	            return;
	        }
	
	        final String host = args[0];
	        final int port = Integer.parseInt(args[1]);
	        new EchoClient(host, port).start();
	    }
	}
```



### 2. 基于Zookeeper、Netty和Spring的轻量级的分布式RPC框架
##### 简易RPC有如下特性：

- 服务异步调用的支持，回调函数callback的支持
- 客户端使用长连接（在多次调用共享连接）
- 服务端异步多线程处理RPC请求
- 服务发布与订阅：服务端使用Zookeeper注册服务地址，客户端从Zookeeper获取可用的服务地址。
- 通信：使用Netty作为通信框架
- Spring：使用Spring配置服务，加载Bean，扫描注解
- 动态代理：客户端使用代理模式透明化服务调用
- 消息编解码：使用Protostuff序列化和反序列化消息

##### RPC介绍
RPC，即 Remote Procedure Call（远程过程调用），调用远程计算机上的服务，就像调用本地服务一样。RPC可以很好的解耦系统，如WebService就是一种基于Http协议的RPC。
![](https://ws1.sinaimg.cn/large/005Vjva3gy1g2qj3z3tr0j30er089dfu.jpg)

- 服务端发布服务

##### 服务注解：

```
	@Target({ElementType.TYPE})
	@Retention(RetentionPolicy.RUNTIME)
	@Component
	public @interface RpcService {
	    Class<?> value();
	}
```

##### 一个服务接口：
​		public interface HelloService {
​		

```
	    String hello(String name);
	
	    String hello(Person person);
	}
```

##### 一个服务实现：使用注解标注：
​		@RpcService(HelloService.class)
​		public class HelloServiceImpl implements HelloService {
​		

```
	    @Override
	    public String hello(String name) {
	        return "Hello! " + name;
	    }
	
	    @Override
	    public String hello(Person person) {
	        return "Hello! " + person.getFirstName() + " " + person.getLastName();
	    }
	}
```

##### 服务在启动的时候扫描得到所有的服务接口及其实现：

```
	@Override
	    public void setApplicationContext(ApplicationContext ctx) throws BeansException {
	        Map<String, Object> serviceBeanMap = ctx.getBeansWithAnnotation(RpcService.class);
	        if (MapUtils.isNotEmpty(serviceBeanMap)) {
	            for (Object serviceBean : serviceBeanMap.values()) {
	                String interfaceName = serviceBean.getClass().getAnnotation(RpcService.class).value().getName();
	                handlerMap.put(interfaceName, serviceBean);
	            }
	        }
	    }
```

##### 在Zookeeper集群上注册服务地址：
​		public class ServiceRegistry {
​		

```
	    private static final Logger LOGGER = LoggerFactory.getLogger(ServiceRegistry.class);
	
	    private CountDownLatch latch = new CountDownLatch(1);
	
	    private String registryAddress;
	
	    public ServiceRegistry(String registryAddress) {
	        this.registryAddress = registryAddress;
	    }
	
	    public void register(String data) {
	        if (data != null) {
	            ZooKeeper zk = connectServer();
	            if (zk != null) {
	                AddRootNode(zk); // Add root node if not exist
	                createNode(zk, data);
	            }
	        }
	    }
	
	    private ZooKeeper connectServer() {
	        ZooKeeper zk = null;
	        try {
	            zk = new ZooKeeper(registryAddress, Constant.ZK_SESSION_TIMEOUT, new Watcher() {
	                @Override
	                public void process(WatchedEvent event) {
	                    if (event.getState() == Event.KeeperState.SyncConnected) {
	                        latch.countDown();
	                    }
	                }
	            });
	            latch.await();
	        } catch (IOException e) {
	            LOGGER.error("", e);
	        }
	        catch (InterruptedException ex){
	            LOGGER.error("", ex);
	        }
	        return zk;
	    }
	
	    private void AddRootNode(ZooKeeper zk){
	        try {
	            Stat s = zk.exists(Constant.ZK_REGISTRY_PATH, false);
	            if (s == null) {
	                zk.create(Constant.ZK_REGISTRY_PATH, new byte[0], ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
	            }
	        } catch (KeeperException e) {
	            LOGGER.error(e.toString());
	        } catch (InterruptedException e) {
	            LOGGER.error(e.toString());
	        }
	    }
	
	    private void createNode(ZooKeeper zk, String data) {
	        try {
	            byte[] bytes = data.getBytes();
	            String path = zk.create(Constant.ZK_DATA_PATH, bytes, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
	            LOGGER.debug("create zookeeper node ({} => {})", path, data);
	        } catch (KeeperException e) {
	            LOGGER.error("", e);
	        }
	        catch (InterruptedException ex){
	            LOGGER.error("", ex);
	        }
	    }
	}
```



- 客户端调用服务
  ##### 使用代理模式调用服务：
  ```
  public class RpcProxy {
      private String serverAddress;
      private ServiceDiscovery serviceDiscovery;
  
      public RpcProxy(String serverAddress) {
          this.serverAddress = serverAddress;
      }
  
      public RpcProxy(ServiceDiscovery serviceDiscovery) {
          this.serviceDiscovery = serviceDiscovery;
      }
  
      @SuppressWarnings("unchecked")
      public <T> T create(Class<?> interfaceClass) {
          return (T) Proxy.newProxyInstance(
                  interfaceClass.getClassLoader(),
                  new Class<?>[]{interfaceClass},
                  new InvocationHandler() {
                      @Override
                      public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                          RpcRequest request = new RpcRequest();
                          request.setRequestId(UUID.randomUUID().toString());
                          request.setClassName(method.getDeclaringClass().getName());
                          request.setMethodName(method.getName());
                          request.setParameterTypes(method.getParameterTypes());
                          request.setParameters(args);
  
                          if (serviceDiscovery != null) {
                              serverAddress = serviceDiscovery.discover();
                          }
                          if(serverAddress != null){
                              String[] array = serverAddress.split(":");
                              String host = array[0];
                              int port = Integer.parseInt(array[1]);
  
                              RpcClient client = new RpcClient(host, port);
                              RpcResponse response = client.send(request);
  
                              if (response.isError()) {
                                  throw new RuntimeException("Response error.",new Throwable(response.getError()));
                              } else {
                                  return response.getResult();
                              }
                          }
                          else{
                              throw new RuntimeException("No server address found!");
                          }
                      }
                  }
          );
      }
  }
  
  ```

##### 从Zookeeper上获取服务地址：
		
```
	public class ServiceDiscovery {
	    private static final Logger LOGGER = LoggerFactory.getLogger(ServiceDiscovery.class);
	
	    private CountDownLatch latch = new CountDownLatch(1);
	
	    private volatile List<String> dataList = new ArrayList<>();
	
	    private String registryAddress;
	
	    public ServiceDiscovery(String registryAddress) {
	        this.registryAddress = registryAddress;
	        ZooKeeper zk = connectServer();
	        if (zk != null) {
	            watchNode(zk);
	        }
	    }
	
	    public String discover() {
	        String data = null;
	        int size = dataList.size();
	        if (size > 0) {
	            if (size == 1) {
	                data = dataList.get(0);
	                LOGGER.debug("using only data: {}", data);
	            } else {
	                data = dataList.get(ThreadLocalRandom.current().nextInt(size));
	                LOGGER.debug("using random data: {}", data);
	            }
	        }
	        return data;
	    }
	
	    private ZooKeeper connectServer() {
	        ZooKeeper zk = null;
	        try {
	            zk = new ZooKeeper(registryAddress, Constant.ZK_SESSION_TIMEOUT, new Watcher() {
	                @Override
	                public void process(WatchedEvent event) {
	                    if (event.getState() == Event.KeeperState.SyncConnected) {
	                        latch.countDown();
	                    }
	                }
	            });
	            latch.await();
	        } catch (IOException | InterruptedException e) {
	            LOGGER.error("", e);
	        }
	        return zk;
	    }
	
	    private void watchNode(final ZooKeeper zk) {
	        try {
	            List<String> nodeList = zk.getChildren(Constant.ZK_REGISTRY_PATH, new Watcher() {
	                @Override
	                public void process(WatchedEvent event) {
	                    if (event.getType() == Event.EventType.NodeChildrenChanged) {
	                        watchNode(zk);
	                    }
	                }
	            });
	            List<String> dataList = new ArrayList<>();
	            for (String node : nodeList) {
	                byte[] bytes = zk.getData(Constant.ZK_REGISTRY_PATH + "/" + node, false, null);
	                dataList.add(new String(bytes));
	            }
	            LOGGER.debug("node data: {}", dataList);
	            this.dataList = dataList;
	        } catch (KeeperException | InterruptedException e) {
	            LOGGER.error("", e);
	        }
	    }
	}
```

- 消息编码
  ##### 请求消息：
  ```
  public class RpcRequest {	
      private String requestId;
      private String className;
      private String methodName;
      private Class<?>[] parameterTypes;
      private Object[] parameters;
  
      public String getRequestId() {
          return requestId;
      }
  
      public void setRequestId(String requestId) {
          this.requestId = requestId;
      }
  
      public String getClassName() {
          return className;
      }
  
      public void setClassName(String className) {
          this.className = className;
      }
  
      public String getMethodName() {
          return methodName;
      }
  
      public void setMethodName(String methodName) {
          this.methodName = methodName;
      }
  
      public Class<?>[] getParameterTypes() {
          return parameterTypes;
      }
  
      public void setParameterTypes(Class<?>[] parameterTypes) {
          this.parameterTypes = parameterTypes;
      }
  
      public Object[] getParameters() {
          return parameters;
      }
  
      public void setParameters(Object[] parameters) {
          this.parameters = parameters;
      }
  }
  
  ```

##### 响应消息：
```
	public class RpcResponse {
	    private String requestId;
	    private String error;
	    private Object result;
	
	    public boolean isError() {
	        return error != null;
	    }
	
	    public String getRequestId() {
	        return requestId;
	    }
	
	    public void setRequestId(String requestId) {
	        this.requestId = requestId;
	    }
	
	    public String getError() {
	        return error;
	    }
	
	    public void setError(String error) {
	        this.error = error;
	    }
	
	    public Object getResult() {
	        return result;
	    }
	
	    public void setResult(Object result) {
	        this.result = result;
	    }
	}

```

##### 消息序列化和反序列化工具：（基于 Protostuff 实现）
```
	public class SerializationUtil {
	    private static Map<Class<?>, Schema<?>> cachedSchema = new ConcurrentHashMap<>();
	
	    private static Objenesis objenesis = new ObjenesisStd(true);
	
	    private SerializationUtil() {
	    }
	
	    @SuppressWarnings("unchecked")
	    private static <T> Schema<T> getSchema(Class<T> cls) {
	        Schema<T> schema = (Schema<T>) cachedSchema.get(cls);
	        if (schema == null) {
	            schema = RuntimeSchema.createFrom(cls);
	            if (schema != null) {
	                cachedSchema.put(cls, schema);
	            }
	        }
	        return schema;
	    }
	
	    /**
	     * 序列化（对象 -> 字节数组）
	     */
	    @SuppressWarnings("unchecked")
	    public static <T> byte[] serialize(T obj) {
	        Class<T> cls = (Class<T>) obj.getClass();
	        LinkedBuffer buffer = LinkedBuffer.allocate(LinkedBuffer.DEFAULT_BUFFER_SIZE);
	        try {
	            Schema<T> schema = getSchema(cls);
	            return ProtostuffIOUtil.toByteArray(obj, schema, buffer);
	        } catch (Exception e) {
	            throw new IllegalStateException(e.getMessage(), e);
	        } finally {
	            buffer.clear();
	        }
	    }
	
	    /**
	     * 反序列化（字节数组 -> 对象）
	     */
	    public static <T> T deserialize(byte[] data, Class<T> cls) {
	        try {
	            T message = (T) objenesis.newInstance(cls);
	            Schema<T> schema = getSchema(cls);
	            ProtostuffIOUtil.mergeFrom(data, message, schema);
	            return message;
	        } catch (Exception e) {
	            throw new IllegalStateException(e.getMessage(), e);
	        }
	    }
	}

```

- 性能改进
  ##### 服务端请求异步处理

  ```java
  public void channelRead0(final ChannelHandlerContext ctx,final RpcRequest request) throws Exception{
  	        RpcServer.submit(new Runnable() {
  	            @Override
  	            public void run() {
  	                LOGGER.debug("Receive request " + request.getRequestId());
  	                RpcResponse response = new RpcResponse();
  	                response.setRequestId(request.getRequestId());
  	                try {
  	                    Object result = handle(request);
  	                    response.setResult(result);
  	                } catch (Throwable t) {
  	                    response.setError(t.toString());
  	                    LOGGER.error("RPC Server handle request error",t);
  	                }
   ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE).addListener(new ChannelFutureListener() {
  	                    @Override
  	                    public void operationComplete(ChannelFuture channelFuture) throws Exception{
  	                        LOGGER.debug("Send response for request " + request.getRequestId());
  	                    }
  	                });
  	            }
  	        });
  	    }
  ```

##### 服务端长连接的管理
客户端保持和服务进行**长连接**，不需要每次调用服务的时候进行连接，长连接的管理（通过Zookeeper获取有效的地址）。  
通过监听Zookeeper服务节点值的变化，动态更新客户端和服务端保持的长连接。这个事情现在放在客户端在做，客户端保持了和所有可用服务的长连接，给客户端和服务端都造成了压力，需要解耦这个实现。

##### 客户端请求异步处理
**客户端请求异步处理的支持，不需要同步等待：发送一个异步请求，返回Future，通过Future的callback机制获取结果。**

```
	IAsyncObjectProxy client = rpcClient.createAsync(HelloService.class);
	RPCFuture helloFuture = client.call("hello", Integer.toString(i));
	String result = (String) helloFuture.get(3000, TimeUnit.MILLISECONDS);
```



