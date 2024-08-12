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

# Mongo数据库连接池

每个 `MongoClient` 实例都管理创建 `MongoClient` 时指定的 MongoDB 集群或节点的连接池。`MongoClient` 对象在大多数驱动程序中都具有线程安全性

```xml
# Configure spring.data.mongodbDB Pool
# 每个host的TCP连接数
spring.data.mongodb.connectionsPerHost=10
# 每个host的最小TCP连接数
spring.data.mongodb.minConnectionsPerHost=10
# 计算允许多少个线程阻塞等待可用TCP连接时的乘数（ 连接池里的连接数 ）
# 算法：threadsAllowedToBlockForConnectionMultiplier*connectionsPerHost
spring.data.mongodb.threadsAllowedToBlockForConnectionMultiplier=1
spring.data.mongodb.serverSelectionTimeout=5000
# 当连接池无可用连接时客户端阻塞等待的时长，单位毫秒
spring.data.mongodb.maxWaitTime=15000
# TCP连接闲置时间,单位毫秒
spring.data.mongodb.maxConnectionIdleTime=3600000
# TCP连接最多可以使用多久,单位毫秒
spring.data.mongodb.maxConnectionLifeTime=3600000
# 连接超时时间,必须大于0,单位毫秒
spring.data.mongodb.connectTimeout=5000
# socket超时时间,单位毫秒
spring.data.mongodb.socketTimeout=5000
spring.data.mongodb.socketKeepAlive=true
spring.data.mongodb.sslEnabled=false
spring.data.mongodb.sslInvalidHostNameAllowed=false
spring.data.mongodb.alwaysUseMBeans=false
# 设置心跳频率。 这是驱动程序将尝试确定群集中每个服务器的当前状态的频率。 默认值为10,000毫秒
spring.data.mongodb.heartbeatFrequency=10000
# 设置最小心跳频率。 如果驱动程序必须经常重新检查服务器的可用性，它将至少在上一次检查后等待很长时间，以避免浪费精力。 默认值为500毫秒。
spring.data.mongodb.minHeartbeatFrequency=500
# 设置用于集群心跳的连接的连接超时,单位毫秒
spring.data.mongodb.heartbeatConnectTimeout=20000
# 设置用于集群心跳的连接的套接字超时,单位毫秒
spring.data.mongodb.heartbeatSocketTimeout=20000
# 本地阈值
spring.data.mongodb.localThreshold=15
```

# 系统架构

 MongoDB 与 MySQL 架构相差不多，底层都使用了『可插拔』的存储引擎以满足用户的不同需要

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407311121691.webp)

# [高可用架构](https://mongoing.com/archives/78417)

## WHAT

高可用性 HA（High Availability）指的是缩短因**正常运维**或者**非预期故障**而导致的停机时间，提高系统可用性

那么问题来了，都说自己的服务高可用，高可用能量化衡量吗？能不能比出个高低呢？

**可以，这里引出一个 SLA 的概念**。SLA 是 Service Level Agreement 的缩写，中文含义：服务等级协议。SLA 就是用来量化可用性的协议，在双方认可的前提条件下，服务提供商与用户间定义的一种双方认可的协定。SLA 是判定服务质量的重要指标。

问题来了，SLA 是怎么量化的？其实就是按照停服时间算的。怎么算的？举个例子：

**1 年 = 365 天 = 8760 小时** 
**99.9 停服时间：8760 \* 0.1% = 8760 \* 0.001 = 8.76小时** 
**99.99 停服时间：8760 \* 0.0001 = 0.876 小时 = 52.6 分钟** 
**99.999 停服时间：8760 \* 0.00001 = 0.0876 小时 = 5.26分钟**

也就是说，如果一家公有云厂商提供对象存储的服务，SLA 协议指明提供 5 个 9 的高可用服务，那就要保证一年的时间内对象存储的停服时间少于 5.26 分钟，如果超过这个时间，就算违背了 SLA 协议，可以找公有云提出赔偿。

说回高可用的话题，大白话就是，无论出啥事都不能让承载的业务受影响，这就是高可用。

前面我们说过，无论是数据的**高可靠**，还是组件的**高可用**全都是一个解决方案：冗余。我们通过多个组件和备份导致对外提供一致性和不中断的服务。**冗余是根本，但是怎么来使用冗余则各有不同**。

以下我们就按照不同的冗余处理策略，可以总结出 MongoDB 几个特定的模式，这个也是通用性质的架构，在其他的分布式系统也是常见的。

我们从 Mongo 的三种高可用模式逐一介绍，这三种模式也代表了通用分布式系统下高可用架构的进化史，分别是 Master-Slave，Replica Set，Sharding 模式

## **Master-Slave 模式**

------

Mongodb 提供的第一种冗余策略就是 Master-Slave 策略，这个也是分布式系统最开始的冗余策略，这种是一种热备策略。

Master-Slave 架构一般用于备份或者做读写分离，一般是一主一从设计和一主多从设计。

Master-Slave 由主从角色构成：

**Master ( 主 )**

可读可写，当数据有修改的时候，会将 Oplog 同步到所有连接的Salve 上去。

**Slave ( 从 )**

只读，所有的 Slave 从 Master 同步数据，从节点与从节点之间不感知。

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041614023.png)

通过上面的图，这是一种典型的扇形结构。

### Master-Slave 对读写分离的思考

Master 对外提供读写服务，有多个 Slave 节点的话，可以用 Slave 节点来提供读服务的节点。

#### **思考，这种读写分离有什么问题？**

有一个不可逾越的问题：**数据不一致问题**。根本原因在于只有 Master 节点可以写，Slave 节点只能同步 Master 数据并对外提供读服务，所以你会发现这个是一个异步的过程。

虽然最终数据会被 Slave 同步到，在数据完全一致之前，数据是不一致的，这个时候去 Slave 节点读就会读到旧的数据。所以，总结来说：**读写分离的结构只适合特定场景，对于必须需要数据强一致的场景是不合适这种读写分离的**。

### Master-Slave 对容灾的思考

当 Master 节点出现故障的时候，由于 Slave 节点有备份数据，有数据就好办呀。只要有数据还在，对用户就有交代。这种 Master 故障的时候，可以通过人为 Check 和操作，手动把 Slave 节点指定为 Master 节点，这样又能对外提供服务了。

思考下这种模式有什么特点？

1. Master-Slave 只区分两种角色：Master 节点，Slave 节点；
2. Master-Slave 的角色是静态配置的，不能自动切换角色，必须人为指定；
3. 用户只能写 Master 节点，Slave 节点只能从 Master 拉数据；
4. 还有一个关键点：Slave 节点只和 Master 通信，Slave 之间相互不感知，这种好处对于 Master 来说优点是非常轻量，缺点是：系统明显存在单点，那么多 Slave 只能从 Master 拉数据，而无法提供自己的判断；

**以上特点存在什么问题？**

**最大的第一个问题就是可用性差**。因为很容易理解，因为主节点挂掉的时候，必须要人为操作处理，这里就是一个巨大的停服窗口；

### Master-Slave 的现状

MongoDB 3.6 起已不推荐使用主从模式，自 MongoDB 3.2 起，分片群集组件已弃用主从复制。因为 Master-Slave 其中 Master 宕机后不能自动恢复，只能靠人为操作，可靠性也差，操作不当就存在丢数据的风险。

## Replica Set 副本集模式

  ### **Replica Set 模式角色**

  Replica Set 是 mongod 的实例集合，包含三类节点角色：

**Primary（ 主节点 ）**

  只有 Primary 是可读可写的，Primary 接收所有的写请求，然后把数据同步到所有 Secondary 。一个 Replica Set 只有一个 Primary 节点，当 Primary 挂掉后，其他 Secondary 或者 Arbiter 节点会**重新选举**出来一个 Primary 节点，这样就又可以提供服务了。

  读请求默认是发到 Primary 节点处理，如果需要故意转发到 Secondary 需要客户端修改一下配置（注意：是客户端配置，决策权在客户端）。

  那有人又会想了，这里也存在 Primary 和 Secondary 节点角色的分类，岂不是也存在单点问题？

  这里和 Master-Slave 模式的最大区别在于，Primary 角色是通过整个集群共同选举出来的，人人都可能成为 Primary ，人人最开始只是  Secondary ，而这个选举过程完全自动，不需要人为参与。

**Secondary（ 副本节点 ）**

  数据副本节点，当主节点挂掉的时候，参与选主。

- 思考一个问题：Secondary 和 Master-Slave 模式的 Slave 角色有什么区别？

  最根本的一个不同在于：Secondary 相互有心跳，Secondary 可以作为数据源，Replica 可以是一种链式的复制模式。

**Arbiter（ 仲裁者 ）**

  不存数据，不会被选为主，只进行选主投票。使用 Arbiter 可以减轻在减少数据的冗余备份，又能提供高可用的能力。

  ![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041621896.png)

### **副本集模式特点思考**

  MongoDB 的 Replica Set 副本集模式主要有以下几个特点：

  - 数据多副本，在故障的时候，可以使用完的副本恢复服务。注意：**这里是故障自动恢复**；
  - 读写分离，读的请求分流到副本上，减轻主（Primary）的读压力；
  - 节点直接互有心跳，可以感知集群的整体状态；

  **思考：这种有什么优缺点呢？**

  可用性大大增强，因为故障时自动恢复的，主节点故障，立马就能选出一个新的 Primary 节点。但是有一个要注意的点：每两个节点之间互有心跳，这种模式会导致节点的心跳几何倍数增大，单个 Replica Set 集群规模不能太大，一般来讲最大不要超过 50 个节点。

  **思考：节点数有讲究吗？**

  有的，参与投票节点数要是**奇数**，这个非常重要。为什么，因为偶数会导致脑裂，也就是投票数对等的情况，无法选出 Primary。

  举个例子，如果有 3 张票，那么一定是 2:1 ，有一个人一定会是多数票，如果是 4 张票，那么很有可能是 2:2 ，那么就有平票的现象。

## Sharding 模式

按道理 Replica Set 模式已经非常好的解决了可用性问题，为什么还会往后演进呢？因为在当今大数据时代，有一个必须要考虑的问题：**就是数据量**。

  用户的数据量是永远都在增加的，理论是没有上限的，但 Replica Set 却是有上限的。怎么说？

  举个例子，假设说你的单机有 10TiB 的空间，内存是 500 GiB，网卡是 40 G，这个就是单机的物理极限。当数据量超过 10 TiB，这个 Replica Set 就无法提供服务了。你可能会说，那就加磁盘喽，把磁盘的容量加大喽。是可以，但是单机的容量和性能一定是有物理极限的（比如说你的磁盘槽位可能最多就 60 盘）。单机存在瓶颈怎么办？

  **解决方案就是：利用分布式技术**。

  解决性能和容量瓶颈一般来说优化有两个方向：

  1. 纵向优化
  2. 横向优化

  **纵向优化**是传统企业最常见的思路，持续不断的加大单个磁盘和机器的容量和性能。CPU 主频不断的提升，核数也不断地加，磁盘容量从 128 GiB 变成当今普遍的 12 TiB，内存容量从以前的 M 级别变成现在上百 G 。带宽从以前百兆网卡变成现在的普遍的万兆网卡，但这些提升终究追不上用互联网数据规模的增加量级。

  **横向优化**通俗来讲就是加节点，横向扩容来解决问题。业务上要划分系统数据集，并在多台服务器上处理，做到容量和能力跟机器数量成正比。单台计算机的整体速度或容量可能不高，但是每台计算机只能处理全部工作量的一部分，因此与单台高速大容量服务器相比，可能提供更高的效率。

  扩展的容量仅需要根据需要添加其他服务器，这比一台高端硬件的机器成本还低，代价就是软件的基础结构要支持，部署维护要复杂。

  那么，实际情况下，哪一种更具可行性呢？

  自然是分布式技术的方案，纵向优化的方案非常容易到达物理极限，横向优化则对个体要求不高，而是群体发挥效果（但是对软件架构提出更高的要求）。

  2003年，Google 发布 Google File System 论文，这是一个可扩展的分布式文件系统，用于大型的、分布式的、对大量数据进行访问的应用。它运行于廉价的普通硬件上，提供分布式容错功能。GFS 正式拉开分布式技术应用的大门。

  MongoDB 的 Sharding 模式就是 MongoDB 横向扩容的一个架构实现。我们下面就看一下 Sharding 模式和之前 Replica Set 模式有什么特殊之处吧。

### **Sharding 模式角色**

  Sharding 模式下按照层次划分可以分为 3 个大模块：

  1. 代理层：mongos
  2. 配置中心：副本集群（mongod）
  3. 数据层：Shard 集群

  简要如下图：[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041621456.png)](https://mongoing.com/wp-content/uploads/2021/04/6f512a45003edb8.png)

  **代理层**：

  代理层的组件也就是 mongos ，这是个无状态的组件，纯粹是路由功能。向上对接 Client ，收到 Client 写请求的时候，按照特定算法均衡散列到某一个 Shard 集群，然后数据就写到 Shard 集群了。收到读请求的时候，定位找到这个要读的对象在哪个 Shard 上，就把请求转发到这个 Shard 上，就能读到数据了。

  **数据层**：

  数据层是啥？就是存储数据的地方。你会惊奇的发现，其实数据层就是由一个个 Replica Set 集群组成。在前面我们说过，单个 Replica Set 是有极限的，怎么办？那就搞多个 Replica Set ，**这样的一个 Replica Set 我们就叫做 Shard** 。理论上，Replica Set 的集群的个数是可以无限增长的。

  **配置中心**：

  代理层是无状态的模块，数据层的每一个 Shard 是各自独立的，那总要有一个集群统配管理的地方，这个地方就是配置中心。里面记录的是什么呢？

  比如：有多少个 Shard，每个 Shard 集群又是由哪些节点组成的。每个 Shard 里大概存储了多少数据量（以便做均衡）。这些东西就是在配置中心的。

  配置中心存储的就是集群拓扑，管理的配置信息。这些信息也非常重要，所以也不能单点存储，怎么办？**配置中心也是一个 Replica Set 集群，数据也是多副本的**

  详细架构图：[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041621680.png)](https://mongoing.com/wp-content/uploads/2021/04/8224336dded02c8.png)

### **Sharding 模式怎么存储数据？**

  我们说过，纵向优化是对硬件使用者最友好的，**横向优化则对硬件使用者提出了更高的要求，也就是说软件架构要适配**。

  单 Shard 集群是有限的，但 Shard 数量是无限的，Mongo 理论上能够提供近乎无限的空间，能够不断的横向扩容。那么现在唯一要解决的就是怎么去把用户数据存到这些 Shard 里？MongDB 是怎么做的？

  首先，要选一个字段（或者多个字段组合也可以）用来做 Key，这个 Key 可以是你任意指定的一个字段。我们现在就是要使用这个 Key 来，通过**某种策略**算出发往哪个 Shard 上。这个策略叫做：**Sharding Strategy** ，也就是分片策略。

  我们把 Sharding Key 作为输入，按照特点的 Sharding Strategy 计算出一个值，**值的集合形成了一个值域**，我们按照固定步长去切分这个值域，**每一个片叫做 Chunk** ，**每个 Chunk 出生的时候就和某个 Shard 绑定起来，这个绑定关系存储在配置中心里。**

  所以，我们看到 MongoDB 的用 Chunk 再做了一层抽象层，隔离了用户数据和 Shard 的位置，用户数据先按照分片策略算出落在哪个 Chunk 上，由于 Chunk 某一时刻只属于某一个 Shard，所以自然就知道用户数据存到哪个 Shard 了。

  Sharding 模式下数据写入过程：[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041622462.gif)](https://mongoing.com/wp-content/uploads/2021/04/88d9f53c52df97f.gif)Sharding 模式下数据读取过程：[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041622820.gif)](https://mongoing.com/wp-content/uploads/2021/04/d0286a4c6a73311.gif)

  通过上图我们也看出来了，mongos 作为路由模块其实就是寻路的组件，写的时候先算出用户 key 属于哪个 Chunk，然后找出这个 Chunk 属于哪个 Shard，最后把请求发给这个 Shard ，就能把数据写下去。读的时候也是类似，先算出用户 key 属于哪个 Chunk，然后找出这个 Chunk 属于哪个 Shard，最后把请求发给这个 Shard ，就能把数据读上来。

  实际情况下，mongos 不需要每次都和 Config Server 交互，大部分情况下只需要把 Chunk 的映射表 cache 一份在 mongos 的内存，就能减少一次网络交互，提高性能。

  **为什么要多一层 Chunk 这个抽象？**

  **为了灵活**，因为一旦是用户数据直接映射到 Shard 上，那就相当于是用户数据和底下的物理位置绑定起来了，这个万一 Shard 空间已经满了，怎么办？

  存储不了呀，又不能存储到其他地方去。有同学就会想了，那我可以把这个变化的映射记录下来呀，记录下来理论上行得通，但是每一个用户数据记录一条到 Shard 的映射，这个量级是非常大的，实际中没有可行性。

  而现在多了一层 Chunk 空间，就灵活了。用户数据不再和物理位置绑定，而是只映射到 Chunk 上就可以了。如果某个 Shard 数据不均衡，那么可以把 Chunk 空间分裂开，迁走一半的数据到其他 Shard ，修改下 Chunk 到 Shard 的映射，**Chunk 到 Shard 的映射条目很少**，完全 Hold 住，并且这种均衡过程用户完全不感知。

  讲回 Sharding Strategy 是什么？**本质上 Sharding Strategy 是形成值域的策略而已**，MongoDB 支持两种 Sharding Strategy：

  1. Hashed Sharding 的方式
  2. Range Sharding 的方式

  **Hashed Sharding**

  把 Key 作为输入，输入到一个 Hash 函数中，计算出一个整数值，值的集合形成了一个值域，我们按照固定步长去切分这个值域，每一个片叫做 Chunk ，**这里的 Chunk 则就是整数的一段范围而已**。[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041622080.gif)](https://mongoing.com/wp-content/uploads/2021/04/a52df3fc2c48e57.gif)

  这种计算值域的方式有什么优缺点呢？

  好处是：

  - 计算速度快
  - **均衡性好**，纯随机

  坏处是：

  - 正因为纯随机，排序**列举的性能极差**，比如你如果按照 name 这个字段去列举数据，你会发现几乎所有的 Shard 都要参与进来；

  **Range Sharding**

  **Range 的方式本质上是直接用 Key 本身来做值，形成的 Key Space 。**[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041622438.gif)](https://mongoing.com/wp-content/uploads/2021/04/fdcbfedb396b612.gif)

  如上图例子，Sharding Key 选为 name 这个字段，对于 “test_0″，”test_1″，”test_2” 这样的 key 排序就是挨着的，所以就全都分配在一个 Chunk 里。

  这 3 条 Docuement 大概率是在一个 Chunk 上，因为我们就是按照 Name 来排序的。这种方式有什么优缺点？

  好处是：

  - 对排序**列举场景非常友好**，因为数据本来就是按照顺序依次放在 Shard 上的，排序列举的时候，顺序读即可，非常快速；

  坏处是：

  - **容易导致热点**，举个例子，如果 Sharding Key 都有相同前缀，那么大概率会分配到同一个 Shard 上，就盯着这个 Shard 写，其他 Shard 空闲的很，却帮不上忙；

### **可用性的进一步提升**

  为什么说 Sharding 模式不仅是容量问题得到解决，可用性也进一步提升？

  因为 Shard（Replica Set）集群个数多了，即使一个或多个 Shard 不可用，Mongo 集群对外仍可以 提供读取和写入服务。因为每一个 Shard 都有一个 Primary 节点，都可以提供写服务，可用性进一步提升。

## **推荐使用姿势**

  上面已经介绍了历史演进的 3 种高可用模式，Master-Slave 模式已经在不推荐了，Relicate Set 和 Sharding 模式都可以保证数据的高可靠和高可用，但是在我们实践过程中，发现**客户端存在非常大的配置权限**，也就是说如果用户在使用 MongoDB 的时候使用姿势不对，可能会导致达不到你的预期。

### 使用姿势一：怎么保证高可用？

------

  如果是 Replicate Set 模式，那么客户端要主动感知主从切换。以前用过 Go 语言某个版本的 MongoDB client SDK，发现在主从切换的时候，并没有主动感知，导致请求还一直发到已经故障的节点，从而导致服务不可用。

  所以针对这种形式要怎么做？有两个方案：

  1. 用 Sharding 模式，因为 Sharding 模式下，用户打交道的是 mongos ，这个是一个代理，帮你屏端自己感知，
  1. 客户端自己感知，定期刷新（这种就相对麻烦）

### 使用姿势二：怎么保证数据的高可靠？

  客户端配置写多数成功才算成功。没错，**这个权限交由由客户端配置**。如果没有配置写多数成功，那么很可能写一份数据成功就成功了，这个时候如果发生故障，或者切主，那么数据可能丢失或者被主节点 rollback ，也等同用户数据丢失。

  mongodb 有完善的 rollback 及写入策略(WriteConcern)机制，但是也要使用得当。怎么保证高可靠？一定要写多数成功才算成功。

### 使用姿势三：怎么保证数据的强一致性？

  客户端要配置两个东西：

  1. 写多数成功，才算成功；
  2. 读使用 strong 模式，也就是只从主节点读；

  只有这两个配置一起上，才能保证用户数据的绝对安全，并且对外提供数据的强一致性。

## **总结**

  1. 本文介绍了 3 种 MongoDB 的高可用架构，Master-Slave 模式，Replica Set 模式，Sharding 模式，这也是常见的架构演进的过程；
  2. MongdbDB Master-Slave 已经不推荐，甚至新版已经不支持这种冗余模式；
  3. Replica Set 通过数据多副本，组件冗余提高了可靠性，并且通过分布式自动选主算法，减少了停服时间窗，提高了可用性；
  4. Sharding 模式通过横向扩容的方式，为用户提供了近乎无限的空间；
  5. MongoDB 客户端掌握了很大的配置权限，通过指定写多数策略和 strong 模式（只从主节点读数据）能保证数据的高可靠和强一致性。

# [MongoDB 一致性模型设计与实现](https://mongoing.com/archives/77853)

## MongoDB 可调一致性（Tunable Consistency）概念及理论支撑

我们都知道，早期的数据库系统往往是部署在单机上的，随着业务的发展，对可用性和性能的要求也越来越高，数据库系统也进而演进为一种分布式的架构。这种架构通常表现为由多个单机数据库节点通过某种复制协议组成一个整体，称之为「Shared-nothing」，典型的如 MySQL，PG，**MongoDB**。

另外一种值得一提是，伴随着「云」的普及，为了发挥云环境下资源池化的优势而出现的「云原生」的架构，典型的如 Aurora，PolarDB，因这种架构通常采用存储计算分离和存储资源共享，所以称之为「Shared-storage」。

不管是哪种架构，在分布式环境下，根据大家耳熟能详的 [CAP](https://en.wikipedia.org/wiki/CAP_theorem) 理论，都要解决所谓的一致性（Consistency）问题，**即在读写发生在不同节点的情况下，怎么保证每次读取都能获取到最新写入的数据**。**这个一致性即是我们今天要讨论的MongoDB 可调一致性模型中的一致性**，区别于单机数据库系统中经常提到的 ACID 理论中的一致性。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041631428.png)](https://mongoing.com/wp-content/uploads/2021/03/527c6edfbe919bd.png)

CAP 理论中的一致性直观来看是强调读取数据的**新近度**（**Recency**），但个人认为也隐含了对**持久性**（**Durability**）的要求，即，当前如果已经读取了最新的数据，不能因为节点故障或网络分区，导致已经读到的更新丢失。关于这一点，我们后面讨论具体设计的时候也能看到 **MongoDB 的一致性模型对持久性的关注**。

既然标题提到了是可调（Tunable）一致性，那这个**可调性具体又指的是什么呢**？

这里就不得不提分布式系统中的另外一个理论，[PACELC](https://en.wikipedia.org/wiki/PACELC_theorem)。PACELC 在 CAP 提出 10 年之后，即 2012 年，在一篇 [Paper](http://www.cs.umd.edu/~abadi/papers/abadi-pacelc.pdf) 中被正式提出，其**核心观点是**，根据 CAP，在一个存在网络分区（`P`）的分布式系统中，我们面临在可用性（`A`）和一致性（`C`）之间的选择，但除此之外（`E`），即使暂时没有网络分区的存在，在实际系统中，我们也要面临在访问延迟（`L`）和一致性（`C`）之间的抉择。所以，PACELC 理论是结合现实情况，对 CAP 理论的一种扩展。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041631816.png)](https://mongoing.com/wp-content/uploads/2021/03/bebc8af89d29a7c.png)

而我们今天要讨论的 MongoDB 一致性模型的**可调之处**，指的就是调节 MongoDB 读写操作对 L 和 C 的选择，或者更具体的来说，是**调节对性能（Performance——Latency、Throughput）和正确性（Correctness——Recency、Durability）的选择（Tradeoff）**。

## MongoDB 一致性模型设计

在讨论具体的实现之前，我们先来尝试从功能设计的角度，理解 MongoDB 的可调一致性模型，这样的好处是可以对其有一个比较全局的认知，后续也可以帮助我们更好的理解它的实现机制。

在学术中，对一致性模型有一些标准的划分和定义，比如我们听到过的线性一致性（Linearizable Consistency），因果一致性（Causal Consistency）等都在这个标准当中，MongoDB 的一致性模型设计自然也不能脱离这个标准。

但是，和很多其他的数据库系统一样，设计上需要综合考虑和其他子系统的关联，比如**复制**、**存储引擎**，具体的实现往往和标准又不是完全一致的。下面的第一个小节，我们就详细探讨标准的一致性模型和 MongoDB 一致性模型的关系，以对其有一个基本的认识。

在这个基础上，我们再来看在具体的功能设计上，MongoDB 的一致性模型是怎么做的，以及在实际的业务场景中是如何被使用的。

### 标准一致性模型和 MongoDB 一致性模型的关系

以复制为基础构建的分布式系统中，一致性模型通常可按照「**以数据为中心（Data-centric）**」和「**以客户端为中心（Client-centric）**」来[划分](https://en.wikipedia.org/wiki/Consistency_model#Consistency_and_replication)，下图中的「Linearizable」，「Sequential」，「Causal」，「Eventual」即属于 Data-centric 的范畴，对一致性的保证也是由强到弱。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041634558.png)](https://mongoing.com/wp-content/uploads/2021/03/8a86253ca52e42a.png)

Data-centric 的一致性模型要求我们站在整个系统的角度看，所有访问进程（客户端）的读写顺序满足同一个特定的约束，比如，对于线性一致性（Linearizable）来说，它要求这个读写顺序和操作真实发生的时间（[Real Time](https://en.wikipedia.org/wiki/Consistency_model#Sequential_consistency_2)）完全一致，是最强的一致性模型，实际系统中很难做到，而对于因果一致性来说，只约束了存在因果关系的操作之间的顺序。

Data-centric 一致性模型虽然对访问进程提供了全局一致的视图，但是在真实的系统中，不同的读写进程（客户端）访问的往往是不同的数据，**维护这样的全局视图会产生不必要的代价**。举个例子，在因果一致性模型下，P1 执行了 `Write1(X=1)`，P2 执行了 `Read1(X=1),Write2(X=3)`，那么 P1 和 P2 之间就产生了因果关系，进而导致`P1:Write1(X=1)` 和 `P2:Write2(X=3)` 的可见顺序存在一个约束，即，需要其他访问进程看到的这两个写操作顺序是一样的，且 Write1 在前，但如果其他进程读的不是 X，显然再提供这种全局一致视图就没有必要了。

由此，为了简化这种全局的一致性约束，就有了 Client-centric 一致性模型，相比于 Data-centric 一致性模型，它**只要求提供单客户端维度的一致性视图**，对单客户端的读写操作提供这几个一致性承诺：「RYW（Read Your Write）」，「MR（Monotonic Read）」，「MW（Monotonic Write）」，「WFR（Write Follow Read）」。关于这些一致性模型的概念和划分，本文不做太详细介绍，感兴趣的可以看 CMU 的这两篇 Lecture（[Lec1](https://web2.qatar.cmu.edu/~msakr/15440-f11/lectures/Lecture11_15440_VKO_10Oct_2011.pptx)，[Lec2](https://web2.qatar.cmu.edu/~msakr/15440-f11/lectures/Lecture11_15440_VKO_10Oct_2011.pptx)），讲的很清晰。

MongoDB 的 [Causal Consistency Session](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#causal-consistency) 即提供了上述几个承诺：RYW，MR，MW，WFR。但是，**这里是 MongoDB 和标准不太一样的地方**，MongoDB 的因果一致性提供的是 Client-centric 一致性模型下的承诺，而非 Data-centric。这么做主要还是从系统开销角度考虑，实现 Data-centric 下的因果一致性所需要的全局一致性视图代价过高，在真实的场景中，Client-centric 一致性模型往往足够了，关于这一点的详细论述可参考 MongoDB 官方在 SIGMOD’19 上 [Paper](https://dl.acm.org/doi/pdf/10.1145/3299869.3314049) 的 2.3 节。

Causal Consistency 在 MongoDB 中是相对比较独立一块实现，**只有当客户端读写的时候开启 Causal Consistency Session 才提供相应承诺**。

**没有开启 Causal Consistency Session 时，MongoDB 通过 [writeConcern](https://docs.mongodb.com/manual/reference/write-concern/) 和 [readConcern](https://docs.mongodb.com/manual/reference/read-concern/) 接口提供了可调一致性，具体来说，包括线性一致性和最终一致性**。最终一致性在标准中的**定义**是非常宽松的，是最弱的一致性模型，但是在这个一致性级别下 MongoDB 也通过 writeConcern 和 readConcern 接口的配合使用，**提供了丰富的对性能和正确性的选择**，从而贴近真实的业务场景。

### MongoDB 可调一致性模型功能接口 —— writeConcern 和 readConcern

在 MongoDB 中，[writeConcern](https://docs.mongodb.com/manual/reference/write-concern/) 是针对写操作的配置，[readConcern](https://docs.mongodb.com/manual/reference/read-concern/) 是针对读操作的配置，而且都支持在**单操作粒度（Operation Level）** 上调整这些配置，使用起来非常的灵活。writeConcern 和 readConcern 互相配合，共同构成了 MongoDB 可调一致性模型的对外功能接口。

#### writeConcern —— 唯一关心的就是写入数据的持久性（Durability）

我们首先来看针对写操作的 writeConcern，写操作改变了数据库的状态，才有了读操作的一致性问题。同时，我们在后面章节也会看到，MongoDB 一些 readConcern 级别的实现也强依赖 writeConcern 的实现。

MongoDB writeConcern 包含如下选项，

```
{ w: <value>, j: <boolean>, wtimeout: <number> }
```

- `w`，指定了这次的写操作需要复制并应用到多少个副本集成员才能返回成功，可以为数字或 “majority”（为了避免引入过多的复杂性，这里忽略[基于 tag 的自定义 writeConcern](https://docs.mongodb.com/manual/tutorial/configure-replica-set-tag-sets/#configure-custom-write-concern)）。`w:0` 时比较特殊，即客户端不需要收到任何有关写操作是否执行成功的确认，具有最高性能。`w: majority` 需要收到**多数派**节点（含 Primary）关于操作执行成功的确认，具体个数由 MongoDB 根据副本集配置自动得出。
- `j`，额外要求节点回复确认时，写操作对应的修改已经被持久化到存储引擎日志中。
- `wtimeout`，Primary 节点在等待足够数量的确认时的超时时间，超时返回错误，但并不代表写操作已经执行失败。

从上面的定义我们可以看出，**writeConcern 唯一关心的就是写操作的持久性，这个持久性不仅仅包含由 `j` 决定、传统的单机数据库层面的持久性，更重要的是包含了由 `w` 决定、整个副本集（Cluster）层面的持久性**。`w` 决定了当副本集发生重新选主时，已经返回写成功的修改是否会“丢失”，在 MongoDB 中，我们称之为**被回滚**。`w` 值越大，对客户端来说，数据的持久性保证越强，写操作的延迟越大。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041635319.png)](https://mongoing.com/wp-content/uploads/2021/03/52a49fe15308ab0.png)

这里还要提及两个概念，「**local committed**」 和 「**majority committed**」，对应到 writeConcern 分别为 `w:1` 和 `w: majority`，它们在后续实现分析中会多次涉及。每个 MongoDB 的写操作会开启底层 WiredTiger 引擎上的一个事务，如下图，`w:1` 要求事务只要在本地成功提交（local committed）即可，而 `w: majority` 要求事务在副本集的多数派节点提交成功（majority committed）。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041636894.png)](https://mongoing.com/wp-content/uploads/2021/03/5d2bc537d944da4.png)

#### readConcern —— 关心读取数据的新近度（Recency）和持久性（Durability）

在 MongoDB 4.2 中包含 5 种 readConcern 级别，我们先来看前 4 种：「local」, 「available」, 「majority」, 「linearizable」，它们对一致性的承诺依次由弱到强。**其中，「linearizable」即对应我们前面提到的标准一致性模型中的线性一致性，另外 3 种 readConcern 级别代表了 MongoDB 在最终一致性模型下，对 Latency 和 Consistency(Recency & Durability) 的取舍**。

下面我们结合一个三节点副本集复制架构图，来简要说明这几个 readConcern 级别的含义。在这个图中，oplog 代表了MongoDB 的复制日志，类似于 MySQL 中的 binlog，复制日志上最新的`x=<value>`，表示了节点的复制进度。

[![img](https://mongoing.com/wp-content/uploads/2021/03/f1dd6b20a5b1a01.png)](https://mongoing.com/wp-content/uploads/2021/03/f1dd6b20a5b1a01.png)

- **local/available**：local 和 available 的语义基本一致，都是读操作直接读取本地最新的数据。但是，available 使用在 MongoDB 分片集群场景下，含[特殊语义](https://docs.mongodb.com/manual/reference/read-concern-available/#readconcern."available")（为了保证性能，可以返回孤儿文档），这个特殊语义和本文的主题关联不大，所以后面我们**只讨论 local readConcern**。在这个级别下，发生重新选主时，**已经读到的数据可能会被回滚掉**。
- **majority**：读取「majority committed」的数据，可以保证读取的数据不会被回滚，但是**并不能保证读到本地最新的数据**。比如，对于上图中的 Primary 节点读，虽然 `x=5` 已经是最新的已提交值，但是由于不是「majority committed」，所以当读操作使用 majority readConcern 时，只返回`x=4`。
- **linearizable**：承诺线性一致性，即，既保证能读取到最新的数据（Recency Guarantee），也保证读到数据不会被回滚（Durability Guarantee）。前面我们说了，线性一致性在真实系统中很难实现，MongoDB 在这里采用了一个相当简化的设计，当读操作指定 linearizable readConcern level 时，读操作只能读取 Primary 节点，而考虑到写操作也只能发生在 Primary，**相当于 MongoDB 的线性一致性承诺被限定在了单机环境下，而非分布式环境**，实现上自然就简单很多。考虑到会有重新选主的情况，**MongoDB 在这个 readConcern level 下唯一需要解决的问题就是，确保每次读发生在真正的 Primary 节点上**。后面分析具体实现我们可以看到，解决这个问题是以增加读延迟为代价的。

以上各 readConcern level 在 Latency、Durability、Recency 上的 Tradeoff 如下，

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041636641.png)](https://mongoing.com/wp-content/uploads/2021/03/a0c6420cc62ad8c.png)

我们还有最后一种 readConcern level 没有提及，即「snapshot readConcern」，放在这里单独讨论的原因是，「snapshot readConcern」是伴随着 4.0 中新出现的[多文档事务](https://www.mongodb.com/blog/post/mongodb-multi-document-acid-transactions-general-availability)（ multi-document transaction，其他系统也常称之为多行事务）而设计的，只能用在显式开启的多文档事务中。而在 4.0 之前的版本中，对于一条读写操作，MongoDB 默认只支持单文档上的事务性语义（单行事务），前面提到的 4 种 readConcern level 正是为这些普通的读写操作（未显式开启多文档事务）而设计的。

「snapshot readConcern」从定义上来看，和 majority readConcern 比较相似，即，读取「majority committed」的数据，也可能读不到最新的已提交数据，但是**其特殊性在于，当用在多文档事务中时，它承诺真正的一致性快照语义**，而其他的 readConcern level 并不提供，关于这一点，我们在后面的实现部分再详细探讨。

#### writeConcern 和 readConcern 的关系

在分布式系统中，当我们讨论一致性的时候，通常指的是读操作对数据的关注，即「what read concerns」，那为什么在 MongoDB 中我们还要单独讨论 writeConcern 呢？从一致性承诺的角度来看，writeConcern 从如下两方面会对 readConcern 产生影响，

- 「linearizable readConcern」读取的数据需要是以「majority writeConcern」写入且持久化到日志中，才能提供[真正的](https://docs.mongodb.com/manual/reference/read-concern/#real-time-order)「线性一致性」语义。考虑如下情况，数据写入到 majority 节点后，没有在日志中持久化，当 majority 节点发生重启恢复，那么之前使用 「linearizable readConcern」读取到的数据就可能丢失，显然和「线性一致性」的语义不相符。在 MongoDB 中，[writeConcernMajorityJournalDefault](https://docs.mongodb.com/manual/reference/replica-configuration/#rsconf.writeConcernMajorityJournalDefault) 参数控制了，当写操作指定 「majority writeConcern」的时候，是否保证写操作在日志中持久化，该参数默认为 true。另外一种情况是，写操作持久化到了日志中，但是没有复制到 majority 节点，在重新选主后，同样可能会发生数据丢失，违背一致性承诺。
- 「majority readConcern」要求读取 majority committed 的数据，所以受限于不同节点的复制进度，可能会读取到更旧的值。但是如果数据是以更高的 writeConcern `w` 值写入的，即写操作在扩散到更多的副本集节点上之后才返回写成功，显然之后再去读取，「majority readConcern」能有更大的概率读到最新写入的值（More Recency Guarantee）。

所以，writeConcern 虽然只关注了写入数据的持久化程度，但是作为读操作的数据来源，也间接的也影响了 MongoDB 对读操作的一致性承诺。

#### writeConcern 和 readConcern 在实际业务中的应用

前面是对 writeConcern 和 readConcern 在功能定义上的介绍，可以看到，读写采用不同的配置，每个配置下面又包含不同的级别，这个接口设计对于使用者来说还是稍显复杂的（社区中也有不少类似的反馈），下面我们就来了解一下 writeConcern 和 readConcern 在真实业务中的统计数据以及几个典型应用场景，以加深对它们的理解。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041636506.png)](https://mongoing.com/wp-content/uploads/2021/03/3c69eae7a8c6f92.png)

上面的统计数据来自于 MongoDB 自己的 Atlas 云服务中用户 Driver 上报的数据，统计样本在百亿量级，所以准确性是可以保证的，从数据中我们可以分析出如下结论，

- 大部分的用户实际上只是单纯的使用默认值
- 在读取数据时，99% 以上的用户都只关心能否尽可能快的读取数据，即使用 local readConcern
- 在写入数据时，虽然大部分用户也只要求写操作在本地写成功即可，但仍然有不小的比例使用了 majority writeConcern（16%，远高于使用 majority readConcern 的比例），因为写操作被回滚对用户来说通常都是更影响体验的。

此外，MongoDB 的默认配置（{w:1} writeConcern, local readConcern）都是更倾向于保护 Latency 的，主要是基于这样的一个事实：**主备切换事件发生的概率比较低，即使发生了丢数据的概率也不大**。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041636422.png)](https://mongoing.com/wp-content/uploads/2021/03/c065b54a2824f30.png)

统计数据给了我们一个 MongoDB readConcern/writeConcern 在真实业务场景下使用情况的直观认识，即，**大部分用户更关注 Latency，而不是 Consistency**。但是，统计数据同时也说明 readConcern/writeConcern 的使用组合是非常丰富的，用户通过使用不同的配置值来满足需求各异的业务场景对一致性和性能的要求，比如如下几个实际业务场景中的应用案例（均来自于 Atlas 云服务中的用户使用场景），

- **Majority reads and writes**：在这个组合下，意味着对数据安全性的关注是第一优先级的。考虑一个助学贷款的网站，网站的访问流量并不高，大约每分钟两次写入（提交申请），对于一个申请贷款的学生来说，显然不能接受自己成功提交的申请在后台 MongoDB 数据库发生重新选主时数据“丢失”，同样也不能接受获取到申请通过结果的情况下，再次查询，可能因为读取的数据被回滚，结果发生变化的情形，所以业务选择使用 majority readConcern & writeConcern 的组合，通过牺牲读写延迟来换取数据的安全性。
- **Local reads and Majority writes**：考虑一个餐饮评价的 App，比如大众点评，用户可能要花很大的精力来编辑一条精彩的评价，如果因为后端 MongoDB 实例发生主备切换导致评论丢失，对用户来说显然是不可接受的，所以用户评价的提交（写）需要使用 majority writeConcern，但是读到一条可能后续会因为回滚而“消失”的评价，对用户来说往往是可以接受，考虑到要兼顾性能，使用 local readConcern 显然是一个更优的选择。
- **Multiple Write Concern Values**：在同一个业务场景中，也不用只局限于一种 writeConcern/readConcern value，可以在不同的条件下使用不同的值来兼顾性能和一致性。比如，考虑一个文档系统，通常这样的系统在用户编辑文档时，会提供自动保存功能，对于非用户主动触发的发布或保存，自动保存的结果如果产生丢失，用户往往是感知不到的，而自动保存功能相对又是会比较频繁的触发（写压力更大），所以这种写动作使用 local writeConcern 显然更合理，写延迟更低，而低频的主动保存或发布，应该使用 majority writeConcern，因为这种情况用户对要保存的数据有明确的感知，很难接受数据的丢失。

### MongoDB 因果一致性模型功能接口 —— Causal Consistency Session

前面已经提及了，相比于 writeConcern/readConcern 构建的可调一致性模型，MongoDB 的因果一致性模型是另外一块相对比较独立的实现，有自己专门的功能接口。MongoDB 的因果一致性是借助于客户端的 [causally consistent session](https://docs.mongodb.com/manual/core/read-isolation-consistency-recency/#sessions) 来实现的，causally consistent session 可以理解为，**维护一系列存在因果关系的读写操作间的因果一致性的执行载体**。

causally consistent session 通过维护 Server 端返回的一些操作执行的元信息（主要是关于操作定序的信息），再结合 Server 端的实现来提供 MongoDB Causal Consistency 所定义的一致性承诺（RYW，MR，MW，WFR），具体原理我们在后面的实现部分再详述。

针对 causally consistent session，我们可以看一个简单的例子，比如现在有一个订单集合 orders，用于存储用户的订单信息，为了扩展读流量，客户端采用主库写入从库读取的方式，用户希望自己在提交订单之后总是能够读取到最新的订单信息（Read Your Write），为了满足这个条件，客户端就可以通过 causally consistent session 来实现这个目的，

```
""" new order """
with client.start_session(causal_consistency=True) as s1:
    orders = client.get_database(
        'test', read_concern=ReadConcern('majority'),
        write_concern=WriteConcern('majority', wtimeout=1000)).orders
    orders.insert_one(
        {'order_id': "123", 'user': "tony", 'order_info': {}}, session=s1)

""" another session get user orders """
with client.start_session(causal_consistency=True) as s2:
    s2.advance_cluster_time(s1.cluster_time) # hybird logical clock
    s2.advance_operation_time(s1.operation_time)
    orders = client.get_database(
        'test', read_preference=ReadPreference.SECONDARY,
        read_concern=ReadConcern('majority'),
        write_concern=WriteConcern('majority', wtimeout=1000)).orders
    for order in orders.find({'user': "tony"}, session=s2):
        print(order)
```

从上面的例子我们可以看到，使用 causally consistent session，仍然需要指定合适的 readConcern/writeConcern value，原因是，**只有指定 majority writeConcern & readConcern，MongoDB 才能提供完整的 Causal Consistency 语义**，即同时满足前面定义的 4 个承诺（RYW，MR，MW，WFR）。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041637282.png)](https://mongoing.com/wp-content/uploads/2021/03/e92ea293c029da1.png)

简单起见，我们只举例其中的一种情况：为什么在 {w: 1} writeConcern 和 majority readConcern 下，不能满足 RYW（Read Your Write）？

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041637993.png)](https://mongoing.com/wp-content/uploads/2021/03/fd29228ac33ac4e.png)

上图是一个 5 节点的副本集，当发生网络分区时（P~old~, S~1~ 和 P~new~, S~2~, S~3~ 分区）,在 P~old~ 上发生的 W~1~ 写入因为使用了 {w:1} writeConcern ，会向客户端返回成功，但是因为没有复制到多数派节点，最终会在网络恢复后被回滚掉，R~1~ 虽然发生在 W~1~ 之后，但是从 S~2~ 并不能读取到 W~1~ 的结果，不符合 RYW 语义。其他情况下为什么不能满足 Causal Consistency 语义，可以参考[官方文档](https://docs.mongodb.com/manual/core/causal-consistency-read-write-concerns/)，有非常详细的说明。

## MongoDB 一致性模型实现机制及优化

前面对 MongoDB 的可调一致性和因果一致性模型，在理论以及具体的功能设计层面做了一个总体的阐述，下面我们就深入到内核层面，来看下 MongoDB 的一致性模型的具体实现机制以及在其中做了哪些优化。

### writeConcern

在 MongoDB 中，writeConcern 的实现相对比较简单，因为**不同的 writeConcern value 实际上只是决定了写操作返回的快慢**。`w <= 1` 时，写操作的执行及返回的流程只发生在本地，并不会涉及等待副本集其他成员确认的情况，比较简单，所以我们只探讨 `w > 1` 时 writeConcern 的实现。

#### w>1 时 writeConcern 的实现

每一个用户的写操作会开启 WiredTiger 引擎层的一个事务，这个事务在提交时会顺便记录本次写操作对应的 Oplog Entry 的时间戳（Oplog 可理解为 MongoDB 的复制日志，这里不做详细介绍，可参考[文档](https://docs.mongodb.com/manual/core/replica-set-oplog/)），这个时间戳在代码里面称之为`lastOpTime`。

```c++
// mongo::RecoveryUnit::OnCommitChange::commit -> mongo::repl::ReplClientInfo::setLastOp
void ReplClientInfo::setLastOp(OperationContext* opCtx, const OpTime& ot) {
    invariant(ot >= _lastOp);
    _lastOp = ot;
    lastOpInfo(opCtx).lastOpSetExplicitly = true;
}
```

引擎层事务提交后，相当于本地已经完成了本次写操作，对于 `w:1` 的 writeConcern，已经可以直接向客户端返回成功，但是当 `w > 1` 时就需要等待足够多的 Secondary 节点也确认写操作执行成功，这个时候 MongoDB 会通过执行 `ReplicationCoordinatorImpl::_awaitReplication_inlock` 阻塞在一个条件变量上，等待被唤醒，被阻塞的用户线程会被加入到 `_replicationWaiterList` 中。

Secondary 在拉取到 Primary 上的这个写操作对应的 Oplog 并且 Apply 完成后，会更新自身的位点信息，并通知另外一个后台线程汇报自己的 `appliedOpTime` 和 `durableOpTime` 等信息给 upstream（主要的方式，还有其他一些特殊的汇报时机）。

```c++
void ReplicationCoordinatorImpl::setMyLastAppliedOpTimeAndWallTimeForward(
    ...
    if (opTime > myLastAppliedOpTime) {
        _setMyLastAppliedOpTimeAndWallTime(lock, opTimeAndWallTime, false, consistency);
        _reportUpstream_inlock(std::move(lock)); // 这里是向 sync source 汇报自己的 oplog apply 进度信息
    }
    ...
}
```

`appliedOpTime` 和 `durableOpTime` 的含义和区别如下，

- `appliedOpTime`：Secondary 上 Apply 完一批 Oplog 后，最新的 Oplog Entry 的时间戳。
- `durableOpTime`：Secondary 上 Apply 完成并在 Disk 上持久化的 Oplog Entry 最新的时间戳， Oplog 也是作为 WiredTiger 引擎的一个 Table 来实现的，但 WT 引擎的 WAL sync 策略默认是 100ms 一次，所以**这个时间戳通常滞后于appliedOpTime**。

上述信息的汇报是通过给 upstream 发送 `replSetUpdatePosition` 命令来完成的，upstream 在收到该命令后，通过比较如果发现某个副本集成员汇报过来的时间戳信息比上次新，就会触发，唤醒等待 writeConcern 的用户线程的逻辑。

唤醒逻辑会去比较用户线程等待的 `lastOptime` 是否小于等于 Secondary 汇报过来的时间戳 TS，如果是，表示有一个 Secondary 节点满足了本次 writeConcern 的要求。那么，TS 要使用 Secondary 汇报过来的那个时间戳呢？如果 writeConcern 中 `j` 参数指定的是 false，意味着本次写操作并不关注是否在 Disk 上持久化，那么 TS 使用 `appliedOpTime`， 否则使用 `durableOpTime` 。当有指定的 `w` 个节点（含 Primary 自身）汇报的 TS 大于等于 `lastOptime`，用户线程即可被唤醒，向客户端返回成功。

```c++
// TopologyCoordinator::haveNumNodesReachedOpTime
    for (auto&& memberData : _memberData) {
        const OpTime& memberOpTime =
            durablyWritten ? memberData.getLastDurableOpTime() : memberData.getLastAppliedOpTime();
        if (memberOpTime >= targetOpTime) {
            --numNodes;
        }
        if (numNodes <= 0) {
            return true;
        }
    }
```

到这里，用户线程因 writeConcern 被阻塞到唤醒的基本流程就完成了，但是我们还需要思考一个问题，MongoDB 是支持链式复制的，即， **P->S1->S2** 这种复制拓扑，如果在 P 上执行了写操作，且使用了 writeConcern w:3，即，要求得到三个节点的确认，而 **S2 并不直接向 P 汇报自己的 Oplog Apply 信息，那这种场景下 writeConcern 要如何满足**？

MongoDB 采用了信息转发的方式来解决这个问题，当 S1 收到 S2 汇报过来的 `replSetUpdatePosition` 命令，进行处理时（`processReplSetUpdatePosition()`），如果发现自己不是 Primary 角色，会立刻触发一个 `forwardSlaveProgress` 任务，即，把自己的 Oplog Apply 信息，连同自己的 Secondary 汇报过来的，构造一个 `replSetUpdatePosition` 命令，发往上游，从而保证，当任一个 Secondary 节点的 Oplog Apply 进度推进，Primary 都能够及时的收到消息，尽可能降低 w>1 时，因 writeConcern 而带来的写操作延迟。

### readConcern

readConcern 的实现相比于 writeConcern，要复杂很多，因为它和存储引擎的关联要更为紧密，在某些情况下，还要依赖于 writeConcern 的实现，同时部分 readConcern level 的实现还要依赖 MongoDB 的复制机制和存储引擎共同提供支持。

另外，MongoDB 为了在满足指定 readConcern level 要求的前提下，尽量降低读操作的延迟和事务执行效率，也做了一些优化。下面我们就结合不同的 readConcern level 来分别描述它们的实现原理和优化手段。

#### “majority” readConcern

“majority” readConcern 的语义前面的章节已经介绍，这里不再赘述。为了保证客户端读到 **majority committed** 数据，根据存储引擎能力的不同，MongoDB 分别实现了两种机制用于提供该承诺。

##### 依赖 WiredTiger 存储引擎快照的实现方式

WiredTiger 为了保证并发事务在执行时，不同事务的读写不会互相 block，提升事务执行性能，也采用了 [MVCC](https://en.wikipedia.org/wiki/Multiversion_concurrency_control) 的并发控制策略，即不同的写事务在提交时，会生成多个版本的数据，每个版本的数据由一个时间戳（commit_ts）来标识。所谓的存储引擎**快照（Snapshot）**，实际上就是**在某个时间点看到的，由历史版本数据所组成的一致性数据视图**。所以，在引擎内部，**快照也是由一个时间戳来标识的**。

前面我们已经提到，由于 MongoDB 采用异步复制的机制，不同节点的复制进度会有差异。如果我们在某个副本集节点直接读取最新的已提交数据，如果它还没有复制到大多数节点，显然就不满足 “majority” readConcern 语义。

这个时候可以采取一个办法，就是仍然读取最新的数据，但是在返回 Client 前等待其他节点确认本次读取的数据已经 apply 完成了，但是这样显然会大幅的增加读操作的延迟（虽然这种情况下，一致性体验反而更好了，因为能读到更新的数据，但是前面我们已经分析了，绝大部分用户在读取时，希望更快的返回的数据，而不是追求一致性）。

所以，MongoDB 采用的做法是在存储引擎层面维护一个 majority committed 数据视图（快照），**这个快照对应的时间戳在 MongoDB 里面称之为 majority committed point**（后面简称 **mcp**）。当 Client 指定 majority 读时，通过直接读取这个快照，来快速的返回数据，无需等待。需要注意的一点是，由于复制进度的差异，mcp 并不能反映当前最新的已提交数据，即，这个方法是通过牺牲 Recency 来换取更低的 Latency。

```
// 以 getMore 命令举例
void applyCursorReadConcern(OperationContext* opCtx, repl::ReadConcernArgs rcArgs) {
        ... 
        switch (rcArgs.getMajorityReadMechanism()) {
            case repl::ReadConcernArgs::MajorityReadMechanism::kMajoritySnapshot: {
                // Make sure we read from the majority snapshot.
                opCtx->recoveryUnit()->setTimestampReadSource(
                    RecoveryUnit::ReadSource::kMajorityCommitted);
                // 获取 majority committed snapshot
                uassertStatusOK(opCtx->recoveryUnit()->obtainMajorityCommittedSnapshot());
                break;
        ...
}
```

但基于 mcp 快照的实现方式需要解决一个问题，即，**如何保证这个快照的有效性？** 进一步来说， 如何保证 mcp 视图所依赖的历史版本数据不会被 WiredTiger 引擎清理掉？

正常情况下，WiredTiger 会根据事务的提交情况自动的去清理多版本的数据，只要当前的活跃事务对某个历史版本的数据没有依赖，即可以从内存中的 MVCC List 里面删掉（不考虑 [LAS 机制](https://mongoing.com/archives/74507)，WT 的多版本数据设计上只存放在内存中）。但是，所谓的 majority committed point，实际上是 Server 层的概念，引擎层并不感知，如果只根据事务的依赖来清理历史版本数据，mcp 依赖的历史版本版本数据可能就会被提前清理掉。

举个例子，在下图的三节点副本集中，如果 Client 从 Primary 节点读取并且指定了 majority readConcern，由于 `mcp = 4`，那么 MongoDB 只能向 Client 返回 `commit_ts = 4` 的历史值。但是，对于 WiredTiger 引擎来说，当前活跃的事务列表中只有 T1，commit_ts = 4 的历史版本是可以被清理的，但清理掉该版本，mcp 所依赖的 snapshot 显然就无法保证。所以，**需要 WiredTiger 引擎层提供一个新机制，根据 Server 层告知的复制进度，即， mcp 位点，来清理历史版本数据**。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041638071.png)](https://mongoing.com/wp-content/uploads/2021/03/09f8c4e58d29e1b.png)

在 WiredTiger 3.0 版本中，开始提供「[Application-specified Transaction Timestamps](https://source.wiredtiger.com/develop/transactions.html#transaction_timestamps)」功能，来解决 Server 层对事务提交顺序（基于 Application Timestamp）的需求和 WiredTiger 引擎层内部的事务提交顺序（基于 Internal Transaction ID）不一致的问题（根源来自于基于 Oplog 的复制机制，这里不作展开）。进一步，在这个功能的基础上，WT 也提供了所谓的「read “as of” a timestamp」功能（也有文章称之为 「Time Travel Query」），即支持从某个指定的 Timestamp 进行快照读，而这个特性正是前面提到的基于 mcp 位点实现 “majority” readConcern 的功能基础。

WiredTiger 对外提供了 [set_timestamp()](https://source.wiredtiger.com/develop/struct_w_t___c_o_n_n_e_c_t_i_o_n.html#ad082439541b1b95d6aae6c15026fe512) 的 API，用于 Server 层来更新相关的 Application Timestamp。WT 目前包含如下语义的 Application Timestamp，

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041638763.png)](https://mongoing.com/wp-content/uploads/2021/03/2c5890ffd3cd3cc.png)

要回答前面提到的关于 mcp snapshot 有效性保证的问题，我们需要重点关注红框中的几个 Timestamp。

首先，`stable` timestamp 在 MongoDB 中含义是，在这个时间戳之前提交的写，不会被回滚，所以**它和 majority commit point（mcp） 的语义是一致的**。`stable` timestamp 对应的快照被存储引擎持久化后，称之为「**stable checkpoint**」，这个 checkpoint 在 MongoDB 中也有重要的意义，在后面的「”local” readConcern」章节我们再详述。

MongoDB 在 Crash Recovery 时，总是从 stable checkpoint 初始化，然后重新应用增量的 Oplog 来完成一次恢复。所以为了提升 Crash Recovery 效率及回收日志空间，引擎层需要定期的产生新的 stable checkpoint，也就意味着`stable` timestamp 也需要不断的被 Server 层推进（更新）。而 MongoDB 在更新 `stable` timestamp 的同时，也会顺便去基于该时间戳去更新 `oldest` timestamp，所以，**在基于快照的实现机制下，oldest timestamp 和 stable timestamp 的语义也是一致的**。

```c++
...
->ReplicationCoordinatorImpl::_updateLastCommittedOpTimeAndWallTime()
->ReplicationCoordinatorImpl::_setStableTimestampForStorage()
->WiredTigerKVEngine::setStableTimestamp()
->WiredTigerKVEngine::setOldestTimestampFromStable()
->WiredTigerKVEngine::setOldestTimestamp()
```

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041638031.png)](https://mongoing.com/wp-content/uploads/2021/03/1dd5df174103422.png)

当前 WiredTiger 收到新的 `oldest` timestamp 时，会结合当前的活跃事务（`oldest_reader`）和 `oldest` timestamp 来计算**新的**全局 `pinned` timestamp，当进行历史版本数据的清理时，**pinned timestamp 之后的版本不会被清理**，从而保证了 mcp snapshot 的有效性。

```c++
// 计算新的全局 pinned timestamp
__conn_set_timestamp->__wt_txn_global_set_timestamp->__wt_txn_update_pinned_timestamp->
__wt_txn_get_pinned_timestamp {
...
    tmp_ts = include_oldest ? txn_global->oldest_timestamp : 0;
...
    if (!include_oldest && tmp_ts == 0)
        return (WT_NOTFOUND);
    *tsp = tmp_ts;
...
}

// 判断历史版本是否可清理
static inline bool
__wt_txn_visible_all(WT_SESSION_IMPL *session, uint64_t id, wt_timestamp_t timestamp)
{
...
    __wt_txn_pinned_timestamp(session, &pinned_ts);
    return (timestamp <= pinned_ts);
}
```

在分析了 mcp snapshot 有效性保证的机制之后，我们还需要回答下面两个关键问题，整个细节才算完整。

1. Secondary 的复制进度，以及进一步由复制进度计算出的 mcp 是由 oplog 中的 ts 字段来标识的，而数据的版本号是由 commit_ts 来标识的，他们之间有什么关系，为什么是可比的？
2. 前面提到了引擎的 Crash Recovery 需要 stable timestamp（mcp）不断的推进来产生新的 stable checkpoint，那 mcp 具体是如何推进的？

**要回答第一个问题**，我们需要先看下，对于一条 insert 操作，它所对应的 oplog entry 的 ts 字段值是怎么来的，以及这条 oplog 和 insert 操作的关系。

首先，当 Server 层收到一条 insert 操作后，会提前调用 `LocalOplogInfo::getNextOpTimes()` 来给其即将要写的 oplog entry 生成 ts 值，获取这个 ts 是需要加锁的，避免并发的写操作产生同样的 ts。然后， Server 层会调用 `WiredTigerRecoveryUnit::setTimestamp` 开启 WiredTiger 引擎层的事务，**并且把这个事务中后续写操作的 `commit_ts` 都设置为 oplog entry 的 ts**，insert 操作在引擎层执行完成后，会把其对应的 oplog entry 也通过同一事务写到 WiredTiger Table 中，之后事务才提交。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639481.png)](https://mongoing.com/wp-content/uploads/2021/03/20c6b40491783c4.png)

也就是说 **MongoDB 是通过把写 oplog 和写操作放到同一个事务中，来保证复制日志和实际数据之间的一致性**，同时也确保了，**oplog entry ts 和写操作本身所产生修改的版本号是一致的**。

**对于第二个问题**，mcp 如何推进，在前面的 writeConcern 实现章节我们提到了，downstream 在 apply 完一批 oplog 之后会向 upstream 汇报自己的 apply 进度信息，upstream 同时也会向自己的 upstream 转发这个信息，基于这个机制，对 Primary 来说，显然最终它能不断的获取到整个副本集所有成员的 oplog apply 进度信息，进而推进自己的 majority commit point（计算的方式比较简单，具体见`TopologyCoordinator::updateLastCommittedOpTimeAndWallTime`）。

但是，上述是一个**单向**传播的机制，而副本集的 Secondary 节点也是能够提供读的，同样需要获取其他节点的 oplog apply 信息来更新 mcp 视图，所以 MongoDB 也提供了如下两种机制来保证 Secondary 节点的 mcp 是可以不断推进的：

1. 基于副本集高可用的心跳机制

   ：

   i. 默认情况下，每个副本集节点都会每 2 秒向其他成员发送心跳（

   ```
   replSetHeartBeat
   ```

    命令）

   ii. 其他成员返回的信息中会包含 

   ```
   $replData
   ```

    元信息，Secondary 节点会根据其中的 

   ```
   lastOpCommitted
   ```

    直接推进自己的 mcp

   

   ```
   $replData: { term: 147, lastOpCommitted: { ts: Timestamp(1598455722, 1), t: 147 } ...
   ```

2. **基于副本集的增量同步机制**：
   i. 基于心跳机制的 mcp 推进方式，显然实时性是不够的，Primary 计算出新的 mcp 后，最多要等 2 秒，下游才能更新自己的 mcp
   ii. 所以，MongoDB 在 oplog 增量同步的过程中，upstream 同样会在向 downstream 返回的 oplog batch 中夹带 `$replData` 元信息，下游节点收到这个信息后同样会根据其中的 `lastOpCommitted` 直接推进自己的 mcp
   iii. 由于 Secondary 节点的 oplog fetcher 线程是持续不断的从上游拉取 oplog，只要有新的写入，导致 Primary mcp 推进，那么下游就会立刻拉取新的 oplog，可以保证在 ms 级别同步推进自己的 mcp

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639237.png)](https://mongoing.com/wp-content/uploads/2021/03/b741ac7368bf204.png)

另外一点需要说明的是，心跳回复中实际上也包含了目标节点的 `lastAppliedOpTime` 和 `lastDurableOpTime` 信息，但是 Secondary 节点并不会根据这些信息自行计算新的 mcp，而是总是等待 Primary 把 `lastOpCommittedOpTime` 传播过来，直接 set 自己的 mcp。

##### Speculative Read —— 不依赖快照的实现方式

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639593.png)](https://mongoing.com/wp-content/uploads/2021/03/67257b036d8caea.png)

类似于 MySQL，MongoDB 也是**支持插件式的存储引擎体系**的，但是并非每个支持的存储引擎都实现了 MVCC，即具备快照能力，比如在 MongoDb 3.2 之前默认的 MMAPv1 引擎就不具备。

此外，即使对于具备 MVCC 的 WiredTiger 引擎，维护 majority commit point 对应的 snapshot 是会带来存储引擎 cache 压力上涨的，所以 MongoDB 提供了 [`replication.enableMajorityReadConcern`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern) 参数用于关闭这个机制。

所以，结合以上两方面的原因，MongoDB 需要提供一种不依赖快照的机制来实现 majority readConcern，MongoDB 把这个机制称之为 **Speculative Read** ，中文上我觉得可以称为“未决读”。

Speculative Read 的实现方式非常简单，上一小节实际上也基本描述了，就是直接读当前最新的数据，但是在实际返回 Client 前，会等待读到的数据在多数节点 apply 完成，故可以满足 majority readConcern 语义。本质上，这是一种后验的机制，在其他的数据库系统中，比如 [Hekaton](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.310.6603&rep=rep1&type=pdf)，VoltDB ，事务的并发控制中也有类似的做法。

在具体的实现上，首先在命令实际执行前会通过 `WiredTigerRecoveryUnit::setTimestampReadSource()` 设置自己的读时间戳，即 readTs，读事务在执行的过程中只会读到 readTs 或之前的版本。

在命令执行完成后，会调用 `waitForSpeculativeMajorityReadConcern()` 确保 readTs 对应的时间点及之前的 oplog 在 majority 节点应用完成。这里实际上最终也是通过调用 `ReplicationCoordinatorImpl::_awaitReplication_inlock` 阻塞在一个条件变量上，等待足够多的 Secondary 节点汇报自己的复制进度信息后才被唤醒，**完全复用了 majority writeConcern 的实现**。所以，writeConcern，readConcern 除了在功能设计上有强关联，在内部实现上也有互相依赖。

需要注意的是，`Speculative Read` 机制 MongoDB 并不打算提供给普通用户使用，如果把 `replication.enableMajorityReadConcern` 设置为 false 之后，继续使用 majority readConcern，MongoDB 会返回 `ReadConcernMajorityNotEnabled` 错误。目前在一些内部命令的场景下才会使用该机制，测试目的的话，可以在 `find` 命令中加一个特殊参数： `allowSpeculativeMajorityRead: true`，强制开启 `Speculative Read` 的支持。

#### 针对 readConcern 的优化 —— Query Yielding

考虑到后文逻辑上的依赖，在分析其他 readConcern level 之前，需要先看一个 MongoDB 针对 readConcern 的优化措施。

**默认情况下，MongoDB Server 层面所有的读操作在 WiredTiger 上都会开启一个事务，并且采用 snapshot 隔离级别**。在 [snapshot isolation](https://en.wikipedia.org/wiki/Snapshot_isolation) 下，事务需要读到一个一致性的快照，且读取的数据是事务开始时最新提交的数据。而 WiredTiger 目前的多版本数据只能存放在内存中，所以在这个规则下，执行时间太久的事务会导致 WiredTiger 的内存压力升高，进一步会影响事务的执行性能。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639283.png)](https://mongoing.com/wp-content/uploads/2021/03/606e58d800032cb.png)

比如，在上图中，事务 T1 开始后，根据 majority commit point 读取自己可见的版本，x=1，其他的事务继续对 x 产生修改并且提交，会产生的新的版本 x=2，x=3……，T1 只要不提交，那么 x=2 及之后的版本都不能从内存中清理，否则就会违反 snapshot isolation 的[语义](https://infoscience.epfl.ch/record/53561/files/srds2005-gsi.pdf)。

面对上述情况，MongoDB 采用了一种称之为「**Query Yielding**」的手段来“优化” 这个问题。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639251.png)](https://mongoing.com/wp-content/uploads/2021/03/679149677b9c61c.png)

「Query Yielding」的思路其实非常简单，就是在事务执行的过程中，定期的进行 `yield`，即释放锁，abort 当前的 WiredTiger 事务，释放 hold 的 snapshot，然后重新打开事务，获取新的 snapshot。显然，通过这种方式，对于一个执行时间很长的 MongoDB 读操作，**它在引擎层事务的 read_ts 是不断推进的**，进而保证 read_ts 之后的版本能够被及时从内存中清理。

之所以在优化前面加一个引号的原因是，这种方式虽然解决了长事务场景下，WT 内存压力上涨的问题，但是**是以牺牲快照隔离级别的语义为代价的（降级为 read committed 隔离级别）**，又是一个典型的牺牲一致性来换取更好的访问性能的应用案例。

**“local” 和 “majority” readConcern 都应用了「Query Yielding」机制**，他们的主要区别是，”majority” readConcern 在 reopen 事务时采用新推进的 mcp 对应的 snapshot，而 “local” readConcern 采用最新的时间点对应的 snapshot。

Server 层在一个 Query 正常执行的过程中（`getNext()`），会不断的调用 `_yieldPolicy->shouldYieldOrInterrupt()` 来判定是否需要 yield，目前主要由如下两个因素共同决定是否 yield：

- `internalQueryExecYieldIterations`：`shouldYieldOrInterrupt()` 调用累积次数超过该配置值会主动 yield，默认为 128，本质上反映的是从索引或者表上获取了多少条数据后主动 yield。yield 之后该累积次数清零。
- `internalQueryExecYieldPeriodMS`：从上次 yield 到现在的时间间隔超过该配置值，主动 yield，默认为 10ms，本质上反映的是当前线程获取数据的行为持续了多久需要 yield。

最后，除了根据上述配置主动的 yield 行为，存储引擎层面也会因为一些原因，比如需要从 disk load page，事务冲突等，告知计划执行器（PlanExecutor）需要 yield。MongoDB 的慢查询日志中会输出一些有关执行计划的信息，其中一项就是 Query 执行期间 yield 的次数，如果数据集不变的情况下，执行时长差别比较大，那么就可能和要访问的 page 在 WiredTiger Cache 中的命中率相关，可以通过 yield 次数来进行一定的判断。

#### “snapshot” readConcern

前面我们已经提到了 “snapshot” readConcern 是专门用于 MongoDB 的多文档事务的，MongoDB 多文档事务提供类似于传统关系型数据库的事务模型（Conversational Transaction），即通过 `begin transaction` 语句显示开启事务， 根据业务逻辑执行不同的操作序列，然后通过 `commit transaction` 语句提交事务。”snapshot” readConcern 除了包含 “majority” readConcern 提供的语义，同时**它还提供真正的一致性快照语义**，因为**多文档事务中的多个操作只会对应到一个 WiredTiger 引擎事务，并不会应用「Query Yielding」**。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639074.png)](https://mongoing.com/wp-content/uploads/2021/03/32328efd8f4cce5.png)

这里这么设计的主要考虑是，和默认情况下为了保证性能而采用单文档事务不同，当应用显示启用多文档事务时，往往意味着它希望 MongoDB 提供类似关系型数据库的，更强的一致性保证，「Query Yielding」导致的 snapshot “漂移”显然是无法接受的。而且在目前的实现中，如果应用使用了多文档事务，即使指定 “majority” 或 “local” readConcern，也会被强制提升为 “snapshot” readConcern。

```c++
// If "startTransaction" is present, it must be true due to the parsing above.
const bool upconvertToSnapshot(sessionOptions.getStartTransaction());
auto newReadConcernArgs = uassertStatusOK(
  _extractReadConcern(invocation.get(), request.body, upconvertToSnapshot)); // 这里强制提升为 "snapshot" readConcern
```

不采用 「Query Yielding」也就意味着存在上节所说的“WiredTiger Cache 压力过大”的问题，在 “snapshot” readConcern 下，当前版本没有太好的解法（在 4.4 中会通过 [durable history](https://jira.mongodb.org/browse/WT-5672)，即支持把多版本数据写到磁盘，而不是只保存在内存中来解决这个问题）。MongoDB 目前采用了另外一个比较简单粗暴的方式来缓解这个问题，即限制事务执行的时长，[`transactionLifetimeLimitSeconds`](https://docs.mongodb.com/manual/reference/parameters/#param.transactionLifetimeLimitSeconds) 配置的值决定了多文档事务的最大执行时长，默认为 60 秒。

超出最大执行时长的事务由[后台线程](https://github.com/mongodb/mongo/blob/master/src/mongo/db/periodic_runner_job_abort_expired_transactions.h)负责清理，默认每 30 秒进行一次清理动作。每个多文档事务都会和一个 [Logical Session](https://www.mongodb.com/blog/post/transactions-background-part-2-logical-sessions-in-mongodb) 关联，清理线程会遍历内存中的 [`SessionCatalog`](https://github.com/mongodb/mongo/blob/master/src/mongo/db/session_catalog.h) 缓存找到所有过期事务，清理和事务关联的 Session，然后 `abortTransaction`（具体可参考[`killAllExpiredTransactions()`](https://github.com/mongodb/mongo/blob/0f1e2d30e3c7d891f2b14c2c92ea630227298e4a/src/mongo/db/kill_sessions_local.cpp#L118-L149)）。

“snapshot” readConcern 为了**同时维持分布式环境下的 “majority” read 语义和事务本地执行的一致性快照语义**，还会带来另外一个问题：**事务因为写冲突而 abort 的概率提升**。

在单机环境下，事务的写冲突往往是因为并发事务的执行修改了同一份数据，进而导致后提交的事务需要 abort（first-writer-win）。但是通过后面的解释我们会看到，**“snapshot” readConcern 为了同时维持两种语义，即使在单机环境下看起来是非并发的事务，也会因为写冲突而 abort**。

要说明这个问题，先来简单看下事务在 snapshot isolation 下的读写[规则](https://infoscience.epfl.ch/record/53561/files/srds2005-gsi.pdf)。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639916.png)](https://mongoing.com/wp-content/uploads/2021/03/4c57014a4bccb4f.png)[![img](https://mongoing.com/wp-content/uploads/2021/03/a9e59d50fbbb1b9.png)](https://mongoing.com/wp-content/uploads/2021/03/a9e59d50fbbb1b9.png)

- 对于读：
  - 对任意事务 $T_i$ ，如果它读到了数据 $X$ 的版本 $X_j$，而 $X_j$ 是由事务 $T_j$ 修改产生，则 $T_j$ 一定已经提交，且 $T_j$ 的提交时间戳一定小于事务 $T_i$ 的快照读时间戳，即只有这样， $T_j$ 的修改对 $T_i$ 才是可见的。**这个规则保证了事务只能读取到自己可见范围内的数据**。
  - 另外，对任意事务 $T_k$，如果它修改了 $X$ 并且产生了新的版本 $X_k$，且 $T_k$ 已提交，那么 $T_k$ 要么在事务 $T_j$ 之前提交（$commit(T_k) < commit(T_j)$），要么在事务 $T_i$ 的快照读时间戳之后提交。**这个规则保证了事务在可见范围内读取最新的数据**。
- 对于写：
  - 对于任意事务 $T_i$ 和 $T_j$，他们都成功提交的前提是没有产生冲突。
  - 冲突的定义：如果 $T_j$ 的提交时间戳在事务 $T_i$ 的观测时间段（[$snapshot(T_i)$, $commit(T_i)$]）内，且二者的修改数据集存在交集，则二者存在冲突。这种情况下 $T_i$ 需要 abort。
  - 对这个规则可以有一个通俗的理解，即事务的并发控制存在一个基本原则：「**过去不能修改将来**」，$snapshot(T_i) < commit(T_j)$ 表明 $T_i$ 相对于 $T_j$ 发生在过去（此时 $T_i$ 看不到 $T_j$ 产生的修改）， $T_i$ 如果正常提交，因为 $commit(T_i) > commit(T_j)$，也就意味着发生在过去的 $T_i$ 的写会覆盖将来的 $T_j$。

然后再回到前面的问题：为什么在 “snapshot” readConcern 下事务冲突 abort 的概率会提升？这里我们结合一个例子来进行说明，

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041639070.png)](https://mongoing.com/wp-content/uploads/2021/03/1833e50c380ad53.png)

上图中，C1 发起的事务 T1 在主节点（P）上提交后，需要复制到一个从节点（S） 并且 apply 完成才算是 majority committed。在事务从 local committed 变为 majority committed 这个延迟内（上图中的红圈），如果 C2 也发起了一个事务 T2，**虽然 T2 是在 T1 提交之后才开始的，但根据 “majority” read 语义的要求，T2 不能够读取 T1 刚提交的修改**，而是基于 mcp 读取 T1 修改前的版本，这个是符合前面的 snapshot read rule 的（ D1 规则）。

但是，如果 T2 读取了这个更早的版本并且做了修改，因为 T2 的 `commit_ts`（有递增要求） 大于 T1 的，根据前面的 snapshot commit rule（D2 规则），T2 需要 abort。

需要说明的是，应用对数据的访问在时间和空间上往往呈现一定的局部性，所以**上述这种 back-to-back transaction workload（T1 本地修改完成后，T2 接着修改同一份数据）在实际场景中是比较常见的**，所以很有必要对这个问题作出优化。

MongoDB 对这个问题的优化也比较简单，采用了和 “majority” readConcern 一样的实现思路，即「speculative read」。MongoDB 把这种基于「speculative read」机制实现的 snapshot isolation 称之为「speculative snapshot isolation」。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640953.png)](https://mongoing.com/wp-content/uploads/2021/03/1c1201616a7b300.png)

仍然使用上面的例子，在「speculative snapshot isolation」机制下，事务 T2 在开始时不再基于 mcp 读取 T1 提交前的版本，而是直接读取最新的已提交值（T1 提交），这样 $snapshot(T_2) >= commit(T_1)$ ，即使 T2 修改了同一条数据，也不会违反 D2 规则。

但是此时 T1 还没有被复制到 majority 节点，T2 如果直接返回客户端成功，显然违反了 “majority” read 的语义。MongoDB 的做法是，**在事务 T2 提交时，如果要维持 “majority” read 的语义，其必须也以 “majority” writeConcern 提交**。这样，如果 T2 产生了修改，在其等待自身的修改成为 majority committed 时，发生它之前的事务 T1 的修改显然也已经是 majority committed（这个是由 MongoDB 复制协议的顺序性和 batch 并发 apply 的原子性保证的），所以自然可保证 T2 读取到的最新值满足 “majority” 语义。

这个方式**本质上是一种牺牲 Latency 换取 Consistency 的做法，和基于 snapshot 的 “majority” readConcern 做法正好相反**。这里这么设计的原因，并不是有目的的去提供更好的一致性，主要还是为了降低事务冲突 abort 的概率，这个对 MongoDB 自身性能和业务的影响非常大，在这个基础上，也可以说，保证业务读取到最新的数据总是更有用的。

关于牺牲 Latency，实际上上述实现机制，对于写事务来说并没有导致额外的延迟，因为事务自身以 “majority” writeConcern 提交进行等待以满足自身写的 majority committed 要求时，也顺便满足了 「speculative read」对等待的需求，缺点就是事务的提交必须要和 “majority” readConcern 强绑定，但是从多文档事务隐含了对一致性有更高的要求来看，这种绑定也是合理的，避免了已提交事务的修改在重新选主后被回滚。

真正产生额外延迟的是只读事务，因为事务本身没有做任何修改，仍然需要等待。实际上这个延迟也可以被优化掉，因为事务如果只是只读，不管读取了哪个时间点的快照，都不会和其他写事务形成冲突，但是 MongoDB 目前并没有提供标记多文档事务为只读事务的接口，期待后续的优化。

#### “local” readConcern

“local” readConcern 在 MongoDB 里面的语义最为简单，即直接读取本地最新的已提交数据，但是它在 MongoDB 里面的实现却相对复杂。

首先我们需要了解的是 MongoDB 的复制协议是一种类似于 [Raft](https://raft.github.io/) 的复制状态机（**R**eplicated **S**tate **M**achine）协议，但它和 Raft 最大区别是，**Raft 先把日志复制到多数派节点，然后再 Apply RSM，而 MongoDB 是先 Apply RSM，然后再异步的把日志复制到 Follower（Secondary） 去 Apply**。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640438.png)](https://mongoing.com/wp-content/uploads/2021/03/e4d902e4d189c73.png)

这种实现方式除了可以降低写操作（在 default writeConcern下）的延迟，也为实现 “local” readConcern 提供了机会，而 Recency，前面的统计数据已经分析了，正是大部分的业务所更加关注的。

MongoDB 的这种设计虽然更贴近于用户需求，但也为它的 RSM 协议引入了额外的复杂性，这点主要体现在重新选举时。

重新选主时可能会发生，已经在之前的 Primary 上追加的部分 log entry 没有来及复制到新的 Primary 节点，那么在前任 Primary重新加入集群时，需要把这部分多余的 log entry 回滚掉（**注**：这种情况，除了旧主可能发生，其他节点也可能发生）。对于 Raft 来说这个回滚动作特别简单，只需对 replicated log 执行 truncate，移除尾部多余的 log entry，然后重新从现任 Primary 追日志即可。

但是，对于 MongoDB 来说，由于在追加日志前就已经对状态机进行了 apply，所以除了 Log Truncation，还需要一个状态机回滚（Data Rollback）流程。Data Rollback 是一个代价比较大的过程，而 MongoDB 本身的日志复制是通常是很快的，真正在发生重新选举时，未及时同步到新主的 log entry 是比较少的，所以**如果能够让新主在接受写操作之前，把旧主上“多余”的日志重新拉取过来并应用**，显然可以避免旧主的 Data Rollback。

> 关于 MongoDB 基于 Raft 协议修改的延伸阅读：[4 modifications for Raft consensus](https://www.openlife.cc/blogs/2015/september/4-modifications-raft-consensus)

##### 重选举时的 Catchup Phase

MongoDB 从 3.4 版本开始实现了上述机制（`catchup phase`），流程如下，

1. 候选节点在成功收到多数派节点的投票后，会通过心跳（`replSetHeartBeat` 命令）向其他节点广播自己当选的消息；
2. 其他节点的的 heartbeat response 中会包含自己最新的 applied opTime，当选节点会把其中最大的 opTIme 作为自己 catchup 的 `targetOpTime`；
3. 从 applied opTime 最大的节点或其下游节点同步数据，这个过程和正常的基于 oplog 的增量复制没有太大区别；
4. 如果在超时时间（由 [`settings.catchUpTimeoutMillis`](https://docs.mongodb.com/v3.4/reference/replica-configuration/#rsconf.settings.catchUpTimeoutMillis) 决定，3.4 默认 60 秒）内追上了 `targetOpTime`，catchup 完成；
5. 如果超时，当选节点并不会 stepDown，而是继续作为新的 Primary 节点。

```c++
void ReplicationCoordinatorImpl::CatchupState::signalHeartbeatUpdate_inlock() {
    auto targetOpTime = _repl->_topCoord->latestKnownOpTimeSinceHeartbeatRestart();
    ...
    ReplicationMetrics::get(getGlobalServiceContext()).setTargetCatchupOpTime(targetOpTime.get());
    log() << "Heartbeats updated catchup target optime to " << *targetOpTime;
    ...
}
```

上述第 5 步意味着，catchup 过程中如果有超时发生，其他节点仍然需要回滚，所以在 3.6 版本中，MongoDB 对这个机制进行了强化。3.6 把 [`settings.catchUpTimeoutMillis`](https://docs.mongodb.com/v3.4/reference/replica-configuration/#rsconf.settings.catchUpTimeoutMillis) 的默认值调整为 -1，即不超时。但为了避免 `catchup phase` 无限进行，影响可用性（集群不可写），增加了 `catchup takeover` 机制，即**集群当前正在被当选节点作为同步源 catchup 的节点，在等待一定的时间后，会主动发起选举投票，来使“不合格”的当选节点下台**，从而减少 Data Rollback 的几率和保证集群尽快可用。

这个等待时间由副本集的 [`settings.catchUpTakeoverDelayMillis`](https://docs.mongodb.com/v3.6/reference/replica-configuration/#rsconf.settings.catchUpTakeoverDelayMillis) 配置决定，默认为 30 秒。

```c++
stdx::unique_lock<stdx::mutex> ReplicationCoordinatorImpl::_handleHeartbeatResponseAction_inlock(
    ...
        case HeartbeatResponseAction::CatchupTakeover: {
            // Don't schedule a catchup takeover if any takeover is already scheduled.
            if (!_catchupTakeoverCbh.isValid() && !_priorityTakeoverCbh.isValid()) {
                Milliseconds catchupTakeoverDelay = _rsConfig.getCatchUpTakeoverDelay();
                _catchupTakeoverWhen = _replExecutor->now() + catchupTakeoverDelay;
                LOG_FOR_ELECTION(0) << "Scheduling catchup takeover at " << _catchupTakeoverWhen;
                _catchupTakeoverCbh = _scheduleWorkAt(
                    _catchupTakeoverWhen, [=](const mongo::executor::TaskExecutor::CallbackArgs&) {
                        _startElectSelfIfEligibleV1(StartElectionReasonEnum::kCatchupTakeover); // 主动发起选举
                    });
            }
    ...
```

Data Rollback 是无法彻底避免的，因为 `catchup phase` 也只能发生在拥有最新 log entry 的节点在线的情况下，即能够向当选节点恢复心跳包，如果在选举完成后，节点才重新加入集群，仍然需要回滚。

MongoDB 目前存在两种 Data Rollback 机制：「Refeched Based Rollback」 和 「Recover To Timestamp Rollback」，其中后一种是在 4.0 及之后的版本，伴随着 WiredTiger 存储引擎能力的提升而演进出来的，下面就简要描述一下它们的实现方式及关联。

##### Refeched Based Rollback

「Refeched Based Rollback」 可以称之为**逻辑回滚**，下面这个图是逻辑回滚的流程图，

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640427.png)](https://mongoing.com/wp-content/uploads/2021/03/9a0e48407dafccc.png)

首先待回滚的旧主，需要确认重新选主后，自己的 oplog 历史和新主的 oplog 历史发生“分叉”的时间点，在这个时间点之前，新主和旧主的 oplog 是一致的，所以这个点也被称之为「**common point**」。旧主上从「common point」开始到自己最新的时间点之间的 oplog 就是未来及复制到新主的“多余”部分，需要回滚掉。

common point 的查找逻辑在 [`syncRollBackLocalOperations()`](https://github.com/mongodb/mongo/blob/54e87285f802a49456c02c80cb0ddb0fbb54c88a/src/mongo/db/repl/roll_back_local_operations.cpp#L164) 中实现，大致流程为，由新到老（反向）从同步源节点获取每条 oplog，然后和自己本地的 oplog 进行比对。本地 oplog 的扫描同样为反向，由于 oplog 的时间戳可以保证递增，扫描时可以通过保存中间位点的方式来减少重复扫描。如果最终在本地找到一条 oplog 的时间戳和 `term` 和同步源的完全一样，那么这条 oplog 即为 common point。由于在分布式环境下，不同节点的时钟不能做到完全实时同步，而 **term 可以唯一标识一个主节点在任期间的修改（oplog）历史**，所以需要把 oplog ts 和 term 结合起来进行 common point 的查找。

在找到 common point 之后，待回滚节点需要把当前最新的时间戳到 common point 之间的 oplog 都回滚掉，由于回滚采用逻辑的方式，整个流程还是比较复杂的。

首先，MongoDB 的 oplog 本质上是一种 redo log，可以通过重新 apply 来进行数据恢复，而且 oplog 记录时对部分操作进行了重写，比如 `{$inc : {quantity : 1}}` 重写为 `{$set : {quantity : val}}` 等，**来保证 oplog 的幂等性，按序重复应用 oplog，并不会导致数据不一致**。但是 oplog 并不包含 undo 信息，所以对于部分操作来说，无法实现基于本地信息直接回滚，比如对于 delete，dropCollection 等操作，删除掉的文档在 oplog 并无记录，显然无法直接回滚。

对于上述情况，MongoDB 采用了所谓「refetch」的方式进行回滚，**即重新从同步源获取无法在本地直接回滚的文档**，但是这个方式的问题在于 oplog 回滚到 tcommon 时，节点可能处于一个不一致的状态。举个例子，在 tcommon 时旧主上存在两条文档 `{x : 10}` 和 `{y : 20}`，在重新选主之后，旧主上对 `x` 的 delete 操作并未同步到新主，在新主新的历史中，客户端先后对 x 和 y 做了更新：`{$set : {y : 200}} ; {$set : {x : 100}}`。在旧主通过「refetch」的方式完成回滚后，它在 tcommon 的状态为： `{x : 100}` 和 `{y : 20}`，显然这个状态对于客户端来说是不一致的。

这个问题的根本原因在于，**「refetch」时只能获取到被删除文档当前最新的状态，而不是被删除前的状态，这个方式破坏了在客户端看来可能存在因果关系的不同文档间的一致性状态**。我们具体上面的例子来说，回滚节点在「refetch」时相当于直接获取了 `{$set : {x : 100}}` 的状态变更操作，而跳过了 `{$set : {y : 200}}`，如果要达到一致性状态，看起来只要重新应用 `{$set : {y : 200}}` 即可。但是回滚节点基于现有信息是无法分析出来跳过了哪些状态的，对于这个问题，直接但是有效的做法是，把同步源从 tcommon 之后的 oplog 都重新拉取并「**reapply**」一遍，显然可以把跳过的状态补齐。而这中间也可能存在对部分状态变更操作的重复应用，比如 `{$set : {x : 100}}`，这个时候 oplog 的幂等性就发挥作用了，可以保证数据在最终「reapply」完后的一致性不受影响。

剩下的问题就是，拉取到同步源 oplog 的什么位置为止？对于回滚节点来说，导致状态被跳过的原因是进行了「refetch」，所以只需要记录每次「refetch」时同步源最新的 oplog 时间戳，「reapply」时拉取到最后一次「refetch」对应的这个同步源时间戳就可以保证状态的正确补齐，MongoDB 在实现中把这个时间戳称之为 `minValid`。

MongoDB 在逻辑回滚的过程中也进行了一些优化，比如在「refetch」之前，会扫描一遍需要回滚的操作（这个不需要专门来做，在查找 common point 的过程即可实现），对于一些存在“互斥”关系的操作，比如 `{insert : {_id:1}` 和 `{delete : {_id:1}}`，就没必要先 refetch 再 delete 了，直接忽略回滚处理即可。但是从上面整体流程看，「Refeched Based Rollback」仍然复杂且代价高：

- 「refetch」阶段需要和同步源通信，并进行数据拉取，如果回滚的是删表操作，代价很大
- 「reapply」阶段也需要和同步源通信，如果「refetch」阶段比较慢，需要拉取和重新应用的 oplog 也比较多
- 实现上复杂，每种可能出现在 oplog 中的操作都需要有对应的回滚逻辑，新增类型时同样需要考虑，代码维护代价高

所以在 4.0 版本中，随着 WiredTiger 引擎提供了回滚到指定的 Timestamp 的功能后，MongoDB 也用物理回滚的机制取代了上述逻辑回滚的机制，但在某些特殊情况下，逻辑回滚仍然有用武之地，下面就对这些做简要分析。

##### Recover To Timestamp Rollback

「Recover To Timestamp Rollback」是借助于存储引擎把物理数据直接回滚到某个指定的时间点，所以这里把它称之为**物理回滚**，下面是 MongoDB 物理回滚的一个简化的流程图，

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640725.png)](https://mongoing.com/wp-content/uploads/2021/03/da8a042a7924367.png)

前面已经提到了 stable timestamp 的语义，这里不再赘述，MongoDB 有一个后台线程（`WTCheckpointThread`）会定期（默认情况下每 60 秒，由 `storage.syncPeriodSecs` 配置决定）根据 stable timestamp 触发新的 checkpoint 创建，这个 checkpoint 在实现中被称为 「**stable checkpoint**」。

```c++
class WiredTigerKVEngine::WiredTigerCheckpointThread : public BackgroundJob {
public:
...
    virtual void run() {
            ...
            {
                stdx::unique_lock<stdx::mutex> lock(_mutex);
                MONGO_IDLE_THREAD_BLOCK;
                _condvar.wait_for(lock,
                                  stdx::chrono::seconds(static_cast<std::int64_t>(
                                      wiredTigerGlobalOptions.checkpointDelaySecs)));
            }
            ...    
                    UniqueWiredTigerSession session = _sessionCache->getSession();
                    WT_SESSION* s = session->getSession();
                    invariantWTOK(s->checkpoint(s, "use_timestamp=true"));
            ...
    }
...    
}
```

stable checkpoint 本质上是一个持久化的历史快照，它所包含的数据修改已经复制到多数派节点，所以不会发生重新选主后修改被回滚。其实 WiredTiger 本身也可以配置根据生成的 WAL 大小或时间来[自动触发](https://source.wiredtiger.com/develop/checkpoint.html)创建新的 checkpoint，但是 Server 层并没有使用，原因就在于 MongoDB 需要保证在回滚到上一个 checkpoint 时，状态机肯定是 “stable” 的，不需要回滚。

WiredTiger 在创建 stable checkpoint 时也是开启一个带时间戳的事务来保证 checkpoint 的一致性，checkpoint 线程会把事务可见范围内的脏页刷盘，最后对应到磁盘上就是一个由多个变长数据块（WT 中称之为`extent`）构成的 BTree。

回滚时，同样要先确定 common point，这个流程和逻辑回滚没有区别，之后， Server 层会首先 abort 掉所有活跃事务，接着调用 WT 提供的 `rollback_to_stable()` 接口把数据库回滚到 stable checkpoint 对应的状态，这个动作主要是重新打开 checkpoint 对应的 BTree，并重新初始化 catalog 信息，`rollback_to_stable()` 执行完后会向 Server 层返回对应的 stable timestamp。

考虑到 stable checkpoint 触发的间隔较大，通常 common point 总是大于 stable checkpoint 对应的时间戳，所以 Server 层在拿到引擎返回的时间戳之后会还需要从其开始重新 apply 本地的 oplog 到 common point 为止，然后把 common point 之后的 oplog truncate 掉，从而达到和新的同步源一致的状态。这个流程主要在 `RollbackImpl::_runRollbackCriticalSection()` 中实现，

```c++
Status RollbackImpl::_runRollbackCriticalSection(
    OperationContext* opCtx,
    RollBackLocalOperations::RollbackCommonPoint commonPoint) noexcept try {
    ...
    killSessionsAbortAllPreparedTransactions(opCtx); // abort 活跃事务
    ...
    auto stableTimestampSW = _recoverToStableTimestamp(opCtx); // 引擎层回滚
    ...
    Timestamp truncatePoint = _findTruncateTimestamp(opCtx, commonPoint); // 查找并设置 truncate 位点
    _replicationProcess->getConsistencyMarkers()->setOplogTruncateAfterPoint(opCtx, truncatePoint);
    ...
    // Run the recovery process. // 这里会进行 reapply oplog 和 truncate oplog
    _replicationProcess->getReplicationRecovery()->recoverFromOplog(opCtx,
                                                                    stableTimestampSW.getValue());
    ...                                                                    
}
```

此外，为了确保回滚可以正常进行，Server 层在 oplog 的自动回收时还需要考虑 stable checkpoint 对部分 oplog 的依赖。通常来说，stable timestamp 之前的 oplog 可以安全的回收，但是在 4.2 中 MongoDB 增加了对大事务（对应的 oplog 大小超过 16MB）和分布式事务的支持，在 stable timestamp 之前的 oplog 在回滚 reapply oplog 的过程中也可能是需要的，所以在 4.2 中 oplog 的回收需要综合考虑当前最老的活跃事务和 stable timestamp。

```c++
StatusWith<Timestamp> WiredTigerKVEngine::getOplogNeededForRollback() const {
    ...
    if (oldestActiveTransactionTimestamp) {
        return std::min(oldestActiveTransactionTimestamp.value(), Timestamp(stableTimestamp));
    } else {
        return Timestamp(stableTimestamp);
    }
}
```

整体上来说，基于引擎 stable checkpoint 的**物理回滚方式在回滚效率和回滚逻辑复杂性上都要优于逻辑回滚**。但是 stable checkpoint 的推进要依赖 Server 层 majority commit point 的推进，而 majority commit point 的推进受限于各个节点的复制进度，所以复制慢时可能会导致 Primary 节点 cache 压力过大，所以 MongoDB 提供了 [`replication.enableMajorityReadConcern`](https://docs.mongodb.com/manual/reference/configuration-options/#replication.enableMajorityReadConcern) 参数用于控制是否维护 mcp，关闭后存储引擎也不再维护 stable checkpoint，此时回滚就仍然需要进行逻辑回滚，这也是在 4.2 中仍然保留「Refeched Based Rollback」的原因。

#### “linearizable” readConcern

**在一个分布式系统中，如果总是把可用性摆在第一位，那么因果一致性是其能够实现的最高一致性级别**（证明可见[此处](https://www.cs.utexas.edu/users/dahlin/papers/cac-tr.pdf)）。前面我们也通过统计数据分析了在大部分情况下用户总是更关注延迟（可用性）而不是一致性，而 MongoDB 副本集，正是从用户需求角度出发，被设计成了一个在默认情况下总是优先保证可用性的分布式系统，下图是一个简单的例证。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640685.png)](https://mongoing.com/wp-content/uploads/2021/03/5dc6fe28dd04b78.png)

既然如此，那 MongoDB 是如何实现 “linearizable” readConcern，即更高级别的线性一致性呢？ MongoDB 的策略很简单，就是**把它退化到几乎是单机环境下的问题**，即只允许客户端在 Primary 节点上进行 “linearizable” 读。说是“几乎”，因为这个策略仍然需要解决如下两个在副本集这个分布式环境下存在的问题，

1. Primary 角色可能会发生变化，“linearizable” readConcern 需要保证每次读取总是能够从当前的 Primary 读取，而不是被取代的旧主。
2. 需要保证读取到读操作开始前最新的写，而且读到的结果不会在重新选主后发生回滚。

**MongoDB 采用同一个手段解决了上述两个问题**，当客户端采用 “linearizable” readConcern 时，在读取完 Primary 上最新的数据后，**在返回前会向 Oplog 中显示的写一条 `noop` 的操作，然后等待这条操作在多数派节点复制成功**。显然，如果当前读取的节点并不是真正的主，那么这条 `noop` 操作就不可能在 majority 节点复制成功，同时，如果 `noop` 操作在 majority 节点复制成功，也就意味着之前读取的在 `noop` 之前写入的数据也已经复制到多数派节点，确保了读到的数据不会被回滚。

```c++
// src/mongo/db/read_concern_mongod.cpp:waitForLinearizableReadConcern()
...
        writeConflictRetry(
            opCtx,
            "waitForLinearizableReadConcern",
            NamespaceString::kRsOplogNamespace.ns(),
            [&opCtx] {
                WriteUnitOfWork uow(opCtx);
                opCtx->getClient()->getServiceContext()->getOpObserver()->onOpMessage(
                    opCtx,
                    BSON("msg"
                         << "linearizable read")); // 写 noop 操作
                uow.commit();
            });
...
    auto awaitReplResult = replCoord->awaitReplication(opCtx, lastOpApplied, wc); // 等待 noop 操作 majority committed
```

这个方案的缺点比较明显，单纯的读操作既产生了额外的写开销，也增加了延迟，但是这个是选择最高的一致性级别所需要付出的代价。

### Causal Consistency

**前面几个章节描述的由 writeConcern 和 readConcern 所构成的 MongoDB 可调一致性模型，仍然是属于最终一致性的范畴**（特殊实现的 “linearizable” readConcern 除外）。虽然最终一致性对于大部分业务场景来说已经足够了，但是在某些情况下仍然需要更高的一致性级别，比如在下图这个经典的银行存款业务中，如果只有最终一致性，那么就可能导致客户看到的账户余额异常。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640050.png)](https://mongoing.com/wp-content/uploads/2021/03/14cbae23aa883a0.png)

这个问题虽然可以在业务端通过记录一些额外的状态和重试来解决，但是显然会导致业务逻辑过于复杂，所以 MongoDB 实现了「Causal Consistency Session」功能来帮助降低业务复杂度。

Causal Consistency 定义了分布式系统上的**读写操作需要满足一个偏序（Partial Order）关系，即只部分操作发生的先后顺序可比**。这个部分操作，进一步来说，指的是存在因果关系的操作，在 MongoDB 的「Causal Consistency Session」实现中，什么样的操作间算是存在因果关系，可参考前文提到的 Client-centric Consistency Model 下的 4 个一致性承诺分别对应的读写场景，此处不再赘述。

所以，要实现因果一致性，MongoDB **首要**解决的问题是如何**给分布式环境下存在因果关系的操作定序**，这里 MongoDB 借鉴了 [Hybrid Logical Clock](https://cse.buffalo.edu/~demirbas/publications/hlc.pdf) 的设计，实现了自己的 ClusterTime 机制，下面就对其实现进行分析。

#### 分布式系统中存在因果关系的操作定序

关于分布式系统中的事件如何定序的论述，最有影响力的当属 Leslie Lamport 的这篇 《
[Time, Clocks, and the Ordering of Events in a Distributed System](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)》，其中提到了一种 Logical Clock 用来确定不同事件的全序，后人也把它称为 Lamport Clock。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041640779.png)](https://mongoing.com/wp-content/uploads/2021/03/c59684a0831ee78.png)

Lamport Clock 只用一个单纯的变量（scalar value）来表示，所以它的**缺点之一是无法识别存在并发的事件（independent event）**，而这个会在实际的系统带来一些问题，比如在支持多点写入的系统中，无法基于 Lamport Clock 对存在写冲突的事件进行识别和处理。所以，后面又衍生出了 [vector clock](https://en.wikipedia.org/wiki/Vector_clock) 来解决这一问题，但 vector clock 会存储数据的多个版本，数据量和系统中的节点数成正比，所以实际使用会带来一些扩展性的问题。

Lamport Clock 存在的**另外一个缺点是，它完全是一个逻辑意义上的值，和具体的物理时钟没有关联**，而在现实的应用场景中，存在一些需要基于真实的物理时间进行访问的场景，比如数据的备份和恢复。Google 在其 [Spanner](https://research.google.com/archive/spanner-osdi2012.pdf) 分布式数据库中提到了一种称之为 TrueTime 的分布式时钟设计，为事务执行提供时间戳。**TrueTime 和真实物理时钟关联**，但是需要特殊的硬件（原子钟/GPS）支持，MongoDB 作为一款开源软件，需要做到通用的部署，显然无法采用该方案。

考虑到 MongoDB 本身不支持 「Multi-Master」 架构，而上述的分布式时钟方案均存在一些 MongoDB 在设计上需要规避的问题，所以 MongoDB 采用了一种所谓的混合逻辑时钟（**H**ybrid **L**ogical **C**lock）的方案。**HLC** 设计上基于 Lamport Clock，只使用单个时钟变量，**在具备给因果操作定序的能力同时，也能够（尽可能）接近真实的物理时钟**。

##### Hybrid Logical Clock 基本原理

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041641833.png)](https://mongoing.com/wp-content/uploads/2021/03/0ce42ee8250b9f8.png)

先来了解一下 HLC 中几个基本的概念，

- `pt`：节点本地的物理时钟，通常是基于 ntp 进行时钟同步，HLC 只会读取该值，不会对该值做修改。
- `l`：HLC 的高位部分，是 HLC 的物理部分，和`pt`关联，HLC 保证 `l` 和 `pt`间的差值不会无限增长（bounded）。
- `c`：HLC 的低位部分，是 HLC 的逻辑部分。

从上面的 HLC 时钟推进图中，可以看到，如果不考虑 `l` 部分（假设 `l` 总是不变），则 `c` 等同于 Lamport Clock，如果考虑 `l` 的变化，因为 `l` 是高位部分，只需要保证 `if e hb f, l.e <= l.f`，仍然可以确定存在因果关系的事件的先后顺序，具体的更新规则可以参考上面的算法。

但是 `l` 的更新机制也决定了其他节点的时钟出现跳变或不同步，会导致 HLC 被推进，进而导致和 `pt` 产生误差，但 HLC 的机制决定了这个误差是有限的。上面的图就是一个很好的案例，假设当前的真实物理时钟是 0，而 0 号节点的时钟出现了跳变，变为 10，则在后续的时钟推进中，`l` 部分不会再增长，只会增加 `c` 部分，直到真实的物理时钟推进到 10，`l` 才会关联新的 `pt` 。

MongoDB 在实现 Causal Consistency 之前就已经在副本集同步的 oplog 时间戳中使用了类似的设计，选择 HLC，也是为了方便和现有设计集成。Causal Consistency 不仅是在单一的副本集层面使用，在基于副本集构建的分片集群中同样有需求，所以这个新的分布式时钟，在 v3.6 中被称为 **ClusterTime**。

##### MongoDB ClusterTime 实现

MongoDB ClusterTime 基本上是严格按照 HLC 的思路来实现的，但它和 HLC 最大的一点不同是，在 HLC 或 Lamport Clock 中，消息的发送和接受都被认为是一个事件，会导致时钟值增加，但在 MongoDB ClusterTime 实现中，**只有会改变数据库状态的操作发生才会导致 ClusterTime 增加**，比如通常的写操作，这么做的目的还是为了和现有的 oplog 中的混合时间戳机制集成，避免更大的重构开销和由此带来的兼容性问题，同时这么做也并不会影响 ClusterTime 在逻辑上的正确性。

因为有了上述区别，ClusterTime 的实现就可以被分为两部分，一个是 ClusterTime 的增加（**Tick**），一个是 ClusterTime 的推进（**Advance**）。

**ClusterTime 的 Tick** 发生在 MongoDB 接收到写操作时，ClusterTime 由 `<Time><Counter>` 来表示，是一个 64bit 的整数，其中高 32 位对应到 HLC 中的物理部分，低 32 位对应到 HLC 中的逻辑部分。而每一个写操作在执行前都会为即将要写的 oplog 提前申请对应的 **OpTime**（调用 `getNextOpTimes()` 来完成），OpTime 由 `<Time><Counter><ElectionTerm>` 来表示，`ElectionTerm` 和 MongoDB 的复制协议相关，是一个本地的状态值，不需要被包含到 ClusterTime 中，所以原有的 OpTime 在新版本中实际上是可以由 ClusterTime 直接转化得来，而 ClusterTime 也会随着 Oplog 写到磁盘而被持久化。

```c++
std::vector<OplogSlot> LocalOplogInfo::getNextOpTimes(OperationContext* opCtx, std::size_t count) {
...
        // 申请 OpTime 时会 Tick ClusterTime 并获取 Tick 后的值
        ts = LogicalClock::get(opCtx)->reserveTicks(count).asTimestamp();
        const bool orderedCommit = false;
...
    std::vector<OplogSlot> oplogSlots(count);
    for (std::size_t i = 0; i < count; i++) {
        oplogSlots[i] = {Timestamp(ts.asULL() + i), term}; // 把 ClusterTime 转化为 OpTime
...
    return oplogSlots;
}
// src/mongo/db/logical_clock.cpp:LogicalClock::reserveTicks() 包含了 Tick 的逻辑，和 HLC paper 一致，主要逻辑如下
{
    newCounter = 0;
    wallClockSecs = now();
    // _clusterTime is a current local value of node’s ClusterTime
    currentSecs = _clusterTime.getSecs();
    if (currentSecs > wallClockSecs) {
        newSecs = currentSecs;
        newCounter = _clusterTime.getCounter() + 1;
    } else {
        newSecs = wallClockSecs;
    }
    _clusterTime = ClusterTime(newSecs, newCounter);
    return _clusterTime;
}
```

**ClusterTime 的 Advance** 逻辑比较简单，MongoDB 会在每个请求的回复中带上当前节点最新的 ClusterTime，如下，

```
"$clusterTime" : {
    "clusterTime" : Timestamp(1495470881, 5),
    "signature" : {
        "hash" : BinData(0, "7olYjQCLtnfORsI9IAhdsftESR4="),
        "keyId" : "6422998367101517844"
    }
}
```

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041641294.png)](https://mongoing.com/wp-content/uploads/2021/03/961471c215bfd24.png)

接收到该 ClusterTime 的角色（mongos，client）如果发现更新的 ClusterTime，就会更新本地的值，同时在和别的节点通信的时候，带上这个新 ClusterTime，从而推进其他节点上的 ClusterTime，这个流程实际上**是一种类似于 Gossip 的消息传播机制**。

因为 Client 会参与到 ClusterTime 的推进（Advance），如果有恶意的 Client 篡改了自己收到的 ClusterTime，比如把高位和低位部分都改成了 UINT32_MAX，则收到该 ClusterTime 的节点后续就无法再进行 Tick，这个会导致整个服务不可用，所以 **MongoDB 的 ClusterTime 实现增加了签名机制（这个安全方面的增强 HLC 没有提及）**，上面的`signature` 字段即对应该功能，mongos 或 mongod 在收到 Client 发送过来的 `$ClusterTime` 时，会根据 config server 上存储的 key 来进行签名校验，如果 ClusterTime 被篡改，则签名不匹配，就不会推进本地时钟。

除了恶意的 Client，操作失误也可能导致 mongod 节点的 wall clock 被更新为一个极大的值，同样会导致 ClusterTime 不能 Tick，针对这个问题，MongoDB 做了一个限制，新的 ClusterTime 和当前 ClusterTime 的差值如果超出 [`maxAcceptableLogicalClockDriftSecs`](https://docs.mongodb.com/manual/reference/parameters/#param.maxAcceptableLogicalClockDriftSecs)，默认为 1 年，则当前的 ClusterTime 不会被推进。

#### MongoDB Causal Consistency 实现

在 ClusterTime 机制的基础上，我们就可以给不同的读写操作定序，但是操作对应的 ClusterTime 是在其被发送到数据节点（mongod）上之后才被赋予的，如果要实现 Causal Consistency 的承诺，比如前面提到的「Read Your Own Write」，显然我们需要 Client 也知道写操作在主节点执行完后对应的 ClusterTime。

```c++
        ...
        "operationTime" : Timestamp(1612418230, 1), # Stable ClusterTime
        "ok" : 1,
        "$clusterTime" : { ... }
```

所以 MongoDB 在请求的回复中除了带上 `$clusterTIme` 用于帮助推进混合逻辑时钟，还会带上另外一个字段 **`operationTime`** 用来表明这个请求包含的操作对应的 ClusterTime，`operationTime` 在 MongoDB 中也被称之为 「**Stable ClusterTime**」，它的准确含义是操作执行完成时，当前最新的 Oplog 时间戳（OpTime）。所以对于写操作来说，`operationTime` 就是这个写操作本身对应的 Oplog 的 OpTime，而对于读操作，取决于并发的写操作的执行情况。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041641214.png)](https://mongoing.com/wp-content/uploads/2021/03/7dea09babf633cd.png)

Client 在收到这个 `operationTime` 后，如果要实现因果一致，就会在发送给其他节点的请求的 `afterClusterTime` 字段中带上这个 `operationTime`，其他节点在处理这个请求时，只会读取 `afterClusterTime` 之后的数据状态，这个过程是通过显式的**等待同步位点推进**来实现的，等待的逻辑和前面提到的 speculative “majority” readConcern 实现类似。上图是 MongoDB 副本集实现「Read Your Own Write」的基本流程。

如果是在分片集群形态下，由于混合逻辑时钟的推进依赖于各个参与方（client/mongos/mongd）的交互，所以会暂时出现不同分片间的逻辑时钟不一致的情况，所以在这个架构下，我们**需要解决某个分片的逻辑时钟滞后于 `afterClusterTime` 而且一直没有新的写入，导致请求持续被阻塞的问题**，MongoDB 的做法是，在这种情况下显式的写一条 `noop` 操作到 oplog 中，相当于强制把这个分片的数据状态推进到 `afterClusterTime` 之后，从而确保操作能够尽快返回，同时也符合因果一致性的要求。

[![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408041641027.png)](https://mongoing.com/wp-content/uploads/2021/03/2659bcbac4bba53.png)

## 总结

本文对 MongoDB 一致性模型在设计上的一些考虑和主要的实现机制进行了分析，这其中包括由 writeConcern 和 readConcern 机制构建的可调一致性模型，对应到标准模型中就是最终一致性和线性一致性，但是 MongoDB 借助read/write concern 这两者的配合，为用户提供更丰富的一致性和性能间的选择。此外，我们也分析了 MongoDB 如何基于 ClusterTime 混合逻辑时钟机制来给分布式环境下的读写操作定序，进而实现因果一致性。

从功能和设计思路来看，MongoDB 无疑是丰富和先进的，但是在接口层面，读写采用不同的配置和级别，事务和非事务的概念区分，Causal Consistency Session 对 read/writeConcern的依赖等，都为用户的实际使用增加了门槛，当然这些也是 MongoDB 在易用性、功能性和性能多方取舍的结果，相信 MongoDB 后续会持续的做出改进。

最后，伴随着 NewSQL 概念的兴起，「分布式+横向扩展+事务能力」逐渐成为新数据库系统的标配，MongoDB 也不例外。当我们在传统单机数据库环境下谈论一致性，更多指的是事务间的隔离性（Isolation），**如果把隔离性这个概念映射到分布式架构下**，可以容易看出，MongoDB 的 “local” readConcern 即对应 read uncommitted，”majority” readConcern 即对应 read committed，而 “snapshot” readConcern 对应的就是分布式的全局快照隔离，即这些新的概念部分也是来自于经典的 ACID 理论在分布式环境下的延伸，带上这样的视角可以让我们更容易理解 MongoDB 的一致性模型设计。


# 如何保证mongodb和数据库双写数据一致性？

## 与Redis和数据库的数据双写一致性问题差异

### 我们是如何用缓存的？

Redis缓存能提升我们系统的性能。

一般情况下，如果有用户请求过来，先查缓存，如果缓存中存在数据，则直接返回。如果缓存中不存在，则再查数据库，如果数据库中存在，则将数据放入缓存，然后返回。如果数据库中也不存在，则直接返回失败。

流程图如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121651250.webp)

有了缓存之后，能够减轻数据库的压力，提升系统性能。

通常情况下，保证缓存和数据双写数据一致性，最常用的技术方案是：`延迟双删`

### 我们是如何用MongoDB的？

`MongoDB`是一个高可用、分布式的`文档数据库`，用于大容量数据存储。文档存储一般用类似`json`的格式存储，存储的内容是文档型的。

通常情况下，我们用来存储大数据或者json格式的数据。

用户写数据的请求，`核心数据`会被写入数据库，json格式的`非核心数据`，可能会写入MongoDB

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121651422.webp)

在数据库的表中，保存了MongoDB相关文档的id。

用户读数据的请求，会先读数据库中的数据，然后通过文档的id，读取MongoDB中的数据

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121652446.webp)

这样可以保证核心属性不会丢失，同时存储用户传入的较大的数据，两全其美。

Redis和MongoDB在我们实际工作中的用途不一样，导致了它们双写数据一致性问题的解决方案是不一样的

##  如何保证双写一致性？

目前双写MongoDB和数据库的数据，用的最多的就是下面这两种方案

### 先写数据库，再写MongoDB

该方案最简单，先在数据库中写入核心数据，再在MongoDB中写入非核心数据

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121653366.webp)

如果有些业务场景，对数据的完整性要求不高，即非核心数据可有可无，使用该方案也是可以的

但如果有些业务场景，对数据完整性要求比较高，用这套方案可能会有问题

当数据库刚保存了核心数据，此时网络出现异常，程序保存MongoDB的非核心数据时失败了

但MongoDB并没有抛出异常，数据库中已经保存的数据没法回滚，这样会出现数据库中保存了数据，而MongoDB中没保存数据的情况，从而导致MongoDB中的非核心数据丢失的问题

所以这套方案，在实际工作中使用不多

### 先写MongoDB，再写数据库

在该方案中，先在MongoDB中写入非核心数据，再在数据库中写入核心数据

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121653116.webp)

关键问题来了：如果MongoDB中非核心数据写入成功了，但数据库中的核心数据写入失败了怎么办？

这时候MongoDB中非核心数据不会回滚，可能存在MongoDB中保存了数据，而数据库中没保存数据的问题，同样会出现数据不一致的问题。

答：我们忘了一个前提，查询MongoDB文档中的数据，必须通过数据库的表中保存的`mongo id`。但如果这个`mongo id`在数据库中都没有保存成功，那么，在MongoDB文档中的数据是永远都查询不到的。

也就是说，这种情况下MongoDB文档中保存的是`垃圾数据`，但对实际业务并没有影响。

这套方案可以解决双写数据一致性问题，但它同时也带来了两个新问题：

- 用户修改操作如何保存数据？
- 如何清理垃圾数据？

## 用户修改操作如何保存数据？

我之前聊的先写MongoDB，再写数据库，这套方案中的流程图，其实主要说的是新增数据的场景。

但如果在用户修改数据的操作中，用户先修改MongoDB文档中的数据，再修改数据库表中的数据

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121654679.webp)

如果出现MongoDB文档中的数据修改成功了，但数据库表中的数据修改失败了，不也出现问题了？

那么，用户修改操作时如何保存数据呢？

这就需要把流程调整一下，在修改MongoDB文档时，还是新增一条数据，不直接修改，生成一个新的mongo id。然后在修改数据库表中的数据时，同时更新mongo id字段为这个新值。

流程图如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121655773.webp)

这样如果新增MongoDB文档中的数据成功了，但修改数据库表中的数据失败了，也没有关系，因为数据库中老的数据，保存的是老的mongo id。通过该id，依然能从MongoDB文档中查询出数据。

使用该方案能够解决修改数据时，数据一致性问题，但同样会存在垃圾数据。

其实这个垃圾数据是可以即使删除的，具体流程图如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121655497.webp)

在之前的流程中，修改完数据库，更新了mongo id为新值，接下来，就把MongoDB文档中的那条老数据直接删了。

该方案可以解决用户修改操作中，99%的的垃圾数据，但还有那1%的情况，即如果最后删除失败该怎么办？

答：这就需要加`重试机制`了。

我们可以使用`job`或者`mq`进行重试，优先推荐使用mq增加重试功能。特别是想`RocketMQ`，自带了失败重试机制，有专门的`重试队列`，我们可以设置`重试次数`。

流程图优化如下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408121656796.webp)

将之前删除MongoDB文档中的数据操作，改成发送mq消息，有个专门的mq消费者，负责删除数据工作，可以做成共用的功能。它包含了失败重试机制，如果删除5次还是失败，则会把该消息保存到`死信队列`中。

然后专门有个程序监控死信队列中的数据，如果发现有数据，则发`报警邮件`。

这样基本可以解决修改删除垃圾数据失败的问题。

## 如何清理新增的垃圾数据？

还有一种垃圾数据还没处理，即在用户新增数据时，如果写入MongoDB文档成功了，但写入数据库表失败了。由于MongoDB不会回滚数据，这时候MongoDB文档就保存了垃圾数据，那么这种数据该如何清理呢？

### 定时删除

我们可以使用job定时扫描，比如：`每天`扫描一次MongoDB文档，将mongo id取出来，到数据库查询数据，如果能查出数据，则保留MongoDB文档中的数据。如果在数据库中该mongo id不存在，则删除MongoDB文档中的数据

如果MongoDB文档中的数据量不多，是可以这样处理的。但如果数据量太大，这样处理会有性能问题。这就需要做优化，常见的做法是：`缩小扫描数据的范围`

比如：扫描MongoDB文档数据时，根据创建时间，只查最近24小时的数据，查出来之后，用mongo id去数据库查询数据。如果直接查最近24小时的数据，会有问题，会把刚写入MongoDB文档，但还没来得及写入数据库的数据也查出来，这种数据可能会被误删。

可以把时间再整体提前一小时，例如获取25小时前到1小时前的数据。这样可以解决大部分系统中，因为数据量过多，在一个定时任务的执行周期内，job处理不完的问题

但如果根据时间缩小范围之后，数据量还是太大，job还是处理不完该怎么办？

- 答：我们可以在job用`多线程`删除数据
- 当然我们还可以将job的执行时间缩短，根据实际情况而定，比如每隔12小时，查询创建时间是13小时前到1小时前的数据
- 或者每隔6小时，查询创建时间是7小时前到1小时前的数据
- 或者每隔1小时，查询创建时间是2小时前到1小时前的数据等等

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

## oplog主从日志

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

## 慢查询日志

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













