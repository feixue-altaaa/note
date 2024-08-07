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

## 安装与连接

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

### 连接

**标准 URI 连接语法**

```bash
mongodb://[username:password@]host1[:port1][,...hostN[:portN]][/[defaultauthdb][?options]]
```

- `mongodb://`：协议头，表示使用 MongoDB
- `[username:password@]`：（可选）认证信息，包括用户名和密码
- `host1[:port1][,...hostN[:portN]]`：服务器地址和端口，可以是一个或多个 MongoDB 服务器的地址和端口
- `/[defaultauthdb]`：（可选）默认认证数据库
- `[?options]`：（可选）连接选项

#### 案例

**案例1-常见连接url**

```bash
//连接到本地 MongoDB 实例
mongodb://localhost

//连接到本地 MongoDB 实例，指定数据库
mongodb://localhost/mydatabase

//使用用户名和密码连接到本地 MongoDB 实例
mongodb://username:password@localhost/mydatabase

//连接到远程 MongoDB 实例
mongodb://remotehost:27017
```

**案例2-Java连接MongoDB**

```java
import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoDatabase;

public class MongoDBConnection {
    public static void main(String[] args) {
        String uri = "mongodb://user:password@localhost:27017/mydatabase?authSource=admin";
        try (MongoClient mongoClient = MongoClients.create(uri)) {
            MongoDatabase database = mongoClient.getDatabase("mydatabase");
            System.out.println("Connected to MongoDB");
        }
    }
}
```

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

### **条件操作符**

#### 比较操作符

| 操作符 | 描述             | 示例                              |
| ------ | ---------------- | --------------------------------- |
| `$eq`  | 等于             | `{ age: { $eq: 25 } }`            |
| `$ne`  | 不等于           | `{ age: { $ne: 25 } }`            |
| `$gt`  | 大于             | `{ age: { $gt: 25 } }`            |
| `$gte` | 大于等于         | `{ age: { $gte: 25 } }`           |
| `$lt`  | 小于             | `{ age: { $lt: 25 } }`            |
| `$lte` | 小于等于         | `{ age: { $lte: 25 } }`           |
| `$in`  | 在指定的数组中   | `{ age: { $in: [25, 30, 35] } }`  |
| `$nin` | 不在指定的数组中 | `{ age: { $nin: [25, 30, 35] } }` |

**案例**-查找年龄大于 25 且城市为 "New York" 的文档

```java
db.myCollection.find({ age: { $gt: 25 }, city: "New York" });
```

#### 逻辑操作符

| 操作符 | 描述                   | 示例                                                       |
| ------ | ---------------------- | ---------------------------------------------------------- |
| `$and` | 逻辑与，符合所有条件   | `{ $and: [ { age: { $gt: 25 } }, { city: "New York" } ] }` |
| `$or`  | 逻辑或，符合任意条件   | `{ $or: [ { age: { $lt: 25 } }, { city: "New York" } ] }`  |
| `$not` | 取反，不符合条件       | `{ age: { $not: { $gt: 25 } } }`                           |
| `$nor` | 逻辑与非，均不符合条件 | `{ $nor: [ { age: { $gt: 25 } }, { city: "New York" } ] }` |

**案例**-查找年龄大于 25 或城市为 "New York" 的文档

```java
db.myCollection.find({ $or: [ { age: { $gt: 25 } }, { city: "New York" } ] });
```

#### 元素操作符

| 操作符    | 描述             | 示例                         |
| --------- | ---------------- | ---------------------------- |
| `$exists` | 字段是否存在     | `{ age: { $exists: true } }` |
| `$type`   | 字段的 BSON 类型 | `{ age: { $type: "int" } }`  |

**案例-查找包含 age 字段的文档**

```java
db.myCollection.find({ age: { $exists: true } });
```

#### 数组操作符

| 操作符       | 描述                     | 示例                                                         |
| ------------ | ------------------------ | ------------------------------------------------------------ |
| `$all`       | 数组包含所有指定的元素   | `{ tags: { $all: ["red", "blue"] } }`                        |
| `$elemMatch` | 数组中的元素匹配指定条件 | `{ results: { $elemMatch: { score: { $gt: 80, $lt: 85 } } } }` |
| `$size`      | 数组的长度等于指定值     | `{ tags: { $size: 3 } }`                                     |

**案例-查找数组 tags 中包含 "red" 和 "blue" 的文档**

```java
db.myCollection.find({ tags: { $all: ["red", "blue"] } });
```

#### 其他操作符

| 操作符   | 描述                               | 示例                               |
| -------- | ---------------------------------- | ---------------------------------- |
| `$regex` | 匹配正则表达式                     | `{ name: { $regex: /^A/ } }`       |
| `$text`  | 进行文本搜索                       | `{ $text: { $search: "coffee" } }` |
| `$where` | 使用 JavaScript 表达式进行条件过滤 | `{ $where: "this.age > 25" }`      |

**案例-查找名字以 "A" 开头的文档**

```java
db.myCollection.find({ name: { $regex: /^A/ } });
```



### 数据库

```mongo
//创建数据库
//当你使用 use 命令来指定一个数据库时，如果该数据库不存在，MongoDB将自动创建它;否则切换到指定数据库
use DATABASE_NAME

//查看所有数据库
//刚创建的数据库并不在数据库的列表中， 要显示它，我们需要向该数据库插入一些数据
show dbs

//显示当前数据库对象或集合
db

//删除数据库
db.dropDatabase()
```

**默认数据库**

MongoDB 中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中

当您通过 shell 连接到 MongoDB 实例时，如果未使用 use 命令切换到其他数据库，则会默认使用 test 数据库

> ### 注意事项
>
> - 数据库名不能包含空格、点（.）或美元符号（$）。
> - 数据库的创建是自动的，不需要显式创建，除非你需要在创建时指定特定的配置选项。
> - 在MongoDB中，只有在数据库中至少有一个集合时，数据库才会在 `show dbs` 命令的输出中显示

### 集合

#### **创建集合**

参数说明

- name: 要创建的集合名称

- options: 可选参数, 指定有关内存大小及索引的选项

|       参数名       |  类型  | 描述                                                         |             示例值              |
| :----------------: | :----: | ------------------------------------------------------------ | :-----------------------------: |
|      `capped`      | 布尔值 | 是否创建一个固定大小的集合。                                 |             `true`              |
|       `size`       |  数值  | 集合的最大大小（以字节为单位）。仅在 `capped` 为 true 时有效。 |        `10485760` (10MB)        |
|       `max`        |  数值  | 集合中允许的最大文档数。仅在 `capped` 为 true 时有效。       |             `5000`              |
|    `validator`     |  对象  | 用于文档验证的表达式。                                       |    `{ $jsonSchema: { ... }}`    |
| `validationLevel`  | 字符串 | 指定文档验证的严格程度。<br>`"off"`：不进行验证。<br>`"strict"`：插入和更新操作都必须通过验证（默认）。<br>`"moderate"`：仅现有文档更新时必须通过验证，插入新文档时不需要。 |           `"strict"`            |
| `validationAction` | 字符串 | 指定文档验证失败时的操作。<br>`"error"`：阻止插入或更新（默认）。<br>`"warn"`：允许插入或更新，但会发出警告。 |            `"error"`            |
|  `storageEngine`   |  对象  | 为集合指定存储引擎配置。                                     |    `{ wiredTiger: { ... }}`     |
|    `collation`     |  对象  | 指定集合的默认排序规则。                                     | `{ locale: "en", strength: 2 }` |

```mongo
//创建集合
db.createCollection(name, options)

db.createCollection("myComplexCollection", {
  capped: true,
  size: 10485760,
  max: 5000,
  validator: { $jsonSchema: {
    bsonType: "object",
    required: ["name", "email"],
    properties: {
      name: {
        bsonType: "string",
        description: "必须为字符串且为必填项"
      },
      email: {
        bsonType: "string",
        pattern: "^.+@.+$",
        description: "必须为有效的电子邮件地址"
      }
    }
  }},
  validationLevel: "strict",
  validationAction: "error",
  storageEngine: {
    wiredTiger: { configString: "block_compressor=zstd" }
  },
  collation: { locale: "en", strength: 2 }
});
```

这个例子创建了一个集合，具有以下特性

- 固定大小，最大 10MB，最多存储 5000 个文档

- 文档必须包含 `name` 和 `email` 字段，其中 `name` 必须是字符串，`email` 必须是有效的电子邮件格式

- 验证级别为严格，验证失败将阻止插入或更新

- 使用 WiredTiger 存储引擎，指定块压缩器为 zstd

- 默认使用英语排序规则

#### 固定集合（Capped Collections）

MongoDB 固定集合（Capped Collections）是性能出色且有着固定大小的集合，对于大小固定，我们可以想象其就像一个环形队列，当集合空间用完后，再插入的元素就会覆盖最初始的头部的元素

**创建固定集合**

我们通过createCollection来创建一个固定集合，且capped选项设置为true

```
db.createCollection("cappedLogCollection",{capped:true,size:10000})

 //还可以指定文档个数,加上max:1000属性
 db.createCollection("cappedLogCollection",{capped:true,size:10000,max:1000})

//判断集合是否为固定集合
>db.cappedLogCollection.isCapped()

//如果需要将已存在的集合转换为固定集合可以使用以下命令
>db.runCommand({"convertToCapped":"posts",size:10000})
```

**固定集合查询**

固定集合文档按照插入顺序储存的,默认情况下查询就是按照插入顺序返回的,也可以使用$natural调整返回顺序

```java
>db.cappedLogCollection.find().sort({$natural:-1})
```

**固定集合的功能特点**

可以插入及更新,但更新不能超出collection的大小,否则更新失败,不允许删除,但是可以调用drop()删除集合中的所有行,但是drop后需要显式地重建集合。

在32位机子上一个cappped collection的最大值约为482.5M,64位上只受系统文件大小的限制。

**固定集合属性及用法**

**属性**

- 属性1:对固定集合进行插入速度极快
- 属性2:按照插入顺序的查询输出速度极快
- 属性3:能够在插入最新数据时,淘汰最早的数据

**用法**

- 用法1:储存日志信息
- 用法2:缓存一些少量的文档

#### 删除与查看

```mongo
//删除集合
//如果成功删除选定集合，则 drop() 方法返回 true，否则返回 false
db.collection.drop()

//查看已有集合
show collections
show tables
```

#### 更名集合名

MongoDB 可以使用 renameCollection 方法来来重命名集合。

renameCollection 方法在 MongoDB 的 admin 数据库中运行，可以将一个集合重命名为另一个名称。renameCollection 命令的语法：

```mongo
db.adminCommand({
  renameCollection: "sourceDb.sourceCollection",
  to: "targetDb.targetCollection",
  dropTarget: <boolean>
})
```

**参数说明**

- **renameCollection**：要重命名的集合的完全限定名称（包括数据库名）
- **to**：目标集合的完全限定名称（包括数据库名）
- **dropTarget**（可选）：布尔值。如果目标集合已经存在，是否删除目标集合。默认值为 `false`

**案例**

将 test 数据库中的 oldCollection 重命名为 newCollection

```mongo
db.adminCommand({ 
  renameCollection: "test.oldCollection", 
  to: "test.newCollection" 
});
```

将 test 数据库中的 oldCollection 重命名为 production 数据库中的 newCollection

```mongo
db.adminCommand({ 
  renameCollection: "test.oldCollection", 
  to: "production.newCollection" 
});
```

> ### 注意事项
>
> - **权限要求**：执行 `renameCollection` 命令需要具有对源数据库和目标数据库的适当权限。通常需要 `dbAdmin` 或 `dbOwner` 角色
> - **目标集合不存在**：目标集合不能已经存在。如果目标集合存在，则会返回错误
> - **索引和数据**：重命名集合会保留所有文档和索引

### 文档

#### 插入

**创建集合**

如果该集合当前不存在，则插入操作将创建该集合

**`_id` 字段**

在 MongoDB 中，存储在集合中的每个文档都需要一个唯一的 [_id](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-_id) 字段作为[主键](https://www.mongodb.com/zh-cn/docs/manual/reference/glossary/#std-term-primary-key)。如果插入的文档省略了 `_id` 字段，MongoDB 驱动程序会自动为 [ObjectId](https://www.mongodb.com/zh-cn/docs/manual/reference/bson-types/#std-label-objectid) 字段生成一个 `_id`

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

**插入单一文档**

> #### 插入文档时的选项
>
> 这些方法的 options 参数通常可以包含以下选项
>
> - ordered（仅适用于 insertMany）：布尔值。如果为 true，则按顺序插入文档，在遇到错误时停止；如果为 false，则尝试插入所有文档，即使遇到错误也继续。默认值为 true。
> - writeConcern：指定写操作的确认级别。
> - bypassDocumentValidation：布尔值。如果为 true，则忽略集合的文档验证规则

```mongo
db.test.insert({
    name:"qqq",
    age:"23"
})

db.myCollection.insertOne({
    name: "Alice",
    age: 25,
    city: "New York"
});
```

```java
db.collection.insertOne(document, options)
//document：要插入的单个文档。
//options（可选）：一个可选参数对象，可以包含 writeConcern 和bypassDocumentValidation 等

//案例
    //同一个文档不能同时插入两次，会报MongoBulkWriteException
Document doc = new Document("name", "zoro").append("age", 26);
collection.insertOne(doc);
```

**插入多个文档**

```mongo
db.myCollection.insertMany([
    { name: "Bob", age: 30, city: "Los Angeles" },
    { name: "Charlie", age: 35, city: "Chicago" }
]);
```

```java
db.collection.insertMany(documents, options)
//options（可选）：一个可选参数对象，可以包含 ordered、writeConcern 和 bypassDocumentValidation 等
 
//案例
Document doc = new Document("name", "zoro").append("age", 26);
Document doc2 = new Document("gender", "male");

List<Document> docList = new ArrayList<Document>();
docList.add(doc);
docList.add(doc2);

collection.insertMany(docList);
```

**保存**

**db.collection.save()**

save() 方法在插入文档时表现得类似于 insertOne()

如果文档包含 _id 字段且已存在，则该文档会被更新；如果文档不包含 _id 字段或 _id 不存在，则会插入一个新文档

```java
//案例
db.myCollection.save({
    _id: ObjectId("60c72b2f9b1d8b5a5f8e2b2d"),
    name: "David",
    age: 40,
    city: "San Francisco"
});
```

**自动增长ID**

MongoDB 没有像 SQL 一样有自动增长的功能， MongoDB 的 _id 是系统自动生成的12字节唯一标识。但在某些情况下，我们可能需要实现 ObjectId 自动增长功能

由于 MongoDB 没有实现这个功能，我们可以通过编程的方式来实现，以下我们将在 counters 集合中实现_id字段自动增长

考虑以下 products 文档。我们希望 _id 字段实现 从 1,2,3,4 到 n 的自动增长功能

```mongo
{
  "_id":1,
  "product_name": "Apple iPhone",
  "category": "mobiles"
}
```

为此，创建 counters 集合，序列字段值可以实现自动长。现在我们向 counters 集合中插入以下文档，使用 productid 作为键值。**该文档起保存自动增长次数作用**。sequence_value 字段是序列通过自动增长后的一个值

```mongo
{
  "_id":"productid",
  "sequence_value": 0
}
```

**创建 Javascript 函数**

现在，我们创建函数 getNextSequenceValue 来作为序列名的输入， 指定的序列会自动增长 1 并返回最新序列值。在本文的实例中序列名为 productid

```javascript
>function getNextSequenceValue(sequenceName){
   var sequenceDocument = db.counters.findAndModify(
      {
         query:{_id: sequenceName },
         update: {$inc:{sequence_value:1}},
         "new":true
      });
   return sequenceDocument.sequence_value;
}
```

**使用 Javascript 函数**

接下来我们将使用 getNextSequenceValue 函数创建一个新的文档， 并设置文档 _id 自动为返回的序列值

```javascript
>db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Apple iPhone",
   "category":"mobiles"})

>db.products.insert({
   "_id":getNextSequenceValue("productid"),
   "product_name":"Samsung S3",
   "category":"mobiles"})
```

就如你所看到的，我们使用 getNextSequenceValue 函数来设置 _id 字段。为了验证函数是否有效，我们可以使用以下命令读取文档

```javascript
>db.products.find()

//以上命令将返回以下结果，我们发现 _id 字段是自增长的
{ "_id" : 1, "product_name" : "Apple iPhone", "category" : "mobiles"}

{ "_id" : 2, "product_name" : "Samsung S3", "category" : "mobiles" }
```

#### 更新

**updateOne()**

updateOne() 方法用于更新匹配过滤器的单个文档

```java
db.collection.updateOne(filter, update, options)
```

- **filter**：用于查找文档的查询条件。
- **update**：指定更新操作的文档或更新操作符。
- **options**：可选参数对象，如 `upsert`、`arrayFilters` 等

> ### 选项参数
>
> 这些更新方法的 options 参数通常可以包含以下选项：
>
> - **upsert**：如果没有匹配的文档，是否插入一个新文档。
> - **arrayFilters**：当更新嵌套数组时，指定应更新的数组元素的条件。
> - **collation**：指定比较字符串时使用的排序规则。
> - **returnDocument**：在 findOneAndUpdate 中使用，指定返回更新前 ("before") 或更新后 ("after") 的文档

**案例**

```java
db.myCollection.updateOne(
    { name: "Alice" },                // 过滤条件
    { $set: { age: 26 } },            // 更新操作
    { upsert: false }                 // 可选参数
);
```

**updateMany()**

updateMany() 方法用于更新所有匹配过滤器的文档

```java
db.collection.updateMany(filter, update, options)
```

**案例**

```java
db.myCollection.updateMany(
    { age: { $lt: 30 } },             // 过滤条件
    { $set: { status: "active" } },   // 更新操作
    { upsert: false }                  // 可选参数
);
```

**replaceOne()**

replaceOne() 方法用于替换匹配过滤器的单个文档，新的文档将完全替换旧的文档

```java
db.collection.replaceOne(filter, replacement, options)
```

- **filter**：用于查找文档的查询条件。

- **replacement**：新的文档，将替换旧的文档。

- **options**：可选参数对象，如 `upsert` 等

**案例**

```java
db.myCollection.replaceOne(
    { name: "Bob" },                  // 过滤条件
    { name: "Bob", age: 31 }          // 新文档
);
```

**findOneAndUpdate()**

findOneAndUpdate() 方法用于查找并更新单个文档，可以选择返回更新前或更新后的文档

```java
db.collection.findOneAndUpdate(filter, update, options)
```

- **filter**：用于查找文档的查询条件。
- **update**：指定更新操作的文档或更新操作符。
- **options**：可选参数对象，如 `projection`、`sort`、`upsert`、`returnDocument` 等

**案例**

```java
db.myCollection.findOneAndUpdate(
    { name: "Charlie" },              // 过滤条件
    { $set: { age: 36 } },            // 更新操作
    { returnDocument: "after" }       // 可选参数，返回更新后的文档
);
```

#### 删除

**drop**

```java
//删除集合中全部数据
collection.drop();
```

**deleteOne()**

deleteOne() 方法用于删除匹配过滤器的单个文档

```java
db.collection.deleteOne(filter, options)
    
//案例
Document query = new Document().append("name", "luosong");
collection.deleteOne(query);
```

- **filter**：用于查找要删除的文档的查询条件。

- **options**（可选）：一个可选参数对象

> ### 删除操作的选项
>
> 这些删除方法的 options 参数通常可以包含以下选项：
>
> - **writeConcern**：指定写操作的确认级别。
> - **collation**：指定比较字符串时使用的排序规则。
> - **projection**（仅适用于 `findOneAndDelete`）：指定返回的字段。
> - **sort**（仅适用于 `findOneAndDelete`）：指定排序顺序以确定要删除的文档

```java
db.myCollection.deleteOne({ name: "Alice" });
```

**deleteMany()**

deleteMany() 方法用于删除所有匹配过滤器的文档

```java
db.collection.deleteMany(filter, options)

//案例
Document query = new Document().append("name", "luosong");
collection.deleteMany(query);
```

**findOneAndDelete()**

findOneAndDelete() 方法用于查找并删除单个文档，并可以选择返回删除的文档

```java
db.collection.findOneAndDelete(filter, options)
    
db.myCollection.findOneAndDelete(
    { name: "Charlie" },
    { projection: { name: 1, age: 1 } }
);    
```

#### 查询

**find()**

```java
db.collection.find(query, projection)
```

- **query**：用于查找文档的查询条件。默认为 `{}`，即匹配所有文档。
- **projection**（可选）：指定返回结果中包含或排除的字段

**案例**

```java
//查找所有文档
db.myCollection.find();

//按条件查找文档
db.myCollection.find({ age: { $gt: 25 } });

//按条件查找文档，并只返回指定字段
db.myCollection.find(
    { age: { $gt: 25 } },
    { name: 1, age: 1, _id: 0 }
);

//如果你需要以易读的方式来读取数据，可以使用 pretty() 方法
db.col.find().pretty()

db.col.find().pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

**findOne()**

findOne() 方法用于查找集合中的单个文档。如果找到多个匹配的文档，它只返回第一个

```java
db.collection.findOne(query, projection)

//案例-查找单个文档，并只返回指定字段
db.myCollection.findOne(
    { name: "Alice" },
    { name: 1, age: 1, _id: 0 }
);    
```

**高级查询方法**

+ **使用比较操作符**

		MongoDB 支持多种比较操作符，如 **$gt、$lt、$gte、$lte、$eq、$ne** 等

	**案例-查找年龄大于 25 的文档**

```java
db.myCollection.find({ age: { $gt: 25 } });
```

+ **使用逻辑操作符**

		MongoDB 支持多种逻辑操作符，如 **$and、$or、$not、$nor** 等
		
		**案例-查询键 by 值为 菜鸟教程 或键 title 值为 MongoDB 教程 的文档**

```java
db.col.find({$or:[{"by":"菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
```

		**案例-查找年龄大于 25 且城市为 "New York" 的文档**

```java
db.myCollection.find({
    $and: [
        { age: { $gt: 25 } },
        { city: "New York" }
    ]
});
```

		**案例-演示 AND 和 OR 联合使用，类似常规 SQL 语句为： 'where likes>50 AND (by = '菜鸟教程' OR title = 'MongoDB 教程')'**

```java
>db.col.find({"likes": {$gt:50}, $or: [{"by": "菜鸟教程"},{"title": "MongoDB 教程"}]}).pretty()
{
        "_id" : ObjectId("56063f17ade2f21f36b03133"),
        "title" : "MongoDB 教程",
        "description" : "MongoDB 是一个 Nosql 数据库",
        "by" : "菜鸟教程",
        "url" : "http://www.runoob.com",
        "tags" : [
                "mongodb",
                "database",
                "NoSQL"
        ],
        "likes" : 100
}
```

+ **使用正则表达式**

		可以使用正则表达式进行模式匹配查询
		
		**案例-查找名字以 "A" 开头的文档**

```java
db.myCollection.find({ name: /^A/ });
```

+ **投影**

		投影用于控制查询结果中返回的字段。可以使用包含字段和排除字段两种方式
		
		**案例-只返回名字和年龄字段**

```java
db.myCollection.find(
    { age: { $gt: 25 } },
    { name: 1, age: 1, _id: 0 }
);
```

+ **排序**

		可以对查询结果进行排序
		
		**案例-按年龄降序排序**

```java
db.myCollection.find().sort({ age: -1 });
```

+ **限制与跳过**

		可以对查询结果进行限制和跳过指定数量的文档。
		
		**案例-返回前 10 个文档**

```java
db.myCollection.find().limit(10);
```

		**案例-跳过前 5 个文档，返回接下来的 10 个文档**

```java
db.myCollection.find().skip(5).limit(10);
```

		**案例-查找年龄大于 25 且城市为 "New York" 的文档，只返回名字和年龄字段，按年龄降序排序，并返回前 10 个文档**

```java
db.myCollection.find(
    {
        $and: [
            { age: { $gt: 25 } },
            { city: "New York" }
        ]
    },
    { name: 1, age: 1, _id: 0 }
).sort({ age: -1 }).limit(10);
```

+ **$type 操作符**

		**$type** 操作符用于查询字段的 BSON 数据类型，它允许您指定一个或多个类型，并返回匹配这些类型的文档

```java
db.collection.find({ field: { $type: <type> } })
```

		**field**：要检查类型的字段
	
		**type**：指定的 BSON 类型，可以是类型的数字代码或类型名称的字符串

**常见的 BSON 类型及其对应的数字代码和字符串名称**

| 类型代码 |      类型名称       |
| :------: | :-----------------: |
|    1     |       double        |
|    2     |       string        |
|    3     |       object        |
|    4     |        array        |
|    5     |       binData       |
|    6     |      undefined      |
|    7     |      objectId       |
|    8     |        bool         |
|    9     |        date         |
|    10    |        null         |
|    11    |        regex        |
|    12    |      dbPointer      |
|    13    |     javascript      |
|    14    |       symbol        |
|    15    | javascriptWithScope |
|    16    |         int         |
|    17    |      timestamp      |
|    18    |        long         |
|    19    |       decimal       |
|   255    |       minKey        |
|   127    |       maxKey        |

**案例**

```
//查找字段类型为字符串的文档
db.myCollection.find({ fieldName: { $type: "string" } })
//或使用类型代码
db.myCollection.find({ fieldName: { $type: 2 } })

//查找字段类型为多种类型的文档，例如，查找 value 字段类型为字符串或整数的文档
db.myCollection.find({ value: { $type: ["string", "int"] } })
//或使用类型代码
db.myCollection.find({ value: { $type: [2, 16] } })
```

+ **Limit 与 Skip 方法**

		limit() 方法用于限制查询结果返回的文档数量，而 skip() 方法用于跳过指定数量的文档。这两个方法通常一起使用，可以用来实现分页查询或在大型数据集上进行分批处理
		
		limit() 方法基本语法如下所示

```java
db.collection.find().limit(<limit>)
    
// 返回前 10 个文档
db.myCollection.find().limit(10);    
```

		skip() 方法语法格式如下

```java
db.collection.find().skip(<skip>)

// 跳过前 10 个文档，返回接下来的 10 个文档
db.myCollection.find().skip(10).limit(10);    
```

		**案例-分页查询**

```java
// 第一页，每页 10 个文档
db.myCollection.find().skip(0).limit(10);

// 第二页，每页 10 个文档
db.myCollection.find().skip(10).limit(10);

// 第三页，每页 10 个文档
db.myCollection.find().skip(20).limit(10);
```

**注:**skip()方法默认参数为 0

> ### 注意事项
>
> - `skip()` 和 `limit()` 方法通常用于配合使用，以实现分页查询。但是在大型数据集上使用 `skip()` 可能会导致性能问题，因为 MongoDB 在执行查询时需要扫描并跳过指定数量的文档，因此建议仅在需要时才使用 `skip()` 方法，尽量避免在大型数据集上连续使用
> - 当结合 `skip()` 和 `limit()` 时，`skip()` 应该在 `limit()` 之前使用，以避免意外行为

#### 正则表达式

正则表达式是使用单个字符串来描述、匹配一系列符合某个句法规则的字符串。

许多程序设计语言都支持利用正则表达式进行字符串操作。

MongoDB 使用 **$regex** 操作符来设置匹配字符串的正则表达式。

MongoDB使用PCRE (Perl Compatible Regular Expression) 作为正则表达式语言。

不同于全文检索，我们使用正则表达式不需要做任何配置。

考虑以下 **posts** 集合的文档结构，该文档包含了文章内容和标签

```
{
   "post_text": "enjoy the mongodb articles on runoob",
   "tags": [
      "mongodb",
      "runoob"
   ]
}
```

以下命令使用正则表达式查找包含 runoob 字符串的文章

```java
db.posts.find({post_text:{$regex:"runoob"}})

//也可以写为
db.posts.find({post_text:/runoob/})    
```

如果检索需要不区分大小写，我们可以设置 $options 为 $i。

以下命令将查找不区分大小写的字符串 runoob

```java
db.posts.find({post_text:{$regex:"runoob",$options:"$i"}})
```

**数组元素使用正则表达式**

我们还可以在数组字段中使用正则表达式来查找内容。 这在标签的实现上非常有用，如果你需要查找包含以 run 开头的标签数据(ru 或 run 或 runoob)， 你可以使用以下代码

```java
>db.posts.find({tags:{$regex:"run"}})
```

**优化正则表达式查询**

- 如果你的文档中字段设置了索引，那么使用索引相比于正则表达式匹配查找所有的数据查询速度更快。
- 如果正则表达式是前缀表达式，所有匹配的数据将以指定的前缀字符串为开始。例如： 如果正则表达式为 **^tut** ，查询语句将查找以 tut 为开头的字符串。

**这里面使用正则表达式有两点需要注意**

正则表达式中使用变量。一定要使用eval将组合的字符串进行转换，不能直接将字符串拼接后传入给表达式。否则没有报错信息，只是结果为空！实例如下

```java
var name=eval("/" + 变量值key +"/i"); 
```

#### 聚合查询

聚合操作主要用于处理数据并返回计算结果。聚合操作将来自多个文档的值组合在一起，按条件分组后，再进行一系列操作（如求和、平均值、最大值、最小值）以返回单个结果

##### **聚合管道**

MongoDB 的聚合框架就是将文档输入处理管道，在管道内完成对文档的操作，最终将文档转换为聚合结果，MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理，管道操作是可以重复的

**聚合流程**

db.collection.aggregate()是基于数据处理的聚合管道，每个文档通过一个由多个阶段（stage）组成的管道，可以对每个阶段的管道进行分组、过滤等功能，然后经过一系列的处理，输出相应的结果

聚合管道方法的流程参见下图

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281447509.png)

上图的聚合操作相当于 MySQL 中的以下语句

```sql
select cust_id as _id, sum(amount) as total from orders where status like "%A%" group by cust_id;
```

**详细流程**

1. `db.collection.aggregate()` 可以用多个构件创建一个管道，对于一连串的文档进行处理。这些构件包括：筛选操作的`match`、映射操作的`project`、分组操作的`group`、排序操作的`sort`、限制操作的`limit`、和跳过操作的`skip`。
2. `db.collection.aggregate()`使用了MongoDB内置的原生操作，聚合效率非常高,支持类似于SQL Group By操作的功能，而不再需要用户编写自定义的JavaScript例程。
3. 每个阶段管道限制为100MB的内存。如果一个节点管道超过这个极限,MongoDB将产生一个错误。为了能够在处理大型数据集,可以设置`allowDiskUse`为`true`来在聚合管道节点把数据写入临时文件。这样就可以解决100MB的内存的限制。
4. `db.collection.aggregate()`可以作用在分片集合，但结果不能输在分片集合，`MapReduce`可以 作用在分片集合，结果也可以输在分片集合。
5. `db.collection.aggregate()`方法可以返回一个指针（`cursor`），数据放在内存中，直接操作。跟Mongo shell 一样指针操作。
6. `db.collection.aggregate()`输出的结果只能保存在一个文档中，`BSON Document`大小限制为16M。可以通过返回指针解决，版本2.6中：`DB.collect.aggregate()`方法返回一个指针，可以返回任何结果集的大小

**聚合语法**

```java
db.collection.aggregate(pipeline, options)
```

| 参数     | 类型     | 描述                                                         |
| -------- | -------- | ------------------------------------------------------------ |
| pipeline | array    | 一系列数据聚合操作或阶段。详见[聚合管道操作符](https://link.segmentfault.com/?enc=yzxRo7axGhcQpUjw8i84vg%3D%3D.tVV5SpuQqSvNDjsfvBsmowLAA2CHIgqeSBHoEs0lTggvXL2ZSOG60tb3E5HZe2sZjbc%2BqWwdm965bBnbPUzf%2Bw%3D%3D) 在版本2.6中更改：该方法仍然可以将流水线阶段作为单独的参数接受，而不是作为数组中的元素;但是，如果不将管道指定为数组，则不能指定options参数 |
| options  | document | 可选。 aggregate()传递给聚合命令的其他选项。 2.6版中的新增功能：仅当将管道指定为数组时才可用 |

**与mysql聚合类比**

| SQL 操作/函数 | mongodb聚合操作 |
| :-----------: | :-------------: |
|     where     |     $match      |
|   group by    |     $group      |
|    having     |     $match      |
|    select     |    $project     |
|   order by    |      $sort      |
|     limit     |     $limit      |
|     sum()     |      $sum       |
|    count()    |      $sum       |
|     join      |     $lookup     |

**$group**

按指定的表达式对文档进行分组

```java
{ $group: { _id: <expression>, <field1>: { <accumulator1> : <expression1> }, ... } }
```

- `_id`字段是必填的;但是，可以指定_id值为null来为整个输入文档计算累计值。
- 剩余的计算字段是可选的，并使用`<accumulator>`运算符进行计算。
- `_id`和`<accumulator>`表达式可以接受任何有效的表达式

**accumulator操作符**

|    名称     | 描述                                                         |
| :---------: | ------------------------------------------------------------ |
|    $avg     | 计算均值                                                     |
|   $first    | 返回每组第一个文档，如果有排序，按照排序，如果没有按照默认的存储的顺序返回第一个文档。 |
|    $last    | 返回每组最后一个文档，如果有排序，按照排序，如果没有按照默认的存储的顺序返回最后一个文档。 |
|    $max     | 根据分组，获取集合中所有文档对应值的最大值。                 |
|    $min     | 根据分组，获取集合中所有文档对应值的最小值。                 |
|    $push    | 将指定的表达式的值添加到一个数组中。                         |
|  $addToSet  | 将表达式的值添加到一个集合中（无重复值，无序）。             |
|    $sum     | 计算总和                                                     |
| $stdDevPop  | 返回输入值的总体标准偏差（population standard deviation）    |
| $stdDevSamp | 返回输入值的样本标准偏差（the sample standard deviation）    |

> **注意：**
>
> - "$addToSet":expr，如果当前数组中不包含expr，那就将它添加到数组中。
> - "$push":expr，不管expr是什么值，都将它添加到数组中，返回包含所有值的数组

**案例-按照`state`分组，并计算每一个state分组的总人数，平均人数以及每个分组的数量**

```java
db.zips.aggregate([
    {
        "$group": {
            "_id": "$state",
            "totalPop": {
                "$sum": "$pop"
            },
            "avglPop": {
                "$avg": "$pop"
            },
            "count": {
                "$sum": 1
            }
        }
    }
])
```

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281501968.png)

**案例-按照`city`分组，并且分组内的`state`字段列表加入到`stateItem`并显示**

```java
db.zips.aggregate([
    {
        "$group": {
            "_id": "$city",
            "stateItem": {
                "$push": "$state"
            }
        }
    }
])
```

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281502599.png)

**案例-state重复个数大于100的城市**

```java
db.zips.aggregate([
    {
        "$group": {
            "_id": "$state",
            "total": {
                "$sum": 1
            }
        }
    },
    {
        "$match": {
            "total": {
                "$gt": 100
            }
        }
    }
])
```

**$match**

过滤文档，仅将符合指定条件的文档传递到下一个管道阶段

```java
{ $match: { <query> } }
```

$match可以使用除了地理空间之外的所有常规查询操作符，**在实际应用中尽可能将$match放在管道的前面位置**。这样有两个好处：

- 一是可以快速将不需要的文档过滤掉，以**减少管道的工作量**
- 二是如果在投射和分组之前执行$match，**查询可以使用索引**

**案例-使用$match管道选择要处理的文档，然后将结果输出到$group管道以计算文档的计数**

```java
db.zips.aggregate([
    {
        "$match": {
            "state": "NY"
        }
    },
    {
        "$group": {
            "_id": null,
            "sum": {
                "$sum": "$pop"
            },
            "avg": {
                "$avg": "$pop"
            },
            "count": {
                "$sum": 1
            }
        }
    }
]).pretty();
```

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281504790.png)

**$unwind**

将数组拆分为单独的文档

**案例-使用$unwind为loc数组中的每个元素输出一个文档**

```java
db.zips.aggregate([
    {
        "$match": {
            "_id": "01002"
        }
    },
    {
        "$unwind": "$loc"
    }
]).pretty();
```

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281506894.png)

**$project**

$project可以从文档中选择想要的字段，和不想要的字段，也可以通过管道表达式进行一些复杂的操作，例如数学操作，日期操作，字符串操作，逻辑操作

**语法**

$project 管道符的作用是选择字段（指定字段，添加字段，不显示字段,_id：0，排除字段等），重命名字段，派生字段

```java
{ $project: { <specification(s)> } }
```

field:1/0，表示选择/不选择

- 默认情况下，_id字段包含在输出文档中。要在输出文档中包含输入文档中的任何其他字段，必须明确指定$project中的包含。 如果指定包含文档中不存在的字段，$project将忽略该字段包含，并且不会将该字段添加到文档中。
- 默认情况下，_id字段包含在输出文档中。要从输出文档中排除_id字段，必须明确指定$project中的_id字段为0。
- v3.4版新增功能-如果指定排除一个或多个字段，则所有其他字段将在输出文档中返回。 如果指定排除_id以外的字段，则不能使用任何其他$project规范表单：即，如果排除字段，则不能指定包含字段，重置现有字段的值或添加新字段。此限制不适用于使用REMOVE变量条件排除字段。

**案例-`_id`字段默认包含在内。要从$ project阶段的输出文档中排除`_id`字段，请在project文档中将`_id`字段设置为0来指定排除_id字段**

```java
db.zips.aggregate([
    {
        "$project": {
            "_id": 0,
            "city": 1,
            "state": 1
        }
    }
]).pretty();
```

![file](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281514618.png)

**$limit**

限制传递到管道中下一阶段的文档数

```java
{ $limit: <positive integer> }
```

**案例-返回管道传递给它的前5个文档。 $limit对其传递的文档内容没有影响**

```java
db.zips.aggregate({
    "$limit": 5
});
```

**$skip**

跳过进入stage的指定数量的文档，并将其余文档传递到管道中的下一个阶段

```java
{ $skip: <positive integer> }
```

**案例-跳过管道传递给它的前5个文档**

```java
db.zips.aggregate({
    "$skip": 5
});
```

**$sort**

对所有输入文档进行排序，并按排序顺序将它们返回到管道

```java
{ $sort: { <field1>: <sort order>, <field2>: <sort order> ... } }
```

$sort指定要排序的字段和相应的排序顺序的文档。 `<sort order>`可以具有以下值之一：

- 1指定升序
- -1指定降序
- {$meta：“textScore”}按照降序排列计算出的textScore元数据

**案例**

```java
db.zips.aggregate([
    {
        "$sort": {
            "pop": -1,
            "city": 1
        }
    }
])
```

##### Map-Reduce函数

MongoDB还提供map-reduce操作来执行聚合。 通常，**map-reduce操作有两个阶段**：**一个map阶段**，它处理每个文档并为每个输入文档发出一个或多个对象，以及**reduce阶段**组合map操作的输出。 可选地，map-reduce可以具有最终化阶段以对结果进行最终修改。 与其他聚合操作一样，map-reduce可以指定查询条件以选择输入文档以及排序和限制结果。

Map-reduce使用自定义JavaScript函数来执行映射和减少操作，以及可选的finalize操作。 虽然自定义JavaScript与聚合管道相比提供了极大的灵活性，但通常，map-reduce比聚合管道效率更低，更复杂。模式如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281541598.webp)

以下是MapReduce**的基本语法**

```java
>db.collection.mapReduce(
   function() {emit(key,value);},  //map 函数
   function(key,values) {return reduceFunction},   //reduce 函数
   {
      out: collection,
      query: document,
      sort: document,
      limit: number
   }
)

```

使用 MapReduce 要实现两个函数 Map 函数和 Reduce 函数,Map 函数调用 emit(key, value), 遍历 collection 中所有的记录, 将 key 与 value 传递给 Reduce 函数进行处理。

Map 函数必须调用 emit(key, value) 返回键值对。

参数说明:

- **map** ：映射函数 (生成键值对序列,作为 reduce 函数参数)。
- **reduce** 统计函数，reduce函数的任务就是将key-values变成key-value，也就是把values数组变成一个单一的值value。。
- **out** 统计结果存放集合 (不指定则使用临时集合,在客户端断开后自动删除)。
- **query** 一个筛选条件，只有满足条件的文档才会调用map函数。（query。limit，sort可以随意组合）
- **sort** 和limit结合的sort排序参数（也是在发往map函数前给文档排序），可以优化分组机制
- **limit** 发往map函数的文档数量的上限（要是没有limit，单独使用sort的用处不大）

**案例-在集合 orders 中查找 status:"A" 的数据，并根据 cust_id 来分组，并计算 amount 的总和**

文档存储了用户的 user_name 和文章的 status 字段

```mongo
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "mark",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"disabled"
})
WriteResult({ "nInserted" : 1 })
>db.posts.insert({
   "post_text": "菜鸟教程，最全的技术文档。",
   "user_name": "runoob",
   "status":"active"
})
WriteResult({ "nInserted" : 1 })
```

现在，我们将在 posts 集合中使用 mapReduce 函数来选取已发布的文章(status:"active")，并通过user_name分组，计算每个用户的文章数

```java
>db.posts.mapReduce( 
   function() { emit(this.user_name,1); }, 
   function(key, values) {return Array.sum(values)}, 
      {  
         query:{status:"active"},  
         out:"post_total" 
      }
)
```

以上 mapReduce 输出结果为

```
{
        "result" : "post_total",
        "timeMillis" : 23,
        "counts" : {
                "input" : 5,
                "emit" : 5,
                "reduce" : 1,
                "output" : 2
        },
        "ok" : 1
}
```

结果表明，共有 5 个符合查询条件（status:"active"）的文档， 在map函数中生成了 5 个键值对文档，最后使用reduce函数将相同的键值分为 2 组

具体参数说明

- result：储存结果的collection的名字,这是个临时集合，MapReduce的连接关闭后自动就被删除了。
- timeMillis：执行花费的时间，毫秒为单位
- input：满足条件被发送到map函数的文档个数
- emit：在map函数中emit被调用的次数，也就是所有集合中的数据总量
- output：结果集合中的文档个数**（count对调试非常有帮助）**
- ok：是否成功，成功为1
- err：如果失败，这里可以有失败原因，不过从经验上来看，原因比较模糊，作用不大

使用 find 操作符来查看 mapReduce 的查询结果

```java
> var map=function() { emit(this.user_name,1); }
> var reduce=function(key, values) {return Array.sum(values)}
> var options={query:{status:"active"},out:"post_total"}
> db.posts.mapReduce(map,reduce,options)
{ "result" : "post_total", "ok" : 1 }
> db.post_total.find();
```

以上查询显示如下结果

```
{ "_id" : "mark", "value" : 4 }
{ "_id" : "runoob", "value" : 1 }
```

##### 单一的聚合命令

  MongoDB还提供了，db.collection.estimatedDocumentCount（），db.collection.count（）和db.collection.distinct（） 所有这些单一的聚合命令。 虽然这些操作提供了对常见聚合过程的简单访问操作，但它们缺乏聚合管道和map-reduce的灵活性和功能。模型如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407281541141.webp)

#### 排序

sort()方法基本语法如下所示

```java
db.collection.find().sort({ field1: 1, field2: -1 })
```

`{ field1: 1, field2: -1 }`：指定要排序的字段及排序顺序。1 表示升序，-1 表示降序

**案例**

```java
// 按 age 字段升序排序
db.myCollection.find().sort({ age: 1 });

// 先按 age 字段升序排序，再按 createdAt 字段降序排序
db.myCollection.find().sort({ age: 1, createdAt: -1 });
```

> ### 注意事项
>
> - MongoDB 在执行排序时会对查询结果进行排序，因此可能会影响性能，特别是在大型数据集上排序操作可能会较慢
> - 如果排序字段上有索引，排序操作可能会更高效。在执行频繁的排序操作时，可以考虑创建适当的索引以提高性能



熟练使用项目

官方文档







pom文件结构

### 什么是POM文件？

POM（Project Object Model）文件是Maven项目的核心文件之一。它是一个XML文件，描述了项目的基本信息、依赖项、构建和发布等信息。POM文件是Maven的重要组成部分，可以帮助开发者管理和构建项目。在使用Maven进行项目构建时，需要根据项目的需要配置POM文件

### POM文件的基本结构

POM文件是一个XML文件，包含多个元素，每个元素代表一个特定的配置项。下面是一个POM文件的基本结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>example-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- 依赖项配置 -->
    </dependencies>

    <build>
        <!-- 构建配置 -->
    </build>

    <repositories>
        <!-- 仓库配置 -->
    </repositories>
</project>
```

## POM文件的常用配置项

### 坐标信息

坐标信息指的是<groupId>、<artifactId>和<version>元素。<groupId>定义了项目的组织ID，<artifactId>定义了项目的唯一标识符，<version>定义了项目的版本号。这些信息对于Maven的依赖管理和构建过程非常重要

```xml
<groupId>com.example</groupId>
<artifactId>example-project</artifactId>
<version>1.0-SNAPSHOT</version>
```

### 依赖项配置

依赖项配置用于定义项目所依赖的外部库。Maven会自动下载并管理这些依赖项。依赖项配置包含在<dependencies>元素中，每个依赖项使用一个<dependency>元素进行描述

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.13</version>
    </dependency>
</dependencies>
```

>  dependencies与dependencyManagement的区别
>
> **dependencyment**
>
> 在我们项目中，我们会发现在父模块的pom文件中常常会出现dependencyMent元素，这是因为我们可以通过其来管理子模块的版本号，也就是说我们在父模块中声明号依赖的版本，但是并不实现引入；
>
> **dependencies**
>
> 上面说到dependencyment只是声明一个依赖，而不实现引入，故我们在子模块中也需要对依赖进行声明，倘若不声明子模块自己的依赖，是不会从父模块中继承的；只有子模块中也声明了依赖。并且没有写对应的版本号它才会从父类中继承；并且version和scope都是取自父类；此外要是子模块中自己定义了自己的版本号，是不会继承自父类的
>
> **总结**
>
> dependencyment只是用来管理依赖，规定未添加版本号的子模块依赖继承自它，dependencies是用来声明子模块自己的依赖，可以在其中来写自己需要的版本号；

### 构建配置

构建配置用于定义项目的构建过程。它包含在<build>元素中，包括了多个子元素。其中比较常用的子元素有

```xml
<build>
    <plugins>
        <!-- 插件配置 -->
    </plugins>
    <resources>
        <!-- 资源文件配置 -->
    </resources>
    <testResources>
        <!-- 测试资源文件配置 -->
    </testResources>
</build>
```

### 插件配置

插件是Maven的一个重要特性，它可以用于扩展Maven的功能。Maven自带了一些插件，比如maven-compiler-plugin、maven-jar-plugin等。插件配置包含在<plugins>元素中，每个插件使用一个<plugin>元素进行描述

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

### 仓库配置

仓库配置用于定义Maven的依赖项下载地址。Maven默认使用中央仓库，但是也可以配置私有仓库或者本地仓库。仓库配置包含在<repositories>元素中，每个仓库使用一个<repository>元素进行描述

```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
</repositories>
```

# 父子 POM 示例

**Maven 父 POM** （或超级 POM）用于构造项目，以**避免重复或重复使用 pom 文件之间的\*继承配置\***。 它有助于长期轻松维护。

如果在父 POM 和子 POM 中都使用不同的值配置了任何依赖项或属性，则子 POM 值将具有优先级

## 父 POM 内容

可以使用包`pom`声明父 POM。 它不打算分发，因为仅从其他项目中引用了它。

Maven 父 pom 可以包含几乎所有内容，并且可以继承到子 pom 文件中，例如：

*   通用数据 – 开发人员的姓名，SCM 地址，分发管理等
*   常数 – 例如版本号
*   共同的依赖项 – 所有子项共同的。 与在单个 pom 文件中多次写入它们具有相同的效果。
*   属性 – 例如插件，声明，执行和 ID。
*   配置
*   资源

## 父 POM 和子 POM 示例

为了匹配父 POM，Maven 使用两个规则：

1.  在项目的根目录或给定的相对路径中有一个 pom 文件。
2.  子 POM 文件中的引用包含与父 POM 文件中所述相同的坐标。

#### 父 POM

此处，父 POM 为 JUnit 和 spring 框架配置了基本项目信息和两个[依赖项](//howtodoinjava.com/maven/maven-dependency-management/)。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd;
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.howtodoinjava.demo</groupId>
	<artifactId>MavenExamples</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<name>MavenExamples Parent</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<junit.version>3.8.1</junit.version>
		<spring.version>4.3.5.RELEASE</spring.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>

	</dependencies>
</project>

```

#### 子 POM

现在，子 POM 需要使用`parent`标签并指定`groupId/artifactId/version`属性来引用父 POM。 这个 pom 文件将从父 POM 继承所有属性和依赖项，并且还可以包括子项目特定的依赖项。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<!--The identifier of the parent POM-->
	<parent>
		<groupId>com.howtodoinjava.demo</groupId>
		<artifactId>MavenExamples</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<modelVersion>4.0.0</modelVersion>
	<artifactId>MavenExamples</artifactId>
	<name>MavenExamples Child POM</name>
	<packaging>jar</packaging>

	<dependencies>		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-security</artifactId>
			<version>${spring.version}</version>
		</dependency>
	</dependencies>

</project>

```

## 父 POM 相对路径

默认情况下，Maven 首先在项目的根目录下查找父 POM，然后在本地仓库中查找，最后在远程仓库中查找。 如果父 POM 文件不在任何其他位置，则可以使用代码标签。 该**相对路径应相对于项目根**。

如果未明确给出相对路径，则默认为`..`，即当前项目的父目录中的 pom。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<!--The identifier of the parent POM-->
	<parent>
		<groupId>com.howtodoinjava.demo</groupId>
		<artifactId>MavenExamples</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<relativePath>../baseapp/pom.xml</relativePath>
	</parent>

	<modelVersion>4.0.0</modelVersion>
	<artifactId>MavenExamples</artifactId>
	<name>MavenExamples Child POM</name>
	<packaging>jar</packaging>

</project>
```

# maven修改镜像地址

在maven中的conf文件中，找到settings.xml文件，将文件中的 mirrors 标签中 添加如下代码，即可将镜像地址改成其他，加快下载速度

```xml
<mirror>
     <id>alimaven</id>
     <name>aliyun maven</name>
     <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     <mirrorOf>central</mirrorOf>
</mirror>
```









MongoClients是怎么实现MongoClient的
