# 总体介绍

Redis 提供了2个不同形式的持久化方式

- RDB（Redis DataBase）
- AOF（Append Of File）

> Redis 持久化是 Redis 和 Memcached 的主要区别之一，因为 Memcached 不具备持久化功能

# RDB（Redis DataBase）

## 定义

+ 在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里

> [磁盘快照](https://baike.baidu.com/item/磁盘快照?fromModule=lemma_inlink)(Snapshot)是针对整个磁盘卷册进行快速的档案系统备份，与其它[备份方式](https://baike.baidu.com/item/备份方式?fromModule=lemma_inlink)最主要的不同点在于「速度」。进行磁盘快照时，并不牵涉到任何档案复制动作。就算数据量再大，一般来说，通常可以在一秒之内完成备份动作
>
> 磁盘快照的基本概念与磁带备份等机制有非常大的不同。在建立磁盘快照时，并不需要复制数据本身，它所作的只是通知LX Series NAS服务器将有数据的磁盘区块全部保留起来，不被覆写。这个通知动作只需花费极短的时间。接下来的档案修改或任何新增、删除动作，均不会覆写原本数据所在的磁盘区块，而是将修改部分写入其它可用的磁盘区块中。所以可以说，数据复制，或者说数据备份，是在平常[档案存取](https://baike.baidu.com/item/档案存取?fromModule=lemma_inlink)时就做好了，而且对效能影响极低。LX Series NAS档案系统内部会建立一份数据结构，纪录[磁盘快照](https://baike.baidu.com/item/磁盘快照?fromModule=lemma_inlink)备份及作用中档案系统所使用到的磁盘区块及指针，让使用者可以同时存取到主要档案系统及过去的磁盘快照版本

## 备份是如何执行的

+ Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能，如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB的缺点是**最后一次持久化后的数据可能丢失

## Fork

- Fork的作用是复制一个与当前进程一样的进程。新进程的所有数据（变量、环境变量、程序计数器等） 数值都和原进程一致，但是是一个全新的进程，并作为原进程的子进程
- 在Linux程序中，fork会产生一个和父进程完全相同的子进程，出于效率考虑，Linux中引入了“**写时复制技术**”
- **一般情况父进程和子进程会共用同一段物理内存**，只有进程空间的各段的内容要发生变化时，才会将父进程的内容复制一份给子进程

# RDB持久化流程



![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132009749.png)

## dump.rdb文件

+ 在redis.conf中配置文件名称，默认为dump.rdb

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011329.png)

## 配置位置

+ rdb文件的保存路径，也可以修改。默认为Redis启动时命令行所在的目录下

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011627.png)

## 如何触发RDB快照、保持策略

### 触发方式

#### 配置文件中默认的快照配置

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011527.png)

#### 自动触发

+ 在 redis.conf 配置文件中的 SNAPSHOTTING 下

> ```bash
> save 3600 1：表示3600 秒内如果至少有 1 个 key 的值变化，则保存
> save 300 10：表示300 秒内如果至少有 10 个 key 的值变化，则保存
> save 60 10000：表示60 秒内如果至少有 10000 个 key 的值变化，则保存
> ```

#### 手动触发

**save**

- save ：save时只管保存，其它不管，**全部阻塞**。手动保存。不建议
- 格式：save 秒钟 写操作次数
- RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件
- 该命令会阻塞当前Redis服务器，执行save命令期间，Redis不能处理其他命令，直到RDB过程完成为止。显然该命令对于内存比较大的实例会造成长时间阻塞，这是致命的缺陷

**bgsave**

+ **bgsave：Redis会在后台异步进行快照操作， 快照同时还可以响应客户端请求**。具体操作是Redis进执行fork操作创建子进程，RDB持久化过程由子进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短
+ 可以通过lastsave 命令获取最后一次成功执行快照的时间

### flushall命令

+ 执行flushall命令，也会产生dump.rdb文件，但里面是空的，无意义

### stop-writes-on-bgsave-error

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011248.png)

+ 当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes

### rdbcompression 压缩文件

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011351.png)

- 对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩
- 如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。推荐yes

### rdbchecksum 检查完整性

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132011470.png)

- 在存储快照后，还可以让redis使用CRC64算法来进行数据校验
- 但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能
- 推荐yes

### rdb的备份

- 先通过config get dir  查询rdb文件的目录 
- 将*.rdb的文件拷贝到别的地方

rdb的恢复

- - - 关闭Redis
    - 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb
    - 启动Redis, 备份数据会直接加载

# 总结

## 优势

- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高更适合使用
- 节省磁盘空间
- 恢复速度快

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132012965.jpeg)

## 劣势

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
- 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能
- 如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改

## 如何停止

```bash
#save后给空值，表示禁用保存策略
#动态停止RDB
redis-cli config set save ""
```

## 图示

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303132012511.jpeg)