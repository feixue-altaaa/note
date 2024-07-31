## 索引

索引通常能够极大的提高查询的效率，如果没有索引，MongoDB在读取数据时必须扫描集合中的每个文件并选取那些符合查询条件的记录

这种扫描全集合的查询效率是非常低的，特别在处理大量的数据时，查询可能要花费几十秒甚至几分钟，这对网站的性能是非常致命的

索引是特殊的数据结构，索引存储在一个易于遍历读取的数据集合中，索引是对数据库表中一列或多列的值进行排序的一种结构

在 MongoDB 中，常见的索引类型包括：

- **单字段索引**：基于单个字段的索引
- **复合索引**：基于多个字段组合的索引
- **文本索引**：用于支持全文搜索。
- **地理空间索引**：用于地理空间数据的查询。
- **哈希索引**：用于对字段值进行哈希处理的索引

### 索引策略

在创建索引时，需要考虑以下因素

- **查询频率**：优先考虑那些经常用于查询的字段
- **字段基数**：字段值的基数越高（即唯一值越多），索引的效果越好
- **索引大小**：索引的大小会影响数据库的内存占用和查询性能

### 索引优化

在对索引进行优化时，可以考虑以下方法

- **选择合适的索引类型**：根据查询需求选择合适的索引类型
- **创建复合索引**：对于经常一起使用的字段，考虑创建复合索引以提高查询效率
- **监控索引性能**：定期监控索引的使用情况，根据实际需求调整索引

### 覆盖索引

官方的MongoDB的文档中说明，覆盖查询是以下的查询：

- 所有的查询字段是索引的一部分
- 所有的查询返回字段在同一个索引中

由于所有出现在查询中的字段是索引的一部分， MongoDB 无需在整个数据文档中检索匹配查询条件和返回使用相同索引的查询结果。

因为索引存在于RAM中，从索引中获取数据比通过扫描文档读取数据要快得多

为了测试覆盖索引查询，使用以下 users 集合

```
{
   "_id": ObjectId("53402597d852426020000002"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "gender": "M",
   "name": "Tom Benzamin",
   "user_name": "tombenzamin"
}
```

我们在 users 集合中创建联合索引，字段为 gender 和 user_name

```java
>db.users.createIndex({gender:1,user_name:1})
```

> **注：**5.0 之前版本可以使用 db.collection.ensureIndex() ，但 ensureIndex() 在 5.0 版本后已被移除，使用 createIndex() 代替

现在，该索引会覆盖以下查询

```java
>db.users.find({gender:"M"},{user_name:1,_id:0})
```

也就是说，对于上述查询，MongoDB的不会去数据库文件中查找。相反，它会从索引中提取数据，这是非常快速的数据查询。

**由于我们的索引中不包括 _id 字段，_id在查询中会默认返回，我们可以在MongoDB的查询结果集中排除它**

如果是以下的查询，不能使用覆盖索引查询

- 所有索引字段是一个数组
- 所有索引字段是一个子文档

-----

### 索引限制

#### 额外开销

每个索引占据一定的存储空间，在进行插入，更新和删除操作时也需要对索引进行操作。所以，如果你很少对集合进行读取操作，建议不使用索引

------

#### 内存(RAM)使用

由于索引是存储在内存(RAM)中,你应该确保该索引的大小不超过内存的限制

如果索引的大小大于内存的限制，MongoDB会删除一些索引，这将导致性能下降

------

#### 查询限制

索引不能被以下的查询使用

- 正则表达式及非操作符，如 $nin, $not, 等
- 算术运算符，如 $mod, 等
- $where 子句

------

#### 索引键限制

从2.6版本开始，如果现有的索引字段的值超过索引键的限制，MongoDB中不会创建索引

------

#### 插入文档超过索引键限制

如果文档的索引字段值超过了索引键的限制，MongoDB不会将任何文档转换成索引的集合。与mongorestore和mongoimport工具类似

------

#### 最大范围

- 集合中索引不能超过64个
- 索引名的长度不能超过128个字符
- 一个复合索引最多可以有31个字段

### 创建索引

MongoDB 使用 createIndex() 方法来创建索引

> *注意在 3.0.0 版本前创建索引方法为 db.collection.ensureIndex()，之后的版本使用了 db.collection.createIndex() 方法，ensureIndex() 还能用，但只是 createIndex() 的别名*

**语法**

createIndex() 方法基本语法格式如下所示

```java
vdb.collection.createIndex( keys, options)
```

- `db`：数据库的引用
- `collection`：集合的名称
- `keys`：一个对象，指定了字段名和索引的排序方向（1 表示升序，-1 表示降序）
- `options`：一个可选参数，可以包含索引的额外选项

options 参数是一个对象，可以包含多种配置选项，以下是一些常用的选项

- `unique`：如果设置为 `true`，则创建唯一索引，确保索引字段的值在集合中是唯一
- `background`：如果设置为 `true`，则索引创建过程在后台运行，不影响其他数据库操作
- `name`：指定索引的名称，如果不指定，MongoDB 会根据索引的字段自动生成一个名称
- `sparse`：如果设置为 `true`，创建稀疏索引，只索引那些包含索引字段的文档
- `expireAfterSeconds`：设置索引字段的过期时间，MongoDB 将自动删除过期的文档
- `v`：索引版本，通常不需要手动设置
- `weights`：**为文本索引指定权重**

**案例-单字段索引**

```java
// 创建 age 字段的升序索引
db.myCollection.createIndex({ age: 1 });

// 创建唯一索引
db.collection.createIndex( { field: 1 }, { unique: true } )

// 创建后台运行的索引
db.collection.createIndex( { field: 1 }, { background: true } )

// 创建稀疏索引
db.collection.createIndex( { field: 1 }, { sparse: true } )

// 创建文本索引并指定权重
db.collection.createIndex( { field: "text" }, { weights: { field: 10 } } )
```

案例-复合索引

**案例-文本索引**

```java
// 创建 name 字段的文本索引
db.myCollection.createIndex({ name: "text" });

//使用全文索引
db.myCollection.find({$text:{$search:"runoob"}})
    
//删除全文索引
//使用 find 命令查找索引名    
db.myCollection.getIndexes()
db.myCollection.dropIndex("text") 
```

**案例-地理空间索引**

```java
//创建地理空间索引
//对于存储地理位置数据的字段，可以使用 2dsphere 或 2d 索引类型来创建地理空间索引。
// 2dsphere 索引，适用于球形地理数据
db.collection.createIndex( { location: "2dsphere" } )

// 2d 索引，适用于平面地理数据
db.collection.createIndex( { location: "2d" } )
```

**案例-哈希索引**

从 MongoDB 3.2 版本开始，可以使用哈希索引对字段进行哈希，以支持大范围的数值查找

```java
db.collection.createIndex( { field: "hashed" } )
```

### 查看索引

```java
db.collection.getIndexes()
```

### 删除索引

使用 dropIndex() 或 dropIndexes() 方法可以删除索引

```java
// 删除指定的索引
db.collection.dropIndex( "indexName" )

// 删除所有索引
db.collection.dropIndexes()
```

### 索引查询分析

MongoDB 查询分析可以确保我们所建立的索引是否有效，是查询语句性能分析的重要工具

MongoDB 查询分析常用函数有：explain() 和 hint()

#### explain()

explain 操作提供了查询信息，使用索引及查询统计等。有利于我们对索引的优化。

接下来我们在 users 集合中创建 gender 和 user_name 的索引

```java
>db.users.find({gender:"M"},{user_name:1,_id:0}).explain()
```

以上的 explain() 查询返回如下结果

```
{
   "cursor" : "BtreeCursor gender_1_user_name_1",
   "isMultiKey" : false,
   "n" : 1,
   "nscannedObjects" : 0,
   "nscanned" : 1,
   "nscannedObjectsAllPlans" : 0,
   "nscannedAllPlans" : 1,
   "scanAndOrder" : false,
   "indexOnly" : true,
   "nYields" : 0,
   "nChunkSkips" : 0,
   "millis" : 0,
   "indexBounds" : {
      "gender" : [
         [
            "M",
            "M"
         ]
      ],
      "user_name" : [
         [
            {
               "$minElement" : 1
            },
            {
               "$maxElement" : 1
            }
         ]
      ]
   }
}
```

现在，我们看看这个结果集的字段

- **indexOnly**: 字段为 true ，表示我们使用了索引
- **cursor**：因为这个查询使用了索引，MongoDB 中索引存储在B树结构中，所以这是也使用了 BtreeCursor 类型的游标。如果没有使用索引，游标的类型是 BasicCursor。这个键还会给出你所使用的索引的名称，你通过这个名称可以查看当前数据库下的system.indexes集合（系统自动创建，由于存储索引信息，这个稍微会提到）来得到索引的详细信息
- **n**：当前查询返回的文档数量
- **nscanned/nscannedObjects**：表明当前这次查询一共扫描了集合中多少个文档，我们的目的是，让这个数值和返回文档的数量越接近越好
- **millis**：当前查询所需时间，毫秒数
- **indexBounds**：当前查询具体使用的索引

#### hint()

虽然MongoDB查询优化器一般工作的很不错，但是也可以使用 hint 来强制 MongoDB 使用一个指定的索引

这种方法某些情形下会提升性能。 一个有索引的 collection 并且执行一个多字段的查询(一些字段已经索引了)

如下查询实例指定了使用 gender 和 user_name 索引字段来查询

```java
>db.users.find({gender:"M"},{user_name:1,_id:0}).hint({gender:1,user_name:1})
```

#### 慢查询日志

##### 开启 Profiling 功能

有两种方式可以控制 Profiling 的开关和级别，第一种是直接在启动参数里直接进行设置。

启动MongoDB时加上–profile=级别 即可。

也可以在客户端调用db.setProfilingLevel(级别) 命令来实时配置。可以通过db.getProfilingLevel()命令来获取当前的Profile级别

```javascript
> db.setProfilingLevel(2); 
{"was" : 0 , "ok" : 1} 
> db.getProfilingLevel()
```

面斜体的级别可以取0，1，2 三个值，他们表示的意义如下：

```
0 – 不开启
1 – 记录慢命令 (默认为>100ms)
2 – 记录所有命令
```

Profile 记录在级别1时会记录慢命令，那么这个慢的定义是什么?上面我们说到其默认为100ms，当然有默认就有设置，其设置方法和级别一样有两种，一种是通过添加–slowms启动参数配置。第二种是调用db.setProfilingLevel时加上第二个参数：

```java
db.setProfilingLevel( level , slowms ) 
db.setProfilingLevel( 1 , 10 );
```

##### 查询 Profiling 记录

与MySQL的慢查询日志不同，Mongo Profile 记录是直接存在系统db里的，记录位置 system.profile，所以，我们只要查询这个Collection的记录就可以获取到我们的 Profile 记录了。

```
> db.system.profile.find()

{"ts" : "Thu Jan 29 2009 15:19:32 GMT-0500 (EST)" , "info" : "query test.$cmd ntoreturn:1 reslen:66 nscanned:0 query: { profile: 2 } nreturned:1 bytes:50" , "millis" : 0} 

db.system.profile.find( { info: /test.foo/ } ) 

{"ts" : "Thu Jan 29 2009 15:19:40 GMT-0500 (EST)" , "info" : "insert test.foo" , "millis" : 0} 
{"ts" : "Thu Jan 29 2009 15:19:42 GMT-0500 (EST)" , "info" : "insert test.foo" , "millis" : 0} 
{"ts" : "Thu Jan 29 2009 15:19:45 GMT-0500 (EST)" , "info" : "query test.foo ntoreturn:0 reslen:102 nscanned:2 query: {} nreturned:2 bytes:86" , "millis" : 0}
{"ts" : "Thu Jan 29 2009 15:21:17 GMT-0500 (EST)" , "info" : "query test.foo ntoreturn:0 reslen:36 nscanned:2 query: { $not: { x: 2 } } nreturned:0 bytes:20" , "millis" : 0}
{"ts" : "Thu Jan 29 2009 15:21:27 GMT-0500 (EST)" , "info" : "query test.foo ntoreturn:0 exception bytes:53" , "millis" : 88}
```

列出执行时间长于某一限度(5ms)的 Profile 记录：

```
> db.system.profile.find( { millis : { $gt : 5 } } ) 
{"ts" : "Thu Jan 29 2009 15:21:27 GMT-0500 (EST)" , "info" : "query test.foo ntoreturn:0 exception bytes:53" , "millis" : 88}
```

查看最新的 Profile 记录：

```
db.system.profile.find().sort({$natural:-1})
```

Mongo Shell 还提供了一个比较简洁的命令show profile，可列出最近5条执行时间超过1ms的 Profile 记录。

##### Profile 信息内容详解

- ts-该命令在何时执行。
- millis Time-该命令执行耗时，以毫秒记.
- info-本命令的详细信息.
- query-表明这是一个query查询操作.
- ntoreturn-本次查询客户端要求返回的记录数.比如, findOne()命令执行时 ntoreturn 为 1.有limit(n) 条件时ntoreturn为n.
- query-具体的查询条件(如x>3).
- nscanned-本次查询扫描的记录数.
- reslen-返回结果集的大小.
- nreturned-本次查询实际返回的结果集.
- update-表明这是一个update更新操作.
- fastmod-Indicates a fast modify operation. See Updates. These operations are normally quite fast.
- fastmodinsert – indicates a fast modify operation that performed an upsert.
- upsert-表明update的upsert参数为true.此参数的功能是如果update的记录不存在，则用update的条件insert一条记录.
- moved-表明本次update是否移动了硬盘上的数据，如果新记录比原记录短，通常不会移动当前记录，如果新记录比原记录长，那么可能会移动记录到其它位置，这时候会导致相关索引的更新.磁盘操作更多，加上索引更新，会使得这样的操作比较慢.
- insert-这是一个insert插入操作.
- getmore-这是一个getmore 操作，getmore通常发生在结果集比较大的查询时，第一个query返回了部分结果，后续的结果是通过getmore来获取的

##### Profiler 的效率

Profiling 功能肯定是会影响效率的，但是不太严重，原因是他使用的是 system.profile 来记录，而 system.profile 是一个capped collection 这种collection 在操作上有一些限制和特点，但是效率更高

### 高级索引

考虑以下文档集合（users ）,文档包含了 address 子文档和 tags 数组

```
{
   "address": {
      "city": "Los Angeles",
      "state": "California",
      "pincode": "123"
   },
   "tags": [
      "music",
      "cricket",
      "blogs"
   ],
   "name": "Tom Benzamin"
}
```

#### 索引数组字段

假设我们基于标签来检索用户，为此我们需要对集合中的数组 tags 建立索引

在数组中创建索引，需要对数组中的每个字段依次建立索引。所以在我们为数组 tags 创建索引时，会为 music、cricket、blogs三个值建立单独的索引

使用以下命令创建数组索引

```java
>db.users.createIndex({"tags":1})
```

创建索引后，我们可以这样检索集合的 tags 字段

```java
>db.users.find({tags:"cricket"})
```

#### 索引子文档字段

假设我们需要通过city、state、pincode字段来检索文档，由于这些字段是子文档的字段，所以我们需要对子文档建立索引

为子文档的三个字段创建索引，命令如下

```java
>db.users.createIndex({"address.city":1,"address.state":1,"address.pincode":1})
```

一旦创建索引，我们可以使用子文档的字段来检索数据

**查询表达不一定遵循指定的索引的顺序，mongodb 会自动优化。所以上面创建的索引将支持以下查询**

```java
>db.users.find({"address.city":"Los Angeles"})   
    
>db.users.find({"address.state":"California","address.city":"Los Angeles"}) 
    
//同样支持以下查询
>db.users.find({"address.city":"Los Angeles","address.state":"California","address.pincode":"123"})
    
```

## MongoDB 复制（副本集）

MongoDB复制是将数据同步在多个服务器的过程。

复制提供了数据的冗余备份，并在多个服务器上存储数据副本，提高了数据的可用性， 并可以保证数据的安全性。

复制还允许您从硬件故障和服务中断中恢复数据

### why

- 保障数据的安全性
- 数据高可用性 (24*7)
- 灾难恢复
- 无需停机维护（如备份，重建索引，压缩）
- 分布式读取数据

### 原理

mongodb的复制至少需要两个节点。其中一个是主节点，负责处理客户端请求，其余的都是从节点，负责复制主节点上的数据。

mongodb各个节点常见的搭配方式为：一主一从、一主多从。

主节点记录在其上的所有操作oplog，从节点定期轮询主节点获取这些操作，然后对自己的数据副本执行这些操作，从而保证从节点的数据与主节点一致。

MongoDB复制结构图如下所示

![MongoDB复制结构图](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407282049970.png)

以上结构图中，客户端从主节点读取数据，在客户端写入数据到主节点时， 主节点与从节点进行数据交互保障数据的一致性。

### 副本集特征

- N 个节点的集群
- 任何节点可作为主节点
- 所有写入操作都在主节点上
- 自动故障转移
- 自动恢复

### 副本集部署

https://www.runoob.com/mongodb/mongodb-replication.html

 ## MongoDB 分片

在Mongodb里面存在另一种集群，就是分片技术,可以满足MongoDB数据量大量增长的需求

当MongoDB存储海量的数据时，一台机器可能不足以存储数据，也可能不足以提供可接受的读写吞吐量。这时，我们就可以通过在多台机器上分割数据，使得数据库系统能存储和处理更多的数据

### why

- 复制所有的写入操作到主节点
- 延迟的敏感数据会在主节点查询
- 单个副本集限制在12个节点
- 当请求量巨大时会出现内存不足。
- 本地磁盘不足
- 垂直扩展价格昂贵

### 分片集群结构分布

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407282056203.png)

上图中主要有如下所述三个主要组件：

- Shard:

  用于存储实际的数据块，实际生产环境中一个shard server角色可由几台机器组个一个replica set承担，防止主机单点故障

- Config Server:

  mongod实例，存储了整个 ClusterMetadata，其中包括 chunk信息。

- Query Routers:

  前端路由，客户端由此接入，且让整个集群看上去像单一数据库，前端应用可以透明使用

### 分片部署

https://www.runoob.com/mongodb/mongodb-sharding.html

## MongoDB 备份(mongodump)与恢复(mongorestore)

https://www.runoob.com/mongodb/mongodb-mongodump-mongorestore.html

## MongoDB GridFS

GridFS 用于存储和恢复那些超过16M（BSON文件限制）的文件(如：图片、音频、视频等)。

GridFS 也是文件存储的一种方式，但是它是存储在MonoDB的集合中

GridFS 可以更好的存储大于16M的文件

GridFS 会将大文件对象分割成多个小的chunk(文件片段),一般为256k/个,每个chunk将作为MongoDB的一个文档(document)被存储在chunks集合中

GridFS 用两个集合来存储一个文件：fs.files与fs.chunks

每个文件的实际内容被存在chunks(二进制数据)中,和文件有关的meta数据(filename,content_type,还有用户自定义的属性)将会被存在files集合中

以下是简单的 fs.files 集合文档

```
{
   "filename": "test.txt",
   "chunkSize": NumberInt(261120),
   "uploadDate": ISODate("2014-04-13T11:32:33.557Z"),
   "md5": "7b762939321e146569b07f72c62cca4f",
   "length": NumberInt(646)
}
```

以下是简单的 fs.chunks 集合文档

```
{
   "files_id": ObjectId("534a75d19f54bfec8a2fe44b"),
   "n": NumberInt(0),
   "data": "Mongo Binary Data"
}
```







## MongoDB 监控

在你已经安装部署并允许MongoDB服务后，你必须要了解MongoDB的运行情况，并查看MongoDB的性能。这样在大流量得情况下可以很好的应对并保证MongoDB正常运作。

MongoDB中提供了mongostat 和 mongotop 两个命令来监控MongoDB的运行情况

### mongostat 命令

mongostat是mongodb自带的状态检测工具，在命令行下使用。它会间隔固定时间获取mongodb的当前运行状态，并输出。如果你发现数据库突然变慢或者有其他问题的话，你第一手的操作就考虑采用mongostat来查看mongo的状态。

启动你的Mongod服务，进入到你安装的MongoDB目录下的bin目录， 然后输入mongostat命令，如下所示

### mongotop 命令

mongotop也是mongodb下的一个内置工具，mongotop提供了一个方法，用来跟踪一个MongoDB的实例，查看哪些大量的时间花费在读取和写入数据。 mongotop提供每个集合的水平的统计数据。默认情况下，mongotop返回值的每一秒。

启动你的Mongod服务，进入到你安装的MongoDB目录下的bin目录， 然后输入mongotop命令，如下所示



https://www.runoob.com/mongodb/mongodb-mongostat-mongotop.html

## MongoDB 关系

MongoDB 的关系表示多个文档之间在逻辑上的相互联系。文档间可以通过嵌入和引用来建立联系。MongoDB 中的关系可以是：

- 1:1 (1对1)
- 1: N (1对多)
- N: 1 (多对1)
- N: N (多对多)

接下来我们来考虑下用户与用户地址的关系。一个用户可以有多个地址，所以是一对多的关系。以下是 **user** 文档的简单结构

```mongo
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "name": "Tom Hanks",
   "contact": "987654321",
   "dob": "01-01-1991"
}
```

以下是 **address** 文档的简单结构

```mongo
{
   "_id":ObjectId("52ffc4a5d85242602e000000"),
   "building": "22 A, Indiana Apt",
   "pincode": 123456,
   "city": "Los Angeles",
   "state": "California"
} 
```

### 嵌入式关系

使用嵌入式方法，我们可以把用户地址嵌入到用户的文档中

```mongo
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address": [
      {
         "building": "22 A, Indiana Apt",
         "pincode": 123456,
         "city": "Los Angeles",
         "state": "California"
      },
      {
         "building": "170 A, Acropolis Apt",
         "pincode": 456789,
         "city": "Chicago",
         "state": "Illinois"
      }]
} 
```

以上数据保存在单一的文档中，可以比较容易的获取和维护数据。 你可以这样查询用户的地址

```java
>db.users.findOne({"name":"Tom Benzamin"},{"address":1})
```

注意：以上查询中 **db** 和 **users** 表示数据库和集合。

这种数据结构的缺点是，如果用户和用户地址在不断增加，数据量不断变大，会影响读写性能

### 引用式关系

引用式关系是设计数据库时经常用到的方法，这种方法把用户数据文档和用户地址数据文档分开，通过引用文档的 **id** 字段来建立关系

```mongo
{
   "_id":ObjectId("52ffc33cd85242f436000001"),
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin",
   "address_ids": [
      ObjectId("52ffc4a5d85242602e000000"),
      ObjectId("52ffc4a5d85242602e000001")
   ]
}
```

以上实例中，用户文档的 **address_ids** 字段包含用户地址的对象id（ObjectId）数组。

我们可以读取这些用户地址的对象id（ObjectId）来获取用户的详细地址信息。

这种方法需要两次查询，第一次查询用户地址的对象id（ObjectId），第二次通过查询的id获取用户的详细地址信息

```java
>var result = db.users.findOne({"name":"Tom Benzamin"},{"address_ids":1})
>var addresses = db.address.find({"_id":{"$in":result["address_ids"]}})
```

## MongoDB 数据库引用

在上一章节MongoDB关系中我们提到了MongoDB的引用来规范数据结构文档。

MongoDB 引用有两种

- 手动引用（Manual References）
- DBRefs

### why

考虑这样的一个场景，我们在不同的集合中 (address_home, address_office, address_mailing, 等)存储不同的地址（住址，办公室地址，邮件地址等）

这样，我们**在调用不同集合地址时，也需要指定集合，手动指定集合名称太繁琐。一个文档从多个集合引用文档，我们应该使用 DBRefs**

### 使用 DBRefs

DBRef的形式

```
{ $ref : , $id : , $db :  }
```

三个字段表示的意义为

- $ref：集合名称
- $id：引用的id
- $db:数据库名称，可选参数

以下实例中用户数据文档使用了 DBRef, 字段 address

```
{
   "_id":ObjectId("53402597d852426020000002"),
   "address": {
   "$ref": "address_home",
   "$id": ObjectId("534009e4d852427820000002"),
   "$db": "runoob"},
   "contact": "987654321",
   "dob": "01-01-1991",
   "name": "Tom Benzamin"
}
```

**address** DBRef 字段指定了引用的地址文档是在 runoob 数据库下的 address_home 集合，id 为 534009e4d852427820000002。

以下代码中，我们通过指定 $ref 参数（address_home 集合）来查找集合中指定id的用户地址信息

```java
>var user = db.users.findOne({"name":"Tom Benzamin"})
>var dbRef = user.address
>db[dbRef.$ref].findOne({"_id":(dbRef.$id)})
```

以上实例返回了 address_home 集合中的地址数据

```
{
   "_id" : ObjectId("534009e4d852427820000002"),
   "building" : "22 A, Indiana Apt",
   "pincode" : 123456,
   "city" : "Los Angeles",
   "state" : "California"
}
```

# 系统架构

 MongoDB 与 MySQL 架构相差不多，底层都使用了『可插拔』的存储引擎以满足用户的不同需要

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311121691.webp)

# 存储引擎

## Wiredtiger

从MongoDB 3.2开始，WiredTiger存储引擎开始作为默认的存储引擎(之前为MMAPV1引擎)。 对于现有部署，如果未指定参数--storageEngine或storage.engine设置，则版本3.2+ mongod实例可以自动确定用于在--dbpath或storage.dbPath中创建数据文件的存储引擎

### Transport Layer业务层

`Transport Layer`是处理请求的基本单位。Mongo有专门的`listener`线程，每次有连接进来，`listener`会创建一个新的线程`conn`负责与客户端交互，它把具体的查询请求交给`network`线程，真正到数据库里查询由`TaskExecutor`来进行

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311124866.webp)

### **快照与检查点**

WiredTiger使用MultiVersion并发控制（MVCC）方式。 在操作开始时，WiredTiger为操作提供数据的时间点快照。 快照提供了内存数据的一致视图

写入磁盘时，WiredTiger将所有数据文件中的快照中的所有数据以一致的方式写入磁盘。 现在持久的数据充当数据文件中的检查点。 该检查点可确保数据文件直到最后一个检查点（包括最后一个检查点）都保持一致； 即检查点可以充当恢复点

从3.6版本开始，MongoDB配置WiredTiger以60秒的间隔创建检查点（即将快照数据写入磁盘）。 在早期版本中，MongoDB将检查点设置为在WiredTiger中以60秒的间隔或在写入2GB日志数据时对用户数据进行检查，以先到者为准

在写入新检查点期间，先前的检查点仍然有效。 这样，即使MongoDB在写入新检查点时终止或遇到错误，重启后，MongoDB仍可从上一个有效检查点恢复

   WiredTiger的写操作会默认写入`Cache`,并持久化到`WAL`(Write Ahead Log)，每60s做一次`checkpoint`，产生快照文件。WiredTiger初始化时，恢复至最新的快照状态，然后根据WAL恢复数据，保证数据的完整性。 Cache是基于BTree的，节点是一个page，root page是根节点，internal page是中间索引节点，leaf page真正存储数据，数据以page为单位与磁道读写。Wiredtiger采用Copy on write的方式管理修改操作（insert、update、delete），修改操作会先缓存在cache里，持久化时，修改操作不会在原来的leaf page上进行，而是写入新分配的page，每次checkpoint都会产生一个新的root page

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97ca47f723444b92ba586c46d76ec180~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311125127.webp)

### Journaling

WiredTiger将预写日志（即日志）与检查点结合使用以确保数据持久性

WiredTiger日记保留检查点之间的所有数据修改。 如果MongoDB在检查点之间退出，它将使用日志重播自上一个检查点以来修改的所有数据

WiredTiger日志使用快速压缩库进行压缩。 要指定其他压缩算法或不进行压缩，请使用storage.wiredTiger.engineConfig.journalCompressor设置参数

> 注意
>
> 如果日志记录小于或等于128字节（WiredTiger的最小日志记录大小），则WiredTiger不会压缩该记录。
>
> 您可以通过将storage.journal.enabled设置为false来禁用独立实例的日志记录，这可以减少维护日志记录的开销。 对于独立实例，不使用日志意味着MongoDB意外退出时，您将丢失最后一个检查点之后的所有数据修改信息

#### 日志记录进程

通过日志记录，WiredTiger 为每个客户端发起的写入操作创建一条日志记录。日志记录包括由初始写入引起的任何内部写入操作。例如，对集合中文档的更新可能会导致对索引的修改；WiredTiger 创建一条日志记录，其中包括更新操作及其关联的索引修改。

MongoDB 配置 WiredTiger 使用内存缓冲来存储日志记录。线程进行协调以分配并复制到它们的缓冲区部分。所有不超过 128 kB 的日记记录都会被缓冲。

当满足以下任一条件时，WiredTiger 会将缓冲的日志记录同步到磁盘：

- 对于副本集成员（主节点和从节点成员）：
  - 如果写入操作包含或暗示 [`j: true` 的写关注。](https://www.mongodb.com/zh-cn/docs/manual/reference/write-concern/#mongodb-writeconcern-writeconcern.j)
  - 此外，对于从节点，在每次批量应用 oplog 条目之后进行

- 每 100 毫秒一次（请参阅 [`storage.journal.commitIntervalMs`](https://www.mongodb.com/zh-cn/docs/manual/reference/configuration-options/#mongodb-setting-storage.journal.commitIntervalMs)）
- 当 WiredTiger 创建新的日志文件时。由于 MongoDB 使用上限 100 MB 的日志文件，所以 WiredTiger 大约每 100 MB 数据创建一份新日志文件

#### 日志文件大小限制

WiredTiger 日志文件的大小限制约为 100 MB。一旦文件超过该限制，WiredTiger 就会创建一个新的日志文件

WiredTiger 会自动删除旧日志文件，仅保留从上一个检查点恢复所需的文件

#### 实现原理

**存储视图**

Journaling功能用到了两个重要的内存视图：共享视图(shared view)和私有视图(private view)。这两个内存视图都是通过MMAP（内存映射）来实现的

- private view映射内存的修改不会(直接)影响磁盘；private view是为读操作保存数据的位置，同时也是mongodb保存新的写操作的第一个地方
- shared view中的数据变化会影响到磁盘上的文件，系统会周期性的刷新shared view中的数据到磁盘

​    启动mongod的时候，数据文件会被映射到共享视图(shared view)。此时操作系统只是完成映射，并不会立即加载数据到内存，mongdb会根据需要加载数据到shared view(按需加载)

> 注：关于上述MMAP可能有人会有疑问,磁盘那么大内存那么小怎么可能都映射过来？这里之说一点就好了：映射到的是虚拟内存，而不是物理内存。当真正需要访问某数据的时候才通过触发“缺页中断”的方式把实际数据加载到物理内存；物理内存实际映射的虚拟页也是会不断轮替的

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311520637.png)

**工作原理**

写操作或修改发生。进程会修改内存中的数据，此时磁盘上的文件数据就与内存中的数据不一致了。此时根据Journaling功能是否打开分为两种情况

1.   没打开。操作系统每60秒将内存中的变化刷新到磁盘。显然这个不是我们讨论的重点
2.  启用journaling的时候，mongod会再做一次映射，映射出一个私有视图(private view)，如图2所示。值得注意的是私有视图没有映射到数据文件，所以操作系统不能直接的将private视图中的任何改变刷新到磁盘

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311521580.png)

当写操作发生时，mongdb首先将数据写到内存中的private视图处，如图3所示。然后mongdb将写操作批量复制到journal，journal会将写操作存储到磁盘文件上使其持久化保存(默认间隔100ms)。journal日志文件上的每一个条目都描述了写操作更改了数据文件的那些字节，如图4所示。**关于这一步mongdb采用的group commit是的方式**

 Mongdb采用group commits的方式将写操作批量复制到journal日志文件中。**group commit提交方式能够最小化journal日志机制对性能的影响**(意思是说:如果每次操作都同步Journal的话对性能影响就太大了,这个是不可接受的)。因为group commit方式在提交过程中必须阻塞所有写入操作

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311523993.png)



**由于数据文件的变化被持久化到了journal日志文件中，即便此时mongodb服务器意外崩溃，写操作也是安全的。因为当数据库重新启动的时候会去读journal日志文件，并将写操作引起的变化重新同步到数据文件中去。**

上面步骤完成后，mongodb会利用journal日志中的写操作记录引起的数据变化来更新shared视图中的数据(执行journal的写操作到shared视图)，如图6。当所有的变化操作都更新到shared视图后，mongodb将重新利用shared视图来映射出一个全新的private视图，防止private视图变得“太脏”，这个时候其内存空间恢复到初始值，约为0(这里应该也是写时复制copy-on-write的意思，当被修改的时候才回去真正的同shared view分开)。

此时shared视图内存中的数据与磁盘上的数据变得不一致了。mongodb会周期性的要求操作系统将shard视图中变化的数据刷新(flush)到磁盘上，使磁盘上的数据与内存中的数据保持一致，如图7。如果系统内存资源不足的时候，操作系统就会选择以更高的频率刷入shared视图到磁盘

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311523271.png)

当执行完刷新内存中变化的数据到磁盘后，mongdb会删除点journal中的这个时间点之后的所有写操作，这一点与关系型数据中的checkpoint类似。

mongodb的Journaling日志功能是默认开启的。上面的过程涉及两个周期。一个是将写操作周期性同步到journal日志文件的周期，这个周期大小通过journal Commitlnterval来控制，默认值是100ms。另一个是mongdb经过60s的周期刷新内存中的变化数据到磁盘，这个值是通过可选参数syncdely来控制的。这些默认值一般适应于大多数情况，不要轻易更改。其实通过上面分析我们知道数据库服务器仍然有100ms时段的数据丢失风险，因为Journaling日志写到磁盘的周期是100ms，倘若刚好一批写操作还在内存没等到下一个100ms同步到journal磁盘文件服务器就故障了，这些还在内存中的数据就会丢失

#### **journal日志带来的问题**

**影响一：对mongdb写入性能的影响**

​    由于提交journal日志会产生写入阻塞，所以他会对写入操作的性能产生影响；不过好在对读操作不会有影响。另外在生产环境开启Journaling是很有必要的

**影响二：对磁盘空间的影响**

​    Journal就是一个磁盘上的持久化文件，显然也会产生磁盘容量的消耗。这也是mongodb吃磁盘容量的原因之一。

​    开启journal日志功能，mongodb会在数据目录下创建一个journal文件夹，用来存放预写重放日志。如果mongdb安全关闭，会自动删除目录下的文件；如果崩溃导致的关闭则不会删除以供重启时自动修复数据。journal日志文件是一个不断往文件尾追加内容的文件，他的命令是j._开头后面接一个数字作为序列号。如果文件超过100MB，mongodb会新建journal文件。对于已经flush到磁盘的数据文件，mongodb会删除或这种利用，总之不会保留因为这些数据已经不会被用来数据恢复了

总结：其实这样就说通了。每60秒间隔数据就会刷新到磁盘中，这个被称为检查点，这些数据已经持久化存储没什么可担心的。此外还有更细粒度的保障那就是100ms周期的预写入日志Journaling。每100ms修改就会持久化到journal中，显然持久化进去后数据也是安全的。唯一不安全的就是两次批量提交journal的这100ms间隔的修改。这个没办法解决，group commit降低journal对性能的影响的必要手段，当然你可以通过调整这个Commitlnterval来实现权衡，但100ms已经是相对最优的间隔



### 数据持久性

**WiredTiger将预写日志（即日志）与检查点结合使用以确保数据持久性**

为了在数据库宕机保证 MongoDB 中数据的持久性，MongoDB 使用了 **Write Ahead Logging** 向磁盘上的 journal 文件预先进行写入；除了 journal 日志，MongoDB 还使用**检查点（Checkpoint）**来保证数据的一致性，当数据库发生宕机时，我们就需要 Checkpoint 和 journal 文件协作完成数据的恢复工作：

- 在数据文件中查找上一个检查点的标识符；
- 在 journal 文件中查找标识符对应的记录；
- 重做对应记录之后的全部操作；

MongoDB 会每隔 60s设置一次检查点，当然我们也可以通过在写入时传入 `j: true` 的参数强制 journal 文件的同步

### **内存使用**

 关于mongodb内存的占用，其实包括两块(或者说叫两级缓存)。一级是WT cache(WiredTiger内部缓存)，另一级是filesystem cache(文件系统缓存或者fs.cache)。其中前者的占用是可调的，后者具体占用多少物理内存应该就是操作系统的事情了。对于Linux系统来说其默认设置倾向于把内存尽可能的用于文件cache

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311536618.png)

从MongoDB 3.4开始，默认的WiredTiger内部缓存大小是以下两者中的较大者： 50％（RAM-1 GB）或256 MB

例如，在总共有4GB RAM的系统上，WiredTiger缓存将使用1.5GB RAM（0.5 *（4 GB-1 GB）= 1.5 GB）。 相反，总内存为1.25 GB的系统将为WiredTiger缓存分配256 MB，因为这是总RAM的一半以上减去一GB（0.5* （1.25 GB-1 GB）= 128 MB

> **关于文件系统缓存**
>
> ​    操作系统为了提高IO的性能而引入了文件系统缓存(File System Cache),系统会将一批IO操作的修改暂存到缓存，周期性的提交给磁盘。
>
> ​    cache的规则很简单。我们肯定希望cache能够尽可能满足正在执行的工作，从应用程序栈的角度考虑越往下层cache越不那么高效。所以，**如果你的应用程序有能力去cache，最好就把内存更多的留给应用程序而不是文件系统**。不过文件系统缓存在一些场合仍然是有用的，比如写日志。如果让你做一个选择是给数据库10G内存还是给文件系统cache10G内存，显然应该把内存给数据库。**所以\**高效的磁盘数据库往往自己提供了缓存，而不是依赖于文件系统缓存\****。从理论上讲，Mysql Innodb实现了自己缓存的数据库会更智能，应该比Mongodb这样依赖于文件系统来刷新数据的数据库要来的高

### 一致性

1. WiredTiger使用`Copy on Write`管理修改操作。修改先放在cache中，并持久化，不直接作用在原leaf page，而是写入新分配的page，每次checkpoint产生新page。 相关文件

- WiredTiger.basecfg: 存储基本配置信息，与ConfigServer有关系
- WiredTiger.lock: 定义锁操作
- table*.wt: 存储各张表的数据
- WiredTiger.wt: 存储table* 的元数据
- Wile: 存储WiredTiger.wt的元数据
- journal: 存储WAL
- 一次Checkpoint的大致流程如下:

对所有的table进行一次Checkpoint，每个table的Checkpoint的元数据更新至WiredTiger.wt 对WiredTiger.wt进行Checkpoint，将该table Checkpoint的元数据更新至临时文件WiredTiger.turtle.set 将WiredTiger.turtle.set重命名为WiredTiger.turtle。 上述过程如中间失败，Wiredtiger在下次连接初始化时，首先将数据恢复至最新的快照状态，然后根据WAL恢复数据，以保证存储可靠性

# 日志

## 系统日志

  系统日志在MongoDB中十分重要，它记录MongoDB启动和停止的操作，以及服务器在运行过程中发生的任何异常信息；配置系统日志非常简单，在运行mongod时候增加额外参数或者写入配置文件即可

```bash
#在启动mongoDB时使用-logpath指定日志的路径，-logappend表示使用追加的方式写日志
mongod -logpath='/usr/local/mongodb/log/rs1.log' -logappend
#也可以在配置文件中指定参数进行配置：
logpath=/usr/local/mongodb/log/rs1.log #日志文件路径
logappend=true #表示使用追加的方式写日志
verbose=true #表示会打印debug信息
vv=true #表示debug级别，有vv-vvvvv，v越多则记录的日志信息越详细。
#quiet=true #quiet=true，表示安静地输出，不会再有debug信息，日志中只会打印一些关键的信息，比如自动故障切换，系统错误等信息，相当于error log。这时需要注释掉verbose参数。
```

## Journal日志

  不开启journal日志，写入wiredtiger的数据，并不会立即持久化存储；而是每分钟会做一次全量的checkpoint，将所有的数据持久化。
  开启journal日志后，数据的更新就先写入Journal日志（其中包含了此次写入操作具体更改的磁盘地址和字节），定期集中提交，然后在正式数据执行更改。这样即使出现宕机，启动时Wiredtiger会先将数据恢复到最近的一次checkpoint的点，然后重放后续的journal操作日志来恢复数据。
  由于MongoDB使用的journal文件大小限制为100MB,因此WiredTiger大约每100MB数据创建一个新的日志文件。
  向MongoDB中写入数据是先写入内存，然后每隔60s在刷盘，同样写入journal,也是先写入对应的buffer，然后每隔50ms在刷盘到磁盘的journal文件。
  需要注意的是如果客户端的写入速度超过了日志的刷新速度，mongod则会限制写入操作，直到日志完成磁盘的写入。这是mongod会限制写入的唯一情况。
  Jouranl日志就是预写入的redo日志，是用于数据故障恢复和持久化数据的，为mongodb增加了额外的可靠性保障。journal除了故障恢复的作用之外，还可以提高写入的性能，批量提交。在这个过程中，所有的写入都可以一次提交，是单事务的，全部成功或者全部失败。启动数据库的Journal功能非常简单，只需写入配置文件即可

```bash
#在启动时指定--journal代表启动journal日志。
mongod --journal
journal=true #启用journal日志
journalCommitInterval=100 #刷写提交机制。可修改范围是2~300，值越低，刷新输出频率越高，数据安全度也就越高，但磁盘性能上的开销也更高。
```

### oplog主从日志

  MongoDB的高可用复制策略有一个叫做副本集。副本集的复制过程中有一个服务器充当主服务器，而一个或多个充当从服务器，主服务器将更新写入oplog，oplog记录着发生在主服务器的更新操作，oplog是主节点的local数据库中的固定集合(oplog.rs)中，并将这些操作分发到从服务器上。
  每个备份节点都维护着自己的oplog,记录着每一次从主节点复制数据的操作。这样，每个成员都可以作为同步源给其他成员使用。
  备份节点从当前使用的同步源中获取需要执行的操作，然后在自己的数据集上执行这些操作，最后再将这些操作写入自己的oplog,如果遇到某个操作失败的情况(只有当同步源的数据损坏或者数据与主节点不一致时才可能发生),那么备份节点就会停止从当前的同步源复制数据。
  oplog中按顺序保存着所有执行过的写操作，副本集中每个成员都维护着一份自己的oplog，每个成员的oplog都应该跟主节点的oplog完全一致。
  如果某个备份节点由于某些原因挂了，但它重新启动后，就会自动从oplog中最后一个操作开始进行同步。由于复制操作的过程是复制数据再写入oplog,所以备份节点可能会在已经同步过的数据上再次执行复制操作。MongoDB在设计之初就考虑到了这种情况:将oplog中的同一个操作执行多次，与只执行一次的效果是一样的。
  oplog是固定集合，MongoDB默认将其大小设置为可用disk空间的5%（默认最小为1G，最大为50G），也可以在MongoDB复制集实例初始化之前将mongo.conf中oplogSize设置为我们需要的值。它只能保持特定数量的操作日志，并且如果执行大量的批量操作，oplog很快就会被填满

```bash
#oplog各字段详解
ts: 8字节的时间戳，由4字节unix timestamp + 4字节自增计数表示。这个值很重要，在选举(如master宕机时)新primary时，会选择ts最大的那个secondary作为新primary
op：1字节的操作类型
"i"： insert
"u"： update
"d"： delete
"c"： db cmd
"db"：声明当前数据库 (其中ns 被设置成为=>数据库名称+ '.')
"n": no op,即空操作，其会定期执行以确保时效性
ns：操作所在的namespace
o：操作所对应的document，即当前操作的内容（比如更新操作时要更新的的字段和值）
o2: 在执行更新操作时的where条件，仅限于update时才有该属性
```

### 慢查询日志

说到MongoDB的慢查询日志，就不得不提到profile分析器，profile分析器将记录的慢日志写到当前库下的system.profile集合下，这个集合是一个固定集合，使用db.system.profile.find()进行查询,慢日志可以在启动时或者在配置文件中进行指定

```bash
#在启动mongoDB时使用--profile=1启动慢日志，--slowms表示慢日志阈值时间
mongod --profile=1 --slowms=5 
#也可以在配置文件中指定参数
slowms=100 #单位是毫秒
profile = 1#0：关闭，不收集任何数据。1：收集慢查询数据，默认是100毫秒。2：收集所有数据
```

```bash
#慢日志内容格式详解
{
    "op" : "query",  #操作类型，有insert、query、update、remove、getmore、command   
    "ns" : "test.test", #操作的集合
    "query" : {
        "$query" : {
            "user_id" : 314436841,
            "data_time" : {
                "$gte" : 1436198400
            }
        },
        "$orderby" : {
            "data_time" : 1
        }
    },
    "ntoskip" : 0, #指定跳过skip()方法的文档的数量。
    "nscanned" : 2, #为了执行该操作，MongoDB在index中浏览的文档数。一般来说，如果nscanned值高于nreturned 的值，说明数据库为了找到目标文档扫描了很多文档。这时可以考虑创建索引来提高效率。
    "nscannedObjects" : 1, #为了执行该操作，MongoDB在collection中浏览的文档数。
    "keyUpdates" : 0, #索引更新的数量，改变一个索引键带有一个小的性能开销，因为数据库必须删除旧的key，并插入一个新的key到B-树索引
    "numYield" : 1,  #该操作为了使其他操作完成而放弃的次数。通常来说，当他们需要访问还没有完全读入内存中的数据时，操作将放弃。这使得在MongoDB为了放弃操作进行数据读取的同时，还有数据在内存中的其他操作可以完成
    "lockStats" : {#锁信息，R：全局读锁；W：全局写锁；r：特定数据库的读锁；w：特定数据库的写锁
        "timeLockedMicros" : {  #该操作获取一个级锁花费的时间。对于请求多个锁的操作，比如对local数据库锁来更新oplog，该值比该操作的总长要长（即millis ）
            "r" : NumberLong(1089485),
            "w" : NumberLong(0)
        },
        "timeAcquiringMicros" : {  #该操作等待获取一个级锁花费的时间。
            "r" : NumberLong(102),
            "w" : NumberLong(2)
        }
    },
    "nreturned" : 1,  // 返回的文档数量
    "responseLength" : 1669, // 返回字节长度，如果这个数字很大，考虑值返回所需字段
    "millis" : 544, #消耗的时间（毫秒）
    "execStats" : {  #一个文档,其中包含执行 查询 的操作，对于其他操作,这个值是一个空文件， system.profile.execStats 显示了就像树一样的统计结构，每个节点提供了在执行阶段的查询操作情况。
        "type" : "LIMIT", ##使用limit限制返回数  
        "works" : 2,
        "yields" : 1,
        "unyields" : 1,
        "invalidates" : 0,
        "advanced" : 1,
        "needTime" : 0,
        "needFetch" : 0,
        "isEOF" : 1,  #是否为文件结束符
        "children" : [
            {
                "type" : "FETCH",  #根据索引去检索指定document
                "works" : 1,
                "yields" : 1,
                "unyields" : 1,
                "invalidates" : 0,
                "advanced" : 1,
                "needTime" : 0,
                "needFetch" : 0,
                "isEOF" : 0,
                "alreadyHasObj" : 0,
                "forcedFetches" : 0,
                "matchTested" : 0,
                "children" : [
                    {
                        "type" : "IXSCAN", #扫描索引键
                        "works" : 1,
                        "yields" : 1,
                        "unyields" : 1,
                        "invalidates" : 0,
                        "advanced" : 1,
                        "needTime" : 0,
                        "needFetch" : 0,
                        "isEOF" : 0,
                        "keyPattern" : "{ user_id: 1.0, data_time: -1.0 }",
                        "boundsVerbose" : "field #0['user_id']: [314436841, 314436841], field #1['data_time']: [1436198400, inf.0]",
                        "isMultiKey" : 0,
                        "yieldMovedCursor" : 0,
                        "dupsTested" : 0,
                        "dupsDropped" : 0,
                        "seenInvalidated" : 0,
                        "matchTested" : 0,
                        "keysExamined" : 2,
                        "children" : [ ]
                    }
                ]
            }
        ]
    },
    "ts": ISODate("2019-08-31T10:05:21.401Z"), #登陆时间
    "client": "127.0.0.1", #登陆IP或地址
    "appName": "MongoDB Shell",
    "allUsers": [],
    "user": ""
}
```













