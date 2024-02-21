# MQ简介

`MQ`，Message Queue，是一种提供消息队列服务的中间件，也称为消息中间件，是一套提供了**消息生**
**产、存储、消费**全过程API的软件系统。消息即数据。一般消息的体量不会很大

![image-20211006220855697](https://cdn.itzhai.com/image-20211006220855697-a.png-itzhai)

# MQ用途

## 限流削峰

+ MQ可以将系统的超量请求暂存其中，以便系统后期可以慢慢进行处理，从而避免了请求的丢失或系统
  被压垮

![1](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211800072.jpg)

![2](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211800682.jpg)

### 实现

```java
@Service
@RocketMQMessageListener(topic = RocketConstant.Topic.PRAISE_TOPIC, consumerGroup = RocketConstant.ConsumerGroup.PRAISE_CONSUMER)
@Slf4j
public class PraiseListener implements RocketMQListener<PraiseRecordVO>, RocketMQPushConsumerLifecycleListener {
    @Resource
    private PraiseRecordService praiseRecordService;

    @Override
    public void onMessage(PraiseRecordVO vo) {
        praiseRecordService.insert(vo.copyProperties(PraiseRecord::new));
    }

    @Override
    public void prepareStart(DefaultMQPushConsumer consumer) {
        // 每次拉取的间隔，单位为毫秒
        consumer.setPullInterval(2000);
        // 设置每次从队列中拉取的消息数为16
        consumer.setPullBatchSize(16);
    }
}
```

在MQ削峰的配置参数里，以下几个DefaultMQPushConsumer的参数是需要注意一下的：

- `pullInterval`：每次从Broker拉取消息的间隔，单位为毫秒
- `pullBatchSize`：每次从Broker队列拉取到的消息数，是从每个队列的拉取数，即Consume每次拉取的消息总数如下： 
- EachPullTotal=所有Broker上的写队列数和(writeQueueNums=readQueueNums) * pullBatchSize
- 例如：PraiseListener中设置了每次拉取的间隔为2s，每次从队列拉取的消息数为16，在搭建了2master broker且broker上writeQueueNums=readQueueNums=4的环境下每次拉取的消息理论数值为`16 * 2 * 4 = 128`

[超级简单的 RocketMQ 流量削峰实战][https://cloud.tencent.com/developer/article/1629610]

## 异步

### 相比如同步通信优点

+ 比如我们有一个购票系统，需求是用户在购买完之后能接收到购买完成的短信

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211800278.webp)

+ 我们省略中间的网络通信时间消耗，假如购票系统处理需要 150ms ，短信系统处理需要 200ms ，那么整个处理流程的时间消耗就是 150ms + 200ms = 350ms。

+ 当然，乍看没什么问题。可是仔细一想你就感觉有点问题，我用户购票在购票系统的时候其实就已经完成了购买，而我现在通过同步调用非要让整个请求拉长时间，而短息系统这玩意又不是很有必要，它仅仅是一个辅助功能增强用户体验感而已。我现在整个调用流程就有点 **头重脚轻** 的感觉了，购票是一个不太耗时的流程，而我现在因为同步调用，非要等待发送短信这个比较耗时的操作才返回结果。那我如果再加一个发送邮件呢？

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211800138.webp)

+ 这样整个系统的调用链又变长了，整个时间就变成了550ms

+ 为了解决这一个问题，聪明的程序员在中间也加了个类似于服务员的中间件——消息队列。这个时候我们就可以把模型给改造

![img](https://pic4.zhimg.com/80/v2-36ed7da79108db6011261efbb043991b_1440w.webp)

+ 这样，我们在将消息存入消息队列之后我们就可以直接返回了(我们告诉服务员我们要吃什么然后玩手机)，所以整个耗时只是 150ms + 10ms = 160ms。

> 但是你需要注意的是，整个流程的时长是没变的，就像你仅仅告诉服务员要吃什么是不会影响到做面的速度的。

[RocketMQ(浅谈异步、解耦、削峰&队列模式和主题模式是什么？怎么用？)][https://zhuanlan.zhihu.com/p/342769820]

## 解耦

+ 上游系统对下游系统的调用若为同步调用，则会大大降低系统的吞吐量与并发度，且系统耦合度太高。
  而异步调用则会解决这些问题。所以两层之间若要实现由同步到异步的转化，一般性做法就是，在这两
  层间添加一个MQ层

![3](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211759025.jpg)

### 使用举例

+ 假如现在两个系统，当B系统crash掉，A系统也就凉了，这就是耦合带来的问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224160221120.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4OTg0MQ==,size_16,color_FFFFFF,t_70)

>  使用MQ后

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200224160437348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mzg4OTg0MQ==,size_16,color_FFFFFF,t_70)

+ 即使B系统crash，A系统照常运行，A只需要把消息给到消息队列就可以

## 数据收集

分布式系统会产生海量级数据流，如：业务日志、监控数据、用户行为等。针对这些数据流进行实时或
批量采集汇总，然后对这些数据流进行大数据分析，这是当前互联网平台的必备技术。通过MQ完成此
类数据收集是最好的选择

[Apache ShenYu 集成 RocketMQ 实时采集海量日志的实践][https://juejin.cn/post/7164658231581605919]

# 常见MQ产品

## ActiveMQ

ActiveMQ是使用Java语言开发一款MQ产品。早期很多公司与项目中都在使用。但现在的社区活跃度已
经很低。现在的项目中已经很少使用了。

## RabbitMQ

RabbitMQ是使用ErLang语言开发的一款MQ产品。其吞吐量较Kafka与RocketMQ要低，且由于其不是
Java语言开发，所以公司内部对其实现定制化开发难度较大。

## Kafka

Kafka是使用Scala/Java语言开发的一款MQ产品。其最大的特点就是高吞吐率，常用于大数据领域的实
时计算、日志采集等场景。其没有遵循任何常见的MQ协议，而是使用自研协议。对于Spring Cloud
Netflix，其仅支持RabbitMQ与Kafka

### 适用场景

Kafka主要特点是基于Pull的模式来处理消息消费，追求高吞吐量，一开始的目的就是用于日志采集和传输，而日志采集重吞吐，数据丢失关系不大。0.8版本开始支持复制，不支持事务，对消息的重复、丢失、错误没有严格要求，适合产生大量数据的互联网服务的数据收集业务

### kafka高性能原因

#### 生产者

Kafka会把收到的消息都写入到硬盘中，它绝对不会丢失数据。为了优化写入速度Kafak采用了两个技术，顺序写入和MMFile

**顺序写入**

- 因为硬盘是机械结构，每次读写都会寻址->写入，其中寻址是一个“机械动作”，它是最耗时的。所以硬盘最“讨厌”随机I/O，最喜欢顺序I/O。为了提高读写硬盘的速度，Kafka就是使用顺序I/O

- 收到消息后Kafka会把数据插入到文件末尾。这种方法有一个缺陷——没有办法删除数据，所以Kafka是不会删除数据的，它会把所有的数据都保留下来，每个消费者（Consumer）对每个Topic都有一个offset用来表示读取到了第几条数据

**Memory Mapped Files**

- Kafka的数据并不是实时的写入硬盘，它充分利用了现代操作系统分页存储来利用内存提高I/O效率

- Memory Mapped Files也被翻译成内存映射文件，在64位操作系统中一般可以表示20G的数据文件，它的工作原理是直接利用操作系统的Page来实现文件到物理内存的直接映射。完成映射之后你对物理内存的操作会被同步到硬盘上（操作系统在适当的时候）

- 这种方法也有一个很明显的缺陷——不可靠，写到mmap中的数据并没有被真正的写到硬盘，操作系统会在程序主动调用flush的时候才把数据真正的写到硬盘。Kafka提供了一个参数——producer.type来控制是不是主动flush，如果Kafka写入到mmap之后就立即flush然后再返回Producer叫同步(sync)；写入mmap之后立即返回Producer不调用flush叫异步(async)

#### 消费者

**零拷贝**

Kafka 的索引文件使用的是 mmap + write 方式，数据文件使用的是 sendfile 方式。适用于系统日志消息这种高吞吐量的大块文件的数据持久化和传输

- 传统read/write方式进行网络文件传输的方式，文件数据实际上是经过了四次copy操作

- 硬盘—>内核buf—>用户buf—>socket相关缓冲区—>协议引擎

- kafka基于sendfile实现Zero Copy，直接从内核空间（DMA的）到内核空间（Socket的），然后发送网卡

追溯 Kafka 文件传输的代码，最终发现调用了 Java NIO 库里的 `transferTo` 方法

```java
@Overridepublic 
long transferFrom(FileChannel fileChannel, long position, long count) throws IOException { 
    return fileChannel.transferTo(position, count, socketChannel);
}
```

如果 Linux 系统支持 `sendfile()` 系统调用，那么 `transferTo()` 实际上最后就会使用到 `sendfile()` 系统调用函数

**批量压缩**

- 在很多情况下，系统的瓶颈不是CPU或磁盘，而是网络IO。进行数据压缩会消耗少量的CPU资源，不过对于kafka而言，网络IO更应该需要考虑

- Kafka使用了批量压缩，即将多个消息一起压缩而不是单个消息压缩

- Kafka允许使用递归的消息集合，批量的消息可以通过压缩的形式传输并且在日志中也可以保持压缩格式，直到被消费者解压缩

- Kafka支持多种压缩协议，包括Gzip和Snappy压缩协议

## RocketMQ

RocketMQ是使用Java语言开发的一款MQ产品。经过数年阿里双11的考验，性能与稳定性非常高。其
没有遵循任何常见的MQ协议，而是使用自研协议。对于Spring Cloud Alibaba，其支持RabbitMQ、
Kafka，但提倡使用RocketMQ

### rocketMq高性能原因

#### 生产者

**顺序写入**

- 消息存储是由ConsumeQueue和CommitLog配合完成的。一个Topic里面有多个MessageQueue，每个MessageQueue对应一个ConsumeQueue

- ConsumeQueue里记录着消息物理存储地址

- CommitLog就存储文件具体的字节信息。文件大小默认1g，文件名称20位数，左边补0右边为偏移量。消息顺序写入文件，文件满了则写入下一个文件


#### 消费者

**随机读**

每次读消息时先读逻辑队列consumQue中的元数据，再从commitlog中找到消息体。但是入口处rocketmq采用package机制，可以批量地从磁盘读取，作为cache存到内存中，加速后续的读取速度

随机读具体流程

- Consumer每20s重新做一次负载均衡更新，根据从Broker存储的ConsumerGroup和Topic信息，把MessageQueue分发给不同的Consumer，负载策略默认是分页
- 每个MessageQueue对应一个pullRequest，全部存储到该Consumer的pullRequestQueue队列里面
- Consumer启动独立后台PullMessageService线程，不停的尝试从pullRequestQueue.take()获取PullRequest
- 捞取到PullRequest会先做缓存校验（默认一个Queue里面缓存待处理消息个数不超过1000个，消息大小不超过100M，否则会延迟50ms再重试），从而保证客户端的缓存负载不会过高
- PullRequest发送给Broker，如果Broker发现该Queue有待处理的消息，就会直接返回给Consumer，Consumer接收响应以后，重新把该PullRequest丢到自己的pullRequestQueue队列里面,从而重复执行捞取消息的动作，保证消息的及时性
- PullRequest发送给Broker，如果Broker发现该Queue没有待处理的消息，则会Hold住这个请求，暂不响应给Consumer，默认长轮询是5s重试获取一次待处理消息，如果有新的待处理消息则立刻Response给Consumer，当客户端检测到消息挂起超时（客户端有默认参数 响应超时时间 20s），会重新发起PullRequest给Broker

**消费模型**

常见消费模型有以下几种

- push：producer发送消息后，broker马上把消息投递给consumer。这种方式好在实时性比较高，但是会增加broker的负载；而且消费端能力不同，如果push推送过快，消费端会出现很多问题
- pull：producer发送消息后，broker什么也不做，等着consumer自己来读取。它的优点在于主动权在消费者端，可控性好；但是间隔时间不好设置，间隔太短浪费资源，间隔太长又会消费不及时
- 长轮询：当consumer过来请求时，broker会保持当前连接一段时间 默认15s,如果这段时间内有消息到达，则立刻返回给consumer；15s没消息的话则返回空然后重新请求。这种方式的缺点就是服务端要保存consumer状态，客户端过多会一直占用资源
- RocketMQ默认是采用pushConsumer方式消费的，从概念上来说是推送给消费者，它的本质是pull+长轮询。这样既通过长轮询达到了push的实时性，又有了pull的可控性。系统收到消息后会自动处理消息和offset(消息偏移量)，如果期间有新的consumer加入会自动做负载均衡(集群模式下offset存在broker中; 广播模式下offset存在consumer里)。当然我们也可以设置为pullConsumer模式，这样灵活性会提高，但是代码却会很复杂，需要手动维护offset，消息存储和状态


**zero copy**

零拷贝技术有mmap及sendfile，sendfile大文件传输快，mmap小文件传输快。MMQ发送的消息通常都很小，rocketmq就是以mmap+write方式实现的

## RocketQ和kafka对比

|   关键词   | ACTIVEMQ | RABBITMQ |           KAFKA           |         ROCKETMQ          |
| :--------: | :------: | :------: | :-----------------------: | :-----------------------: |
|  开发语言  |   Java   |  ErLang  |           Java            |           Java            |
| 单机吞吐量 |   万级   |   万级   |          十万级           |          十万级           |
|   Topic    |    -     |    -     | 百级Topic会影响系统吞吐量 | 千级Topic会影响系统吞吐量 |
| 社区活跃度 |    低    |    高    |            高             |            高             |

### 为什么kafka比RocketMQ[吞吐量](https://so.csdn.net/so/search?q=吞吐量&spm=1001.2101.3001.7020)更高

- kafka性吞吐量更高主要是由于Producer端将多个小消息合并，批量发向Broker。kafka采用异步发送的机制，当发送一条消息时，消息并没有发送到broker而是缓存起来，然后直接向业务返回成功，当缓存的消息达到一定数量时再批量发送

- 此时减少了网络io，从而提高了消息发送的性能，但是如果消息发送者宕机，会导致消息丢失，业务出错，所以理论上kafka利用此机制提高了io性能却降低了可靠性


### RocketMQ为何无法使用同样的方式

- RocketMQ通常使用的Java语言，缓存过多消息会导致频繁GC
- Producer调用发送消息接口，消息未发送到Broker，向业务返回成功，此时Producer宕机，会导致消息丢失，业务出错
- Producer通常为分布式系统，且每台机器都是多线程发送，我们认为线上的系统单个Producer每秒产生的数据量有限，不可能上万
- 缓存的功能完全可以由上层业务完成

### 为什么阿里会自研RocketMQ？

- Kafka的业务应用场景主要定位于日志传输；对于复杂业务支持不够
- 阿里很多业务场景对数据可靠性、数据实时性、消息队列的个数等方面的要求很高。

  - kafka针对海量数据，但是对数据的正确度要求不是十分严格。而阿里巴巴中用于交易相关的事情较多，对数据的正确性要求极高，Kafka不合适

- 当业务成长到一定规模，采用开源方案的技术成本会变高

  - 开源方案无法满足业务的需要；旧版本、自开发代码与新版本的兼容都可能是问题；运维角度，Kafka使用 scala 编写，而阿里是java系。Kafka 的后续维护是个问题

### 为什么选择RocketMQ

- 当broker里面的topic的partition数量过多时，kafka的性能却不如rocketMq

- kafka和rocketMq都使用文件存储，但是kafka是一个分区一个文件，当topic过多，分区的总量也会增加，kafka中存在过多的文件，当对消息刷盘时，就会出现文件竞争磁盘，出现性能的下降。一个partition（分区）一个文件，顺序读写。一个分区只能被一个消费组中的一个 消费线程进行消费，因此可以同时消费的消费端也比较少

- rocketMq所有的队列都存储在一个文件中，每个队列的存储的消息量也比较小，因此topic的增加对rocketMq的性能的影响较小。rocketMq可以存在的topic比较多，可以适应比较复杂的业务

### 测试对比

对比发送端、接收端共存情况下，Topic数量对Kafka、RocketMQ的性能影响，分区数采用8个分区。这次压测我们只关注服务端的性能指标，所以压测的退出标准是:

**不断增加发送端的压力,直到系统吞吐量不再上升,而响应时间拉长。此时服务端出现性能瓶颈，获取相应的系统最佳吞吐量，整个过程中保证消息没有累积**

#### 测试场景

默认每个Topic的分区数为8，每个Topic对应一个订阅者，逐步增加Topic数量。得到如下数据

![image-20230912195330699](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202309121953044.png)

可以看到，不论Topic数量是多少，Kafka和RocketMQ均能保证发送端和消费端的TPS持平，就是说，保证了消息没有累积。

根据Topic数量的变化，画出二者的消息处理能力的对比曲线如下图

![img](http://static.dingtalk.com/media/lALODSsaM80BuM0DOQ_825_440.png)



从图上可以看出

- Kafka在Topic数量由64增长到256时，吞吐量下降了98.37%
- RocketMQ在Topic数量由64增长到256时，吞吐量只下降了16%

为什么两个产品的表现如此悬殊呢？这是因为Kafka的每个Topic、每个分区都会对应一个物理文件。当Topic数量增加时，消息分散的落盘策略会导致磁盘IO竞争激烈成为瓶颈。而RocketMQ所有的消息是保存在同一个物理文件中的，Topic和分区数对RocketMQ也只是逻辑概念上的划分，所以Topic数的增加对RocketMQ的性能不会造成太大的影响

#### 测试结论

**在消息发送端，消费端共存的场景下，随着Topic数的增加Kafka吞吐量会急剧下降，而RocketMQ则表现稳定。因此Kafka适合Topic和消费端都比较少的业务场景，而RocketMQ更适合多Topic，多消费端的业务场景**

#### 测试环境

服务端为单机部署，机器配置如下

| CPU  |                             24核                             |
| :--: | :----------------------------------------------------------: |
| 内存 |                             94G                              |
| 硬盘 | Seagate Constellation ES (SATA 6Gb/s) 2,000,398,934,016 bytes [2.00 TB] 7202 rpm |
| 网卡 |                           1000Mb/s                           |

应用版本

| 消息中间件 | 版本  |
| :--------: | :---: |
|   Kafka    | 0.8.2 |
|  RocketMQ  | 3.4.6 |

#### 测试脚本

|    压力端     |      Jmeter的java客户端       |
| :-----------: | :---------------------------: |
|   消息大小    |            128字节            |
|    并发数     | 能达到服务端最大TPS的最优并发 |
| Topic分区数量 |               8               |
|   刷盘策略    |           异步落盘            |

# MQ常见协议（不了解）

一般情况下MQ的实现是要遵循一些常规性协议的。常见的协议如下：

## JMS

JMS，Java Messaging Service（Java消息服务）。是Java平台上有关MOM（Message Oriented
Middleware，面向消息的中间件 PO/OO/AO）的技术规范，它便于消息系统中的Java应用程序进行消
息交换，并且通过提供标准的产生、发送、接收消息的接口，简化企业应用的开发。ActiveMQ是该协
议的典型实现。

## STOMP

STOMP，Streaming Text Orientated Message Protocol（面向流文本的消息协议），是一种MOM设计
的简单文本协议。STOMP提供一个可互操作的连接格式，允许客户端与任意STOMP消息代理
（Broker）进行交互。ActiveMQ是该协议的典型实现，RabbitMQ通过插件可以支持该协议。

## AMQP

AMQP，Advanced Message Queuing Protocol（高级消息队列协议），一个提供统一消息服务的应用
层标准，是应用层协议的一个开放标准，是一种MOM设计。基于此协议的客户端与消息中间件可传递
消息，并不受客户端/中间件不同产品，不同开发语言等条件的限制。 RabbitMQ是该协议的典型实
现。

## MQTT

MQTT，Message Queuing Telemetry Transport（消息队列遥测传输），是IBM开发的一个即时通讯协
议，是一种二进制协议，主要用于服务器和低功耗IoT（物联网）设备间的通信。该协议支持所有平
台，几乎可以把所有联网物品和外部连接起来，被用来当做传感器和致动器的通信协议。 RabbitMQ通
过插件可以支持该协议。

# RocketMQ发展历程



![4](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801093.jpg)

- 2007年，阿里开始五彩石项目，Notify作为项目中交易核心消息流转系统，应运而生。Notify系统是
  RocketMQ的雏形
- 2010年，B2B大规模使用ActiveMQ作为阿里的消息内核。阿里急需一个具有海量堆积能力的消息系
  统
- 2011年初，Kafka开源。淘宝中间件团队在对Kafka进行了深入研究后，开发了一款新的MQ，MetaQ。
  2012年，MetaQ发展到了v3.0版本，在它基础上进行了进一步的抽象，形成了RocketMQ，然后就将其
  进行了开源
- 2015年，阿里在RocketMQ的基础上，又推出了一款专门针对阿里云上用户的消息系统Aliware MQ。
  2016年双十一，RocketMQ承载了万亿级消息的流转，跨越了一个新的里程碑。11⽉28⽇，阿⾥巴巴
  向 Apache 软件基⾦会捐赠 RocketMQ，成为 Apache 孵化项⽬
- 2017 年 9 ⽉ 25 ⽇，Apache 宣布 RocketMQ孵化成为 Apache 顶级项⽬（TLP ），成为国内⾸个互联
  ⽹中间件在 Apache 上的顶级项⽬

# RocketMQ基本概念

## 消息（Message）

> 消息系统所传输信息的物理载体，生产和消费数据的最小单位，每条消息必须属于一个主题。每个消息拥有唯一的Message ID，且可以携带具有业务标识的Key。系统提供了通过Message ID和Key查询消息的功能

## 主题（Topic）

>  **表示一类消息的集合，每个主题包含若干条消息，是RocketMQ进行消息订阅的基本单位，一个生产者可以同时发送多种Topic的消息；而一个消费者只对某种特定的Topic感兴趣，即只可以订阅和消费一种Topic的消息**

![5](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801184.jpg)

## 标签（Tag）
> 为消息设置的标签，用于同一主题下区分不同类型的消息。来自同一业务单元的消息，可以根据不同业
> 务目的在同一主题下设置不同标签。标签能够有效地保持代码的清晰度和连贯性，并优化RocketMQ提
> 供的查询系统。消费者可以根据Tag实现对不同子主题的不同消费逻辑，实现更好的扩展性。Topic是消息的一级分类，Tag是消息的二级分类

## 订阅与发布
> 消息的发布是指某个生产者向某个topic发送消息；消息的订阅是指某个消费者关注了某个topic中带有某些tag的消息，进而从该topic消费数据

## 队列（Queue）

> 存储消息的物理实体。一个Topic中可以包含多个Queue，每个Queue中存放的就是该Topic的消息。一
> 个Topic的Queue也被称为一个Topic中消息的分区（Partition）。
> 一个Topic的Queue中的消息只能被一个消费者组中的一个消费者消费。一个Queue中的消息不允许同
> 一个消费者组中的多个消费者同时消费

![6](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801604.jpg)

## 分片（Sharding）

+ 分片不同于分区。在RocketMQ中，分片指的是存放相应Topic的Broker。每个分片中会创建出相应数量的分区，即Queue，每个Queue的大小都是相同的。

![7](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801784.jpg)

## 消息标识（MessageId/Key）

> RocketMQ中每个消息拥有唯一的MessageId，且可以携带具有业务标识的Key，以方便对消息的查询。
> 不过需要注意的是，MessageId有两个：在生产者send()消息时会自动生成一个MessageId（msgId)，
> 当消息到达Broker后，Broker也会自动生成一个MessageId(offsetMsgId)。msgId、offsetMsgId与key都
> 称为消息标识

**msgId**：由producer端生成，其生成规则为：

```bash
producerIp + 进程pid + MessageClientIDSetter类的ClassLoader的hashCode +当前时间 + AutomicInteger自增计数器
```

**offsetMsgId**：由broker端生成，其生成规则为： 

```
brokerIp + 物理分区的offset（Queue中的偏移量）
```

**key**：由用户指定的业务相关的唯一标识



# 系统架构

![8](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801282.jpg)

**RocketMQ架构上主要分为四部分构成**

## Producer

+ 消息生产者，负责生产消息。Producer通过MQ的负载均衡模块选择相应的Broker集群队列进行消息投递，投递的过程支持快速失败并且低延迟

> 例如，业务系统产生的日志写入到MQ的过程，就是消息生产的过程
> 再如，电商平台中用户提交的秒杀请求写入到MQ的过程，就是消息生产的过程

+ RocketMQ中的消息生产者都是以生产者组（Producer Group）的形式出现的。生产者组是同一类生产者的集合，这类Producer发送相同Topic类型的消息。一个生产者组可以同时发送多个主题的消息。

## Consumer

+ 消息消费者，负责消费消息。一个消息消费者会从Broker服务器中获取到消息，并对消息进行相关业务处理

> 例如，QoS系统从MQ中读取日志，并对日志进行解析处理的过程就是消息消费的过程
> 再如，电商平台的业务系统从MQ中读取到秒杀请求，并对请求进行处理的过程就是消息消费的
> 过程

+ RocketMQ中的消息消费者都是以消费者组（Consumer Group）的形式出现的。消费者组是同一类消费者的集合，这类Consumer消费的是同一个Topic类型的消息。消费者组使得在消息消费方面，实现**负载均衡**（将一个Topic中的不同的Queue平均分配给同一个Consumer Group的不同的Consumer，注意，并不是将消息负载均衡）和**容错**（一个Consmer挂了，该Consumer Group中的其它Consumer可以接着消费原Consumer消费的Queue）的目标变得非常容易

![9](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801687.jpg)

+ 消费者组中Consumer的数量应该小于等于订阅Topic的Queue数量。如果超出Queue数量，则多出的
  Consumer将不能消费消息

![10](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801739.jpg)

+ 不过，一个Topic类型的消息可以被多个消费者组同时消费

> 注意，
> 1）消费者组只能消费一个Topic的消息，不能同时消费多个Topic消息
> 2）一个消费者组中的消费者必须订阅完全相同的Topic

## Name Server

### 功能介绍

+ NameServer是一个Broker与Topic路由的注册中心，支持Broker的动态注册与发现

+ RocketMQ的思想来自于Kafka，而Kafka是依赖了Zookeeper的。所以，在RocketMQ的早期版本，即在MetaQ v1.0与v2.0版本中，也是依赖于Zookeeper的。从MetaQ v3.0，即RocketMQ开始去掉了Zookeeper依赖，使用了自己的NameServer

+ 主要包括两个功能

  - **Broker管理**：接受Broker集群的注册信息并且保存下来作为路由信息的基本数据；提供心跳检测
    机制，检查Broker是否还存活

  - **路由信息管理**：每个NameServer中都保存着Broker集群的整个路由信息和用于客户端查询的队列
    信息。Producer和Conumser通过NameServer可以获取整个Broker集群的路由信息，从而进行消
    息的投递和消费

### 路由注册

+ NameServer通常也是以集群的方式部署，不过，NameServer是无状态的，即NameServer集群中的各个节点间是无差异的，各节点间相互不进行信息通讯。那各节点中的数据是如何进行数据同步的呢？在Broker节点启动时，轮询NameServer列表，与每个NameServer节点建立长连接，发起注册请求。在NameServer内部维护着⼀个Broker列表，用来动态存储Broker的信息

> 注意，这是与其它像zk、Eureka、Nacos等注册中心不同的地方。
> 这种NameServer的无状态方式，有什么优缺点：
> 优点：NameServer集群搭建简单，扩容简单。
> 缺点：对于Broker，必须明确指出所有NameServer地址。否则未指出的将不会去注册。也正因为如此，NameServer并不能随便扩容。因为，若Broker不重新配置，新增的NameServer对于Broker来说是不可见的，其不会向这个NameServer进行注册

+ Broker节点为了证明自己是活着的，为了维护与NameServer间的长连接，会将最新的信息以心跳包的方式上报给NameServer，每30秒发送一次心跳。心跳包中包含 BrokerId、Broker地址(IP+Port)、Broker名称、Broker所属集群名称等等。NameServer在接收到心跳包后，会更新心跳时间戳，记录这个Broker的最新存活时间

   ### 路由剔除

- 由于Broker关机、宕机或网络抖动等原因，NameServer没有收到Broker的心跳，NameServer可能会将其从Broker列表中剔除
- NameServer中有⼀个定时任务，每隔10秒就会扫描⼀次Broker表，查看每一个Broker的最新心跳时间戳距离当前时间是否超过120秒，如果超过，则会判定Broker失效，然后将其从Broker列表中剔除

> 扩展：对于RocketMQ日常运维工作，例如Broker升级，需要停掉Broker的工作。OP需要怎么做？
> OP需要将Broker的读写权限禁掉。一旦client(Consumer或Producer)向broker发送请求，都会收到broker的NO_PERMISSION响应，然后client会进行对其它Broker的重试。
> 当OP观察到这个Broker没有流量后，再关闭它，实现Broker从NameServer的移除。
> OP：运维工程师
> SRE：Site Reliability Engineer，现场可靠性工程师

### 路由发现

+ RocketMQ的路由发现采用的是Pull模型。当Topic路由信息出现变化时，NameServer不会主动推送给客户端，而是客户端定时拉取主题最新的路由。默认客户端每30秒会拉取一次最新的路由

> 扩展：
> 1）Push模型：推送模型。其实时性较好，是一个“发布-订阅”模型，需要维护一个长连接。而长连接的维护是需要资源成本的。该模型适合于的场景：实时性要求较高Client数量不多，Server数据变化较频繁
> 2）Pull模型：拉取模型。存在的问题是，实时性较差
> 3）Long Polling模型：长轮询模型。其是对Push与Pull模型的整合，充分利用了这两种模型的优势，屏蔽了它们的劣势

**客户端NameServer选择策略**

+ 这里的客户端指的是Producer与Consumer客户端在配置时必须要写上NameServer集群的地址，那么客户端到底连接的是哪个NameServer节点呢？客户端首先会生产一个随机数，然后再与NameServer节点数量取模，此时得到的就是所要连接的节点索引，然后就会进行连接。如果连接失败，则会采用round-robin策略，逐个尝试着去连接其它节点首先采用的是随机策略进行的选择，失败后采用的是轮询策略

> 扩展：Zookeeper Client是如何选择Zookeeper Server的？
> 简单来说就是，经过两次Shuffle，然后选择第一台Zookeeper Server。详细说就是，将配置文件中的zk server地址进行第一次shuffle，然后随机选择一个。这个选择出的一般都是一个hostname。然后获取到该hostname对应的所有ip，再对这些ip进行第二次shuffle，从shuffle过的结果中取第一个server地址进行连接

## Broker

### 功能介绍

+ Broker充当着消息中转角色，负责存储消息、转发消息。Broker在RocketMQ系统中负责接收并存储从生产者发送来的消息，同时为消费者的拉取请求作准备。Broker同时也存储着消息相关的元数据，包括消费者组消费进度偏移offset、主题、队列等。Kafka 0.8版本之后，offset是存放在Broker中的，之前版本是存放在Zookeeper中的

### 模块构成

+ 下图为Broker Server的功能模块示意图

![11](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211801507.jpg)

- **Remoting Module**：整个Broker的实体，负责处理来自clients端的请求。而这个Broker实体则由以下模块构成
- **Client Manager**：客户端管理器。负责接收、解析客户端(Producer/Consumer)请求，管理客户端。例如，维护Consumer的Topic订阅信息
- **Store Service**：存储服务。提供方便简单的API接口，处理消息存储到物理硬盘和消息查询功能
- **HA Service**：高可用服务，提供Master Broker 和 Slave Broker之间的数据同步功能
- **Index Service**：索引服务。根据特定的Message key，对投递到Broker的消息进行索引服务，同时也提供根据Message Key对消息进行快速查询的功能

## 工作流程

### 具体流程

- 启动NameServer，NameServer启动后开始监听端口，等待Broker、Producer、Consumer连接
- 启动Broker时，Broker会与所有的NameServer建立并保持长连接，然后每30秒向NameServer定时发送心跳包
- 发送消息前，可以先创建Topic，创建Topic时需要指定该Topic要存储在哪些Broker上，当然，在创建Topic时也会将Topic与Broker的关系写入到NameServer中。不过，这步是可选的，也可以在发送消息时自动创建Topic
- Producer发送消息，启动时先跟NameServer集群中的其中一台建立长连接，并从NameServer中获取路由信息，即当前发送的Topic消息的Queue与Broker的地址（IP+Port）的映射关系。然后根据算法策略从队选择一个Queue，与队列所在的Broker建立长连接从而向Broker发消息。当然，在获取到路由信息后，Producer会首先将路由信息缓存到本地，再每30秒从NameServer更新一次路由信息
- Consumer跟Producer类似，跟其中一台NameServer建立长连接，获取其所订阅Topic的路由信息，然后根据算法策略从路由信息中获取到其所要消费的Queue，然后直接跟Broker建立长连接，开始消费其中的消息。Consumer在获取到路由信息后，同样也会每30秒从NameServer更新一次路由信息。不过不同于Producer的是，Consumer还会向Broker发送心跳，以确保Broker的存活状态。Topic的创建模式

> 手动创建Topic时，有两种模式
> 集群模式：该模式下创建的Topic在该集群中，所有Broker中的Queue数量是相同的
> Broker模式：该模式下创建的Topic在该集群中，每个Broker中的Queue数量可以不同
> 自动创建Topic时，默认采用的是Broker模式，会为每个Broker默认创建4个Queue

### 读/写队列

- 从物理上来讲，读/写队列是同一个队列。所以，不存在读/写队列数据同步问题。读/写队列是逻辑上进行区分的概念。一般情况下，读/写队列数量是相同的
- 例如，创建Topic时设置的写队列数量为8，读队列数量为4，此时系统会创建8个Queue，分别是0 1 2 3 4 5 6 7。Producer会将消息写入到这8个队列，但Consumer只会消费0 1 2 3这4个队列中的消息，4 5 6 7中的消息是不会被消费到的
- 再如，创建Topic时设置的写队列数量为4，读队列数量为8，此时系统会创建8个Queue，分别是0 1 2 3 4 5 6 7。Producer会将消息写入到0 1 2 3 这4个队列，但Consumer只会消费0 1 2 3 4 5 6 7这8个队列中的消息，但是4 5 6 7中是没有消息的。此时假设Consumer Group中包含两个Consuer，Consumer1消费0 1 2 3，而Consumer2消费4 5 6 7。但实际情况是，Consumer2是没有消息可消费的
- 也就是说，当读/写队列数量设置不同时，总是有问题的。那么，为什么要这样设计呢？
- 其这样设计的目的是为了，**方便Topic的Queue的缩容**
- 例如，原来创建的Topic中包含16个Queue，如何能够使其Queue缩容为8个，还不会丢失消息？可以动态修改写队列数量为8，读队列数量不变。此时新的消息只能写入到前8个队列，而消费都消费的却是16个队列中的数据。当发现后8个Queue中的消息消费完毕后，就可以再将读队列数量动态设置为8。整个缩容过程，没有丢失任何消息
- perm用于设置对当前创建Topic的操作权限：2表示只写，4表示只读，6表示读写

# RocketMQ用途

## 基于mmap+page cache实现磁盘文件的高性能读写

### **传统文件IO操作的多次数据拷贝问题**

- 首先我们先来给大家分析一下，假设[RocketMQ](https://so.csdn.net/so/search?q=RocketMQ&spm=1001.2101.3001.7020)没有使用mmap技术，就是使用最传统和基本的普通文件IO操作去进行磁盘文件的读写，那么会存在什么样的性能问题？
- 答案是：**多次数据拷贝问题**
- 首先，假设我们有一个程序，这个程序需要对磁盘文件发起IO操作读取他里面的数据到自己这儿来，那么会经过以下一个顺序
- 首先从磁盘上把数据读取到内核IO缓冲区里去，然后再从内核IO缓存区里读取到用户进程私有空间里去，然后我们才能拿到这个文件里的数据

![http://img3.sycdn.imooc.com/5e63497300017cdf03620462.jpg](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzMuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTczMDAwMTdjZGYwMzYyMDQ2Mi5qcGc?x-oss-process=image/format,png)

- 为了读取磁盘文件里的数据，是不是发生了两次数据拷贝？
- 没错，所以这个就是普通的IO操作的一个弊端，必然涉及到两次数据拷贝操作，对磁盘读写性能是有影响的。
- 那么如果我们要将一些数据写入到磁盘文件里去呢？
- 那这个就是一样的过程了，必须先把数据写入到用户进程私有空间里去，然后从这里再进入内核IO缓冲区，最后进入磁盘文件里去

![http://img1.sycdn.imooc.com/5e6349730001914605020462.jpg](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzEuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTczMDAwMTkxNDYwNTAyMDQ2Mi5qcGc?x-oss-process=image/format,png)

+ 在数据进入磁盘文件的过程中，是不是再一次发生了两次数据拷贝？没错，所以这就是传统普通IO的问题，有两次数据拷贝问题

### RocketMQ是如何基于mmap技术+page cache技术优化

- 首先，RocketMQ底层对CommitLog、ConsumeQueue之类的磁盘文件的读写操作，基本上都会采用mmap技术来实现
- 如果具体到代码层面，就是基于JDK NIO包下的MappedByteBuffer的map()函数，来先将一个磁盘文件（比如一个CommitLog文件，或者是一个ConsumeQueue文件）映射到内存里来
- 这里我必须给大家解释一下，这个所谓的内存映射是什么意思
- 其实有的人可能会误以为是直接把那些磁盘文件里的数据给读取到内存里来了，类似这个意思，但是并不完全是对的
- 因为刚开始你建立映射的时候，并没有任何的数据拷贝操作，其实磁盘文件还是停留在那里，只不过他把物理上的磁盘文件的一些地址和用户进程私有空间的一些虚拟内存地址进行了一个映射

![http://img1.sycdn.imooc.com/5e6349740001834a03200298.jpg](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzEuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTc0MDAwMTgzNGEwMzIwMDI5OC5qcGc?x-oss-process=image/format,png)

- 这个地址映射的过程，就是JDK NIO包下的MappedByteBuffer.map()函数干的事情，底层就是基于mmap技术实现的
- 另外这里给大家说明白的一点是，这个mmap技术在进行文件映射的时候，一般有大小限制，在1.5GB~2GB之间
- 所以RocketMQ才让CommitLog单个文件在1GB，ConsumeQueue文件在5.72MB，不会太大。
- 这样限制了RocketMQ底层文件的大小，就可以在进行文件读写的时候，很方便的进行内存映射了。
- 然后接下来要给大家讲的一个概念，就是之前给大家说的PageCache，实际上在这里就是对应于虚拟内存

![img](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzMuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTc0MDAwMWJlZGYwNDgwMDMwNC5qcGc?x-oss-process=image/format,png)

### 基于mmap技术+pagecache技术实现高性能的文件读写

- 接下来就可以对这个已经映射到内存里的磁盘文件进行读写操作了，比如要写入消息到CommitLog文件，你先把一个CommitLog文件通过MappedByteBuffer的map()函数映射其地址到你的虚拟内存地址。
- 接着就可以对这个MappedByteBuffer执行写入操作了，写入的时候他会直接进入PageCache中，然后过一段时间之后，由os的线程异步刷入磁盘中，如下图我们可以看到这个示意

![http://img3.sycdn.imooc.com/5e63497400016f1a06960516.jpg](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzMuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTc0MDAwMTZmMWEwNjk2MDUxNi5qcGc?x-oss-process=image/format,png)

- 上面的图里，只有一次数据拷贝的过程，他就是从PageCache里拷贝到磁盘文件里而已！这个就是你使用mmap技术之后，相比于传统磁盘IO的一个性能优化
- 接着如果我们要从磁盘文件里读取数据呢？
- 那么此时就会判断一下，当前你要读取的数据是否在PageCache里？如果在的话，就可以直接从PageCache里读取了！
- 比如刚写入CommitLog的数据还在PageCache里，此时你Consumer来消费肯定是从PageCache里读取数据的
- 但是如果PageCache里没有你要的数据，那么此时就会从磁盘文件里加载数据到PageCache中去，如下图
- 而且PageCache技术在加载数据的时候，还会将你加载的数据块的临近的其他数据块也一起加载到PageCache里去

![http://img1.sycdn.imooc.com/5e63497400014ec806460528.jpg](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZzEuc3ljZG4uaW1vb2MuY29tLzVlNjM0OTc0MDAwMTRlYzgwNjQ2MDUyOC5qcGc?x-oss-process=image/format,png)

+ 在你读取数据的时候，其实也仅仅发生了一次拷贝，而不是两次拷贝

### 预映射机制 + 文件预热机制

接着给大家说几个Broker针对上述的磁盘文件高性能读写机制做的一些优化

- 内存预映射机制：Broker会针对磁盘上的各种CommitLog、ConsumeQueue文件预先分配好MappedFile，也就是提前对一些可能接下来要读写的磁盘文件，提前使用MappedByteBuffer执行map()函数完成映射，这样后续读写文件的时候，就可以直接执行了
- 文件预热：在提前对一些文件完成映射之后，因为映射不会直接将数据加载到内存里来，那么后续在读取尤其是CommitLog、ConsumeQueue的时候，其实有可能会频繁的从磁盘里加载数据到内存中去

所以其实在执行完map()函数之后，会进行madvise系统调用，就是提前尽可能多的把磁盘文件加载到内存里去。通过上述优化，才真正能实现一个效果，就是写磁盘文件的时候都是进入PageCache的，保证写入高性能；

同时尽可能多的通过map + madvise的映射后预热机制，把磁盘文件里的数据尽可能多的加载到PageCache里来，后续对CosumeQueue、CommitLog进行读取的时候，才能尽可能从内存里读取数据

[RocketMQ 如何基于mmap+page cache实现磁盘文件的高性能读写？](https://blog.csdn.net/qian_348840260/article/details/106720710)

## RocketMQ如何保证消息的可靠性投递

要想保证消息的可靠型投递，无非保证如下3个阶段的正常执行即可

- 生产者将消息成功投递到broker
- broker将投递过程的消息持久化下来
- 消费者能从broker消费到消息

![img](https://ask.qcloudimg.com/http-save/yehe-5457771/0b4bao0b0z.png)

### **发送端消息重试**

producer向broker发送消息后，没有收到broker的ack时，rocketmq会自动重试。重试的次数可以设置，默认为2次

```java
DefaultMQProducer producer = new DefaultMQProducer(RPODUCER_GROUP_NAME);
// 同步发送设置重试次数为5次
producer.setRetryTimesWhenSendFailed(5);
// 异步发送设置重试次数为5次
producer.setRetryTimesWhenSendAsyncFailed(5);
```

### **消息持久化**

![img](https://ask.qcloudimg.com/http-save/yehe-5457771/a5qz2p0g1m.png)我们先来了解一下消息的存储流程，这个知识对后面分析消费端消息重试非常重要

和消息相关的文件有如下几种

- CommitLog：存储消息的元数据
- ConsumerQueue：存储消息在CommitLog的索引
- IndexFile：可以通过Message Key，时间区间快速查找到消息

![img](https://ask.qcloudimg.com/http-save/yehe-5457771/1020fret3v.png)

整个消息的存储流程如下

- Producer将消息顺序写到CommitLog中
- 有一个线程根据消息的队列信息，写入到相关的ConsumerQueue中（minOffset为写入的初始位置，consumerOffset为当前消费到的位置，maxOffset为ConsumerQueue最新写入的位置）和IndexFile
- Consumer从ConsumerQueue的consumerOffset读取到当前应该消费的消息在CommitLog中的偏移量，到CommitLog中找到对应的消息，消费成功后移动consumerOffset

**刷盘机制**

![img](https://ask.qcloudimg.com/http-save/yehe-5457771/qyz9hope6o.png)

- **同步刷盘**：如上图所示，只有在消息真正持久化至磁盘后 RocketMQ 的 `Broker` 端才会真正返回给 Producer 端一个成功的 ACK 响应。同步刷盘对 MQ 消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多
- **异步刷盘**：能够充分利用 OS 的 `PageCache` 的优势，只要消息写入 `PageCache` 即可将成功的 ACK 返回给 Producer 端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了 MQ 的性能和吞吐量

**主从复制**

如果一个broker有master和slave时，就需要将master上的消息复制到slave上，复制的方式有两种

- **「同步复制」**：master和slave均写成功，才返回客户端成功。maste挂了以后可以保证数据不丢失，但是同步复制会增加数据写入延迟，降低吞吐量
- **「异步复制」**：master写成功，返回客户端成功。拥有较低的延迟和较高的吞吐量，但是当master出现故障后，有可能造成数据丢失

### **消费端消息重试**

- Consumer 消费消息其实也是需要一个 A 的操作，即 Comsumer 消费成功后需要返回 `Broker` 一个确认消息，**如果没有返回则 `Broker` 认为这条消息消费失败，失败后会再重试消费该消息**
- RocketMQ 对于重试消息的处理是先保存至 Topic 名称为`SCHEDULE_TOPIC_XXXX`的延迟队列中，后台定时任务按照对应的时间进行 Delay 后重新保存至`%RETRY%+consumerGroup`的重试队列中，RocketMQ默认允许每条消息最多重试**16**次，每次消费失败发送一条延时消息到重试队列，同一条消息失败一次将延时等级提高一次，然后再放到重试队列。重试16次后如果还没有消费成功，则将消息放到死信队列中

**那么重试队列中的消息是如何被消费的？**

消息消费者在启动的时候，会订阅正常的topic和重试队列的topic

![img](https://ask.qcloudimg.com/http-save/yehe-5457771/sbz11jhmat.png)

定时消息的实现逻辑也比较简单，可以归纳为如下几步

- 发送延时消息 
  - 替换topic为SCHEDULE_TOPIC_XXXX，queueId为消息延迟等级（如果不替换topic直接发到对应的consumeQueue中，则消息会被立马消费） 
  - 将消息原来的topic，queueId放到消息扩展属性中
  - 将消息应该执行的时间放到tagsCode中
- 将消息顺序写到CommitLog中
- 将消息对应的信息分发到对应的ConsumerQueue中（topic为SCHEDULE_TOPIC_XXXX总共有18个queue，对应18个延迟级别）
- 定时任务不断判断消息是否到达投递时间，没有到达则后续执行投递
- 如果到达投递时间，则从commitLog中拉取消息的内容，重新设置消息topic，queueId为原来的（原来的topic，queueId在消息扩展属性中），然后将消息投递到commitLog中，此时消息就会被分发到对应的队列中，然后被消费

**死信队列**

当消息达到最大的重试次数之后（默认16次），若消费依然失败，消息就会被投递到 Topic 名称为 `%DLQ%+consumerGroup` 的DLQ 死信队列，**死信队列用于处理无法被正常消费的消息**

RocketMQ 将这种正常情况下无法被消费的消息称为**死信消息（Dead-Letter Message）**，将存储死信消息的特殊队列称为**死信队列（Dead-Letter Queue）**。在 RocketMQ 中，可以通过使用 console 控制台对死信队列中的消息进行重发来使得消费者实例再次进行消费

**为什么重试队列和死信队列要按照Consumer Group来进行划分？**

**因为在RocketMQ的时候使用一定要保持订阅关系一致。即一个Consumer Group订阅的topic和tag要完全一致，不然可能会导致消费逻辑混乱，消息丢失**

如下任意一种情况都表现为订阅关系不一致

- 相同ConsumerGroup下的Consumer实例订阅了不同的Topic。
- 相同ConsumerGroup下的Consumer实例订阅了相同的Topic，但订阅的Tag不一致

## RocketMQ削峰

```java
@Service
@RocketMQMessageListener(topic = RocketConstant.Topic.PRAISE_TOPIC, consumerGroup = RocketConstant.ConsumerGroup.PRAISE_CONSUMER)
@Slf4j
public class PraiseListener implements RocketMQListener<PraiseRecordVO>, RocketMQPushConsumerLifecycleListener {
    @Resource
    private PraiseRecordService praiseRecordService;

    @Override
    public void onMessage(PraiseRecordVO vo) {
        praiseRecordService.insert(vo.copyProperties(PraiseRecord::new));
    }

    @Override
    public void prepareStart(DefaultMQPushConsumer consumer) {
        // 每次拉取的间隔，单位为毫秒
        consumer.setPullInterval(2000);
        // 设置每次从队列中拉取的消息数为16
        consumer.setPullBatchSize(16);
    }
}
```

单次pull消息的最大数目受broker存储的`MessageStoreConfig.maxTransferCountOnMessageInMemory`(默认为32)值限制，即若想要消费者从队列拉取的消息数大于32有效(pullBatchSize>32)则需更改Broker的启动参数`maxTransferCountOnMessageInMemory`值。在MQ削峰的配置参数里，以下几个`DefaultMQPushConsumer`的参数是需要注意一下的

- pullInterval：每次从Broker拉取消息的间隔，单位为毫秒
- pullBatchSize：每次从Broker队列拉取到的消息数，该参数很容易让人误解，一开始我以为是每次拉取的消息总数，但测试过几次后确认了实质上是从每个队列的拉取数(源码上的注释文档真的很差，跟没有一样)，即Consume每次拉取的消息总数如下： `EachPullTotal=所有Broker上的写队列数和(writeQueueNums=readQueueNums) * pullBatchSize`
- consumeMessageBatchMaxSize：每次消费(即将多条消息合并为List消费)的最大消息数目，默认值为1，[rocketmq-spring-boot-starter](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fapache%2Frocketmq-spring) 目前不支持批量消费(2.1.0版本)

在消费者开始消息消费时会先从各队列中拉取一条消息进行消费，消费成功后再以每次pullBatchSize的数目进行拉取

PraiseListener中设置了每次拉取的间隔为2s，每次从队列拉取的消息数为16，在搭建了2master broker且broker上writeQueueNums=readQueueNums=4的环境下每次拉取的消息理论数值为16 * 2 * 4 = 128，在第一次从各队列拉取1条消息(即共8条)后消费成功后会每次就会拉取最多128条消息进行消费，想验证下的可以把onMessage()的insert()改为log.info("1")然后统计单位秒内打印的日志数是否为128

根据以上配置单Conumer情况下每2s理论消费为128，即每2秒数据库新增的点赞数据大概为128条左右，有20%偏差都在个人可接受范围内，然后对点赞接口进行简单压测1s 2000请求校验MQ效果，根据消费配置理论上需要16次拉取即需32s才能消费完，压测后查看数据库校验效果

![praise-jmeter.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/23/171a64bf441c0acf~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)



![praise-jmeter-db.png](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/23/171a64bf443af9bd~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.png)

由上图可以看出除第一次2s和最后一次2s外数据库每2s的插入数据数和一般都在128附近波动，也用了34s(因第一次拉取数较少所以比理论多花费一次拉取)消费的偏差大小可能会受每次拉取数pullBatchSize、Broker上的消息队列数、网络波动等情况影响，但需要的目的已经达到了，我只想把单位时间内过多的数据库操作交给MQ做分隔成多个单位时间内的小批量操作，消息过多就堆积，当请求峰值过了后直到MQ堆积的消息消费完前数据库的插入数依旧会与峰值期的插入数相差不大，达到了MQ削峰填谷的效果

### 上线了但消费效率预估失误如何动态更改消费效率

当把拉取数pullBatchSize设置Broker的默认最大传输值32了，线上又不想重启Broker更改maxTransferCountOnMessageInMemory参数，如有2个Broker且queue都为4，那么拉取消费效率才为32 * 2 * 4 = 256，如果想要动态调整，可以从Broker数或Broker队列数下手，可以将Broker的writeQueueNums、readQueueNums增大，如都改为8，那么效率就成了32 * 2 * 8 = 512。需要注意的是更改完queues后必须去Dashboard的Topic下的CONSUMER MANAGER查看新增的队列上是否都有Consumer成功注册上去了，因为遇到了在测试与生产上使用rocketmq-spring-boot-starter @RocketMQListener标注消费者不会自动注册到新队列上的情况，但没排除是不是RocketMQ版本的原因(个人本地的版本比环境上的高了一个小版本0.0.1，本地没出现没消费者注册到新队列上的问题)，而是使用了自定义DefaultMQPushConsumer bean(原生的方式都是没有问题的)的备用方案。当再启动新的消费者应用时CONSUMER MANAGER(下图)中就会出现 新Consumer数 * 各Broker队列数和的队列行

[实战：RocketMQ削峰，这一篇就够了](https://juejin.cn/post/6844904135884537870)

## 如何保证消息不被重复消费

### **可能会有哪些重复消费的问题**

首先，比如 RabbitMQ、RocketMQ、Kafka，都有可能会出现消息重复消费的问题，正常。因为这问题通常不是 MQ 自己保证的，是由我们开发来保证的。挑一个 Kafka 来举个例子，说说怎么重复消费吧

Kafka 实际上有个 offset 的概念，就是每个消息写进去，都有一个 offset，代表消息的序号，然后 consumer 消费了数据之后，每隔一段时间（定时定期），会把自己消费过的消息的 offset 提交一下，表示“我已经消费过了，下次我要是重启啥的，你就让我继续从上次消费到的 offset 来继续消费吧”

但是凡事总有意外，比如我们之前生产经常遇到的，就是你有时候重启系统，看你怎么重启了，如果碰到点着急的，直接 kill 进程了，再重启。这会导致 consumer 有些消息处理了，但是没来得及提交 offset，尴尬了。重启之后，少数消息会再次消费一次

有这么个场景。数据 1/2/3 依次进入 kafka，kafka 会给这三条数据每条分配一个 offset，代表这条数据的序号，我们就假设分配的 offset 依次是 152/153/154。消费者从 kafka 去消费的时候，也是按照这个顺序去消费。假如当消费者消费了 offset=153 的这条数据，刚准备去提交 offset 到 zookeeper，此时消费者进程被重启了。那么此时消费过的数据 1/2 的 offset 并没有提交，kafka 也就不知道你已经消费了 offset=153 这条数据。那么重启之后，消费者会找 kafka 说，嘿，哥儿们，你给我接着把上次我消费到的那个地方后面的数据继续给我传递过来。由于之前的 offset 没有提交成功，那么数据 1/2 会再次传过来，如果此时消费者没有去重的话，那么就会导致重复消费

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211802714.png)

如果消费者干的事儿是拿一条数据就往数据库里写一条，会导致说，你可能就把数据 1/2 在数据库里插入了 2 次，那么数据就错啦

其实重复消费不可怕，可怕的是你没考虑到重复消费之后，怎么保证**幂等性**

举个例子吧。假设你有个系统，消费一条消息就往数据库里插入一条数据，要是你一个消息重复两次，你不就插入了两条，这数据不就错了？但是你要是消费到第二次的时候，自己判断一下是否已经消费过了，若是就直接扔了，这样不就保留了一条数据，从而保证了数据的正确性

一条数据重复出现两次，数据库里就只有一条数据，这就保证了系统的幂等性

幂等性，通俗点说，就一个数据，或者一个请求，给你重复来多次，你得确保对应的数据是不会改变的，不能出错

### 怎么保证消息队列消费的幂等性

其实还是得结合业务来思考，我这里给几个思路

比如你拿个数据要写库，你先根据主键查一下，如果这数据都有了，你就别插入了，update 一下好吧

比如你是写 Redis，那没问题了，反正每次都是 set，天然幂等性

比如你不是上面两个场景，那做的稍微复杂一点，**常见处理方法：可以给消息设置一个唯一ID，当消费者消费消息时就将这个ID存储在数据库中，每次消费者消费消息次就用这个ID去数据库里查找是否有相同ID来确保是否要消费消息，这样就实现了消息幂等**
