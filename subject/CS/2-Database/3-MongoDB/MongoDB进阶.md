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

