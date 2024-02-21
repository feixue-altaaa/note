



# RocketMQ的核心工作机制

![1674133054337](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803409.png)

- 由上图可以看到RocketMQ存储的文件主要包括**Commitlog文件**、**ConsumeQueue文件**、**Index文件**

## RocketMQ的存储设计介绍

- **CommitLog：RocketMQ将所有主题的消息存储在同一个文件中，确保消息发送时按顺序写文件，尽最大能力确保消息发送的高可用性与高吞吐量**
- **ConsumeQueue**：消息中间件一般都是基于Topic的订阅与发布模式，消息消费时必须按照主题进行筛选消息，显然从Commitlog文件中按照topic去筛选消息会变得及其低效，为了提高根据主题检索消息的效率，RocketMQ引入了ConsumeQueue文件，俗成消费队列文件
- **index文件**：**关系型数据库可以按照字段属性进行记录检索，作为一款主要面向业务开发的消息中间件，RocketMQ也提供了基于消息属性的检索能力，底层的核心设计理念是为Commitlog文件建立哈希索引，并存储在Index文件中**



- 在RocketMQ中顺序写入到Commitlog文件后，ConsumeQueue与Index文件都是异步构建的，其数据流向图如下

![1674133236559](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803370.png)

## Commitlog文件

- 消息主体以及元数据的存储主体，存储Producer端写入的消息主体内容，消息内容不是定长的。单个文件大小默认1G

### CommitLog的命名规则

- 从上面的图中也可以看得出来，其文件的命名也及其巧妙，使用该存储在消息文件中的第一个全局偏移量来命名文件文件名长度为20位，左边补零，剩余为起始偏移量，比如00000000000000000000代表了第一个文件，起始偏移量为0，文件大小为1G=1073741824；消息主要是顺序写入日志文件，当文件满了，写入下一个文件；这样的设计主要是方便根据消息的物理偏移量，快速定位到消息所在的物理文件，第二个文件为00000000001073741824，起始偏移量为1073741824，以此类推

### CommitLog的性能读写

- RocketMQ在消息写入过程中追求极致的磁盘的读写性能。所有主题的消息随着到达Broker的**顺序写入**commitlog文件。Commitlog文件使用顺序写，极大提高了文件的写性能。 当顺序依次追加到文件中，消息一旦写入，就无法再进行修改

- 基于磁盘文件的与基于内存读取机制有一个本质的不同点，就是在内存读取模式下基本上是现成的数据结构，例如，数据、集合或者哈希表等，对数据的读写非常方便，但是针对于磁盘存储读取的Commitlog文件，我们该如何如何搜索，这时候我们引入了ConsumeQueue

- RocketMQ与关系型数据会为每一条数据引入一个ID主键，在基于磁盘的读取机制中，也会为一条Message引入一个唯一标志：**消息物理偏移量**，即消息存储在文件的起始位置
- 正是有了物理偏移量的概念，这也与上面提到的Commitlog的文件名命名相互呼应，这样做的好处是给出任意一个消息的物理偏移量，例如消息偏移量为 12345678，可以通过二分法进行查找，快速定位这个文件在第一个文件中，然后用消息的物理偏移量减去该文件的名称所得到的差值，就是在该文件中的绝对地址

![1674133537310](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803391.png)

## ConsumeQueue的读取模式

- ConsumeQueue消息消费队列，引入的目的主要是提高消息消费的性能，由于RocketMQ是基于主题topic的订阅模式，消息消费是针对主题进行的，如果要遍历commitlog文件中根据topic检索消息是非常低效的

- ConsumeQueue文件是Commitlog文件基于Topic的索引文件，主要用于消费者根据Topic消费消息，其组织方式为/topic/queue，同一个队列中存在多个文件
- Consumer即可根据ConsumeQueue来查找待消费的消息。其中，ConsumeQueue（逻辑消费队列）作为消费消息的索引，保存了指定Topic下的队列消息在**CommitLog中的起始物理偏移量offset**，**消息大小size**和**消息Tag的HashCode值**

> 注意：ConsumeQueue中的每个条目长度固定20个字节（**8字节commitlog物理偏移量**、**4字节消息长度**、**8字节tag hashcode**（**这里不是存储tag的原始字符串，而选择存储hashcode**）），每个条目的长度固定，可以使用访问类似数组下标的方式快速定位条目，极大地提高了ConsumeQueue文件的读取性能

### ConsumeQueue的读取消息偏移量

- 首先，消息消费者根据topic、消息消费进度（ConsumeQueue逻辑偏移量），即第几个ConsumeQueue条目，这样的消费进度去访问消息的方法为使用逻辑偏移量logicOffset * 20即可找到该条目的起始偏移量（ConsumeQueue文件中的偏移量），然后读取该偏移量后20个字节即得到一个条目，**无须遍历ConsumeQueue文件**

![1674133660257](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803700.png)

> - 第N个消ConsumeQueue的元素数据的索引开始：（N-1）* 20+1
> - 第N个消ConsumeQueue的元素数据的索引结束：（N）* 20

- ConsumeQueue文件夹如下：topic/queue/file三层组织结构，地址：$HOME/store/ConsumeQueue/{topic}/{queueId}/{fileName}。单个ConsumerQueue文件由30W个条目组成，可以像数组一样随机访问每一个条目，每个ConsumeQueue文件大小约5.72M

![1674133798688](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803032.png)

## 存储结构分析总结

- 数据单独存储到一个Commit Log完全
- 队列实际只存储消息在Commit Log的位置信息，并且串行方式刷盘

### 这样做的优点

- 队列轻量化，单个队列数据量非常少
- 对磁盘的访问串行化，避免磁盘竟争，不会因为队列增加导致 IOWAIT 增高

### 这样做的缺点

- 写虽然完全是顺序写，但是读却变成了完全的随机读
- 读一条消息，会先读 Consume Queue，再读 Commit Log，增加了开销
- 要保证 Commit Log 与 Consume Queue 完全的一致，增加了编程的复杂度

#### 以上缺点如何克服

- **随机读，尽可能让读命中PAGECACHE，减少 IO 读操作，所以内存越大越好**
- **访问 PAGECACHE 时，即使只访问1k 的消息，系统也会提前预读出更多数据，在下次读时，就可能命中内存**
- **随机访问 Commit Log 磁盘数据，系统 IO 调度算法设置为` NOOP `方式，会在一定程度上将完全的随机读变成顺序跳跃方式，而顺序跳跃方式读较完全的随机读性能会高5倍以上**

> **4k的消息在完全随机访问情况下，仍然可以达到8K次每秒以上的读性能**。**RocketMQ与Kafka相比具有一个强大的优势，就是支持按消息属性检索消息，引入consumequeue文件解决了基于topic查找的问题，但如果想基于消息的某一个属性查找消息，ConsumeQueue文件就无能为力了**，为了解决此问题RocketMQ引入了Index索引文件，实现基于文件的哈希索引

## IndexFile文件

- IndexFile文件基于物理磁盘文件实现Hash索引。其文件由40字节的文件头、500万个哈希槽，每个哈希槽4个字节，最后由2000万个Index条目，每个条目由20个字节构成，分别为4字节索引key的hashcode、8字节消息物理偏移量、4字节时间戳、4字节的前一个Index条目（哈希冲突的链表结构）

- IndexFile的文件存储结构如下图所示

![1674134078438](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803695.png)

> 其原理和读取方式与ConsumerQueue较为相似，至此不过多赘述。对于IndexFile文件和ConsumerQueue文件都是，Broker端的后台服务线程—ReputMessageService不停地分发请求并异步构建ConsumeQueue（逻辑消费队列）和IndexFile（索引文件）数据

## 页缓存与内存映射

- RocketMQ中，ConsumeQueue逻辑消费队列存储的数据较少，并且是顺序读取，在PageCache机制的预读取作用下，ConsumeQueue文件的读性能几乎接近读内存，即使在有消息堆积情况下也不会影响性能。而对于CommitLog消息存储的日志数据文件来说，读取消息内容时候会产生较多的随机访问读取，严重影响性能。（此时块存储采用SSD的话），随机读的性能也会有所提升

### 内存映射mmap

- 虽然基于磁盘的顺序写可以极大提高IO的写效率，但如果基于文件的存储采用常规的JAVA文件操作API，例如 FileOutputStream等，其性能提升会很有限，RocketMQ引入了内存映射，将磁盘文件映射到内存中，以操作内存的方式操作磁盘，性能又提升了一个档次

> **JAVA中可通过FileChannel的map方法创建内存映射文件。在Linux服务器中由该方法创建的文件使用的是操作系统的pagecache，即页缓存**

- **Linux操作系统中的内存使用策略时会尽可能地利用机器的物理内存，并常驻内存中，就是所谓的页缓存。在操作系统的内存不够的情况下，采用缓存置换算法，例如LRU将不常用的页缓存回收，即操作系统会自动管理这部分内存**
- **如果RocketMQ Broker进程异常退出，存储在页缓存中的数据并不会丢失，操作系统会定时将页缓存中的数据持久化到磁盘，做到数据安全可靠。不过如果是机器断电等异常情况，存储在页缓存中的数据就有可能丢失**

### 页缓存PageCache

页缓存（PageCache)是OS对文件的缓存，用于加速对文件的读写

- 程序对文件进行顺序读写的速度几乎接近于内存的读写速度，主要原因就是由于OS使用PageCache机制对读写访问操作进行了性能优化，将一部分的内存用作PageCache
- 对于数据的写入，OS会先写入至Cache内，随后通过异步的方式由pdflush内核线程将Cache内的数据刷盘至物理磁盘上
- 对于数据的读取，如果一次读取文件时出现未命中PageCache的情况，OS从物理磁盘上访问读取文件的同时，会顺序对其他相邻块的数据文件进行预读取

### 是否使用堆外内存

- RocketMQ为了降低PageCache的使用压力引入了transientStorePoolEnable机制，即内存级别的读写分离机制。默认情况下RocketMQ将消息写入PageCache，消息消费时从PageCache中读取，这样在高并发时PageCache的压力会比较大，容易出现瞬时broker busy

![1674134283222](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803827.png)

- RocketMQ还引入了transientStorePoolEnable=true，将消息先写入堆外内存并立即返回，然后异步将堆外内存中的数据提交到pagecache，再异步刷盘到磁盘中。其工作机制如下图所示

![1674134303685](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803407.png)

> **消息在消费读取时不会尝试从堆外内存中读，而是从PageCache中读取，这样就形成了内存级别的读写分离，即消息写入时主要面对堆外内存，而读消息时主要面对pagecache**。

- **优点是消息是直接写入堆外内存，然后异步写入pagecache。相比每条消息追加直接写入pagechae，其最大的优势是将消息写入PageCache操作批量化**
- **缺点是如果由于某些意外操作导致Broker进程异常退出，那么存储在堆外内存的数据会丢失，但如果是放入pagecache，broker异常退出并不会丢失消息**

## 刷盘机制

- 有了顺序写和内存映射的加持，RocketMQ的写入性能得到了极大的保证，但凡事都有利弊，引入了内存映射和页缓存机制，消息会先写入到页缓存，此时消息并没有真正持久化到磁盘。那么broker收到客户端的消息发送后，是存储到页缓存中就直接返回成功，还是要持久化到磁盘中才返回成功呢？

- 这是一个“艰难”的抉择，是在性能与消息可靠性方面进行权衡。为此，RocketMQ提供了多种策略：同步刷盘、异步刷盘

### 同步刷盘

- 同步刷盘在RocketMQ的实现中成为组提交，并不是每一条消息都必须刷盘。采用同步刷盘，每一个线程将数据追到到内存后，并向刷盘线程提交刷盘请求，然后会阻塞；刷盘线程从任务队列中获取一个任务，然后触发一次刷盘，但并不只刷与请求相关的消息，而是会直接将内存中待刷盘的所有消息一次批量刷盘，然后就可以唤醒一组请求线程，实现组刷盘

- 同步刷盘的优点是能保证消息不丢失，即向客户端返回成功就代表这条消息已被持久化到磁盘，即消息非常可靠，但这是以牺牲写入响应延迟性能为代价的，由于RocketMQ的消息是先写入pagecache，故消息丢失的可能性较小，如果能容忍一定几率的消息丢失，可以考虑使用异步刷盘。

![1674134418997](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803605.png)

- 如上图所示，只有在消息真正持久化至磁盘后RocketMQ的Broker端才会真正返回给Producer端一个成功的ACK响应。同步刷盘对MQ消息可靠性来说是一种不错的保障，但是性能上会有较大影响，一般适用于金融业务应用该模式较多

### 异步刷盘

- 异步刷盘指的是broker将消息存储到pagecache后就立即返回成功，然后开启一个异步线程定时执行FileChannel的forece方法，将内存中的数据定时刷写到磁盘，默认间隔为500ms

![1674134456279](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803507.png)

- 能够充分利用OS的PageCache的优势，只要消息写入PageCache即可将成功的ACK返回给Producer端。消息刷盘采用后台异步线程提交的方式进行，降低了读写延迟，提高了MQ的性能和吞吐量。

![1674134473843](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211803031.png)

> **GroupCommitService 从队列中拿出待刷盘请求request， 然后执行刷盘动作， 此时会将write指针与flush指针之间的所有数据刷写到磁盘中，即这里并不只是将request l对应的那一条消息刷写到磁盘**

## RocketMQ的持久化存储机制总结

- RocketMQ采用的是混合型的存储结构，即为Broker单个实例下所有的队列共用一个日志数据文件（即为CommitLog）来存储。混合型存储结构(多个Topic的消息实体内容都存储于一个CommitLog中)，针对Producer和Consumer分别采用了数据和索引部分相分离的存储结构

- Producer发送消息至Broker端，然后Broker端使用同步或者异步的方式对消息刷盘持久化，保存至CommitLog中。只要消息被刷盘持久化至磁盘文件CommitLog中，那么Producer发送的消息就不会丢失
- Consumer也就肯定有机会去消费这条消息。当无法拉取到消息后，可以等下一次消息拉取，同时服务端也支持长轮询模式，如果一个消息拉取请求未拉取到消息，Broker允许等待30s的时间，只要这段时间内有新消息到达，将直接返回给消费端

# 消息的生产

## 消息的生产过程

- Producer可以将消息写入到某Broker中的某Queue中，其经历了如下过程：
- Producer发送消息之前，会先向NameServer发出获取消息Topic的路由信息的请求
- NameServer返回该Topic的路由表及Broker列表
- Producer根据代码中指定的Queue选择策略，从Queue列表中选出一个队列，用于后续存储消息
- Produer对消息做一些特殊处理，例如，消息本身超过4M，则会对其进行压缩
- Producer向选择出的Queue所在的Broker发出RPC请求，将消息发送到选择出的Queue

> 路由表：实际是一个Map，key为Topic名称，value是一个QueueData实例列表。QueueData并不是一个Queue对应一个QueueData，而是一个Broker中该Topic的所有Queue对应一个QueueData。即，只要涉及到该Topic的Broker，一个Broker对应一个QueueData。QueueData中包含brokerName。简单来说，路由表的key为Topic名称，value则为所有涉及该Topic的BrokerName列表。

> Broker列表：其实际也是一个Map。key为brokerName，value为BrokerData。一个Broker对应一个BrokerData实例，对吗？不对。一套brokerName名称相同的Master-Slave小集群对应一个BrokerData。BrokerData中包含brokerName及一个map。该map的key为brokerId，value为该broker对应的地址。brokerId为0表示该broker为Master，非0表示Slave。

## Queue选择算法

+ 对于无序消息，其Queue选择算法，也称为消息投递算法，常见的有两种：

### 轮询算法

+ 默认选择算法。该算法保证了每个Queue中可以均匀的获取到消息。

+ 该算法存在一个问题：由于某些原因，在某些Broker上的Queue可能投递延迟较严重。从而导致Producer的缓存队列中出现较大的消息积压，影响消息的投递性能。

### 最小投递延迟算法
- 该算法会统计每次消息投递的时间延迟，然后根据统计出的结果将消息投递到时间延迟最小的Queue。如果延迟相同，则采用轮询算法投递。该算法可以有效提升消息的投递性能。
- 该算法也存在一个问题：消息在Queue上的分配不均匀。投递延迟小的Queue其可能会存在大量的消息。而对该Queue的消费者压力会增大，降低消息的消费能力，可能会导致MQ中消息的堆积

##  消息的存储

+ RocketMQ中的消息存储在本地文件系统中，这些相关文件默认在当前用户主目录下的store目录中

![1674108727072](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804832.png)

- abort：该文件在Broker启动后会自动创建，正常关闭Broker，该文件会自动消失。若在没有启动Broker的情况下，发现这个文件是存在的，则说明之前Broker的关闭是非正常关闭。
- checkpoint：其中存储着commitlog、consumequeue、index文件的最后刷盘时间戳
- commitlog：其中存放着commitlog文件，而消息是写在commitlog文件中的
- config：存放着Broker运行期间的一些配置数据
- consumequeue：其中存放着consumequeue文件，队列就存放在这个目录中
- index：其中存放着消息索引文件indexFile
- lock：运行期间使用到的全局资源锁

### commitlog文件

> 说明：在很多资料中commitlog目录中的文件简单就称为commitlog文件。但在源码中，该文件
> 被命名为mappedFile

#### 目录与文件

- commitlog目录中存放着很多的mappedFile文件，当前Broker中的所有消息都是落盘到这些mappedFile文件中的。mappedFile文件大小为1G（小于等于1G），文件名由20位十进制数构成，表示当前文件的第一条消息的起始位移偏移量
- 第一个文件名一定是20位0构成的。因为第一个文件的第一条消息的偏移量commitlog offset为0当第一个文件放满时，则会自动生成第二个文件继续存放消息。假设第一个文件大小是1073741820字节（1G = 1073741824字节），则第二个文件名就是00000000001073741824。以此类推，第n个文件名应该是前n-1个文件大小之和
- 一个Broker中所有mappedFile文件的commitlog offset是连续的需要注意的是，一个Broker中仅包含一个commitlog目录，所有的mappedFile文件都是存放在该目录中的。即无论当前Broker中存放着多少Topic的消息，这些消息都是被顺序写入到了mappedFile文件中的。也就是说，这些消息在Broker中存放时并没有被按照Topic进行分类存放。mappedFile文件是顺序读写的文件，所有其访问效率很高。无论是SSD磁盘还是SATA磁盘，通常情况下，顺序存取效率都会高于随机存取

#### 消息单元

+ **mappedFile**

![1674110088861](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804439.png)

+ mappedFile文件内容由一个个的消息单元构成。每个消息单元中包含消息总长度MsgLen、消息的物理位置physicalOffset、消息体内容Body、消息体长度BodyLength、消息主题Topic、Topic长度TopicLength、消息生产者BornHost、消息发送时间戳BornTimestamp、消息所在的队列QueueId、消息在Queue中存储的偏移量QueueOffset等近20余项消息相关属性。

> 需要注意到，消息单元中是包含Queue相关属性的。所以，我们在后续的学习中，就需要十分留意commitlog与queue间的关系是什么？
>
> 一个mappedFile文件中第m+1个消息单元的commitlog offset偏移量L(m+1) = L(m) + MsgLen(m) (m >= 0)

## 定时消息

**定时消息（延迟队列）是指消息发送到 broker 后，不会立即被消费，等待特定时间投递给真正的 topic**。 broker 有配置项`messageDelayLevel`，默认值为“1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h”，18个level。可以配置自定义 `messageDelayLevel`。注意，`messageDelayLevel `是 broker 的属性，不属于某个 topic。发消息时，设置 delayLevel 等级即可：msg.setDelayLevel(level)。level有以下三种情况：

- level == 0，消息为非延迟消息
- 1<=level<=maxLevel，消息延迟特定时间，例如level==1，延迟1s
- level > maxLevel，则level== maxLevel，例如level==20，延迟2h

定时消息会暂存在名为 `SCHEDULE_TOPIC_XXXX` 的 topic 中，并根据 delayTimeLevel 存入特定的 queue，queueId = delayTimeLevel – 1，即一个 queue 只存相同延迟的消息，保证具有相同发送延迟的消息能够顺序消费。broker会调度地消费 `SCHEDULE_TOPIC_XXXX`，将消息写入真实的 topic。

需要注意的是，定时消息会在第一次写入和调度写入真实topic时都会计数，因此发送数量、tps都会变高

# indexFile

+ 除了通过通常的指定Topic进行消息消费外，RocketMQ还提供了根据key进行消息查询的功能。该查询是通过store目录中的index子目录中的indexFile进行索引实现的快速查询。当然，这个indexFile中的索引数据是在包含了key的消息被发送到Broker时写入的。如果消息中没有包含key，则不会写入

## 索引条目结构

+ 每个Broker中会包含一组indexFile，每个indexFile都是以一个时间戳命名的（这个indexFile被创建时的时间戳）。每个indexFile文件由三部分构成：indexHeader，slots槽位，indexes索引数据。每个indexFile文件中包含500w个slot槽。而每个slot槽又可能会挂载很多的index索引单元

![1674189773703](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804124.png)

+ indexHeader固定40个字节，其中存放着如下数据

![1674189793562](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804805.png)

- beginTimestamp：该indexFile中第一条消息的存储时间
- endTimestamp：该indexFile中最后一条消息存储时间
- beginPhyoffset：该indexFile中第一条消息在commitlog中的偏移量commitlog offset
- endPhyoffset：该indexFile中最后一条消息在commitlog中的偏移量commitlog offset
- hashSlotCount：已经填充有index的slot数量（并不是每个slot槽下都挂载有index索引单元，这里统计的是所有挂载了index索引单元的slot槽的数量）
- indexCount：该indexFile中包含的索引单元个数（统计出当前indexFile中所有slot槽下挂载的所有index索引单元的数量之和）
- indexFile中最复杂的是Slots与Indexes间的关系。在实际存储时，Indexes是在Slots后面的，但为了便于理解，将它们的关系展示为如下形式

![1674189896174](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804280.png)

+ key的hash值 % 500w 的结果即为slot槽位，然后将该slot值修改为该index索引单元的indexNo，根据这个indexNo可以计算出该index单元在indexFile中的位置。不过，该取模结果的重复率是很高的，为了解决该问题，在每个index索引单元中增加了preIndexNo，用于指定该slot中当前index索引单元的前一个index索引单元。而slot中始终存放的是其下最新的index索引单元的indexNo，这样的话，只要找到了slot就可以找到其最新的index索引单元，而通过这个index索引单元就可以找到其之前的所有index索引单元。

> indexNo是一个在indexFile中的流水号，从0开始依次递增。即在一个indexFile中所有indexNo是以此递增的。indexNo在index索引单元中是没有体现的，其是通过indexes中依次数出来的。index索引单元默写20个字节，其中存放着以下四个属

![1674189935255](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804673.png)

- keyHash：消息中指定的业务key的hash值
- phyOffset：当前key对应的消息在commitlog中的偏移量commitlog offset
- timeDiff：当前key对应消息的存储时间与当前indexFile创建时间的时间差
- preIndexNo：当前slot下当前index索引单元的前一个index索引单元的indexNo

## indexFile的创建

- indexFile的文件名为当前文件被创建时的时间戳。这个时间戳有什么用处呢？
- 根据业务key进行查询时，查询条件除了key之外，还需要指定一个要查询的时间戳，表示要查询不大于该时间戳的最新的消息，即查询指定时间戳之前存储的最新消息。这个时间戳文件名可以简化查询，提高查询效率。具体后面会详细讲解
- indexFile文件是何时创建的？其创建的条件（时机）有两个：
- 当第一条带key的消息发送来后，系统发现没有indexFile，此时会创建第一个indexFile文件当一个indexFile中挂载的index索引单元数量超出2000w个时，会创建新的indexFile。当带key的消息发送到来后，系统会找到最新的indexFile，并从其indexHeader的最后4字节中读取到indexCount。若indexCount >= 2000w时，会创建新的indexFile
- 由于可以推算出，一个indexFile的最大大小是：(40 + 500w * 4 + 2000w * 20)字节

## 查询流程

+ 当消费者通过业务key来查询相应的消息时，其需要经过一个相对较复杂的查询流程。不过，在分析查询流程之前，首先要清楚几个定位计算式子

$$
计算指定消息key的slot槽位序号：
slot槽位序号 = key的hash \%\ 500w
$$

$$
计算槽位序号为n的slot在indexFile中的起始位置：
slot(n)位置 = 40 + (n - 1) * 4
$$

$$
计算indexNo为m的index在indexFile中的位置：
index(m)位置 = 40 + 500w * 4 + (m - 1) * 20
$$

> 40为indexFile中indexHeader的字节数
> 500w * 4 是所有slots所占的字节数

+ 具体查询流程如下

![1674190223610](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804089.png)

# 消息的消费

+ 消费者从Broker中获取消息的方式有两种：pull拉取方式和push推动方式。消费者组对于消息消费的模式又分为两种：集群消费Clustering和广播消费Broadcasting。

## 获取消费类型

### 拉取式消费

+ Consumer主动从Broker中拉取消息，主动权由Consumer控制。一旦获取了批量消息，就会启动消费过程。不过，该方式的实时性较弱，即Broker中有了新的消息时消费者并不能及时发现并消费。由于拉取时间间隔是由用户指定的，所以在设置该间隔时需要注意平稳：间隔太短，空请求比例会增加；间隔太长，消息的实时性太差

### 推送式消费

+ 该模式下Broker收到数据后会主动推送给Consumer。该获取方式一般实时性较高。该获取方式是典型的发布-订阅模式，即Consumer向其关联的Queue注册了监听器，一旦发现有新的消息到来就会触发回调的执行，回调方法是Consumer去Queue中拉取消息。而这些都是基于Consumer与Broker间的长连接的。长连接的维护是需要消耗系统资源的

### 对比

- pull：需要应用去实现对关联Queue的遍历，实时性差；但便于应用控制消息的拉取
- push：封装了对关联Queue的遍历，实时性强，但会占用较多的系统资源

## 消费模式

### 广播消费

![1674191476889](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804304.png)

+ 广播消费模式下，相同Consumer Group的每个Consumer实例都接收同一个Topic的全量消息。即每条消息都会被发送到Consumer Group中的每个Consumer

### 集群消费

![1674191507205](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804281.png)

+ 集群消费模式下，相同Consumer Group的每个Consumer实例平均分摊同一个Topic的消息。即每条消息只会被发送到Consumer Group中的某个Consumer

### 消息进度保存

- 广播模式：消费进度保存在consumer端。因为广播模式下consumer group中每个consumer都会消费所有消息，但它们的消费进度是不同。所以consumer各自保存各自的消费进度
- 集群模式：消费进度保存在broker中。consumer group中的所有consumer共同消费同一个Topic中的消息，同一条消息只会被消费一次。消费进度会参与到了消费的负载均衡中，故消费进度是需要共享的。下图是broker中存放的各个Topic的各个Queue的消费进

![1674191555609](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804515.png)

### Rebalance机制

+ Rebalance机制讨论的前提是：集群消费

> 什么是Rebalance
>
> Rebalance即再均衡，指的是，将⼀个Topic下的多个Queue在同⼀个Consumer Group中的多个Consumer间进行重新分配的过程

![1674191600123](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804744.png)

+ Rebalance机制的本意是为了提升消息的并行消费能力。例如，⼀个Topic下5个队列，在只有1个消费者的情况下，这个消费者将负责消费这5个队列的消息。如果此时我们增加⼀个消费者，那么就可以给其中⼀个消费者分配2个队列，给另⼀个分配3个队列，从而提升消息的并行消费能力

#### Rebalance限制

+ 由于⼀个队列最多分配给⼀个消费者，因此当某个消费者组下的消费者实例数量大于队列的数量时，多余的消费者实例将分配不到任何队列

#### Rebalance危害

- Rebalance的在提升消费能力的同时，也带来一些问题
- 消费暂停：在只有一个Consumer时，其负责消费所有队列；在新增了一个Consumer后会触发Rebalance的发生。此时原Consumer就需要暂停部分队列的消费，等到这些队列分配给新的Consumer后，这些暂停消费的队列才能继续被消费
- 消费重复：Consumer 在消费新分配给自己的队列时，必须接着之前Consumer 提交的消费进度的offset继续消费。然而默认情况下，offset是异步提交的，这个异步性导致提交到Broker的offset与Consumer实际消费的消息并不一致。这个不一致的差值就是可能会重复消费的消息

> 同步提交：consumer提交了其消费完毕的一批消息的offset给broker后，需要等待broker的成功ACK。当收到ACK后，consumer才会继续获取并消费下一批消息。在等待ACK期间，consumer是阻塞的
>
> 异步提交：consumer提交了其消费完毕的一批消息的offset给broker后，不需要等待broker的成功ACK。consumer可以直接获取并消费下一批消息
>
> 对于一次性读取消息的数量，需要根据具体业务场景选择一个相对均衡的是很有必要的。因为数量过大，系统性能提升了，但产生重复消费的消息数量可能会增加；数量过小，系统性能会下降，但被重复消费的消息数量可能会减少

+ 消费突刺：由于Rebalance可能导致重复消费，如果需要重复消费的消息过多，或者因为Rebalance暂停时间过长从而导致积压了部分消息。那么有可能会导致在Rebalance结束之后瞬间需要消费很多消息。

#### Rebalance产生的原因

+ 导致Rebalance产生的原因，无非就两个：消费者所订阅Topic的Queue数量发生变化，或消费者组中消费者的数量发生变化

> 1）Queue数量发生变化的场景：
>
> Broker扩容或缩容
>
> Broker升级运维
>
> Broker与NameServer间的网络异常
>
> Queue扩容或缩容
>
> 2）消费者数量发生变化的场景：
>
> Consumer Group扩容或缩容
>
> Consumer升级运维
>
> Consumer与NameServer间网络异常

#### Rebalance过程

+ 在Broker中维护着多个Map集合，这些集合中动态存放着当前Topic中Queue的信息、Consumer Group中Consumer实例的信息。一旦发现消费者所订阅的Queue数量发生变化，或消费者组中消费者的数量发生变化，立即向Consumer Group中的每个实例发出Rebalance通知

> TopicConfigManager：key是topic名称，value是TopicConfig。TopicConfig中维护着该Topic中所有Queue的数据
>
> ConsumerManager：key是Consumser Group Id，value是ConsumerGroupInfo。
>
> ConsumerGroupInfo中维护着该Group中所有Consumer实例数据。
>
> ConsumerOffsetManager：key为Topic与订阅该Topic的Group的组合,即topic@group，value是一个内层Map。内层Map的key为QueueId，内层Map的value为该Queue的消费进度offset

+ Consumer实例在接收到通知后会采用Queue分配算法自己获取到相应的Queue，即由Consumer实例自主进行Rebalance

#### Queue分配算法

+ 一个Topic中的Queue只能由Consumer Group中的一个Consumer进行消费，而一个Consumer可以同时消费多个Queue中的消息。那么Queue与Consumer间的配对关系是如何确定的，即Queue要分配给哪个Consumer进行消费，也是有算法策略的。常见的有四种策略。这些策略是通过在创建Consumer时的构造器传进去的

##### 平均分配策略

![1674192063624](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211804085.png)

- 该算法是要根据avg = QueueCount / ConsumerCount 的计算结果进行分配的。如果能够整除，则按顺序将avg个Queue逐个分配Consumer；如果不能整除，则将多余出的Queue按照Consumer顺序逐个分配
- 该算法即，先计算好每个Consumer应该分得几个Queue，然后再依次将这些数量的Queue逐个分配个Consumer

##### 环形平均策略

![1674192122702](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805022.png)

+ 环形平均算法是指，根据消费者的顺序，依次在由queue队列组成的环形图中逐个分配。该算法不用事先计算每个Consumer需要分配几个Queue，直接一个一个分即可

##### 一致性hash策略

![1674192158826](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805422.png)

- 该算法会将consumer的hash值作为Node节点存放到hash环上，然后将queue的hash值也放到hash环上，通过顺时针方向，距离queue最近的那个consumer就是该queue要分配的consumer
- 该算法存在的问题：分配不均

##### 同机房策略

![1674192189195](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805272.png)

+ 该算法会根据queue的部署机房位置和consumer的位置，过滤出当前consumer相同机房的queue。然后按照平均分配策略或环形平均策略对同机房queue进行分配。如果没有同机房queue，则按照平均分配策略或环形平均策略对所有queue进行分配

##### 对比

- 一致性hash算法存在的问题
  - 两种平均分配策略的分配效率较高，一致性hash策略的较低。因为一致性hash算法较复杂。另外，一致性hash策略分配的结果也很大可能上存在不平均的情况
- 一致性hash算法存在的意义
  - 其可以有效减少由于消费者组扩容或缩容所带来的大量的Rebalance

![1674192265183](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805152.png)

+ 一致性hash算法的应用场景
  - Consumer数量变化较频繁的场景

#### 至少一次原则

- RocketMQ有一个原则：每条消息必须要被成功消费一次
- 那么什么是成功消费呢？Consumer在消费完消息后会向其消费进度记录器提交其消费消息的offset，offset被成功记录到记录器中，那么这条消费就被成功消费了

> 什么是消费进度记录器？
>
> 对于广播消费模式来说，Consumer本身就是消费进度记录器
>
> 对于集群消费模式来说，Broker是消费进度记录器

## 订阅关系的一致性

+ 订阅关系的一致性指的是，同一个消费者组（Group ID相同）下所有Consumer实例所订阅的Topic与Tag及对消息的处理逻辑必须完全一致。否则，消息消费的逻辑就会混乱，甚至导致消息丢失。

### 正确订阅关系

+ 多个消费者组订阅了多个Topic，并且每个消费者组里的多个消费者实例的订阅关系保持了一致

![1674196601337](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805426.png)

### 错误订阅关系

+ 一个消费者组订阅了多个Topic，但是该消费者组里的多个Consumer实例的订阅关系并没有保持一致

**错误类型**

+ 订阅了不同Topic
+ 订阅了不同Tag
+ 订阅了不同数量的Topic

## offset管理

- 这里的offset指的是Consumer的消费进度offset
- 消费进度offset是用来记录每个Queue的不同消费组的消费进度的。根据消费进度记录器的不同，可以分为两种模式：本地模式和远程模式

### offset本地管理模式

- 当消费模式为广播消费时，offset使用本地模式存储。因为每条消息会被所有的消费者消费，每个消费者管理自己的消费进度，各个消费者之间不存在消费进度的交集
- Consumer在广播消费模式下offset相关数据以json的形式持久化到Consumer本地磁盘文件中，默认文件路径为当前用户主目录下的.rocketmq_offsets/${clientId}/${group}/Offsets.json 。其中${clientId}为当前消费者id，默认为ip@DEFAULT；${group}为消费者组名称

### offset远程管理模式

- 当消费模式为集群消费时，offset使用远程模式管理。因为所有Cosnumer实例对消息采用的是均衡消费，所有Consumer共享Queue的消费进度。Consumer在集群消费模式下offset相关数据以json的形式持久化到Broker磁盘文件中，文件路径为当前用户主目录下的store/config/consumerOffset.json 
- Broker启动时会加载这个文件，并写入到一个双层Map（ConsumerOffsetManager）。外层map的key为topic@group，value为内层map。内层map的key为queueId，value为offset。当发生Rebalance时，新的Consumer会从该Map中获取到相应的数据来继续消费。集群模式下offset采用远程管理模式，主要是为了保证Rebalance机制

### offset用途

消费者是如何从最开始持续消费消息的？消费者要消费的第一条消息的起始位置是用户自己通过consumer.setConsumeFromWhere()方法指定的

在Consumer启动后，其要消费的第一条消息的起始位置常用的有三种，这三种位置可以通过枚举类型常量设置。这个枚举类型为ConsumeFromWhere

![1674197190671](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805275.png)

> CONSUME_FROM_LAST_OFFSET：从queue的当前最后一条消息开始消费
>
> CONSUME_FROM_FIRST_OFFSET：从queue的第一条消息开始消费
>
> CONSUME_FROM_TIMESTAMP：从指定的具体时间戳位置的消息开始消费。这个具体时间戳是通过另外一个语句指定的 
>
> consumer.setConsumeTimestamp(“20210701080000”) yyyyMMddHHmmss

+ 当消费完一批消息后，Consumer会提交其消费进度offset给Broker，Broker在收到消费进度后会将其更新到那个双层Map（ConsumerOffsetManager）及consumerOffset.json文件中，然后向该Consumer进行ACK，而ACK内容中包含三项数据：当前消费队列的最小offset（minOffset）、最大offset（maxOffset）、及下次消费的起始offset（nextBeginOffset）

### 重试队列

![1674197251508](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805525.png)

+ 当rocketMQ对消息的消费出现异常时，会将发生异常的消息的offset提交到Broker中的重试队列。系统在发生消息消费异常时会为当前的topic@group创建一个重试队列，该队列以%RETRY%开头，到达重试时间后进行消费重试

### offset的同步提交与异步提交

- 集群消费模式下，Consumer消费完消息后会向Broker提交消费进度offset，其提交方式分为两种
- 同步提交： 消费者在消费完一批消息后会向broker提交这些消息的offset，然后等待broker的成功响应。若在等待超时之前收到了成功响应，则继续读取下一批消息进行消费（从ACK中获取nextBeginOffset）。若没有收到响应，则会重新提交，直到获取到响应。而在这个等待过程中，消费者是阻塞的。其严重影响了消费者的吞吐量
- 异步提交： 消费者在消费完一批消息后向broker提交offset，但无需等待Broker的成功响应，可以继续读取并消费下一批消息。这种方式增加了消费者的吞吐量。但需要注意，broker在收到提交的offset后，还是会向消费者进行响应的。可能还没有收到ACK，此时Consumer会从Broker中直接获取nextBeginOffset

## 消费幂等

### 什么是消费幂等

- 当出现消费者对某条消息重复消费的情况时，重复消费的结果与消费一次的结果是相同的，并且多次消费并未对业务系统产生任何负面影响，那么这个消费过程就是消费幂等的
- 幂等：若某操作执行多次与执行一次对系统产生的影响是相同的，则称该操作是幂等的。在互联网应用中，尤其在网络不稳定的情况下，消息很有可能会出现重复发送或重复消费。如果重复的消息可能会影响业务处理，那么就应该对消息做幂等处理

### 消息重复的场景分析

+ 什么情况下可能会出现消息被重复消费呢？最常见的有以下三种情况：

#### 发送时消息重复

+ 当一条消息已被成功发送到Broker并完成持久化，此时出现了网络闪断，从而导致Broker对Producer应答失败。 如果此时Producer意识到消息发送失败并尝试再次发送消息，此时Broker中就可能会出现两条内容相同并且Message ID也相同的消息，那么后续Consumer就一定会消费两次该消息

#### 消费时消息重复

+ 消息已投递到Consumer并完成业务处理，当Consumer给Broker反馈应答时网络闪断，Broker没有接收到消费成功响应。为了保证消息至少被消费一次的原则，Broker将在网络恢复后再次尝试投递之前已被处理过的消息。此时消费者就会收到与之前处理过的内容相同、Message ID也相同的消息

#### Rebalance时消息重复

+ 当Consumer Group中的Consumer数量发生变化时，或其订阅的Topic的Queue数量发生变化时，会触发Rebalance，此时Consumer可能会收到曾经被消费过的消息

### 通用解决方案

#### 两要素

- 幂等解决方案的设计中涉及到两项要素：幂等令牌，与唯一性处理。只要充分利用好这两要素，就可以设计出好的幂等解决方案
- 幂等令牌：是生产者和消费者两者中的既定协议，通常指具备唯⼀业务标识的字符串。例如，订单号、流水号。一般由Producer随着消息一同发送来的
- 唯一性处理：服务端通过采用⼀定的算法策略，保证同⼀个业务逻辑不会被重复执行成功多次。例如，对同一笔订单的多次支付操作，只会成功一次

#### 解决方案

- 对于常见的系统，幂等性操作的通用性解决方案是：
- 首先通过缓存去重。在缓存中如果已经存在了某幂等令牌，则说明本次操作是重复性操作；若缓
  存没有命中，则进入下一步。
- 在唯一性处理之前，先在数据库中查询幂等令牌作为索引的数据是否存在。若存在，则说明本次
  操作为重复性操作；若不存在，则进入下一步。
- 在同一事务中完成三项操作：唯一性处理后，将幂等令牌写入到缓存，并将幂等令牌作为唯一索
  引的数据写入到DB中。

> 第1步已经判断过是否是重复性操作了，为什么第2步还要再次判断？能够进入第2步，说明已经不是重复操作了，第2次判断是否重复？当然不重复。一般缓存中的数据是具有有效期的。缓存中数据的有效期一旦过期，就是发生缓存穿透，使请求直接就到达了DBMS。

#### 解决方案举例

**以支付场景为例**

- 当支付请求到达后，首先在Redis缓存中却获取key为支付流水号的缓存value。若value不空，则说明本次支付是重复操作，业务系统直接返回调用侧重复支付标识；若value为空，则进入下一步操作
- 到DBMS中根据支付流水号查询是否存在相应实例。若存在，则说明本次支付是重复操作，业务系统直接返回调用侧重复支付标识；若不存在，则说明本次操作是首次操作，进入下一步完成唯一性处理

**在分布式事务中完成三项操作**

- 完成支付任务
- 将当前支付流水号作为key，任意字符串作为value，通过set(key, value, expireTime)将数据写入到Redis缓存
- 将当前支付流水号作为主键，与其它相关数据共同写入到DBMS

#### 消费幂等的实现

- 消费幂等的解决方案很简单：为消息指定不会重复的唯一标识。因为Message ID有可能出现重复的情
  况，所以真正安全的幂等处理，不建议以Message ID作为处理依据。最好的方式是以业务唯一标识作为
  幂等处理的关键依据，而业务的唯一标识可以通过消息Key设置
- 以支付场景为例，可以将消息的Key设置为订单号，作为幂等处理的依据。具体代码示例如下

```java
Message message = new Message();
message.setKey("ORDERID_100");
SendResult sendResult = producer.send(message);
```

+ 消费者收到消息时可以根据消息的Key即订单号来实现消费幂等

```java
consumer.registerMessageListener(new MessageListenerConcurrently() {
    @Override
    public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,
ConsumeConcurrentlyContext context) {
        for(MessageExt msg:msgs){
            String key = msg.getKeys();
            // 根据业务唯一标识Key做幂等处理
            // ……
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
});
```

> RocketMQ能够保证消息不丢失，但不能保证消息不重复

## 消息堆积与消费延迟

### 概念

- 消息处理流程中，如果Consumer的消费速度跟不上Producer的发送速度，MQ中未处理的消息会越来越多（进的多出的少），这部分消息就被称为堆积消息。消息出现堆积进而会造成消息的消费延迟。以下场景需要重点关注消息堆积和消费延迟问题
- 业务系统上下游能力不匹配造成的持续堆积，且无法自行恢复
- 业务系统对消息的消费实时性要求较高，即使是短暂的堆积造成的消费延迟也无法接受

### 产生原因分析

![1674200420262](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805507.png)

+ Consumer使用长轮询Pull模式消费消息时，分为以下两个阶段

#### 消息拉取

- Consumer通过长轮询Pull模式批量拉取的方式从服务端获取消息，将拉取到的消息缓存到本地缓冲队列中。对于拉取式消费，在内网环境下会有很高的吞吐量，所以这一阶段一般不会成为消息堆积的瓶颈

  > 一个单线程单分区的低规格主机(Consumer，4C8G)，其可达到几万的TPS。如果是多个分区多个线程，则可以轻松达到几十万的TPS

#### 消息消费

+ Consumer将本地缓存的消息提交到消费线程中，使用业务消费逻辑对消息进行处理，处理完毕后获取到一个结果。这是真正的消息消费过程。此时Consumer的消费能力就完全依赖于消息的消费耗时和消费并发度了。如果由于业务处理逻辑复杂等原因，导致处理单条消息的耗时较长，则整体的消息吞吐量肯定不会高，此时就会导致Consumer本地缓冲队列达到上限，停止从服务端拉取消息

#### 结论

+ 消息堆积的主要瓶颈在于客户端的消费能力，而消费能力由消费耗时和消费并发度决定。注意，消费耗时的优先级要高于消费并发度。即在保证了消费耗时的合理性前提下，再考虑消费并发度问题

### 消费耗时

- 影响消息处理时长的主要因素是代码逻辑。而代码逻辑中可能会影响处理时长代码主要有两种类型：CPU内部计算型代码和外部I/O操作型代码
- 通常情况下代码中如果没有复杂的递归和循环的话，内部计算耗时相对外部I/O操作来说几乎可以忽略。所以外部IO型代码是影响消息处理时长的主要症结所在

> 外部IO操作型代码举例：
>
> 读写外部数据库，例如对远程MySQL的访问
>
> 读写外部缓存系统，例如对远程Redis的访问
>
> 下游系统调用，例如Dubbo的RPC远程调用，Spring Cloud的对下游系统的Http接口调用关于下游系统调用逻辑需要进行提前梳理，掌握每个调用操作预期的耗时，这样做是为了能够判断消费逻辑中IO操作的耗时是否合理。通常消息堆积是由于下游系统出现了服务异常或达到了DBMS容量限制，导致消费耗时增加
>
> 服务异常，并不仅仅是系统中出现的类似500这样的代码错误，而可能是更加隐蔽的问题。例如，网络带宽问题
>
> 达到了DBMS容量限制，其也会引发消息的消费耗时增加

### 消费并发度

+ 一般情况下，消费者端的消费并发度由单节点线程数和节点数量共同决定，其值为单节点线程数* 节点数量。不过，通常需要优先调整单节点的线程数，若单机硬件资源达到了上限，则需要通过横向扩展来提高消费并发度

> 单节点线程数，即单个Consumer所包含的线程数量
>
> 节点数量，即Consumer Group所包含的Consumer数量
>
> 对于普通消息、延时消息及事务消息，并发度计算都是单节点线程数*节点数量。但对于顺序消息则是不同的。顺序消息的消费并发度等于Topic的Queue分区数量
>
> 1）全局顺序消息：该类型消息的Topic只有一个Queue分区。其可以保证该Topic的所有消息被顺序消费。为了保证这个全局顺序性，Consumer Group中在同一时刻只能有一个Consumer的一个线程进行消费。所以其并发度为1
>
> 2）分区顺序消息：该类型消息的Topic有多个Queue分区。其仅可以保证该Topic的每个Queue分区中的消息被顺序消费，不能保证整个Topic中消息的顺序消费。为了保证这个分区顺序性，每个Queue分区中的消息在Consumer Group中的同一时刻只能有一个Consumer的一个线程进行消费。即，在同一时刻最多会出现多个Queue分蘖有多个Consumer的多个线程并行消费。所以其并发度为Topic的分区数量

### 单机线程数计算

+ 对于一台主机中线程池中线程数的设置需要谨慎，不能盲目直接调大线程数，设置过大的线程数反而会带来大量的线程切换的开销。理想环境下单节点的最优线程数计算模型为：C *（T1 + T2）/ T1。

- C：CPU内核数
- T1：CPU内部逻辑计算耗时
- T2：外部IO操作耗时

> 最优线程数 = C *（T1 + T2）/ T1 = C * T1/T1 + C * T2/T1 = C + C * T2/T1
>
> 注意，该计算出的数值是理想状态下的理论数据，在生产环境中，不建议直接使用。而是根据当前环境，先设置一个比该值小的数值然后观察其压测效果，然后再根据效果逐步调大线程数，直至找到在该环境中性能最佳时的值

### 如何避免

+ 为了避免在业务使用时出现非预期的消息堆积和消费延迟问题，需要在前期设计阶段对整个业务逻辑进行完善的排查和梳理。其中最重要的就是梳理消息的消费耗时和设置消息消费的并发度

#### 梳理消息的消费耗时

+ 通过压测获取消息的消费耗时，并对耗时较高的操作的代码逻辑进行分析。梳理消息的消费耗时需要关注以下信息
  + 消息消费逻辑的计算复杂度是否过高，代码是否存在无限循环和递归等缺陷

  + 消息消费逻辑中的I/O操作是否是必须的，能否用本地缓存等方案规避

  + 消费逻辑中的复杂耗时的操作是否可以做异步化处理。如果可以，是否会造成逻辑错乱

#### 设置消费并发度

+ 对于消息消费并发度的计算，可以通过以下两步实施：
  + 逐步调大单个Consumer节点的线程数，并观测节点的系统指标，得到单个节点最优的消费线程数和消息吞吐量
  + 根据上下游链路的流量峰值计算出需要设置的节点数

> 节点数 = 流量峰值 / 单个节点消息吞吐量

## 回溯消费

回溯消费是指 Consumer **已经消费成功的消息**，由于业务上需求需要**重新消费**，要支持此功能，`Broker` 在向 Consumer 投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度，例如由于 Consumer 系统故障，恢复后需要重新消费1小时前的数据，那么 `Broker` 要提供一种机制，**可以按照时间维度来回退消费进度**。RocketMQ 支持按照时间回溯消费，时间维度精确到毫秒

## 消息的清理

- 消息被消费过后会被清理掉吗？不会的

- 消息是被顺序存储在commitlog文件的，且消息大小不定长，所以消息的清理是不可能以消息为单位进行清理的，而是以commitlog文件为单位进行清理的。否则会急剧下降清理效率，并实现逻辑复杂。commitlog文件存在一个过期时间，默认为72小时，即三天

- 除了用户手动清理外，在以下情况下也会被自动清理，无论文件中的消息是否被消费过

  文件过期，且到达清理时间点（默认为凌晨4点）后，自动清理过期文件

  文件过期，且磁盘空间占用率已达过期清理警戒线（默认75%）后，无论是否达到清理时间点，都会自动清理过期文件

  磁盘占用率达到清理警戒线（默认85%）后，开始按照设定好的规则清理文件，无论是否过期。默认会从最老的文件开始清理

  磁盘占用率达到系统危险警戒线（默认90%）后，Broker将拒绝消息写入

> 需要注意以下几点
>
> 1）对于RocketMQ系统来说，删除一个1G大小的文件，是一个压力巨大的IO操作。在删除过程中，系统性能会骤然下降。所以，其默认清理时间点为凌晨4点，访问量最小的时间。也正因如果，我们要保障磁盘空间的空闲率，不要使系统出现在其它时间点删除commitlog文件的情况
>
> 2）官方建议RocketMQ服务的Linux文件系统采用ext4。因为对于文件删除操作，ext4要比ext3性能更好

# 事务消息

Apache RocketMQ 在 4.3.0 版中已经支持分布式事务消息，这里 RocketMQ 采用了 **2PC** 的思想来实现了提交事务消息，同时增加一个补偿逻辑来处理二阶段超时或者失败的消息，如下图所示

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211805166.webp)

## Half Message（半消息）

+ Producer 把消息发送到 `Broker` 端时，该消息是不能被 Consumer 消费的，需要 Producer 对消息**进行二次确认**后，才能被消费

## 消息回查

**由于网络抖动或者 Producer 重启，导致 Producer 一直没有对 `Half Message` 进行二次确认**。`Broker` 端对未确定状态的消息发起回查，将消息发送到对应的 Producer 端（同一个Group的Producer），由 Producer 根据消息来检查本地事务的状态，进而执行 Commit 或者 Rollback 。值得注意的是，RocketMqQ 并不会无休止的的信息事务状态回查，**默认回查15次**，如果15次回查还是无法得知事务状态，默认回滚该消息



