---
title: storm心得-下
date: 2020-08-06 01:35:16
tags:
---

## dubbo的引入

随着互联网的发展，网站应用的规模不断扩大，常规的垂直应用架构已无法应对，分布式服务架构以及流动计算架构势在必行，亟需一个治理系统确保架构有条不紊的演进
![mark](https://img.jinguo.tech/blog/20200116/wjl7jTS7zaLL.png?imageslim)

- 单一应用架构
  当网站流量很小时，只需一个应用，将所有功能都部署在一起，以减少部署节点和成本。  
  此时，用于简化增删改查工作量的  数据访问框架(ORM)  是关键。  

- 垂直应用架构
  当访问量逐渐增大，单一应用增加机器带来的加速度越来越小，将应用拆成互不相干的几个应用，以提升效率。  
  此时，用于加速前端页面开发的  Web框架(MVC)  是关键。  

- 分布式服务架构
  当垂直应用越来越多，应用之间交互不可避免，将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。  
  此时，用于提高业务复用及整合的  分布式服务框架(RPC)  是关键。  

- 流动计算架构
  当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心基于访问压力实时管理集群容量，提高集群利用率  
  此时，用于提高机器利用率的  资源调度和治理中心(SOA)  是关键。  

  

在大规模服务化之前，应用可能只是通过RMI或Hessian等工具，简单的暴露和引用远程服务，通过配置服务的URL地址进行调用，通过F5等硬件进行负载均衡。  

1. 当服务越来越多时，服务URL配置管理变得非常困难，F5硬件负载均衡器的单点压力也越来越大。  
   此时需要一个服务注册中心，动态的注册和发现服务，使服务的位置透明。并通过在消费方获取服务提供方地址列表，实现软负载均衡和Failover，降低对F5硬件负载均衡器的依赖，也能减少部分成本。  
2. 当进一步发展，服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。  
   这时，需要自动画出应用间的依赖关系图，以理清理关系。  
3. 接着，服务的调用量越来越大，服务的容量问题就暴露出来，这个服务需要多少机器支撑？什么时候该加机器？  
   为了解决这些问题，第一步，要将服务现在每天的调用量，响应时间，都统计出来，作为容量规划的参考指标。     
   其次，要可以动态调整权重，在线上，将某台机器的权重一直加大，并在加大的过程中记录响应时间的变化，直到响应时间到达阀值，记录此时的访问量，再以此访问量乘以机器数反推总容量。  

### Dubbo的工作原理

![mark](https://img.jinguo.tech/blog/20200116/9BmbGSdmqAIw.png?imageslim)

### 节点角色说明：

- Provider:  暴露服务的服务提供方。
- Consumer:  调用远程服务的服务消费方。
- Registry:  服务注册与发现的注册中心。
- Monitor:  统计服务的调用次调和调用时间的监控中心。
- Container:  服务运行容器。

### 调用关系说明：

- 服务容器负责启动，加载，运行服务提供者。
- 服务提供者在启动时，向注册中心注册自己提供的服务。
- 服务消费者在启动时，向注册中心订阅自己所需的服务。
- 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
- 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
- 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## Dubbo-admin管理平台的安装

### dubbo-admin 本地编译打包

https://github.com/alibaba/dubbo/releases  
https://github.com/apache/incubator-dubbo/releases  
解压后，根目录里不存在dubbo-admin，无法编译打包，发现dubbo-admin迁移到新地址  
https://github.com/apache/incubator-dubbo-ops

### 克隆项目

apache 下的dubbo-admin git仓库：  
https://github.com/apache/incubator-dubbo-ops  
先把这个项目用git克隆到本地中  
![mark](https://img.jinguo.tech/blog/20200116/K3cL9oIMDyUc.png?imageslim)

#### application.properties配置

![mark](https://img.jinguo.tech/blog/20200116/OBg6vOx9Hm2v.png?imageslim)

#### application-test.properties配置

![](https://ws1.sinaimg.cn/large/005Vjva3gy1g3a8nhcmhnj30tw04ujrj.jpg)

#### tomcat配置启动项

![mark](https://img.jinguo.tech/blog/20200116/jTIptyM29eyc.png?imageslim)

#### 配置部署war包

![mark](https://img.jinguo.tech/blog/20200116/pMWYodys0jHX.png?imageslim)  
![mark](https://img.jinguo.tech/blog/20200116/815r6xyBFzgC.png?imageslim)

#### 说明：可以发现最新版本的 dubbo-admin 为springboot项目，可以直接打包成jar，使用java -jar xxx.jar 运行。

#### Production Setup

1. Clone source code on develop branch git clone https://github.com/apache/incubator-dubbo-admin.git
2. Specify registry address in dubbo-admin-server/src/main/resources/application.properties
3. Build  
   mvn clean package  
   ![mark](https://img.jinguo.tech/blog/20200116/QW4o2Lu212cH.png?imageslim)
4. Start  
   mvn --projects dubbo-admin-server spring-boot:run  
   **启动Zookeeper集群**
   ![mark](https://img.jinguo.tech/blog/20200116/6S0qSXjElGrA.png?imageslim)  
   ![mark](https://img.jinguo.tech/blog/20200116/KuguVid47fPC.png?imageslim)
5. Visit http://localhost:8080
   ![mark](https://img.jinguo.tech/blog/20200116/LvOOUrd9THDG.png?imageslim)

### 报错1

![mark](https://img.jinguo.tech/blog/20200116/Y5eggYkh3Rcn.png?imageslim)

### 解决办法

如果SpringBoot在子模块，直接main启动子模块会报错。
解决办法就是在IDEA MAVEN Projects->dubbo-admin-server->Plugins->spring-boot->spring-boot:run->run maven build
![mark](https://img.jinguo.tech/blog/20200116/3Ks7eNPV4RNd.png?imageslim)

### 报错2

![mark](https://img.jinguo.tech/blog/20200116/rbB1U2e6QnzT.png?imageslim)

### 解决办法

taskkill /pid 8876 /f

![mark](https://img.jinguo.tech/blog/20200116/C9aKSFeYcYUi.png?imageslim)

## zookeeper与dubbo关系

dubbo是动物园，动物园里有什么动物，有动物园自己说了算，zookeeper只是登记了园里有什么动物可供参观，游客可以参观那个动物，参观人数太多，ZK如何分流等，动物园可以不用ZK做这个工作（能提供这个功能的有很多），可以用别的做这个注册、选举、分流、负载均衡的管理工作，只是大家都用ZK；dubbo中的注册中心用了zookeeper而已，也可以用别的，dubbo有注册中心（使用了ZK）、服务提供者、消费者、运行容器，监视器；

## Netty在Dubbo中的应用

**Dubbo 底层使用的是 Netty 作为网络通信**  

1. dubbo的Consumer消费者如何使用Netty  

### 调用 Spring 容器的 getBean 方法, dubbo 扩展了 FactoryBean，所以，会调用 getObject 方法，该方法会创建代理对象。

```java
// get remote service proxy
DemoService demoService = (DemoService) context.getBean("demoService");
```

### 调用 DubboProtocol 实例的 getClients（URL url） 方法，当这个给定的 URL 的 client 没有初始化则创建，然后放入缓存  

```java
private ExchangeClient getSharedClient(URL url){
	String key=url.getAddress();
	ReferenceCountExchangeClient client=referenceClientMap.get(key);
	if(client!=null){
		if(!=client.isClosed()){
			client.incrementAndGetCount();
			return client;
		}else{
			referenceClientMap.remove(key);
		}
	}
	synchronized(key.intern()){
		//这个initClient()方法是创建Netty的client的
		ExchangeClient exchangeClient=initClient(url);
		client=new ReferenceCountExchangeClient(exchangeClient,ghostClientMap);
		referenceClientMap.put(key,client);
		ghostClientMap.remove(key);
		return client;
	}
}
```

### 最终调用的就是抽象父类AbstractClient的构造方法，构造方法中包含了创建Socket客户端，连接客户端等行为。

```java
public AbstractClient(URL url, ChannelHandler handler) throws RemotingException {
    doOpen();
    connect();
}
```

### doOpent 方法用来创建 Netty 的 bootstrap ：

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
    bootstrap = new ClientBootstrap(channelFactory);
    bootstrap.setOption("keepAlive", true);
    bootstrap.setOption("tcpNoDelay", true);
    bootstrap.setOption("connectTimeoutMillis", getTimeout());
    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        public ChannelPipeline getPipeline() {
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyClient.this);
            ChannelPipeline pipeline = Channels.pipeline();
            pipeline.addLast("decoder", adapter.getDecoder());
            pipeline.addLast("encoder", adapter.getEncoder());
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
}
```

### connect 方法用来连接提供者：

```java
protected void doConnect() throws Throwable {
    long start = System.currentTimeMillis();
	//调用了 bootstrap 的 connect 方法,这里使用的是 jboss 的 netty3,当连接成功后，注册写事件，准备开始向提供者传递数据。 
    ChannelFuture future = bootstrap.connect(getConnectAddress());
    boolean ret = future.awaitUninterruptibly(getConnectTimeout(), TimeUnit.MILLISECONDS);
    if (ret && future.isSuccess()) {
        Channel newChannel = future.getChannel();
        newChannel.setInterestOps(Channel.OP_READ_WRITE);
    } 
}
```

### main 方法最终会调用 HeaderExchangeChannel 的 request 方法，通过 channel 进行请求。  

```java
public ResponseFuture request(Object request, int timeout) throws RemotingException {
    Request req = new Request();
    req.setVersion("2.0.0");
    req.setTwoWay(true);
    req.setData(request);
    DefaultFuture future = new DefaultFuture(channel, req, timeout);
	//send 方法中最后调用 jboss Netty 中继承了 NioSocketChannel 的 NioClientSocketChannel 的 write 方法。完成了一次数据的传输。  
    channel.send(req);
    return future;
}
```

## dubbo 的 Provider 提供者如何使用 Netty  

Provider 作为被访问方，是一个 Server 模式的 Socket。 Spring 容器启动的时候，会调用一些扩展类的初始化方法，比如继承了  InitializingBean，ApplicationContextAware，ApplicationListener。而 dubbo 创建了 ServiceBean 继承了一个监听器。Spring 会调用他的 onApplicationEvent 方法，该类有一个 export 方法，用于打开 ServerSocket 。  然后执行了 DubboProtocol 的 createServer 方法，然后创建了一个 NettyServer 对象。

### NettyServer 对象的构造方法同样是 doOpen 方法。

```java
protected void doOpen() throws Throwable {
    NettyHelper.setNettyLoggerFactory();
	//boss 线程，worker 线程，和 ServerBootstrap
    ExecutorService boss = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerBoss", true));
    ExecutorService worker = Executors.newCachedThreadPool(new NamedThreadFactory("NettyServerWorker", true));
    ChannelFactory channelFactory = new NioServerSocketChannelFactory(boss, worker, getUrl().getPositiveParameter(Constants.IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS));
    bootstrap = new ServerBootstrap(channelFactory);
	//在添加了编解码 handler 之后，添加一个 NettyHandler，最后调用 bind 方法，完成绑定端口的工作。
    final NettyHandler nettyHandler = new NettyHandler(getUrl(), this);
    channels = nettyHandler.getChannels();
    bootstrap.setPipelineFactory(new ChannelPipelineFactory() {
        public ChannelPipeline getPipeline() {
            NettyCodecAdapter adapter = new NettyCodecAdapter(getCodec(), getUrl(), NettyServer.this);
            ChannelPipeline pipeline = Channels.pipeline();
            pipeline.addLast("decoder", adapter.getDecoder());
            pipeline.addLast("encoder", adapter.getEncoder());
            pipeline.addLast("handler", nettyHandler);
            return pipeline;
        }
    });
    channel = bootstrap.bind(getBindAddress());
}
```

### Netty在Dubbo中的应用总结

dubbo中消费者使用 NettyClient，提供者使用 NettyServer，Provider 启动的时候，会开启端口监听。Client 在 Spring getBean 的时候，会创建 Client。当调用远程方法的时候，将数据通过 dubbo 协议编码发送到 NettyServer，然后 NettServer 收到数据后解码，并调用本地方法，并返回数据，完成一次完美的 RPC 调用。

## Apache Storm分布式消息系统

Apache Storm处理实时数据，并且输入通常来自消息排队系统。外部分布式消息系统将提供实时计算所需的输入。Spout将从消息系统读取数据，并将其转换为元组并输入到Apache Storm中。Apache Storm在内部使用其自己的分布式消息传递系统，用于其nimbus和主管之间的通信。  

### 什么是分布式消息系统？  

分布式消息传递基于可靠消息队列的概念。消息在客户端应用程序和消息系统之间异步排队。分布式消息传递系统提供可靠性，可扩展性和持久性的好处。  
大多数消息模式遵循发布 - 订阅模型（简称发布 - 订阅），其中消息的发送者称为发布者，而想要接收消息的那些被称为订阅者。  
一旦消息已经被发​​送者发布，订阅者可以在过滤选项的帮助下接收所选择的消息。通常我们有两种类型的过滤，一种是基于主题的过滤，另一种是基于内容的过滤。  
需要注意的是，pub-sub模型只能通过消息进行通信。它是一个非常松散耦合的架构;甚至发件人不知道他们的订阅者是谁。许多消息模式使消息代理能够交换发布消息以便由许多订户及时访问。

![mark](https://img.jinguo.tech/blog/20200116/lG6PiOTlO76X.png?imageslim)

下表描述了一些流行的高吞吐量消息传递系统 -
![mark](https://img.jinguo.tech/blog/20200116/LFSn4gj0zFXj.png?imageslim)
Thrift在Facebook上构建，用于跨语言服务开发和远程过程调用（RPC）。后来，它成为一个开源的Apache项目。Apache Thrift是一种接口定义语言，允许以容易的方式在定义的数据类型之上定义新的数据类型和服务实现。    
Apache Thrift也是一个支持嵌入式系统，移动应用程序，Web应用程序和许多其他编程语言的通信框架。与Apache Thrift相关的一些关键功能是它的模块化，灵活性和高性能。此外，它可以在分布式应用程序中执行流式处理，消息传递和RPC。  
Storm广泛使用Thrift协议进行内部通信和数据定义。Storm拓扑只是Thrift Structs。在Apache Storm中运行拓扑的Storm Nimbus是一个Thrift服务。

## Storm工作原理

![mark](https://img.jinguo.tech/blog/20200116/79gVc35j1D9V.png?imageslim)

### Storm与传统关系型数据库 

传统关系型数据库是先存后计算，而storm则是先算后存，甚至不存   
传统关系型数据库很难部署实时计算，只能部署定时任务统计分析窗口数据 
关系型数据库重视事务，并发控制，相对来说Storm比较简陋   
Storm与Hadoop，Spark等是流行的大数据方案    
与Storm关系密切的语言：核心代码用clojure书写，实用程序用python开发，使用java开发拓扑  

1. topology  
   Storm集群中有两种组件节点，一种是**控制节点**(Nimbus节点)，另一种是**工作节点**(Supervisor节点)。这两种组件都是快速失败的，没有状态。任务状态和心跳信息等都保存在Zookeeper上的，提交的代码资源都在本地机器的硬盘上。所有Topology任务的 提交必须在Storm客户端节点上进行(需要配置 storm.yaml文件)，由Nimbus节点分配给其他Supervisor节点进行处理。 Nimbus负责在集群里面发送代码，分配工作给机器，并且监控状态。全局只有一个。Nimbus节点首先将提交的Topology进行分片，分成一个个的Task，并将Task和Supervisor相关的信息提交到 zookeeper集群上，Supervisor会去zookeeper集群上认领自己的Task，通知自己的Worker进程进行Task的处理。   
   和同样是计算框架的MapReduce相比，MapReduce集群上运行的是Job，而Storm集群上运行的是Topology。但是Job在运行结束之后会自行结束，Topology却只能被手动的kill掉，否则会一直运行下去  
   数据存储之后的展现，也是需要自己处理的，storm UI 只提供对topology的监控和统计。 
   ![mark](https://img.jinguo.tech/blog/20200116/giIigvaTGzfK.png?imageslim)

2. zookeeper集群  
   storm使用zookeeper来协调整个集群， 但是要注意的是storm并不用zookeeper来传递消息。所以zookeeper上的负载是非常低的，单个节点的zookeeper在大多数情况下 都已经足够了， 但是如果你要部署大一点的storm集群， 那么你需要的zookeeper也要大一点。  
   部署zookeeper有些需要注意的地方：  
   ①对zookeeper做好监控非常重要， zookeeper是fail-fast的系统，只要出现什么错误就会退出， 所以实际场景中要监控   
   ②实际场景中要配置一个cron job来压缩zookeeper的数据和业务日志。zookeeper自己是不会去压缩这些的，所以你如果不设置一个cron job, 磁盘会很快不够用

3. Component
   Storm中，Spout和Bolt都是Component。所以，Storm定义了一个名叫IComponent的总接口 
    全家谱如下：绿色部分是我们最常用、比较简单的部分。红色部分是与事务相关的。
   ![mark](https://img.jinguo.tech/blog/20200116/3ydBPi1GBWs0.png?imageslim)

4. Spout
   Spout是Stream的消息产生源， Spout组件的实现可以通过继承BaseRichSpout类或者其他Spout类来完成，也可以通过实现IRichSpout接口来实现

   ```java
   public interface ISpout extends Serializable { 
     void open(Map conf, TopologyContext context, SpoutOutputCollector collector); 
     void close(); 
     void nextTuple(); 
     void ack(Object msgId); 
     void fail(Object msgId); 
   }   
   ```

    ①open()方法 -- 初始化方法   
    close() -- 在该spout将要关闭时调用。但是不保证其一定被调用，因为在集群中supervisor节点，可以使用kill -9来杀死worker进程。只有当Storm是在本地模式下运行，如果是发送停止命令，可以保证close的执行   
    ②ack(Object msgId) -- 成功处理tuple时回调的方法，通常情况下，此方法的实现是将消息队列中的消息移除，防止消息重放   
    ③fail(Object msgId) -- 处理tuple失败时回调的方法，通常情况下，此方法的实现是将消息放回消息队列中然后在稍后时间里重放   
    ④nextTuple() -- 这是Spout类中最重要的一个方法。发射一个Tuple到Topology都是通过这个方法来实现的。调用此方法时，storm向spout发出请求，让spout发出元组（tuple）到输出器（ouput collector）。这种方法应该是非阻塞的，所以spout如果没有元组发出，这个方法应该返回。nextTuple、ack 和fail 都在spout任务的同一个线程中被循环调用。 当没有元组的发射时，应该让nextTuple睡眠一个很短的时间（如一毫秒），以免浪费太多的CPU。继承了BaseRichSpout后，不用实现close、 activate、 deactivate、 ack、 fail 和 getComponentConfiguration 方法，只关心最基本核心的部分。   通常情况下（Shell和事务型的除外），实现一个Spout，可以直接实现接口IRichSpout，如果不想写多余的代码，可以直接继承BaseRichSpout 

5. Bolt
   Bolt类接收由Spout或者其他上游Bolt类发来的Tuple，对其进行处理。Bolt组件的实现可以通过继承BasicRichBolt类或者IRichBolt接口等来完成  
     prepare方法 -- 此方法和Spout中的open方法类似，在集群中一个worker中的task初始化时调用。 它提供了bolt执行的环境   
     declareOutputFields方法 -- 用于声明当前Bolt发送的Tuple中包含的字段(field)，和Spout中类似   
     cleanup方法 -- 同ISpout的close方法，在关闭前调用。同样不保证其一定执行。   
     execute方法 -- 这是Bolt中最关键的一个方法，对于Tuple的处理都可以放到此方法中进行。具体的发送是通过emit方法来完成的。execute接受一个tuple进行处理，并用prepare方法传入的  OutputCollector的ack方法（表示成功）或fail（表示失败）来反馈处理结果。   
     Storm提供了IBasicBolt接口，其目的就是实现该接口的Bolt不用在代码中提供反馈结果了，Storm内部会自动反馈成功。如果你确实要反馈失败，可以抛出FailedException   
     通常情况下，实现一个Bolt，可以实现IRichBolt接口或继承BaseRichBolt，如果不想自己处理结果反馈，可以实现 IBasicBolt接口或继承BaseBasicBolt，它实际上相当于自动实现了collector.emit.ack(inputTuple) 

6. Topology运行方式
   在开始创建项目之前，了解Storm的操作模式(operation modes)是很重要的。 Storm有两种运行方式 

### 本地运行的提交方式 

```java
LocalCluster cluster = new LocalCluster(); 
cluster.submitTopology(TOPOLOGY_NAME, conf, builder.createTopology()); 
Thread.sleep(2000); 
cluster.shutdown(); 
```

### 分布式提交方式

```java
StormSubmitter.submitTopology（TOPOLOGY_NAME, conf, builder.createTopology()); 
```

  需要注意的是，在Storm代码编写完成之后，需要打包成jar包放到Nimbus中运行，打包的时候，不需要把依赖的jar都打迚去，否则如果把依赖的storm.jar包打进去的话，运行时会出现重复的配置文件错误导致Topology无法运行。因为Topology运行之前，会加载本地的 storm.yaml 配置文件。 

### 运行的命令如下###

```shell
storm jar StormTopology.jar mainclass [args] 
```

## storm守护进程的命令

  Nimbus: storm nimbus 启动nimbus守护进程   
  Supervisor: storm supervisor 启动supervisor守护迚程   
  UI：storm ui 这将启动stormUI的守护进程,为监测storm集群提供一个基于web的用户界面。  
  DRPC: storm drpc 启动DRPC的守护进程  

## storm管理命令

```shell
JAR：storm jar topology_jar topology_class [arguments...] 
```

jar命令是用于提交一个集群拓扑.它运行指定参数的topology_class中的main()方法，上传topology_jar到nimbus，由nimbus发布到集群中。一旦提交，storm将激活拓扑并开始处理topology_class 中的main()方法，main()方法负责调用StormSubmitter.submitTopology()方法，并提供一个唯一的拓扑(集群)的名。如果一个拥有该名称的拓扑已经存在于集群中，jar命令将会失败。常见的做法是在使用命令行参数来指定拓扑名称，以便拓扑在提交的时候被命名。 

```shell
KILL：storm kill topology_name [-w wait_time] 
```

杀死一个拓扑，可以使用kill命令。它会以一种安全的方式销毁一个拓扑，首先停用拓扑，在等待拓扑消息的时间段内允许拓扑完成当前的数据流。执行kill命令时可以通过-w [等待秒数]指定拓扑停用以后的等待时间。也可以在Storm UI 界面上实现同样的功能 

```shell
 Deactivate：storm deactivate topology_name 
```

 停用拓扑时，所有已分发的元组都会得到处理，spouts的nextTuple方法将不会被调用。也可以在Storm UI 界面上实现同样的功能 
	

```shell
  Activate：storm activate topology_name 
```

   启动一个停用的拓扑。也可以在Storm UI 界面上实现同样的功能 

```shell
 Rebalance：storm rebalance topology_name [-w wait_time] [-n worker_count] [-e component_name=executer_count]... 
```

 rebalance使你重新分配集群任务。这是个很强大的命令。比如，你向一个运行中的集群增加了节点。rebalance命令将会停用拓扑，然后在相应超时时间之后重分配worker，并重启拓扑   
	

```shell
storm rebalance wordcount-topology -w 15 -n 5 -e sentence-spout=4 -e split-bolt=8 
```

 还有其他管理命令，如：Remoteconfvalue、REPL、Classpath等 

## Storm与Hadoop的对比

![mark](https://img.jinguo.tech/blog/20200116/inthPTtTa26V.png?imageslim)

## DRPC通过DRPC Server来实现，DRPC Server的整体工作过程如下：  

引入DRPC主要是利用storm的实时计算能力来并行化CPU密集性的计算任务。  

1. 接收到一个RPC调用请求；  
2. 发送请求到Storm上的**拓扑**；  
3. 从Storm上接收计算结果；  
4. 将计算结果返回给客户端。  



## 附录

### maven更新镜像源

```xml
<mirrors>
		<!-- 阿里云仓库 -->
	          <mirror>
	              <id>alimaven</id>
	              <mirrorOf>central</mirrorOf>
	             <name>aliyun maven</name>
	             <url>https://maven.aliyun.com/repository/central</url>
	         </mirror>
	          <!-- 中央仓库1 -->
         <mirror>
             <id>repo1</id>
             <mirrorOf>central</mirrorOf>
             <name>Human Readable Name for this Mirror.</name>
             <url>http://repo1.maven.org/maven2/</url>
         </mirror>
     
         <!-- 中央仓库2 -->
         <mirror>
             <id>repo2</id>
             <mirrorOf>central</mirrorOf>
             <name>Human Readable Name for this Mirror.</name>
             <url>http://repo2.maven.org/maven2/</url>
         </mirror>
     </mirrors> 
  </mirrors>
	         
```

## RPC和MQ对比及其适用/不适用场合

### 系统结构  

**RPC系统结构：**  
Cosume <=> Provider  
Consumer调用的Provider提供的服务  

**Message Queue系统结构：**  
Sender <=> Queue <=> Reciver  
Sender发送消息给Queue；Receiver从Queue拿到消息来处理。

### 功能的特点

在架构上，RPC和Message的差异点是，Message有一个中间结点Message Queue，可以把消息存储。  

### 消息的特点

- Message Queue把请求的压力保存一下，逐渐释放出来，让处理者按照自己的节奏来处理。
- Message Queue引入一下新的结点，让系统的可靠性会受Message Queue结点的影响
- Message Queue是异步单向的消息。发送消息设计成是不需要等待消息处理的完成。
- 所以对于有同步返回需求，用Message Queue则变得麻烦了。

### PRC的特点

- 同步调用，对于要等待返回结果/处理结果的场景，RPC是可以非常自然直觉的使用方式。 
- RPC也可以是异步调用。
- 由于等待结果，Consumer（Client）会有线程消耗。
- 如果以异步RPC的方式使用，Consumer（Client）线程消耗可以去掉。但不能做到像消息一样暂存消息/请求，压力会直接传导到服务Provider。   

### 适用场合说明

- 希望同步得到结果的场合，RPC合适。
- 希望使用简单，则RPC；RPC操作基于接口，使用简单，使用方式模拟本地调用。异步的方式编程比较复杂。  
- 不希望发送端（RPC Consumer、Message Sender）受限于处理端（RPC Provider、Message Receiver）的速度时，使用Message Queue。  
- 随着业务增长，有的处理端处理量会成为瓶颈，会进行同步调用到异步消息的改造。
- 这样的改造实际上有调整业务的使用方式。比如原来一个操作页面提交后就下一个页面会看到处理结果；改造后异步消息后，下一个页面就会变成“操作已提交，完成后会得到通知”。  

### 不适用场合说明 

RPC同步调用使用Message Queue来传输调用信息。  
发送端是在等待，同时占用一个中间点的资源，没有对等的收益。RPC的方式可以保证调用返回即处理完成，使用消息方式后这一点不能保证了。

