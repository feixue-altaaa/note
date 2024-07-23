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

## CRUD

