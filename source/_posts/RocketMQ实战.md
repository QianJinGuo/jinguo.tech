# RocketMQ实战

## 1. RocketMQ 是什么

#### RocketMQ是一款[^低延迟的]、[^高可靠的]、[^可伸缩的]、易于使用的消息中间件

[^低延迟的]:响应时间低，比如一个网页在几秒内打开，越短表示延迟越低
[^高可靠的]: 指的是运行时间能够满足预计时间的一个系统或组件
[^可伸缩的]:      可伸缩性是高性能、低成本和可维护性等多因素的综合考量和平衡



####  RocketMQ具有以下特性

* 支持发布/订阅（Pub/Sub）和点对点（P2P）消息模型
* 在一个队列中可靠的先进先出（FIFO）和严格的顺序传递
* 支持拉（pull）和推（push）两种消息模式
* 单一队列百万消息的堆积能力
* 支持多种消息协议，如 JMS、MQTT 等
* 分布式高可用的部署架构,满足至少一次消息传递语义
* 提供 docker 镜像用于隔离测试和云集群部署
* 提供配置、指标和监控等功能丰富的 Dashboard



> ###  Producer
>
> **消息生产者**，生产者的作用就是将消息发送到 MQ。生产者本身既可以产生消息，如读取文本信息等，也可以对外提供接口，由外部应用调用接口传递消息，再由生产者将收到的消息发送到 MQ。
>
> ### Producer Group
>
> **生产者组**，就是多个发送同一类消息的生产者称之为一个生产者组。



> ### Consumer
>
> **消息消费者**，消费 MQ 上的消息的应用程序就是消费者，至于消息是否进行逻辑处理，还是直接存储到数据库等取决于业务需要。
>
> ### Consumer Group
>
> **消费者组**，消费同一类消息的多个 consumer 实例组成一个消费者组。



>### Topic
>
>*Topic* 是一种消息的逻辑分类。比如说有订单类的消息，也有库存类的消息，那么就需要进行分类，一个是订单 Topic 存放订单相关的消息，一个是库存 Topic 存储库存相关的消息。以此类推
>
>### Message
>
>*Message*是消息的载体。一个 Message 必须指定 topic，相当于寄信的地址。Message 还有一个可选的 tag 设置，以便消费端可以基于 tag 进行过滤消息。也可以添加额外的键值对，例如需要一个业务 key 来查找 broker 上的消息，方便在开发过程中诊断问题。
>
>



> ### Tag
>
> *Tag***标签**可以被认为是对Topic进一步细化。一般在相同业务模块中通过标签来标记不同用途的消息
>
> ### Broker
>
> *Broker*是RocketMQ系统的主要角色，即MQ。Broker接收来自生产者的消息，储存，以及为消费者拉取消息的请求做好准备
>
> ### Name Server
>
> *Name Server* 为 producer 和 consumer 提供路由信息。



### RocketMQ架构

![mark](https://img.jinguo.tech/blog/20191112/nLTqL4TXdKFH.webp)

#### 由这张图可以看到有四个集群，分别是 NameServer 集群、Broker 集群、Producer 集群和 Consumer 集群：

1. *NameServer*: 

   提供轻量级的服务发现和路由。 每个 NameServer 记录完整的路由信息，提供等效的读写服务，并支持快速存储扩展。

2. *Broker*: 

   通过提供轻量级的 Topic 和 Queue 机制来处理消息存储,同时支持推（push）和拉（pull）模式以及主从结构的容错机制。

3. *Producer*：

   生产者，产生消息的实例，拥有相同 Producer Group 的 Producer 组成一个集群。

4. *Consumer*：

   消费者，接收消息进行消费的实例，拥有相同 Consumer Group 的Consumer 组成一个集群。

   

#### 图中结构关系

图中箭头含义，从 Broker 开始，Broker Master1 和 Broker Slave1 是主从结构，它们之间会进行数据同步，即 Date Sync。

同时每个 Broker 与NameServer 集群中的所有节 点建立长连接，定时注册 Topic 信息到所有 NameServer 中。

