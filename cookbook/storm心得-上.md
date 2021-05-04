---
title: storm(心得)
date: 2020-01-16 16:08:52
tags: Storm
categories: 大数据
---

# Storm入门

Storm是一个**分布式的**，可靠的，容错的**数据流处理系统**。它会把工作任务委托给不同类型的组件，每个组件负责处理一项简单特定的任务。   
Storm是Twitter开源的一个分布式的实时计算系统，用于数据的实时分析，持续计算，分布式RPC等等
Storm是一个免费开源、分布式、高容错的实时计算系统。Storm令持续不断的流计算变得容易，弥补了Hadoop批处理所不能满足的实时要求。Storm经常用于在实时分析、在线机器学习、持续计算、分布式远程调用和ETL等领域。Storm的部署管理非常简单，而且，在同类的流式计算工具，Storm的性能也是非常出众的  
Storm集群的输入流由一个被称作spout的组件管理，spout把数据传递给bolt， bolt要么把数据保存到某种存储器，要么把数据传递给其它的bolt。
一个Storm集群就是在一连串的bolt之间转换spout传过来的数据。  
注：Storm中的核心术语  
**spout**龙卷，读取原始数据为bolt提供数据  
**bolt** 雷电，从spout或其它bolt接收数据，并处理数据，处理结果可作为其它bolt的数据源或最终结果    
**nimbus** 雨云，主节点的守护进程，负责为工作节点分发任务  
**topology** 拓扑结构，Storm的一个任务单元  
**define field(s)** 定义域，由spout或bolt提供，被bolt接收  

## Storm应用案例

- 数据处理流，Storm不需要中间队列
- 连续计算。连续发送数据到客户端，使它们能够实时更新并显示结果。
- **分布式远程过程调用**
- 频繁的CPU密集型操作**并行化**。

## Storm组件

在Storm集群中，有两类节点：主节点master node和工作节点worker nodes。  
主节点运行着一个叫做**Nimbus**的守护进程。这个守护进程负责在集群中分发代码，为工作节点分配任务，并监控故障。    
**Supervisor**守护进程作为拓扑的一部分运行在工作节点上。  
一个Storm**拓扑结构**在不同的机器上运行着众多的工作节点。
因为Storm在**Zookeeper**或本地磁盘上**维持所有的集群状态**，守护进程可以是无状态的而且失效或重启时不会影响整个系统的健康  
在系统底层，Storm使用了**zeromq**，这是一种先进的，可嵌入的**网络通讯库**，它提供的绝妙功能使Storm成为可能。其中，Storm只用了push/pull sockets      

### 注：zeromq的特性  

- 一个并发架构的Socket库  
- 对于集群产品和超级计算，比TCP要快    
- 可通过inproc（进程内）, IPC（进程间）, TCP和multicast(多播协议)通信  
- 异步I / O的可扩展的多核消息传递应用程序  
- 利用扇出(fanout), 发布订阅（PUB-SUB）,管道（pipeline）, 请求应答（REQ-REP），等方式实现N-N连接  
  注：最新的Storm已不再必须依赖**ZeroMQ**，各种依赖的库和软件也已经有更新的版本。
  最近版本的Storm支持使用**netty**做消息队列。
  Netty提供**异步的、事件驱动**的网络应用程序框架和工具，用以快速开发**高性能、高可靠性的**网络服务器和客户端程序。正好是 storm所需要的。

## Storm的特性

- 简化编程：使用Storm，实现实时处理的复杂性被大大降低了
- 开发容易：使用一门基于JVM的语言开发会更容易，也可以借助一个小的中间件，在Storm上使用任何语言开发
- 容错：Storm集群会关注工作节点状态，如果宕机了必要的时候会重新分配任务。
- 可扩展：所有需要为扩展集群所做的工作就是增加机器。Storm会在新机器就绪时向它们分配任务。
- 可靠的：所有消息都可保证至少处理一次。如果出错了，消息可能处理不只一次，永远不会丢失消息。
- 快速：速度是驱动Storm设计的一个关键因素
- 事务性：可以为几乎任何计算得到恰好一次消息语义

## 安装Storm集群

要手工安装Storm，需要先安装以下软件  

1. Zookeeper集群
2. Java
3. Python
4. Unzip命令

### 注：

Nimbus和管理进程将要依赖Java、Python和unzip命令

### 前期准备

1. 准备搭建3节点集群,准备3个虚拟机node1,node2,node3  
2. 配置好hosts映射文件和互相的ssh免密登录  
3. 配置好JDK  
   注：storm是依赖于zookeeper的,搭建storm集群前,必须先把zookeeper集群搭建好

### 安装storm

1. 准备好storm安装包
2. 上传解压重命名为storm到/export/server路径下
3. 修改配置文件 storm.yaml

### 运行

- 前台启动 (前台启动会占用窗口)  
  （1）在node1上启动 nimbus进程(主节点) 和 web UI  
  （2）在 node2 和 node3 上启动 supervisor(从节点)
- 后台启动

#### ssh脚本实现一键启动

```shell
.#!/bin/bash
source /etc/profile
nohup /export/server/storm/bin/storm nimbus >/dev/null 2>&1 &
echo "node1 nimbus is running"
nohup /export/server/storm/bin/storm ui >/dev/null 2>&1 &
echo "node1 core is running"
for host in node2 node3
do
{
ssh $host "source /etc/profile;nohup /export/server/storm/bin/storm supervisor >/dev/null 2>&1 &"
echo "$host Supervisor is running"
}
done
```

### 进入web页面查看集群

## 使用入门

### MAVEN依赖

```xml
<dependency>
    <groupId>org.apache.storm</groupId>
    <artifactId>storm-core</artifactId>
    <version>1.1.1</version>
    <!-- 目前<scope>可以使用5个值：
    * compile，缺省值，适用于所有阶段，会随着项目一起发布。
    * provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。
    * runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。
    * test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
    * system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。  -->
    <!--<scope>provided</scope>-->
</dependency>
```

### 编写Spout类读取日志文件中的内容, 并把数据发送给下游Bolt类进行处理

```java
/***
 * Version: 
 * Description: 读取外部文件,把一行一行的数据发送给下游的bolt
 *              类似于hadoop mapreduce的inputformat
 ***/
//BaseBasicSpout
public class ReadFileSpout extends BaseRichSpout {
    private SpoutOutputCollector spoutOutputCollector;
    private BufferedReader bufferedReader;
    /**
     * 初始化方法, 类似于这个类的构造器, 只被运行一次
     * spout组件读取原始数据为bolt提供数据
     * 一般用来打开数据链接, 打开网络连接
     * @param map 传入的是storm集群的配置文件和用户自定义的配置文件, 一般不用
     * @param topologyContext 上下文对象, 一般不用
     * @param spoutOutputCollector 数据输出的收集器,spout把数据传给此参数,由此参数传给storm框架
     */
    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
        try {
        	//本地模式
            //this.bufferedReader = new BufferedReader(new FileReader(new File("D:\\wordcount.txt")));
            //集群模式
            this.bufferedReader = new BufferedReader(new FileReader(new File("//root//stormdata//wordcount.txt")));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        this.spoutOutputCollector = spoutOutputCollector;
    }

    /**
     * 下一个tuple, tuple是数据传送的基本单位
     * 不断地往下一个组件发送tuple消息
     * 这里面是该spout组件的核心逻辑
     * 如从kafka消息队列中拿到数据
     * 后台有个while方法一直调用该方法, 每调用一次就发送一个tuple出去
     */
    public void nextTuple() {
        String line = null;
        try {
        	//一行一行的读取文件内容,并且一行一行的发送
            line = bufferedReader.readLine();
            if (line != null){
				//将信息封装成tuple，发送消息给下一个组件
		        //this.collector.emit(new Value(this.words[index]));

                spoutOutputCollector.emit(Arrays.asList(line));

				//每发送一个消息，休眠500ms
       			// Thread.sleep(500);
				// Utils.sleep(500);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 通过字段声明发出的数据是什么,tuple中的数据的字段名
     * @param outputFieldsDeclarer
     */
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("line"));
    } 
}
```

### 编写Bolt类对出入的内容进行单词切分

```java
/***
 * Description: 输入一行一行的数据
 *              对一行数据进行切割
 *              输出单词及单词出现的次数
 ***
//BaseBasicBolt
public class SplitBolt extends BaseRichBolt {
    private OutputCollector outputCollector;
    /**
     * 初始化方法,只被运行一次
     * @param map 配置文件
     * @param topologyContext 上下文对象
     * @param outputCollector 数据收集器
     */
    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.outputCollector = outputCollector;
    }

    /**
     * 执行业务逻辑的方法
     * @param tuple 获取的上游数据
     */
    @Override
    public void execute(Tuple tuple) {
        //获取上游句子(字段:"line"),从tuple中读取数据
		//获取nextTuple()方法emit()过来的数据	
        String line = tuple.getStringByField("line");
        //对句子进行切割
        String[] words = line.split(" ");
        //发送数据
        for (String word : words) {
            //需要发送单词和单词出现的次数,总共两个字段
            outputCollector.emit(Arrays.asList(word, "1"));
        }
    }

    /**
     * 声明发送出去的数据
     * @param outputFieldsDeclarer
     */
    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word", "num"));
    }
}
```

### 编写Bolt类对单词进行计数

```java
/***
* Description: 负责统计每个单词出现的次数, 类似于hadoop mapreduce的reduce
*              输入单词及单词出现的次数
*              输出打印在控制台
***/
public class WordCountBolt extends BaseRichBolt {
	//定义一个map用于储存单词及其数量
    private Map<String, Integer> wordCountMap = new HashMap<>();

    /**
     * 初始化方法
     * @param map 配置文件
     * @param topologyContext 上下文对象
     * @param outputCollector 数据收集器
     */
    @Override
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        //由于WordCountBolt是最后一个bolt所以不需要提取出OutputCollector
    }

    @Override
    public void execute(Tuple tuple) {
        //获取信息(单词, 数量)
        String word = tuple.getStringByField("word");
        String num = tuple.getStringByField("num");
        //使用map进行记录
        //开始计数
        if (wordCountMap.containsKey(word)){
            //如果map里已经有这个单词,就把数量进行累加
            Integer integer = wordCountMap.get(word);
            wordCountMap.put(word, integer + Integer.parseInt(num));
        }else {
            //如果map里已经没有这个单词,就把单词和数量放入map
            wordCountMap.put(word, Integer.parseInt(num));
        }

        //打印
        System.out.println(wordCountMap);
    }

    @Override
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        //由于不向外发送数据,所以不用写
    }
}
```

### 编写启动类对程序进行整合

```
/***
 * Description: wordcount驱动类,用来提交任务
 ***/
public class WordCountTopology {
    public static void main(String[] args) throws InvalidTopologyException, AuthorizationException, AlreadyAliveException {
        //通过TopologyBuilder 封装任务信息
        TopologyBuilder topologyBuilder = new TopologyBuilder();
```

​     

```java
        //设置spout获取数据
        //SpoutDeclarer setSpout(String id, IRichSpout spout, Number parallelism_hint):参数:自定义id, spout对象, 并发数量 表示用多少个excutor来执行这个组件
		//setNumTasks(8)，设置该组件执行时并发的task数量，也就意味着1个excutor会执行8个task

        topologyBuilder.setSpout("readfilesspout", new ReadFileSpout(), 2);
        //设置splitbolt 对句子进行切割
        topologyBuilder.setBolt("splitbolt", new SplitBolt(), 4).shuffleGrouping("readfilesspout");
        //设置wordcountbolt 对单词进行统计，将bolt设置到topology中，并且指定他接收的消息
        topologyBuilder.setBolt("wordcountbolt", new WordCountBolt(), 2).shuffleGrouping("splitbolt");

        //准备一个配置文件，配置一些topology在集群中运行的参数
        Config config = new Config();

        //启动2个worker!
        config.setNumWorkers(2);

        //任务提交有:本地模式 和 集群模式

        //本地模式
        //LocalCluster localCluster = new LocalCluster();
        //localCluster.submitTopology("wordcount", config, topologyBuilder.createTopology());

        //集群模式,参数:Topology名字, 配置文件, Topology对象
		//用builder来创建topology
        StormSubmitter.submitTopology("wordcount2", config, topologyBuilder.createTopology());
    }
}
```

### 执行程序

1. 选择本地模式运行  
   直接运行驱动类的main方法即可, 统计后的结果会直接打印在控制台
2. 选择上传到集群进行执行  
   首先通过maven的package命名将程序打好jar包  

#### 注：

在storm-core的依赖中加入:<scope>provided</scope>  
在上传到node2或node3上, 在指定路径下要确保存在日志文件

# Storm Distributed RPC（DRPC）

## 分布式远程过程调用

- DRPC的主要作用就是利用Storm的**实时计算**能力来**并行化**CPU intensive的计算。  
- 对于每一次函数调用，Storm topology将函数的参数当成是输入流，并且将函数运行的结果作为输出流。  
- DRPC其实不能算是storm本身的一个特性，它是通过组合storm的**原语**spout，bolt，topology而成的一种模式(pattern)。  
- DRPC通过一个"DRPC server"来进行**协调均衡**。（Storm整合了DRPC server的一个实现）。  
- DRPC server接受一个RPC请求，发送该请求给Storm topology，接受该Storm topology产生的结果，并把结果返回给客户端。  
- 对于客户端来说，一次DRPC调用就像是一次正常的RPC调用一样。

### 客户端使用DRPC来获取以"http://baidu.com"为参数的"reach"函数的返回结果：

```java
DRPCClient client = new DRPCClient("drpc-host", 3772);
String result = client.execute("reach", "http://baidu.com");
```

![mark](https://img.jinguo.tech/blog/20200116/rPnjrFMNdD6e.png?imageslim)  

#### 1. 客户端将要执行的函数名以及相应的参数发送给DRPC server 。实现了这个函数的topology使用  

#### 2. DRPCSpout来接收从DRPC server传来的函数的远程调用流，从而来执行该函数。

#### 3. 每一次函数的远程调用都被DRPC server附上了一个唯一的id。

#### 4. 接下来topology计算结果，在最后topology中的bolt调用ReturnResults来连接DRPC server并将结果及相应的函数远程调用id返回给DRPC server。

#### 5. 接下来DRPC server通过id来匹配相应的客户端，此时客户端还处于等待状态，匹配上后，疏通等待状态的客户端，并开始将结果发送给客户端。

## LinearDRPCTopologyBuilder（线性DRPCTopologyBuilder）

### Storm中有个LinearDRPCTopologyBuilder，实现了几乎所以DRPC步骤的自动化,这些步骤如下:

1. 建立 spout
2. 将结果返回到DRPC server
3. 向bolts提供了在tuples集合上进行有限聚集的功能

#### 创建LinearDRPCTopologyBuilder

```java
public static class ExclaimBolt extends BaseBasicBolt {
    public void execute(Tuple tuple, BasicOutputCollector collector) {
        String input = tuple.getString(1);
		//简单的在元组的第二个字段的值后加了一个"!"
        collector.emit(new Values(tuple.getValue(0), input + "!"));
    }
```

​		

```java
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("id", "result"));
    }
}

public static void main(String[] args) throws Exception {
  //我们将DRPC函数名告诉给topology（本例函数名为exclamation）。	
  //单个DRPC server可以负责处理多个函数，函数之间通过函数名来进行区分。
  //第一个bolt的输入是一个2元组，第一个字段为request id，第二个字段为request对应的参数。
    LinearDRPCTopologyBuilder builder = new LinearDRPCTopologyBuilder("exclamation");
    builder.addBolt(new ExclaimBolt(), 3);
}
```

#### 创建Local mode DRPC

```java
//首先创建一个LocalDRPC对象。该对象将会在进程中模拟一个DRPC server。
LocalDRPC drpc = new LocalDRPC();
//然后创建LocalCluster来以本地模式来运行该topology。
LocalCluster cluster = new LocalCluster();
//LinearDRPCTopologyBuilder有单独的方法来创建本地的topologies以及远程的topologies。
//在本地模式中，LocalDRPC对象不会绑定到任何端口，所以，topology需要知道与其通信的对象
//（即将drpc作为参数传入：builder. createLocalTopology(drpc)）;
cluster.submitTopology("drpc-demo", conf, builder.createLocalTopology(drpc));
//在建立了topology后，我们可以使用LocalDRPC的execute进行DRPC远程调用。
System.out.println("Results for 'hello':" + drpc.execute("exclamation", "hello"));

cluster.shutdown();
drpc.shutdown();
```

#### Remote mode DRPC

1. 建立DRPC servers
2. 配置DRPC servers的位置
3. 向Storm cluster提交DRPC topologies,可用storm脚本建立DRPC server：

##### 1. 用storm脚本建立DRPC server：

```java
bin/storm drpc
```

##### 2. 配置DRPC servers位置，通过storm.yaml来进行配置或者在topology程序中进行配置

```java
drpc.servers:
  - "drpc1.foo.com"
  - "drpc2.foo.com"
```

##### 3. 通过StormSubmitter建立DRPC topologies

```java
StormSubmitter.submitTopology("exclamation-drpc", conf, builder.createRemoteTopology());
```

## Storm DRPC深入

分布式dRPC（distributed RPC，DRPC）用于对Storm上大量的**函数调用**进行**并行计算**。对于每一次函数调用，Storm集群上运行的拓扑接收调用函数的参数信息作为输入流，并将计算结果作为输出流发射出去。  
可概括为：Storm进行计算，根据客户端提交的请求参数，而返回Storm计算的结果。 

### 注：

Storm是一个流式计算框架，数据源源不断的产生，收集，计算。（数据实时产生、实时传输、实时计算、实时展示）  
Storm只负责数据的计算，不负责数据的存储     
2013年前后，阿里巴巴基于storm框架，使用java语言开发了类似的流式计算框架佳作，Jstorm。2016年年底阿里巴巴将源码贡献给了Apache storm，两个项目开始合并，新的项目名字叫做storm2.x 

![mark](https://img.jinguo.tech/blog/20200116/RcA2os1xUqfW.png?imageslim)
**其中:**
Nimbus：负责资源分配和任务调度。  
Supervisor：负责接受nimbus分配的任务，启动和停止属于自己管理的worker进程。  
Worker：运行具体处理组件逻辑的进程。  
Task：worker中每一个spout/bolt的线程称为一个task. 在storm0.8之后，task不再与物理线程对应，同一个spout/bolt的task可能会共享一个物理线程，该线程称为executor。  
![mark](https://img.jinguo.tech/blog/20200116/nC49B0GwGz0o.png?imageslim)

### 注：

**DataSource**: 数据源  
**Spout**：在一个topology中产生源数据流的组件。通常情况下spout会从外部数据源中读取数据，然后转换为topology内部的源数据。Spout是一个主动的角色，其接口中有个nextTuple()函数，storm框架会不停地调用此函数，用户只要在其中生成源数据即可。   
**Bolt**：在一个topology中接受数据然后执行处理的组件。Bolt可以执行过滤、函数操作、合并、写数据库等任何操作。Bolt是一个被动的角色，其接口中有个execute(Tuple input)函数,在接受到消息后会调用此函数，用户可以在其中执行自己想要的操作。   
**Tuple**：一次消息传递的基本单元。本来应该是一个key-value的map，但是由于各个组件间传递的tuple的字段名称已经事先定义好，所以tuple中只要按序填入各个value就行了，所以就是一个value list.  
**Stream**：源源不断传递的tuple就组成了stream。  
**Topology**：Storm中运行的一个实时应用程序，因为各个组件间的消息流动形成逻辑上的一个拓扑结构。

### 分组策略

1. 随机分组(Shuffle grouping)：随机分发tuple到Bolt的任务，保证每个任务获得相等数量的tuple。 跨服务器通信，浪费网络资源，尽量不适用
2. 字段分组(Fields grouping)：根据指定字段分割数据流，并分组。例如，根据“user-id”字段，相同“user-id”的元组总是分发到同一个任务，不同“user-id”的元组可能分发到不同的任务。 跨服务器，除非有必要，才使用这种方式。
3. LocalOrShuffle 分组。 优先将数据发送到本地的Task，节约网络通信的资源。



## zookeeper安装和使用

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。  
ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。  
ZooKeeper的目标就是封装好复杂易出错的关键服务，将简单易用的接口和性能高效、功能稳定的系统提供给用户。 ZooKeeper包含一个简单的原语集，提供Java和C的接口。    

### zoo_sample.cfg文件配置

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=D:\\DevelopSoftware\\zookeeper\\zookeeper-3.4.14\\data
dataLogDir=D:\\DevelopSoftware\\zookeeper\\zookeeper-3.4.14\\log
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
```

### 参数解释

- tickTime：这个时间是作为 Zookeeper 服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个 tickTime 时间就会发送一个心跳。
- initLimit：这个配置项是用来配置 Zookeeper 接受客户端（这里所说的客户端不是用户连接 Zookeeper 服务器的客户端，而是 Zookeeper 服务器集群中连接到 Leader 的 Follower 服务器）初始化连接时最长能忍受多少个心跳时间间隔数。当已经超过 10 个心跳的时间（也就是 tickTime）长度后 Zookeeper 服务器还没有收到客户端的返回信息，那么表明这个客户端连接失败。总的时间长度就是 10*2000=20 秒
- syncLimit：这个配置项标识 Leader 与 Follower 之间发送消息，请求和应答时间长度，最长不能超过多少个 tickTime 的时间长度，总的时间长度就是 5*2000=10 秒
- dataDir：顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存在这个目录里。
- clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户端的访问请求。

### 异常

Socket error occurred: localhost/0:0:0:0:0:0:0:1:2181: Connection refused: no further information  
**解决办法：**将conf下的zoo_sample.cfg文件改成zoo.cfg文件。zkServer启动的时候要找到的zool.cfg而实际上在conf文件夹下面却是zoo_sample.cfg

zookeeper启动成功
![mark](https://img.jinguo.tech/blog/20200116/OVbwuEhqSMG5.png?imageslim)

## Zookeeper伪分布式集群搭建

1. 将Zookeeper解压后，复制三份，分别起名为8001,8002,8003，放到同一个目录中如zk-cluster。   
2. 创建zk-data文件夹，在zk-data中新建8001,8002,8003文件夹。在每个文件夹下都创建data,log文件夹。
3. 在上面创建的data目录下，创建myid文件，文件名就是myid，没有后缀，然后8001下的文件内容为1,8002下的myid内容为2，8003下的myid内容为3.  
4. 修改zk-cluster中8001、8002、8003 下conf目录中的配置文件zoo.cfg ,下面是我8001下的zoo.cfg ,其中和8002，8003略作修改

### zoo.cfg文件如下

```properties
# The number of milliseconds of each tick
# 服务器与客户端之间交互的基本时间单元（ms）
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
# zookeeper所能接受的客户端数量
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
# 服务器与客户端之间请求和应答的时间间隔
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
# 保存zookeeper数据，日志路径
dataDir=D:/DevelopSoftware/zookeeper/zk-data/8001/data
dataLogDir=D:/DevelopSoftware/zookeeper/zk-data/8001/log
# the port at which the clients will connect
# 这是客户端链接的端口号
clientPort=2181										
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

# Cluster Zookeeper Server Address 下面配置不需要修改 要注意的就是，下面server.number (number是1、2、3)分别对应myid中的内容，zookeeper也是通过server后面的数字以及dataDir下的myid内容来判断zookeeper集群的关系的（哪个server对应哪个地址），然后后面两个端口号，一个是跟服务器发送链接的端口，另一个是接受服务器链接的端口
# server.A=B:C:D  其中A是一个数字，代表这是第几号服务器；B是服务器的IP地址；C表示服务器与群集中的“领导者”交换信息的端口；当领导者失效后，D表示用来执行选举时服务器相互通信的端口。
# 客户端与zookeeper相互交互的端口
server.1=127.0.0.1:8001:9001
server.2=127.0.0.1:8002:9002
server.3=127.0.0.1:8003:9003
```

### 报错

```java
 [myid:1] - WARN  [WorkerSender[myid=1]:QuorumCnxManager@584] - Cannot open channel to 3 at election address /127.0.0.1:9003
java.net.ConnectException: Connection refused: connect
        at java.net.DualStackPlainSocketImpl.waitForConnect(Native Method)
        at java.net.DualStackPlainSocketImpl.socketConnect(DualStackPlainSocketImpl.java:85)
        at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350)
        at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206)
        at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188)
        at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:172)
        at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392)
        at java.net.Socket.connect(Socket.java:589)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.connectOne(QuorumCnxManager.java:558)
        at org.apache.zookeeper.server.quorum.QuorumCnxManager.toSend(QuorumCnxManager.java:534)
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.process(FastLeaderElection.java:454)
        at org.apache.zookeeper.server.quorum.FastLeaderElection$Messenger$WorkerSender.run(FastLeaderElection.java:435)
        at java.lang.Thread.run(Thread.java:745)
```

### 报错解决办法

产生上述Waring信息是因为zookeeper服务的每个实例都拥有全局的配置信息，他们在启动的时候需要随时随地的进行leader选举，此时server1就需要和其他两个zookeeper实例进行通信，但是，另外两个zookeeper实例还没有启动起来，因此将会产生上述所示的提示信息。当我们用同样的方式启动server2和server3后就不会再有这样的警告信息了。
