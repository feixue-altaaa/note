# 定义

+ 主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，**Master以写为主，Slave以读为主**

# 能干嘛

- **负载均衡**，读写分离，性能扩展
  - 在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量


![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303141604945.jpeg)

+ 数据热备份
  + 主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式
+ **故障快速恢复**
  + 当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复

# 复制流程

- Slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave，以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave，完成同步
- 但是只要是重新连接master，一次完全同步（全量复制)将被自动执行

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303141604576.jpeg)

# 怎么玩：主从复制

- 拷贝多个redis.conf文件include(写绝对路径)
- 开启daemonize yes
- Pid文件名字pidfile
- 指定端口port
- Log文件名字
- dump.rdb名字dbfilename
- Appendonly 关掉或者换名字

## 新建redis6379.conf，填写以下内容

- include /myredis/redis.conf
- pidfile /var/run/redis_6379.pid
- port 6379
- dbfilename dump6379.rdb

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260823686-fed7fb2f-432d-4991-9838-c66f38c31e17.jpeg)

## 新建redis6380.conf，填写以下内容

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260823926-37de7faa-bc2e-455d-a76b-b49fc30c0e7e.jpeg)

## 新建redis6381.conf，填写以下内容

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260824241-b0882de4-5e12-429e-bc88-bfbd98155c81.jpeg)

- slave-priority 10
- 设置从机的优先级，值越小，优先级越高，用于选举主机时使用。默认100

## 启动三台redis服务器

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260824507-ccd8eda2-b350-452c-9e34-4dd46221aff2.jpeg)

## 查看系统进程，看看三台服务器是否启动

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260824723-eba201b4-675b-47b5-918a-870e49ee7838.jpeg)

## 查看三台主机运行情况

- info replication
- 打印主从复制的相关信息

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260824980-578a7e49-6709-4d33-9f78-088b395d793c.jpeg)

## 配从(库)不配主(库) 

```bash
slaveof  <ip><port>
#成为某个实例的从服务器
```

+ 在6380和6381上执行: slaveof 127.0.0.1 6379

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260825213-70992091-3b94-4ca5-bfe9-3fe1dabde4a1.jpeg)

+ 在主机上写，在从机上可以读取数据

+ 在从机上写数据报错

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260825456-303c8c5c-3ac3-4506-aa58-4bca395089a8.jpeg)

- 主机挂掉，重启就行，一切如初
- 从机重启需重设：slaveof   127.0.0.1 6379
- 可以将配置增加到文件中。永久生效

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260825723-e8715df5-5500-417b-811d-99dc34ad4c5a.jpeg)

# 常用策略

## 一主二仆

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260825965-ffdbe38d-cad5-4a91-8252-2d6f71665b3d.jpeg)

### 原理

- **主机不配置，从机使用slaveof声明所属主机**
- **主机如果宕机，重启后自动恢复到之前的状态，不需要再做其他任何修改，再新增加数据，从机可以读到数据**
- **从机如果宕机，再次重启后，不会自动成为原master的从机。需要使用slaveof再次声明所属主机，声明之后可以再次读取数据**
- **主机可写可读，从机只可以读，不可以写**
- **从机使用slaveof声明所属主机，会发送sync到主机，获取主机的rdb文件（从头开始复制），执行，实现数据同步，以后再增数据，使用增量复制完成同步。如果是宕机后再次声明所属主机，则使用全量复制完成同步**
- **主机shutdown后，从机不会上位，当主机重新连接后，主从关系依旧**

## 薪火相传

+ 上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险

- 用 slaveof  <ip><port>
- 中途变更转向：会清除之前的数据，重新建立拷贝最新的
- 风险是一旦某个slave宕机，后面的slave都没法备份
- 主机挂了，从机还是从机，无法写数据了

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260826274-9f7018d3-817b-4b5e-902e-3e59dba0235b.jpeg)

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260826518-2da4a24a-3ae7-4f0c-a0a7-5c5cf3f6396e.jpeg)

## 反客为主

哨兵节点由两部分组成，哨兵节点和数据节点

- 哨兵节点：哨兵系统由一个或多个哨兵节点组成，哨兵节点是特殊的redis节点，不存储数据
- 数据节点：主节点和从节点都是数据节点

访问redis集群的数据都是通过哨兵集群的，哨兵监控整个redis集群

一旦发现redis集群出现了问题，比如刚刚说的主节点挂了，从节点会顶上来。但是主节点地址变了，这时候应用服务无感知，也不用更改访问地址，因为哨兵才是和应用服务做交互的。Sentinel 很好的解决了故障转移

- 当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改
- 用 slaveof  no one  将从机变为主机

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260826763-aa6ee2df-e0c6-47de-8899-65f257054523.jpeg)

# 哨兵模式(sentinel)

## 定义

**反客为主的自动版**，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260827300-3210f157-9125-48d8-9fe3-21abbfe893e2.jpeg)

## 原理

- 哨兵主要用于管理多个 Redis 服务器，主要有以下三个任务：监控、提醒以及故障转移
- 每个哨兵会向其它哨兵、master、slave 定时发送消息，以确认对方是否还存活。如果发现对方在配置的指定时间内未回应，则暂时认为对方已挂。若“哨兵群”中的多数 sentinel 都报告某一 master 没响应，系统才认为该 master “彻底死亡”，通过一定的 vote 算法从剩下的 slave 节点中选一台提升为 master，然后自动修改相关配置

## 怎么玩(使用步骤)

### 调整为一主二仆模式

+ 6379带着6380、6381

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260827636-11eed497-0987-4fa1-b2b9-2bfd412a499b.jpeg)

### 配置文件

+ 自定义的/myredis目录下新建sentinel.conf文件，名字绝不能错

+ 配置哨兵,填写内容

  sentinel monitor mymaster 127.0.0.1 6379 1

  其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量

其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量

### 启动哨兵

- /usr/local/bin
- redis做压测可以用自带的redis-benchmark工具
- 执行redis-sentinel  /myredis/sentinel.conf 

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260827917-1f28506a-dce4-4e73-b4f2-5749f283b77d.jpeg)

### 当主机挂掉，从机选举中产生新的主机

- (大概10秒左右可以看到哨兵窗口日志，切换了新的主机)
- 哪个从机会被选举为主机呢？根据优先级别：slave-priority 
- 原主机重启后会变为从机

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260828282-23881157-3515-463e-b2e8-767db7615cd2.jpeg)

### 复制延时

+ 由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重

## 故障恢复

![img](https://cdn.nlark.com/yuque/0/2022/jpeg/29671373/1659260828571-c2be13b0-f6e1-4bca-893f-0b087e7e19cf.jpeg)

- 优先级在redis.conf中默认：slave-priority 100，值越小优先级越高
- 偏移量是指获得原主机数据最全的
- 每个redis实例启动后都会随机生成一个40位的runid

