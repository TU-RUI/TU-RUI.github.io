---
title: 什么是Flink、Kafka Consumer与Kafka Stream
date: 2022-06-20 19:11:23.002
updated: 2022-06-21 14:19:18.254
url: /archives/shen-me-shi-flinkkafkaconsumer-yu-kafkastream
categories: 
- 技术类
tags: 
- Flink
- Kakfa
- 流数据处理
---


# 1. 什么是Flink
Flink是一款分布式的数据计算引擎，它支持流式数据处理，即实时地消费和处理一些数据流，实时地产生数据的结果（例如实时消费用户行为数据，每分钟产出一次最近10分钟用户的点击率），同时也支持批处理，即处理静态的数据集、历史的数据集，也可以用来做一些复杂事件处理ECP，例如风控场景可以实时检测登录之后的用户行为，来避免盗号风险。  总而言之，Flink是基于流式数据(Streams)的有状态的(Stateful)计算引擎。这里有两个关键词Streams和Stateful

* **流式数据**，Flink认为有界数据集是无界数据流的一种特例，所以说有界数据集也是一种数据流，事件流也是一种数据流。Everything is streams。
* **有状态**，即有状态计算。有状态计算是最近几年来越来越被用户需求的一个功能。举个例子，一个网站一天内访问UV数，那么这个UV数便为状态。Flink提供了内置的对状态的一致性的处理，即如果任务发生了Failover，其状态不会丢失、不会被多算少算，同时提供了非常高的性能。     

另外Flink受欢迎背后，还离不开其他的很多特性，例如高性能，高可扩展性，支持容错，纯内存计算，支持事件时间和乱序，支持exactly-once语义等等

# 2. 什么是Kafka Stream API

## 2.1 Kafka Consumer API
介绍Kafka Stream API之前，需要先介绍一下Kafka Consumer API，Kafka作为一个消息队列，必然需要写入数据和读取数据，Kakfa官方提供了一套客户端代码帮助我们便捷地进行读取和写入Kafka中数据。这套客户端里包含Kafka Producer和Kafka Consumer。Kafka Consumer API就是Kafka官方提供的对Kafka流中数据进行低级访问和流处理的库，当我们需要消费Kafka内数据时，就可以使用Kafka Consumer API来完成，它包含消费消息，订阅主题，创建消费者，保存和提交偏移等基本功能

## 2.2 Kafka Stream API
使用Kakfa Consumer API，我们能实现基本的消息消费的功能，有时我们需要更加复杂的流式数据处理，例如涉及到一批数据的聚合，不同consumer间数据的协调，容错等功能时，我们往往需要借助其他工具一起使用，例如使用Buffer-Trigger做一批数据的聚合，使用分布式缓存来协调不同的consumer节点，自行实现状态和容错，非常繁琐，为了更好的支持流式数据处理场景，Kafka官方提供了Kafka Stream API，
它建立在Kafka Producer和Consumer之上，提供了一系列比Kafka Consumer更强大也更简单的特性，例如：
* 通过kafka事务机制支持exactly-once语义
* 支持事件时间，处理时间，摄入时间，支持处理乱序数据
* 支持有状态的数据处理，例如流的连接(join)，聚合函数(aggregations)，窗口(windowing)
* 同时支持流式数据(Stream)和表格数据(Table)
* 附带函数式编程风格DSL，支持用于执行复杂事件处理的API(CEP)，甚至可以组合DSL和CEP

# 3 Flink VS Kafka Stream API
## 3.1 主要区别
Kakfa推出Stream API后，支持了事件时间，窗口，聚合函数等等常用流处理中常用功能，流数据处理方面基本已经向Flink靠齐了，下面是一些Kafka和Flink之间最重要的区别

| | Flink | Kafka Stream API |
| --- | --- | --- |
|部署方式 | Flink是一个集群框架，因此应用程序的部署由该框架负责 | Streams API是一个任何标准Java应用程序都可以嵌入的库，因此不会改变你的部署方式，你基本上可以使用任意方式来部署应用程序。|
|代码生命周期 | 流处理代码在Flink集群中作为作业部署和运行 |	用户的流处理代码嵌入在他们的应用程序中运行 |
|作业由谁负责管理 | 一般由数据基础设施或BI团队 | 一般由业务线团队 |
|分布式协调 |	Flink Master（JobManager），流式程序的一部分 | 利用Kafka集群进行协调、负载平衡和容错。 |
|流数据来源 |	Kafka、文件系统、其他消息队列等等 | 严格的Kafka，Kafka中的Connect API服务于解决数据进出Kafka的问题 |
|流数据写入 |	Kafka、其他MQ、文件系统、数据库、K/V存储、流处理器状态和其他外部系统 | Kafka、应用程序状态、操作数据库或任何外部系统 |
|有界和无界数据流 | 无界与有界 | 无界 |
|语义保证 | Flink内部状态恰好一次；使用选定的输入源和输出源时exactly once（例如，Kafka 到 Flink 到 HDFS）；当 Kafka 为输出源时，at least once。将来很可能与 Kafka 端对端Exactly-once |	端到端都是Kafka，Exactly-once |

实时上Flink 和Kafka Stream二者最核心的区别在于Flink和Kafka Stream的部署和管理模式，以及如何协调分布式处理（包括容错）。

## 3.2 部署与管理模式
Flink程序作为独立的作业进行部署和升级，Flink作业可以自行启动和停止，从所有权的角度看，Flink作业的运维通常由拥有框架运行的集群的团队负责，例如数据基础设施团队。

Kafka Stream API是一个标准的库，因此应用程序的生命周期由应用开发人员负责。Streams API并未规定应如何配置、监控或部署应用程序，以及如何与公司现有的打包、部署、监控和运营工具无缝集成。从所有权的角度来看，Streams 应用程序通常由各自的产品团队负责。

## 3.3 分布式协调
另一个区别是Flink有一个专门的主节点进行协调，而Streams API通过Kafka的消费者组协议依赖Kafka broker进行分布式协调和容错。
Flink的容错、扩展、状态的同步都是由主节点(JobManager)进行全局协调的。主节点基于ZooKeeper实现了自己的高可用机制。这种方法可以帮助Flink获得高吞吐量，以及exactly once语义保证，以及Flink的checkPoint功能，并且它支持Flink的exactly-once的输出源（例如HDFS 和 Cassandra，但不包含Kafka)。

Kafka Streams API利用Kafka提供容错、保证连续处理和高可用性。用户的应用程序的每个实例都独立运行，所有协调由kafka broker完成。容错是内置在Kafka协议中的，允许轻量级的集成到用户应用程序中。

# 4 总结
Kafka Streams API使流处理可作为应用程序的一部分运营，由于Kafka Stream API与Kafka中的核心概念紧密集成，应用程序可以利用并受益于Kafka的核心能力——性能、可扩展性、安全性、可靠性 以及端到端恰好一次。另一方面，Flink非常适合能够部署在独立集群中的应用程序，并能让应用程序得到吞吐量，延迟、事件时间语义、保存点和操作特性、应用程序状态的一次性保证、端到端的一次性保证和批处理等功能

---

参考资料  
 https://www.confluent.io/blog/apache-flink-apache-kafka-streams-comparison-guideline-users/  
 https://medium.com/@chandanbaranwal/spark-streaming-vs-flink-vs-storm-vs-kafka-streams-vs-samza-choose-your-stream-processing-91ea3f04675b  
 https://www.baeldung.com/java-kafka-streams-vs-kafka-consumer  
 https://stackoverflow.com/questions/44014975/kafka-consumer-api-vs-streams-api?rq=1  
 https://www.confluent.io/blog/introducing-kafka-streams-stream-processing-made-simple/  
 https://kafka.apache.org/intro  

