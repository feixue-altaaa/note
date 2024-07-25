# what

**文档数据库**

MongoDB 中的记录是一个文档，它是由字段和值对组成的数据结构。MongoDB 文档类似于 JSON 对象。字段值可以包含其他文档、数组和文档数组

![A MongoDB document.](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407222203501.svg+xml)

# why

**使用文档的优点**

- 文档对应于许多编程语言中的原生数据类型。
- 嵌入式文档和数组可以减少成本高昂的的连接操作。
- 动态模式支持流畅的多态性

## 主要功能

### 高性能

MongoDB 提供高性能数据持久性。尤其是

- 对嵌入式数据模型的支持减少了数据库系统上的 I/O 活动。
- 索引支持更快的查询，并且可以包含嵌入式文档和数组的键。

### 查询 API

MongoDB 查询 API 支持[读写操作 (CRUD)](https://www.mongodb.com/zh-cn/docs/manual/crud/#std-label-crud) 以及：

- [数据聚合](https://www.mongodb.com/zh-cn/docs/manual/core/aggregation-pipeline/#std-label-aggregation-pipeline)
- [文本搜索](https://www.mongodb.com/zh-cn/docs/manual/text-search/#std-label-text-search)和[地理空间查询](https://www.mongodb.com/zh-cn/docs/manual/tutorial/geospatial-tutorial/)

### 高可用性

MongoDB 的复制工具称为[副本集](https://www.mongodb.com/zh-cn/docs/manual/replication/)，它提供以下功能：

- *自动*故障转移
- 数据冗余。

[副本集](https://www.mongodb.com/zh-cn/docs/manual/replication/)是一组 MongoDB 服务器，它们维护相同的数据集，并可提供冗余和提高数据可用性。

### 横向可扩展性

MongoDB 的*核心*功能之一是提供横向可扩展性：

- [分片](https://www.mongodb.com/zh-cn/docs/manual/sharding/#std-label-sharding-introduction)会将数据分布在机器集群上。
- 从 3.4 开始，MongoDB 支持基于[分片键](https://www.mongodb.com/zh-cn/docs/manual/core/zone-sharding/#std-label-zone-sharding)创建数据的[区域](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-shard-key)。在均衡的集群中，MongoDB 仅将区域覆盖的读写定向到区域内的那些分片。有关更多信息，请参阅[区域](https://www.mongodb.com/zh-cn/docs/manual/core/zone-sharding/#std-label-zone-sharding)手册页面。

### 支持多种存储引擎

MongoDB 支持[多种存储引擎：](https://www.mongodb.com/zh-cn/docs/manual/core/storage-engines/)

- [WiredTiger Storage Engine](https://www.mongodb.com/zh-cn/docs/manual/core/wiredtiger/)（包括对[静态加密](https://www.mongodb.com/zh-cn/docs/manual/core/security-encryption-at-rest/)
- [内存存储引擎。](https://www.mongodb.com/zh-cn/docs/manual/core/inmemory/)

此外，MongoDB 还提供可插拔的存储引擎 API，从而允许第三方基于 MongoDB 开发存储引擎。

# how

## 安装与启动

### 安装

+ [官网下载](https://www.mongodb.com/try/download/community)

+ 傻瓜式安装，中间需配置数据文档和日志文档地址

+ 安装Mongoshell，**MongoDB6之前shell是直接在里面的6之后需要单独下载**
  + [Mongoshell下载地址](https://www.mongodb.com/try/download/shell)
  + 下载后解压到mongo的安装路径下
+ 配置bin目录到环境变量中
  + ![image-20240725142505188](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407251425239.png)

+ [nosqlbooster下载地址](https://nosqlbooster.com/downloads)

### 启动与退出

**启动：**打开cmd,输入**mongosh**,即可成功运行mongo

![image-20240725142548601](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407251425622.png)

**退出：**mongo使用命令：**quit()**

## 基础概念

### **MongoDB 术语**

| SQL 术语/概念 | MongoDB 术语/概念 |              解释/说明              |
| :-----------: | :---------------: | :---------------------------------: |
|   database    |     database      |               数据库                |
|     table     |    collection     |            数据库表/集合            |
|      row      |     document      |           数据记录行/文档           |
|    column     |       field       |             数据字段/域             |
|     index     |       index       |                索引                 |
|  table joins  |                   |        表连接,MongoDB不支持         |
|  primary key  |    primary key    | 主键,MongoDB自动将_id字段设置为主键 |

**数据库（Database）**：包含一个或多个集合的 MongoDB 实例

**集合（Collection）**：类似于关系型数据库中的表，集合是一组文档的容器。在 MongoDB 中，一个集合中的文档不需要有一个固定的模式

> ### capped collections
>
> Capped collections 就是固定大小的collection。
>
> 它有很高的性能以及队列过期的特性(过期按照插入的顺序). 有点和 "RRD" 概念类似。
>
> Capped collections 是高性能自动的维护对象的插入顺序。它非常适合类似记录日志的功能和标准的 collection 不同，你必须要显式的创建一个capped collection，指定一个 collection 的大小，单位是字节。collection 的数据存储空间值提前分配的。
>
> Capped collections 可以按照文档的插入顺序保存到集合中，而且这些文档在磁盘上存放位置也是按照插入顺序来保存的，所以当我们更新Capped collections 中文档的时候，更新后的文档不可以超过之前文档的大小，这样话就可以确保所有文档在磁盘上的位置一直保持不变。
>
> 由于 Capped collection 是按照文档的插入顺序而不是使用索引确定插入位置，这样的话可以提高增添数据的效率。MongoDB 的操作日志文件 oplog.rs 就是利用 Capped Collection 来实现的。
>
> 要注意的是指定的存储大小包含了数据库的头信息
>
> ```java
> db.createCollection("mycoll", {capped:true, size:100000})
> ```

> - 在 capped collection 中，你能添加新的对象。
> - 能进行更新，然而，对象不会增加存储空间。如果增加，更新就会失败 。
> - 使用 Capped Collection 不能删除一个文档，可以使用 drop() 方法删除 collection 所有的行。
> - 删除之后，你必须显式的重新创建这个 collection。
> - 在32bit机器中，capped collection 最大存储为 1e9( 1X109)个字节

**文档（Document）**：MongoDB 的基本数据单元，通常是一个 JSON-like 的结构，可以包含多种数据类型；文档是一组键值(key-value)对(即 BSON)。MongoDB 的文档不需要设置相同的字段，并且相同的字段不需要相同的数据类型

**BSON**：Binary JSON 的缩写，是 MongoDB 用来存储和传输文档的二进制形式的 JSON

**索引（Index）**：用于优化查询性能的数据结构，可以基于集合中的一个或多个字段创建索引

**分片（Sharding）**：一种分布数据到多个服务器（称为分片）的方法，用于处理大数据集和高吞吐量应用

**副本集（Replica Set）**：一组维护相同数据集的 MongoDB 服务器，提供数据的冗余备份和高可用性

**主节点（Primary）**：副本集中负责处理所有写入操作的服务器

**从节点（Secondary）**：副本集中的服务器，用于读取数据和在主节点故障时接管为主节点

**MongoDB Shell**：MongoDB 提供的命令行界面，用于与 MongoDB 实例交互

**聚合框架（Aggregation Framework）**：用于执行复杂的数据处理和聚合操作的一系列操作

**Map-Reduce**：一种编程模型，用于处理大量数据集的并行计算

**GridFS**：用于存储和检索大于 BSON 文档大小限制的文件的规范

**ObjectId**：MongoDB 为每个文档自动生成的唯一标识符

**事务（Transactions）**：从 MongoDB 4.0 开始支持，允许一组操作作为一个原子单元执行

**连接（Join）**：MongoDB 允许在查询中使用 `$lookup` 操作符来实现类似 SQL 的连接操作

**TTL（Time-To-Live）**：可以为集合中的某些字段设置 TTL，以自动删除旧数据

**存储引擎（Storage Engine）**：MongoDB 用于数据存储和管理的底层技术，如 WiredTiger 和 MongoDB 的旧存储引擎 MMAPv1

**MongoDB Compass**：MongoDB 的图形界面工具，用于可视化和管理 MongoDB 数据

**MongoDB Atlas**：MongoDB 提供的云服务，允许在云中托管 MongoDB 数据库

> **注意**
>
> MongoDB区分类型和大小写
>
> MongoDB的文档不能有重复的键

### 元数据

数据库的信息是存储在集合中。它们使用了系统的命名空间

```mongo
dbname.system.*
```

在MongoDB数据库中名字空间 <dbname>.system.* 是包含多种系统信息的特殊集合(Collection)，如下

|       集合命名空间       |                  描述                   |
| :----------------------: | :-------------------------------------: |
| dbname.system.namespaces |            列出所有名字空间             |
|  dbname.system.indexes   |              列出所有索引               |
|  dbname.system.profile   |       包含数据库概要(profile)信息       |
|   dbname.system.users    |       列出所有可访问数据库的用户        |
|   dbname.local.sources   | 包含复制对端（slave）的服务器信息和状态 |

对于修改系统集合中的对象有如下限制

在{{system.indexes}}插入数据，可以创建索引。但除此之外该表信息是不可变的(特殊的drop index命令将自动更新相关信息)

{{system.users}}是可修改的。 {{system.profile}}是可删除的

### MongoDB 数据类型

| 数据类型           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| String             | 字符串。存储数据常用的数据类型。在 MongoDB 中，UTF-8 编码的字符串才是合法的 |
| Integer            | 整型数值。用于存储数值。根据你所采用的服务器，可分为 32 位或 64 位 |
| Boolean            | 布尔值。用于存储布尔值（真/假）                              |
| Double             | 双精度浮点值。用于存储浮点值                                 |
| Min/Max keys       | 将一个值与 BSON（二进制的 JSON）元素的最低值和最高值相对比   |
| Array              | 用于将数组或列表或多个值存储为一个键                         |
| Timestamp          | 时间戳。记录文档修改或添加的具体时间                         |
| Object             | 用于内嵌文档                                                 |
| Null               | 用于创建空值                                                 |
| Symbol             | 符号。该数据类型基本上等同于字符串类型，但不同的是，它一般用于采用特殊符号类型的语言 |
| Date               | 日期时间。用 UNIX 时间格式来存储当前日期或时间。你可以指定自己的日期时间：创建 Date 对象，传入年月日信息 |
| Object ID          | 对象 ID。用于创建文档的 ID                                   |
| Binary Data        | 二进制数据。用于存储二进制数据                               |
| Code               | 代码类型。用于在文档中存储 JavaScript 代码                   |
| Regular expression | 正则表达式类型。用于存储正则表达式                           |

### ObjectId

ObjectId 类似唯一主键，可以很快的去生成和排序，包含 12 bytes，含义是：

- 前 4 个字节表示创建 **unix** 时间戳,格林尼治时间 **UTC** 时间，比北京时间晚了 8 个小时
- 接下来的 3 个字节是机器标识码
- 紧接的两个字节由进程 id 组成 PID
- 最后三个字节是随机数

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407251600606.jpeg)

MongoDB 中存储的文档必须有一个 _id 键。这个键的值可以是任何类型的，默认是个 ObjectId 对象

由于 ObjectId 中保存了创建的时间戳，所以你不需要为你的文档保存时间戳字段，你可以通过 getTimestamp 函数来获取文档的创建时间

```mongo
var objectId = new ObjectId()
objectId.getTimestamp()
```

ObjectId 转为字符串

```mongo
objectId.toString()
objectId.str
```

### 时间戳

BSON 有一个特殊的时间戳类型用于 MongoDB 内部使用，与普通的 日期 类型不相关。 时间戳值是一个 64 位的值。其中：

- 前32位是一个 time_t 值（与Unix新纪元相差的秒数）
- 后32位是在某秒中操作的一个递增的`序数`

在单个 mongod 实例中，时间戳值通常是唯一的。

在复制集中， oplog 有一个 ts 字段。这个字段中的值使用BSON时间戳表示了操作时间

> *BSON 时间戳类型主要用于 MongoDB 内部使用。在大多数情况下的应用开发中，你可以使用 BSON 日期类型*

### 日期

表示当前距离 Unix新纪元（1970年1月1日）的毫秒数。日期类型是有符号的, 负数表示 1970 年之前的日期

```mongo
var mydate = new Date()
mydate
//ISODate("2024-07-25T16:07:58.218+08:00")
typeof mydate
//"object"
```

## 常见命令

```mongo
//显示所有数据库信息
show dbs
//显示当前数据库对象或集合
db
//连接到一个指定的数据库
use + 数据库名称
```

## CRUD

### 插入操作

#### 基础概念

**创建集合**

如果该集合当前不存在，则插入操作将创建该集合。

**`_id` 字段**

在 MongoDB 中，存储在集合中的每个文档都需要一个唯一的 [_id](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-_id) 字段作为[主键](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-primary-key)。如果插入的文档省略了 `_id` 字段，MongoDB 驱动程序会自动为 [ObjectId](https://www.mongodb.com/zh-cn/docs/manual/reference/bson-types/#std-label-objectid) 字段生成一个 `_id`。

这也适用于通过执行 [upsert: true](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.update/#std-label-upsert-parameter)

**原子性(Atomicity)**

MongoDB 中的所有写入操作在单个文档级别上都是原子操作，即使该操作修改单个文档*中*的多个嵌入文档也是如此

当单个写操作（例如 [`db.collection.updateMany()`](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.updateMany/#mongodb-method-db.collection.updateMany)）修改了多份文档，则每份文档的修改都是原子性的，但整个操作不是原子性的。

在执行多文档写入操作时，无论是通过单次写入操作还是多次写入操作，其他操作都可能会交错进行。

对于需要对多个文档（在单个或多个集合中）原子性读取和写入的情况，MongoDB 支持分布式事务，包括副本集和分片集群上的事务

> ## 并发控制
>
> 并发控制允许多个应用程序同时运行，而不会造成数据不一致或数据冲突。
>
> 对文档的 [`findAndModify`](https://www.mongodb.com/zh-cn/docs/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify) 操作是[原子性](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-atomic-operation)操作：如果查找条件与文档匹配，则对该文档执行更新。在当前更新完成之前，该文档的并发查询和其他更新不会受影响。
>
> 考虑以下示例
>
> + 一个包含两个文档的集合
>
> ```m
> db.myCollection.insertMany( [
>    { _id: 0, a: 1, b: 1 },
>    { _id: 1, a: 1, b: 1 }
> ] )

> + 两个如下 findAndModify 操作同时运行
>
>
> ```
> db.myCollection.findAndModify( {
>  query: { a: 1 },
>  update: { $inc: { b: 1 }, $set: { a: 2 } }
> } )
> ```
>
> 完成 [`findAndModify`](https://www.mongodb.com/zh-cn/docs/manual/reference/command/findAndModify/#mongodb-dbcommand-dbcmd.findAndModify) 操作后，保证两份文档中的 `a` 和 `b` 都设置为`2`

#### 插入单一文档

[`db.collection.insertOne()`](https://www.mongodb.com/zh-cn/docs/manual/reference/method/db.collection.insertOne/#mongodb-method-db.collection.insertOne) 将*单个*[文档](https://www.mongodb.com/zh-cn/docs/manual/core/document/#std-label-bson-document-format)插入到集合中。

以下示例将新文档插入 `inventory` 集合。如果文档未指定 `_id` 字段，MongoDB 会将具有 ObjectId 值的 `_id` 字段添加到新文档中

```mongo
db.test.insert({
    name:"qqq",
    age:"23"
})
```

```java
```
