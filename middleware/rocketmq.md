# RocketMQ

## 基本概念

### 消息模型

- 由 Producer、Broker、Consumer 三部分组成，其中Producer 负责生产消息，Consumer 负责消费消息，Broker 负责存储消息。

### Producer

- 负责生产消息，一般由业务系统负责生产消息，发送到broker。分为同步发送、异步发送、顺序发送、单向发送。

### Consumer

- 负责消费消息，一般是后台系统负责异步消费。两种消费形式：拉取式消费、推动式消费。

### Topic

- 表示一类消息的集合，是RocketMQ进行消息订阅的基本单位。

### Broker Server

- 消息中转角色，负责存储消息、转发消息。

### Name Server

- 充当路由消息的提供者。

### Pull Consumer

- 从Broker服务器拉消息、主动权由应用控制。

### Push Consumer

- Broker收到数据后会主动推送给消费端，实时性较高。

### Producer Group

- 发送同一类消息且发送逻辑一致的producer的集合

### Consumer Group

- Clustering

	- 集群消费模式下,相同Consumer Group的每个Consumer实例平均分摊消息。

- Broadcasting

	- 广播消费模式下，相同Consumer Group的每个Consumer实例都接收全量的消息。

### Normal Ordered Message

- 消费者通过同一个消费队列收到的消息是有顺序的，不同消息队列收到的消息则可能是无序的。

### Strictly Ordered Message

- 消费者收到的所有消息均是有顺序的。

### Message

- 消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。

### Tag

- 为消息设置的标志，用于同一主题下区分不同类型的消息。

## 特性

### 订阅与发布

- 消息的发布是指某个生产者向某个topic发送消息；消息的订阅是指某个消费者关注了某个topic中带有某些tag的消息，进而从该topic消费数据。

### 消息顺序

- 全局顺序消息

	- 某个Topic下的所有消息都要保证顺序，按照严格的先入先出（FIFO）的顺序进行发布和消费

- 分区顺序消息

	- 对于指定的一个 Topic，所有消息根据 sharding key 进行区块分区。 同一个分区内的消息按照严格的 FIFO 顺序进行发布和消费

### 消息过滤

- 可以根据Tag进行消息过滤，也支持自定义属性过滤。在Broker端实现。

### 消息可靠性

- 硬件资源可立即恢复情况，RocketMQ在这种情况下能保证消息不丢。单点故障无法恢复时，消息会全部丢失。

### At least Once

- 每个消息必须消费一次

### 回溯消费

- 已经消费成功的消息，由于某些业务上需求需要重新消费。RocketMQ的broker支持按照时间回溯消费，时间维度精确到毫秒。

### 事务消息

- 应用本地事务和发送消息操作可以被定义到全局事务中，要么同时成功，要么同时失败。

### 定时消息（延迟队列）

- 定时消息会暂存在名为SCHEDULE_TOPIC_XXXX的topic中，并根据delayTimeLevel存入特定的queue，等待特定时间投递给真正的topic

### 消息重试

- Consumer消费消息失败后，要提供一种重试机制，令消息再消费一次

### 消息重投

- 生产者在发送消息时，同步消息失败会重投，异步消息有重试，oneway没有任何保证。

### 流量控制

- 生产者流控，因为broker处理能力达到瓶颈；消费者流控，因为消费能力达到瓶颈。

### 死信队列

- 正常情况下无法被消费的消息称为死信消息（Dead-Letter Message），将存储死信消息的特殊队列称为死信队列（Dead-Letter Queue）

## 架构

### 技术架构

- Producer

	- 消息发布的角色，支持分布式集群方式部署

- Consumer

	- 消息消费的角色，支持分布式集群方式部署

- NameServer

	- 简单的Topic路由注册中心，支持Broker的动态注册与发现，主要功能为Broker管理和路由信息管理

- BrokerServer

	- 负责消息的存储、投递和查询以及服务高可用保证

		- Remoting Module
		- Client Manager
		- Store Service
		- HA Service
		- Index Service

### 部署架构

- https://github.com/apache/rocketmq/blob/master/docs/cn/architecture.md
附上文档链接

## 设计
https://blog.csdn.net/selt791/article/details/104575265

### 消息存储

- 消息存储整体架构

	- CommitLog

		- 消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容,

	- ConsumeQueue

		- 消息消费队列，可以看成是基于topic的commitlog索引文件，存储路径为：$HOME/store/consumequeue/{topic}/{queueId}/{fileName}

	- IndexFile

		- 提供了一种可以通过key或时间区间来查询消息的方法。Index文件的存储位置是：$HOME \store\index${fileName}，

- 页缓存与内存映射

	- RocketMQ主要通过MappedByteBuffer对文件进行读写操作

- 消息刷盘

	- 同步刷盘

		- 可靠性来说是一种不错的保障，但是性能上会有较大影响，适用于金融业务

	- 异步刷盘

		- 采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量

### 通信机制

- 基本通讯流程
- rocketmq-remoting

	- RocketMQ消息队列中负责网络通信的模块，基于Netty

- RemotingCommand

	- 这个类在消息传输过程中对所有数据内容的封装，不但包含了所有的数据结构，还包含了编码解码操作

- 异步通信流程
- Reactor多线程设计

### 消息过滤

- Tag过滤方式
- SQL92的过滤方式

### 负载均衡

- Producer的负载均衡
- Consumer的负载均衡

### 事务消息

- 采用了2PC的思想来实现了提交事务消息，同时增加一个补偿逻辑来处理二阶段超时或者失败的消息

	- 事务消息在一阶段对用户不可见
	- 子主题 2
	- 子主题 3

### 消息查询

- 按照MessageId查询消息

	- Client端从MessageId中解析出Broker的地址（IP地址和端口）和Commit Log的偏移地址后封装成一个RPC请求后通过Remoting通信层发送

- 按照Message Key查询消息

	- 基于RocketMQ的IndexFile索引文件来实现的

参考文章（逻辑清晰）：http://www.dockone.io/article/9726
