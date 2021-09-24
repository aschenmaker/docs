# kafka in action

## 1异步通信

### 1.1观察者模式

**发布-订阅模式**

定义一种一对多的关系，如果一个对象改变了状态，那么依赖他的对象都会得到通知和自动更新

### 1.2生产者消费者模式

- **传统模式**	
  - 生产者直接讲消息传递给指定的消费者
  - 耦合性较高，生产者和消费者逻辑产生了变化都需要修改业务逻辑
  - 生产者和消费者的不一定是 1:1 的生产消费的配比
- **引入消息中间件**
  - 通过一个中间的阻塞队列来进行通讯，实现解耦
    - 生产者 ---> 消息中间件 --> 消费者
- 数据传递流程
  - 可以N个生产者，也可以有N个消费者
  - 生产者向中间件中添加数据
  - 消费者从中间件取出数据， 一般情况下先进先出FIFO的原则

### 1.3消息中间件的作用

- 解耦
- 支持并发
  - 生产者可以持续进行生产
- 削峰
  - 高峰期产生的数据可以进行挤压

### 1.4数据单元

- 关联到业务对象
  - 数据单元必须关联到某一种数据对象
- 完整性
  - 传输过程中需要保证数据的完整性
- 独立性
  - 数据单元之间没有相互的依赖
  - 某个数据单元没有传输完成，不会影响到其他的数据单元
- 颗粒度
  - 颗粒度过小，传输的次数变多
  - 颗粒度过大增加了传输的时间，以及后期消费速度

## 2消息传递原理

消息传递系统负责将数据从一个应用传递到另外一个应用，应用只需要关注数据，无需关注数据在两个和多个应用中是如何传递的。

### 2.1点对点传递

- 消息持久化道一个队列中，有一个或者多个消费者消费队列的数据。但是只能被消费一次。
- 某条数据被消费后应该从队列中删除
- 即使有多个消费者，也需要保证数据的顺序处理
- 在这种模式中由 `消费代理` 来记录消费状态， 但是这种模式无法很好保证消费的处理语义，他是不是真的成功被消费了？

### 2.2发布订阅消息传递

- 消息被持久化道一个topic中
- 消费者可以订阅一种或多种topic，消费者可以消费topic的数据，同一条数据可以被多个消费者消费，数据消费后也不会立马删除
  - 被多个消费者消费，可以理解为消息需要作用多个应用中去。
- kafka采用 `拉取模型`， 由消费者控制消费速度，和消费的进度，消费者可以按照任意格式的偏移量进行消费。

## 3kafka

kafka是一个高吞吐量的分布式消息发布订阅消息系统

### 3.1设计目标

- 时间复杂度为O(1)的方式提供消息持久化的能力
- 高吞吐率
- 支持kafka server之间的消息分区，分布式消费，保证每一个partition内的消息顺序传输
- 支持在线水平扩展
- 同时支持离线数据处理和 **实时数据** 处理

### 3.2优点

- 解耦
- 冗余
  - 持久化数据，直到被完全消费
- 拓展性
- 灵活性和峰值处理
- 可恢复行
- 顺序保证
- 缓冲

## 4kafka系统架构

![kafka架构](https://dist.lyneee.com/blog/2021-09-24-kafka-arch.png)

![kafka2](https://dist.lyneee.com/blog/2021-09-24-kafka-arch2.jpg)

### 4.1broker

kafka集群中包含一个或者多个服务器，每一个服务器节点叫做broker

### 4.2topic

- 消息有不同的类别。被称为topic
- 类似于数据库中表名和ES的index
- 物理上不同的topic分开存储
- 逻辑上一个topic的消息虽然保存在一个或者多个broker上但是用户只需要指定消息的topic就可以进行消费

### 4.3partition 分区

- 一个topic中的数据分割为一个或者多个partition
- 生产者产生数据的时候，根据分配策略，选择分区，然后将消息追加到对应分区的尾端
- 每条消息都有自增的编号
  - 标识顺序
  - 标识消息的偏移量
- 每一个partition使用多个segment文件进行存储
- 单个partition中的数据是有序的
- 如果需要保证数据的完全顺序，那么需要讲partition设置为1

### 4.4leader

每个partition有多个副本，其中一个作为leader，leader是当前负责数据读写的partition

### 4.5follower

- folloer跟随leader，所有写请求都通过leader路由，数据变更，都会广播给所有的follower，follower与leader保持一致
- 如果leader失效，则会从follower总选举产生一个新的leader
- 如果follower挂掉，卡住，或者同步太慢。leader会把这个follower从‘In sync replicas’（ISR）删除，重新创建一个新的follower

### 4.6replication

- partition可能产生损坏
- 我们需要对分区数据进行备份
- 将分区划分为一个Leader和多个follower
  - leader负责写入和读取数据
  - follower负责备份数据
  - 保证了数据的一致性
- 备份数，设置为N，则表示主+备=N

### 4.7producer 

- 生产者讲消息发送到topic中
- broker接收到生产者发送的消息后，将消息追加到用于追加数据的segment中
- 生产者发送的消息，存储到一个partition中，生产者可以指定数据存储的partition 

### 4.8consumer

消费者可以从broker中读取数据，消费者可以消费多个topic中的数据

### 4.9consumer group

- 每一个consumer属于一个特定的consumer group
- 多个消费者集中处理某一个topic的数据，提高消费能力
- 整个消费者组共享一组偏移量，防止数据被重复读，因为一个topic有多个分区

### 4.10offset

- 可以唯一标识的一条数据
- 偏移量决定了数据读取的位置，不会有线程安全的问题，消费者通过偏移量来决定下次读取的位置
- 消息被消费之后，并不会立马删除，这样多个业务可以重复使用kafka的消息
- 可以修改偏移量达到重复读区的数据的目的
- 消息最终会被删除，默认一周

### 4.11zookeeper

- kafka通过zookeeper 来存储集群的meta信息

zookeeper 谁的事务领先就选择谁来作为新的主节点

## 5数据检索机制

- topic在物理层面以partition作为分组，一个topic可以分成若干个partition

- partition还可以细分为Segment，一个partition在物理上有多个segment组成

  - segment的参数主要有两个：
    - log.segment.bytes：单个segment最大可以容纳的数据量，默认1GB
    - log.segment.ms: kafaka在commit一个没写满的segment，等待时间是7天

- LogSegment文件由两部分组成，分别是.index 和.log文件，分别表示segment索引文件和数据文件。

  - partition全局的第一个segment从0开始，后续每个segment文件名为上一个segment文件最后一条消息的offset值
  - 数值大小为64位，20个字符长度，没有数字的用0填充

  ```shell
  # the first segment
  00000000000000000000.index
  00000000000000000000.log
  # the second segment
  00000000000000170410.index
  00000000000000170410.log
  ```


## 6数据安全性

三种模式

- 0， at least once， 消息绝对不会丢，但是可能会重复传输
- 1，at most once，消息绝对可能会丢，但是绝对不会再次传输
- 2，exactly once，每条消息肯定会被传输一次且仅传输一次

###  6.1Producer deliver guarantee

acks：

1. 在isr中的leader 确认后发送一批消息
2. 无需等待broker的回应，发送下一批消息
3. 需要等待所有的follower都接受后才算发送完成，可靠性最高。

### 6.2 ISR机制 In Sync Replica

 目的是确保数据的一致性，快速的选择新的主节点，来进行数据传输。

- 关键词
  - AR Assigned Replicas 用来标识副本的全集
  - OSR out-sync Replicas 离开同步队列的副本
  - ISR in-sync Replicas 在同步队列中的副本
    - ISR = Leader+没有落后太多的follower
    - AR = ISR+OSR
- 数据流通过程
  - Producer -- push --> leader
  - leader <--pull -- follower
  - follower 间隔时间从leader 拉取数据，保持数据同步
- ISR
  - 当主节点挂掉，从ISR中去选择主节点
  - 判断是否在ISR中的标准
    - 超过10s没有同步数据
      - replica.lag.time.max.ms =  10000
    -  和leader差4000条数据
      - replica.lag.max.messages = 4000
  - 脏节点选举
    - kafka的一种降级措施
    - 选择第一个恢复的节点作为leader进行服务，并以它的数据作为基准

###  6.3Broker数据存储机制

无论消息是否被消费，kafka都会保留所有的消息，在消息策略上有两种可以选择的 

1. 基于时间：log.retention.hours=168 # 7days
2. 基于大小: log.retention.bytes=1073741824 # 10GB

### 6.4消费者数据消费保证

- 如果consumer设置为autocommit， consumer一旦读取到数据就进行提交
- 读完消息先commit，在进行处理
  - 如果处理失败，则无法重新再读取到这个数据。 # **漏读**
  - 这对应了at most once
- 读完消息先处理再commit
  - 如如果处理完之后还没来得及commit，consumer崩溃了，会导致consumer重启后，会把该消息重复处理一次。 # **重复读**
  - 这对应了at least once
- 一定要做到exactly once的，需要协调offset和实际操作的输出
  - 两阶段提交

## 7kafka优化

### 7.1partition数目

- 一般来说，每一个partition的吞吐量为 几MB/s，增加更多的partition意味着

  - 更高的并行和吞吐
  - 拓展同一个消费者组的消费者数目
  - 如果有很多限制的brokers，则可以利用到brokers
  - 造成更多的zookeeper选举
  - kafka打开更多的文件

- 调整策略

  - 如果集群较小，则配置2*broker数量的partition数目
  - 如果集群较大，则配置1*broker数量的partition
  - 考虑最高峰需要吞吐的并行consumer数量，调整partition数量，如果应用场景需要20个consumer进行消费，那么设置成20个partition

### 7.2replication factor

决定了副本的数目，更多的副本意味着

- 系统更稳定
- 更多的延迟
- 磁盘使用率会更高

调整：

- 不要在生产环节中将RF设置为1
- 如果replication性能成为了瓶颈，使用更好的broker，而不是降低RF的数量

### 7.3批量写入

为了提高吞吐量，需要定期的去批量的写文件
