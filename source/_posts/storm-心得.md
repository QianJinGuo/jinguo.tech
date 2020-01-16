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

