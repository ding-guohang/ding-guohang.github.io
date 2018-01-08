---
title: 'Kafka Note One: 初次见面, 请多关照'
date: 2017-12-23 21:03:54
updated: 2017-12-24 02:22:22
tags:
- MQ
- Kafka
categories:
- 笔记
comments: true
---

*这是我对于kafka的学习过程中的一些笔记，可能大多数的信息来源于官方网站的翻译，以及网上的其他blog。*
*也许会参杂着一些自己的思考和认知，如果你需要一个入门、基础知识的快速阅览，也许会适合你。*
*resources: [infoQ](http://www.infoq.com/cn/articles/kafka-analysis-part-1/) & [kafka官网](http://kafka.apache.org/)*
<!--more-->


# Topic
* Topic, 也可以被成为subject, 发布-订阅模型中的消息主题
* 不同的Topic在物理上分开存储


# Partition

{% cq %} For each topic, the Kafka cluster maintains a partitioned log {% endcq %}

* 一种物理上的概念, 每个Partition在物理上对应一个文件夹，文件夹下存放的是这个Partition的所有消息和索引文件
* Partition内的消息是有序的
* 这更像是一个日志文件，和对应的多个索引

## 一个Topic，被分为多个Partition
![anatomy of topic](/img/anatomy of topic.png)
* 减小日志文件大小，让服务器可以承担
* 多个partition可以被并行消费

## 一个Partition里的日志文件，被分为多个Segment
![](/img/partition.png)
* 每个日志，或者说每个消息，都具备自己在partition中的一个64byte的唯一offset，而且每个partition是有序的
* 每个Segment以自己包含的第一条消息的offset + ".kafka" 命名
* 有一个索引文件，表明了每个segment下包含的log的offset范围
```
日志的组成
message length ： 4 bytes (value: 1+4+n)
"magic" value ： 1 byte 
CRC 校验码 ： 4 bytes 
消息内容 ： n bytes 
```

## 消息的读取是O(1)的（这句话待求证）
{% cq %} Kafka's performance is effectively constant with respect to data size so storing data for a long time is not a problem {% endcq %}

* Kafka会保留所有的消息，与是否被消费无关
* 可以指定删除旧数据的策略， 时间/partition文件大小
* 文件的大小和多少，不会影响kafka的性能

![](/img/consumer and provider.png)

## Consumer保存自己消费的offset，consumer之间互不影响
* 因此kafka的broker是无状态的，也不需要锁
* consumer可以通过修改自己的offset达到重新消费消息的效果
* consumer去broker pull消息


# Producer
* producer 选择自己的每一条消息记录到哪个partition中
* 最简单的方式就是循环，可以保证负载均衡
* 也可以通过某些消息中的某些元素自定义


# Consumer

{% cq %} The way consumption is implemented in Kafka is by dividing up the partitions in the log over the consumer instances so that each instance is the exclusive consumer of a "fair share" of partitions at any point in time {% endcq %}

![](/img/consumer-groups.png)

* consumer有group的概念，一个group的所有实例，一起消费相同的topic
* 每个消息只会被交付给consumer group中的一个实例
* kafka 关于这个的实现方式是将 partition划分到消费者实例上，使得在任何时间点上，每个实例都是将partitions公平分享的独占消费者
* 关于这一点，newQMQ就有刻意的区别实现，做了动态的关联
* 这个对应关系，是由kafka的协议动态维护的，所以任何消费者实例的变化，都会导致对应关系的变化
* 不需要区分订阅或是广播，所有消息都是订阅，如果每个consumer有独一无二的group，就是广播
* kafka只保证每个partition有序，不保证多个partition的顺序，所以整体来说，其实是不保证消息有序的，如果想要整体顺序消费，那么只能通过一个partitoon的方式实现，这样带来的后果是也只会有一个consumer消费消息
* 关于这一点，newQMQ也有做一些特殊的处理，如果真的想要强顺序的话，貌似是可以实现的，虽然不知道效果是不是和这个一样


# Fault Tolerance
* partition 分布在kafka的集群上，每个server处理数据并请求共享分区，每个 partition 被复制在多个(可配)服务器上以实现容错
* 每个 partition 有一个server是leader，0或多个followers
* leader处理读写请求，其他followers被动复制
* leader挂了，某个followers自动变为leader
* 每个server都扮演一些 partition 的leader，和另一些的follower，因此load在集群中很平衡
* 当然，以此推论，如果只有一个server，则独自扮演所有topic的所有partition的leader，并且没有follower
