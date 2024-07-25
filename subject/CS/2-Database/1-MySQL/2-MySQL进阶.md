# 常用存储引擎

+  存储引擎是[数据库](https://cloud.tencent.com/solution/database?from=10680)的核心，在[MySQL](https://cloud.tencent.com/product/cdb?from=10680)中，存储引擎是以插件的形式运行的。支持的引擎有十几种之多，但我们实战常用到的，大概只有InnoDB、MyISAM 和 Memory

![1674370050093](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303142025301.png)

## MySQL体系结构

![1674370324719](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241715505.png)

### 连接层

+ 最上层是一些客户端和链接服务，包含本地sock 通信和大多数基于客户端/服务端工具实现的类似于TCP/IP的通信。主要完成一些类似于连接处理、授权认证、及相关的安全方案。在该层上引入了线程池的概念，为通过认证安全接入的客户端提供线程。同样在该层上可以实现基于SSL的安全链接。服务器也会为安全接入的每个客户端验证它所具有的操作权限

### 服务层

+ 第二层架构主要完成大多数的核心服务功能，如SQL接口，并完成缓存的查询，SQL的分析和优化，部分内置函数的执行。所有跨存储引擎的功能也在这一层实现，如过程、函数等。在该层，服务器会解析查询并创建相应的内部解析树，并对其完成相应的优化如确定表的查询的顺序，是否利用索引等，最后生成相应的执行操作。如果是select语句，服务器还会查询内部的缓存，如果缓存空间足够大，这样在解决大量读操作的环境中能够很好的提升系统的性能

### 引擎层

+ 存储引擎层， 存储引擎真正的负责了MySQL中数据的存储和提取，服务器通过API和存储引擎进行通信。不同的存储引擎具有不同的功能，这样我们可以根据自己的需要，来选取合适的存储引擎。数据库中的索引是在存储引擎层实现的

### 存储层

+ 数据存储层， 主要是将数据(如: redolog、undolog、数据、索引、二进制日志、错误日志、查询日志、慢查询日志等)存储在文件系统之上，并完成与存储引擎的交互

和其他数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎上，插件式的存储引擎架构，将查询处理和其他的系统任务以及数据的存储提取分离。这种架构可以根据业务的需求和实际需要选择合适的存储引擎

## 存储引擎介绍

+ 存储引擎就是**存储数据、建立索引、更新/查询数据**等技术的实现方式 。存储引擎是**基于表**的，而不是基于库的，所以存储引擎也可被称为表类型。我们可以在创建表的时候，来指定选择的存储引擎，如果没有指定将自动选择默认的存储引擎

```bash
-- 建表时指定存储引擎
CREATE TABLE 表名(
字段1 字段1类型 [ COMMENT 字段1注释 ] ,
......
字段n 字段n类型 [COMMENT 字段n注释 ]
) ENGINE = INNODB [ COMMENT 表注释 ] ;

-- 查询当前数据库支持的存储引擎
show engines;
```

## 存储引擎特点

### InnoDB

#### What

+ InnoDB是一种兼顾**高可靠性和高性能**的通用存储引擎，在 MySQL 5.5 之后，InnoDB是默认的MySQL 存储引擎

#### 特点

- DML操作遵循ACID模型，支持事务；
- 行级锁，提高并发访问性能；
- 支持外键FOREIGN KEY约束，保证数据的完整性和一致性；

> # 外键约束
>
> ## What
>
> - 外键约束（Foreign key）是关系型数据库中的 Table 的一个特殊字段，经常与主键约束（Primary key）一起使用。对于两个具有关联关系的表而言，相关联字段中主键所在的表就是主表（父表），外键所在的表就是从表（子表）。外键约束可以保证引用的完整性（Referential Integrity）
>
> - 引用完整性是数据的属性，如果数据拥有该属性，那么数据中所有的引用都是合法的，在关系型数据库的上下文中，这就意味着关系型数据库中引用另一个表中的值必须存在
>
> - 简而言之，外键约束就是用来建立主表与从表的关联关系，为两个表的数据建立连接，约束两个表中数据的一致性和完整性
>
> - NOTE：一个 Table 可以有一个或多个外键，外键可以为空值，若不为空值，则每一个外键的值必须等于主表中主键的某个值
>
> ## 外键关联
>
> - 所谓外键关联，即：B 存在外键 b_f_k，以 A 表的 a_k 作为参照（References）列，则 A 为主表，B 为从表
> - 若 A、B 关联了 on delete/update 等操作，则 A 中某记录的更新或删除会联动着 B 中外键与其关联对应的记录做更新或删除操作
>
> - 反之，B 怎样变 A 不必跟随变动，且 A 中必须事先存在 B 要插入的数据外键列的值，例如：B.bfk 作为外键参照 A.ak，则 B.bfk 插入的值必须是 A.ak 中已存在的。简而言之，就是若 B 有以 A 作为参照的外键，则 B 中的此字段的取值只能是 A 中存在的值
>
> ## 外键的作用
>
> - 外键用于支持关系型数据库的 “参照完整性”，外键具有保持数据完整性和一致性的机制，对业务处理有着很好的校验作用
> - 举例说明：假设 Table user 的 Column user.id 为主键（Primary key），Table profile 的 Column profile.uid 为主键。以 user 为主表、profile 为关联表、profile.uid 为外键（Foreign key）并将 user.id 作为参考（References），且联动了删除/更新操作（on delete/update cascade）。那么：
>
> - 在 user 中删除 id 为 1 的记录，会联动删除 profile 中 uid 为 1 的记录
> - 在 user 中更新 id 为 1 的记录至 id 为 2，则 profile 中 uid 为 1 的记录也会被联动更新至 uid 为 2
> - 这样即保持了数据的完整性，也保证了数据的一致性。而且这个工作都是交由 RDBMS 内部实现的触发器来完成的，不需要额外的编码
>
> ## 外键的性能问题
>
> - 外键的使用往往会带来性能问题，因为
> - 数据库需要维护外键的内部管理；
> - 外键等于把数据的一致性事务实现，全部交给数据库服务器完成；
> - 涉及外键字段的增，删，更新操作，需要触发相关操作去检查，而不得不消耗资源；
> - 外键还会因为需要请求对其他表内部加锁而容易出现死锁情况

#### 文件

- xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的**表结构（frm-早期的 、sdi-新版的）、数据和索引**
- 参数：innodb_file_per_table

```bash
-- 查看
show variables like 'innodb_1 file_per_table';
```

- 如果该参数开启，代表对于InnoDB引擎的表，每一张表都对应一个ibd文件。 我们直接打开MySQL的数据存放目录： C:\ProgramData\MySQL\MySQL Server 8.0\Data ， 这个目录下有很多文件夹，不同的文件夹代表不同的数据库，我们直接打开itcast文件夹

  ![1674371923347](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241715108.png)

- 可以看到里面有很多的ibd文件，每一个ibd文件就对应一张表，比如：我们有一张表 account，就有这样的一个account.ibd文件，而在这个ibd文件中不仅存放表结构、数据，还会存放该表对应的索引信息。 而该文件是基于二进制存储的，不能直接基于记事本打开，我们可以使用mysql提供的一个指令 ibd2sdi ，通过该指令就可以从ibd文件中提取sdi信息，而sdi数据字典信息中就包含该表的表结构

#### 逻辑存储结构

![1674371950846](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241715369.png)

- 表空间 : InnoDB存储引擎逻辑结构的最高层，ibd文件其实就是表空间文件，在表空间中可以包含多个Segment段
- 段 : 表空间是由各个段组成的， 常见的段有**数据段、索引段、回滚段**等。InnoDB中对于段的管理，都是引擎自身完成，不需要人为对其控制，一个段中包含多个区
  - 为什么要引入段呢，这要从索引说起。我们都知道索引的目的是为了加快查找速度，是一种典型的用空间换时间的方法
  - B+ 树的叶子节点存放的是我们的具体数据，非叶子结点是索引页。所以 B+ 树将数据分为了两部分，叶子节点部分和非叶子节点部分，也就我们要介绍的段 Segment，也就是说 InnoBD 中每一个索引都会创建两个 Segment 来存放对应的两部分数据

- 区 : 区是表空间的单元结构，**每个区的大小为1M**。 默认情况下， InnoDB存储引擎页大小为16K， 即一个区中一共有**64个连续的页**
  - 如果只有页这一个层次的话，页的个数是非常多的，存储空间的分配和回收都会很麻烦，因为要维护这么多的页的状态是非常麻烦的所以，InnoDB 又引入了区（Extent) 的概念。一个区默认是 64 个连续的页组成的，也就是 1MB。通过 Extent 对存储空间的分配和回收就比较容易

- 页 : 页是组成区的最小单元，**页也是InnoDB 存储引擎磁盘管理的最小单元**，**每个页的大小默认为 16KB**。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区
- 行 : InnoDB 存储引擎是面向行的，也就是说数据是按行进行存放的，在每一行中除了定义表时所指定的字段以外，还包含两个隐藏字段

### MyISAM

#### What

+ MyISAM是MySQL早期的默认存储引擎

#### 特点

- 不支持事务，不支持外键
- 支持表锁，不支持行锁
- 访问速度快

> **为什么MyISAM查询性能比Innodb快**
>
> ![img](https://mmbiz.qpic.cn/sz_mmbiz_png/tJJDa2wmJiaqJOuaia7EA8nH3QMxjEsqiavWORTuOlhQnwTHNcoFebic8xglH39EGfRfQSY0NY56iaR2ziabib9Vsqe1Q/640?wx_fmt=png)
>
> 通过上面表格对比， **InnoDB在做SELECT的时候，要维护的东西比MYISAM引擎多很多，**影响查询速度有
>
> - 数据块，InnoDB要缓存，MyISAM只缓存索引块， 这中间还有换进换出的减少
> - InnoDB寻址要映射到块，再到行，MyISAM记录的直接是文件的OFFSET，定位比InnoDB要快
> - InnoDB还需要维护MVCC一致； 虽然你的场景没有，但他还是需要去检查和维护

#### 文件

- xxx.sdi：存储表结构信息
- xxx.MYD: 存储数据
- xxx.MYI: 存储索引

![1674372880753](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241716939.png)

### Memory

#### 介绍

+ Memory引擎的表数据是**存储在内存**中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用

#### 特点

+ 内存存放

+ hash索引（默认）

#### 文件

+ xxx.sdi：存储表结构信息

> 面试题:
> InnoDB引擎与MyISAM引擎的区别 ?
> ①. InnoDB引擎, 支持事务, 而MyISAM不支持。
> ②. InnoDB引擎, 支持行锁和表锁, 而MyISAM仅支持表锁, 不支持行锁。
> ③. InnoDB引擎, 支持外键, 而MyISAM是不支持的。
> 主要是上述三点区别，当然也可以从索引结构、存储限制等方面，更加深入的回答，具体参
> 考如下官方文档：
> https://dev.mysql.com/doc/refman/8.0/en/innodb-introduction.html
> https://dev.mysql.com/doc/refman/8.0/en/myisam-storage-engine.html

### 存储引擎选择

- 在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合
- InnoDB: 是Mysql的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择
- MyISAM ： 如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那么选择这个存储引擎是非常合适的
- MEMORY：将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORY的缺陷就是对表的大小有限制，太大的表无法缓存在内存中，而且无法保障数据的安全性

| 存储引擎 | 特点                                                         |
| :------: | :----------------------------------------------------------- |
|  InnoDb  | 支持事务； 支持表锁、行级锁，提高并发访问性能； 支持外键FOREIGN KEY约束，保证数据的完整性和一致性； |
|  MyISAM  | 不支持事务，不支持外键 支持表锁，不支持行锁 访问速度快       |
|  Memory  | Memory引擎的表数据是**存储在内存**中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用；hash索引（默认） |

# 索引

## What

+ 索引（index）是帮助MySQL高效获取数据的**数据结构**(有序)。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据， 这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引

![image-20230124182522418](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301241825474.png)



## Why

+ 加快查询速度（无索引时为全表扫描）

### 演示

+ 表结构及其数据如下

![image-20230125122904412](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251229460.png)

+ 假如我们要执行的SQL语句为 ： select * from user where age = 45;

#### 无索引情况

![image-20230125123010923](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251230985.png)

+ 在无索引情况下，就需要从第一行开始扫描，一直扫描到最后一行，我们称之为 全表扫描，性能很低

#### 有索引情况

+ 如果我们针对于这张表建立了索引，假设索引结构就是二叉树，那么也就意味着，会对age这个字段建
  立一个二叉树的索引结构

![image-20230125123146547](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251231604.png)

+ 此时我们在进行查询时，只需要扫描三次就可以找到数据了，极大的提高的查询的效率

> 备注： 这里我们只是假设索引的结构是二叉树，介绍一下索引的大概原理，只是一个示意图，并不是索引的真实结构

### 优劣

|                            优势                             |                             劣势                             |
| :---------------------------------------------------------: | :----------------------------------------------------------: |
|           提高数据检索的效率，降低数据库的IO成本            |                    索引列也是要占用空间的                    |
| 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗 | 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低 |

## 索引结构

### What

+ MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的索引结构，主要包含以下几种

|      索引结构       |                             描述                             |
| :-----------------: | :----------------------------------------------------------: |
|     B+Tree索引      |         最常见的索引类型，大部分引擎都支持 B+ 树索引         |
|      Hash索引       | 底层数据结构是用哈希表实现的, 只有精确匹配索引列的查询才有效, 不支持范围查询 |
|  R-tree(空间索引)   | 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少 |
| Full-text(全文索引) | 是一种通过建立倒排索引,快速匹配文档的方式。类似于Lucene,Solr,ES |

> 全文索引是一种用于文本搜索的技术，它可以在大量文档中快速查找特定的关键词或短语。全文索引会对文档中的所有单词进行索引，而不仅仅是文档标题或标签等元数据。它的工作原理是将每个文档转换成一系列单词，然后将这些单词存储在一个专门的索引中。当用户输入一个查询时，全文索引可以快速地在索引中查找匹配的单词，并返回相关的文档列表。全文索引常用于搜索引擎、数据库和文本编辑器等应用程序中

+ 上述是MySQL中所支持的所有的索引结构，接下来，我们再来看看不同的存储引擎对于索引结构的支持情况

|    索引    |     InnoDE      | MyISAM | Memory |
| :--------: | :-------------: | :----: | :----: |
| B+tree素引 |      支持       |  支持  |  支持  |
|  Hash索引  |     不支持      | 不支持 |  支持  |
| R-tree索引 |     不支持      |  支持  | 不支持 |
| Full-text  | 5.6版本之后支持 |  支持  | 不支持 |

+ 注意： 我们平常所说的索引，如果没有特别指明，都是指B+树结构组织的索引

### 二叉树
+ 假如说MySQL的索引结构采用二叉树的数据结构，比较理想的结构如下

![image-20230125124611336](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251246383.png)

+ 如果主键是顺序插入的，则会形成一个单向链表，结构如下

![image-20230125125058193](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251250237.png)

+ 所以，如果选择二叉树作为索引结构，会存在以下缺点

  - 顺序插入时，会形成一个链表，查询性能大大降低


  - 大数据量情况下，层级较深，检索速度慢


### 红黑树

+ 红黑树是一颗自平衡二叉树，那这样即使是顺序插入数据，最终形成的数据结构也是一颗平衡的二叉树,结构如下

![image-20230125125350475](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251253518.png)

- 但是，即使如此，由于红黑树也是一颗二叉树，所以也会存在一个缺点：**大数据量情况下，层级较深，检索速度慢**
- 所以，在MySQL的索引结构中，并没有选择二叉树或者红黑树，而选择的是B+Tree

> #### 红黑树定义和性质
>
> 红黑树是一种含有红黑结点并能自平衡的二叉查找树。它必须满足下面性质：
>
> - 性质1：每个节点要么是黑色，要么是红色
> - 性质2：根节点是黑色
> - 性质3：每个叶子节点（NIL）是黑色
> - 性质4：每个红色结点的两个子结点一定都是黑色
> - **性质5：任意一结点到每个叶子结点的路径都包含数量相同的黑结点**

### B-Tree

+ B-Tree，B树是一种多路平衡查找树，相对于二叉树，B树每个节点可以有多个分支，即多叉。以一颗最大度数（max-degree）为5(5阶)的b-tree为例，那这个B树每个节点最多存储4个key，5个指针

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251255520.jpg)

> 知识小贴士: 树的度数指的是一个节点的子节点个数

#### 特点

- 5阶的B树，每一个节点最多存储4个key，对应5个指针
- 一旦节点存储的key数量到达5，就会裂变，中间元素向上分裂
- 在B树中，非叶子节点和叶子节点都会存放数据

### B+Tree

+ B+Tree是B-Tree的变种，我们以一颗最大度数（max-degree）为4（4阶）的b+tree为例，来看一下其结构示意图

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251257962.jpg)

- 我们可以看到，两部分
- 绿色框框起来的部分，是索引部分，仅仅起到索引数据的作用，不存储数据
- 红色框框起来的部分，是数据存储部分，在其叶子节点中要存储具体的数据

**最终我们看到，B+Tree 与 B-Tree相比，主要有以下三点区别**

- 所有的数据都会出现在叶子节点
- 叶子节点形成一个单向链表
- 非叶子节点仅仅起到索引数据作用，具体的数据都是在叶子节点存放的

**MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能，利于排序**

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251301517.jpg)

**B+树的优点**

- 
  小树高：B+ 树每个节点中只包含键值信息，内部节点不保存数据记录，这样每个节点就可以保存更多的键值，使得树的高度比较小，从而能够快速地定位到指定记录

- 
  批量读写：InnoDB 存储引擎利用 B+ 树索引的性质，使用批量读写方式提高数据表的读写效率。在读取数据时，直接预读取下一个节点的数据块，使得实际访问 I/O 次数大大减少，提高了读取的效率

- 
  范围查询：B+ 树的叶子节点之间形成链表，可以对关键字进行排序，方便范围查询。由于叶子节点保存了所有数据记录，所以在查找一连串符合条件的记录时，可以顺着链表顺序把记录输出，不用跳来跳去地查找

- 
  可预测性：B+ 树的节点大小比较稳定，因此树的层数可以预估，从而可以有效地分配磁盘空间，避免频繁的磁盘空间分配和回收，提高了数据表管理的效率

- 
  插入删除优化：B+ 树的内部指针通常比叶子节点指针多，对于插入和删除操作，都是对一个局部区域进行操作，不会对整棵树产生大规模扰动，可以利用局部性原理，将树的状态缓存在缓存中，避免频繁地从磁盘读写节点信息，提高数据表更新的效率

### B-树和B+树对比

#### B-树

+ B-Tree中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个3阶的B-Tree

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251318988.png)

+ **每个节点占用一个盘块的磁盘空间**，一个节点上有`两个升序排序的关键字`和`三个指向子树根节点的指针`，指针存储的是子节点所在磁盘块的地址。两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域。以根节点为例，关键字为17和35，P1指针指向的子树的数据范围为小于17，P2指针指向的子树的数据范围为17~35，P3指针指向的子树的数据范围为大于35

**模拟查找关键字29的过程：**

> 根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
> 比较关键字29在区间（17,35），找到磁盘块1的指针P2。
> 根据P2指针找到磁盘块3，读入内存。【磁盘I/O操作第2次】
> 比较关键字29在区间（26,30），找到磁盘块3的指针P2。
> 根据P2指针找到磁盘块8，读入内存。【磁盘I/O操作第3次】
> 在磁盘块8中的关键字列表中找到关键字29

+ 分析上面过程，发现需要`3次磁盘I/O`操作，和`3次内存查找`操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而3次磁盘I/O操作是影响整个B-Tree查找效率的决定因素

#### B+树

+ 从**B-Tree**结构图中可以看到每个节点中`不仅包含数据的key值`，还有`data值`。而每一个页的存储空间是有限的，如果data数据较大时将会导致每个节点（即一个页）能存储的key的数量很小，当存储的数据量很大时同样会导致B-Tree的深度较大，增大查询时的磁盘I/O次数，进而影响查询效率。在**B+Tree**中，所有`数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上，而非叶子节点上只存储key值信息`，这样可以大大加大每个节点存储的key值数量，降低B+Tree的高度

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251338842.png)

> InnoDB存储引擎中页的大小为16KB，一般表的主键类型为INT（占用4个字节）或BIGINT（占用8个字节）-----这里使用BIGINT来计算，指针类型也一般为6个字节，也就是说根节点一个页中大概存储16KB/(8B+6B)=1170个键值，而叶子节点的一个页（因为InnoDB存储引擎叶子节点还要存储数据）大概存储16KB/1KB（假设为1kb --索引+数据）= 16。也就是说一个深度为3的B+Tree索引可以维护1170 * 1170 * 16 = 2000多万条记录。所以当Mysql数据个数为2000多万（反正数据很大时）查询性能会急剧下降

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251343644.png)

+ 实际情况中每个节点可能不能填充满，因此在数据库中，B+Tree的高度一般都在2 ~ 4层。MySQL的InnoDB存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要1~3次磁盘I/O操作

### Hash

+ MySQL中除了支持B+Tree索引，还支持一种索引类型---Hash索引

#### 结构

+ 哈希索引就是采用一定的hash算法，将键值换算成新的hash值，映射到对应的槽位上，然后存储在hash表中

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251347354.jpg)

+ 如果两个(或多个)键值，映射到一个相同的槽位上，他们就产生了hash冲突（也称为hash碰撞），可以通过链表来解决

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251348015.jpg)

#### 特点

- Hash索引只能用于对等比较(=，in)，不支持范围查询（between，>，< ，...）
- 无法利用索引完成排序操作
- 查询效率高，通常(不存在hash冲突的情况)只需要一次检索就可以了，效率通常要高于B+tree索引

#### 存储引擎支持

+ 在MySQL中，支持hash索引的是Memory存储引擎。而InnoDB中具有**自适应hash功能**，hash索引是InnoDB存储引擎根据B+Tree索引在指定条件下自动构建的

> 思考题： 为什么InnoDB存储引擎选择使用B+tree索引结构?
>
> 相对于二叉树，层级更少，搜索效率高；
>
> 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
>
> 相对Hash索引，B+tree支持范围匹配及排序操作

## 索引分类

### 索引分类

+ 在MySQL数据库，将索引的具体类型主要分为以下几类：主键索引、唯一索引、常规索引、全文索引

|   分类   |                        含义                         |           特点           |  关键字  |
| :------: | :-------------------------------------------------: | :----------------------: | :------: |
| 主键索引 |              针对于表中主键创建的索引               | 默认自动创建，只能有一个 | PRIMARY  |
| 唯一索引 |          避免同一个表中某数据列中的值重复           |        可以有多个        |  UNIQUE  |
| 常规索引 |                  快速定位特定数据                   |        可以有多个        |          |
| 全文索引 | 全文索引查找的是文本中的关键词,而不是比较索引中的值 |        可以有多个        | FULLTEXT |

**innodb的表必须指定主键吗**

不是，**推荐使用整型的自增主键**

如果设置了主键，那么InnoDB会选择主键作为聚集索引、如果没有显式定义主键，则InnoDB会选择第一个不包含有NULL值的唯一索引作为主键索引、如果也没有这样的唯一索引，则InnoDB会选择内置6字节长的ROWID作为隐含的聚集索引(ROWID随着行记录的写入而主键递增)

如果表使用自增主键，那么每次插入新的记录，记录就会顺序添加到当前索引节点的后续位置，主键的顺序按照数据记录的插入顺序排列，自动有序。当一页写满，就会自动开辟一个新的页，减少了页分裂

如果使用非自增主键（如果身份证号或学号等），由于每次插入主键的值近似于随机，因此每次新纪录都要被插到现有索引页得中间某个位置，此时MySQL不得不为了将新记录插到合适位置而移动数据，甚至目标页面可能已经被回写到磁盘上而从缓存中清掉，此时又要从磁盘上读回来，这增加了很多开销，同时频繁的移动、分页操作造成了大量的碎片，得到了不够紧凑的索引结构，后续不得不通过OPTIMIZE TABLE来重建表并优化填充页面

### 聚集索引&二级索引

+ 而在在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种

| 分类                      |                           含义                            |        特点         |
| :------------------------ | :-------------------------------------------------------: | :-----------------: |
| 聚集索引(Clustered Index) | 将数据存储与索引放到了一块,索引结构的叶子节点保存了行数据 | 必须有,而且只有一个 |
| 二级索引(secondary ndex)  | 将数据与索引分开存储,索引结构的叶子节点关联的是对应的主键 |    可以存在多个     |

> 聚集索引（Clustered Index）和二级索引（Secondary Index，也称为非聚集索引 Non-Clustered Index）都是数据库中用于加速数据检索的索引类型。它们有一些共同点，但在实现原理和使用场景上存在一些重要的区别。下面是它们的异同
>
> **相同点**
>
> - 都是索引：聚集索引和二级索引都是数据库中用于加速数据检索的数据结构
> - 提高查询速度：两种索引都可以显著提高数据查询速度，避免全表扫描
> - 逻辑顺序：它们都可以根据索引键值的逻辑顺序对数据进行排序
>
> **不同点**
>
> - 存储数据方式：聚集索引将数据行存储在索引中，按照索引键值的顺序进行排序。数据行只存在于聚集索引中，不存在于其他地方。而二级索引存储的是指向实际数据行的指针（如聚集索引键值或行ID），而不是数据行本身
> - 索引数量：一个表只能有一个聚集索引，因为数据行本身只能按照一种顺序进行存储。但是，一个表可以有多个二级索引
> - 效率：聚集索引的查询效率通常比二级索引高，因为在聚集索引中，索引和数据行是在一起的，无需额外的数据查找。而在二级索引中，查询需要执行两个步骤：首先通过索引找到对应的聚集索引键值或行ID，然后再通过这个键值或行ID找到实际的数据行
> - 范围查询：对于范围查询，聚集索引的效果更好，因为它们按顺序存储数据行，可以更高效地执行范围扫描。而二级索引可能需要多次查找实际数据行，对于范围查询效率较低
> - 更新开销：聚集索引的更新开销通常较大，因为它需要维护数据行的物理顺序。而在二级索引中，仅需要更新索引及指针，开销较小
>
> 总之，聚集索引和二级索引都是数据库中用于提高数据查询速度的重要数据结构。聚集索引将数据行存储在索引中，按照索引键值的顺序进行排序。而二级索引存储的是指向实际数据行的指针。一个表只能有一个聚集索引，但可以有多个二级索引。在选择索引类型时，需要根据实际应用场景和查询需求来权衡

- 聚集索引选取规则
  - 如果存在主键，主键索引就是聚集索引
  - 如果不存在主键，将使用第一个唯一（UNIQUE）索引作为聚集索引
  - 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引

**聚集索引和二级索引的具体结构如下**

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251402465.jpg)

- 聚集索引的叶子节点下挂的是这一行的数据 
- 二级索引的叶子节点下挂的是该字段值对应的主键值

**接下来，我们来分析一下，当我们执行如下的SQL语句时，具体的查找过程是什么样子的**

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251413283.jpg)

- 具体过程如下
  - 由于是根据name字段进行查询，所以先根据name='Arm'到name字段的二级索引中进行匹配查找。但是在二级索引中只能查找到 Arm 对应的主键值 10
  - 由于查询返回的数据是*，所以此时，还需要根据主键值10，到聚集索引中查找10对应的记录，最终找到10对应的行row
  - 最终拿到这一行的数据，直接返回即可

> 回表查询： 这种先到二级索引中查找数据，找到主键值，然后再到聚集索引中根据主键值，获取数据的方式，就称之为回表查询

> 思考题：
>
> 以下两条SQL语句，那个执行效率高? 为什么?
>
> A. select * from user where id = 10 ;
>
> B. select * from user where name = 'Arm' ;
>
> 备注: id为主键，name字段创建的有索引；
>
> 解答：
>
> A 语句的执行性能要高于B 语句
>
> 因为A语句直接走聚集索引，直接返回数据。 而B语句需要先查询name字段的二级索引，然后再查询聚集索引，也就是需要进行回表查询

> 思考题
>
> InnoDB主键索引的B+tree高度为多高呢?
>
> ![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251417171.jpg)
>
> 假设
>
> 一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6个字节的空间，主键即使为bigint，占用字节数为8
>
> 高度为2：
>
> n * 8 + (n + 1) * 6 = 16 * *1024 , 算出n约为 1170*
>
> 1171 *  16 = 18736
>
> 也就是说，如果树的高度为2，则可以存储 18000 多条记录
>
> 高度为3：
>
> 1171 * 1171 * 16 = 21939856
>
> 也就是说，如果树的高度为3，则可以存储 2200w 左右的记录

### 索引语法

```bash
#创建索引
CREATE [ UNIQUE | FULLTEXT ] INDEX index_name ON table_name (index_col_name,... ) ;
#查看索引
SHOW INDEX 1 FROM table_name ;
#删除索引
DROP INDEX index_name ON table_name ;
```

## 索引使用

### 最左前缀法则

+ 如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。如果跳跃某一列，索引将会部分失效(后面的字段索引失效)。以 tb_user 表为例，我们先来查看一下之前 tb_user 表所创建的索引

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251534797.jpg)

- 在 tb_user 表中，有一个联合索引，这个联合索引涉及到三个字段，顺序分别为：profession，age，status
- 对于最左前缀法则指的是，查询时，最左变的列，也就是profession必须存在，否则索引全部失效。而且中间不能跳过某一列，否则该列后面的字段索引将失效

> 注意 ： 最左前缀法则中指的最左边的列，是指在查询时，联合索引的最左边的字段(即是第一个字段)必须存在，与我们编写SQL时，条件编写的先后顺序无关

### 范围查询

+ 联合索引中，出现范围查询(>,<)，范围查询右侧的列索引失效

- 当范围查询使用>= 或 <= 时，走联合索引了，但是索引的长度为54，就说明所有的字段都是走索引的
- 所以，在业务允许的情况下，尽可能的使用类似于 >= 或 <= 这类的范围查询，而避免使用 > 或 <

### 索引失效情况

#### 索引列运算

+ 不要在索引列上进行运算操作， 索引将失效

```bash
#索引失效
explain select * from tb_user where substring(phone,10,2) = '15';
```

#### 字符串不加引号

+ 字符串类型字段使用时，不加引号，索引将失效

#### 模糊查询

+ 如果仅仅是**尾部模糊匹配，索引不会失效**。如果是头部模糊匹配，索引失效
+ 接下来，我们来看一下这三条SQL语句的执行效果，查看一下其执行计划：由于下面查询语句中，都是根据profession字段查询，符合最左前缀法则，联合索引是可以生效的，我们主要看一下，模糊查询时，%加在关键字之前，和加在关键字之后的影响

```bash
explain select * from tb_user where profession like '软件%';
#失效
explain select * from tb_user where profession like '%工程';
#失效
explain select * from tb_user where profession like '%工%';
```

#### or连接条件

+ 用or分割开的条件， 如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到

+ **当or连接的条件，左右两侧字段都有索引时，索引才会生效**

#### 数据分布影响

+ 如果MySQL评估使用索引比全表更慢，则不使用索引

#### is null 与 is not null

+ 查询时MySQL会评估，走索引快，还是全表扫描快，如果全表扫描更快，则放弃索引走全表扫描。 因此，is null 、is not null是否走索引，得具体情况具体分析，并不是固定的

### SQL提示
+ SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的

```bash
#use index ： 建议MySQL使用哪一个索引完成此次查询（仅仅是建议，mysql内部还会再次进行评估）
explain select * from tb_user use index(idx_user_pro) where profession = '软件工程';

#ignore index ： 忽略指定的索引
explain select * from tb_user ignore index(idx_user_pro) where profession = '软件工程';

#force index ： 强制使用索引
explain select * from tb_user force index(idx_user_pro) where profession = '软件工程';
```

### 覆盖索引

+ 尽量使用覆盖索引，减少select *。 覆盖索引是指查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到

|          Extra           |                             含义                             |
| :----------------------: | :----------------------------------------------------------: |
| Using where; Using Index | 查找使用了索引,但是需要的数据都在索引列中能找到,所以不需要回表查询数据 |
|  Using index condition   |             查找使用了索引,但是需要回表查询数据              |

+ 表结构及索引示意图

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251557024.jpg)

- id是主键，是一个聚集索引。 name字段建立了普通索引，是一个二级索引（辅助索引）
- 执行SQL : select * from tb_user where id = 2;

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251558783.jpg)

+ 根据id查询，直接走聚集索引查询，一次索引扫描，直接返回数据，性能高

+ 执行SQL：selet id,name from tb_user where name = 'Arm';

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251600170.jpg)

+ 虽然是根据name字段查询，查询二级索引，但是由于查询返回字段为 id，name，在name的二级索引中，这两个值都是可以直接获取到的，因为覆盖索引，所以不需要回表查询，性能高

+ 执行SQL：selet id,name,gender from tb_user where name = 'Arm';

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251601008.jpg)

+ 由于在name的二级索引中，不包含gender，所以，需要两次索引扫描，也就是需要回表查询，性能相对较差一点

> 思考题：
>
> 一张表, 有四个字段(id, username, password, status), 由于数据量大, 需要对以下SQL语句进行优化, 该如何进行才是最优方案:
>
> select id,username,password from tb_user where username ='itcast';
>
> 答案: 针对于 username, password建立联合索引, sql为: 
>
> create index idx_user_name_pass on tb_user(username,password);
>
> 这样可以避免上述的SQL语句，在查询的过程中，出现回表查询

### 前缀索引

+ 当字段类型为字符串（varchar，text，longtext等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO， 影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率

+ 前缀索引**说白了就是对文本的前几个字符建立索引（具体是几个字符在建立索引时指定）**，这样建立起来的索引更小，所以查询更快。这有点类似于 Oracle 中对字段使用 Left 函数来建立函数索引，只不过 MySQL 的这个前缀索引在查询时是内部自动完成匹配的，并不需要使用 Left 函数

+ 那么为什么不对整个字段建立索引呢？一般来说使用前缀索引，可能都是因为整个字段的数据量太大，没有必要针对整个字段建立索引，前缀索引仅仅是选择一个字段的部分字符作为索引，这样一方面可以节约索引空间，另一方面则可以提高索引效率，当然很明显，这种方式也会降低索引的选择性

> 关于索引的选择性（Index Selectivity），它是指不重复的索引值（也称为基数 cardinality)和数据表的记录总数的比值，取值范围在 `[0,1]` 之间。索引的选择性越高则查询效率越高，因为选择性高的索引可以让 MySQL 在查找时过滤掉更多的行
>
> 是不是选择性越高的索引越好呢？当然不是！索引选择性最高为 1，如果索引选择性为 1，就是唯一索引了，搜索的时候就能直接通过搜索条件定位到具体一行记录！这个时候虽然性能最好，但是也是最费空间的，**这不符合我们创建前缀索引的初衷**
>
> 我们一开始之所以要创建前缀索引而不是唯一索引，**就是希望能够在索引的性能和空间之间找到一个平衡**，我们希望能够选择足够长的前缀以保证较高的选择性（这样在查询的过程中就不需要扫描很多行），但是又希望索引不要太过于占用存储空间
>
> 那么我们该如何选择一个合适的索引选择性呢？索引前缀应该足够长，以便前缀索引的选择性接近于索引的整个列，即前缀的基数应该接近于完整列的基数

```bash
create index idx_xxxx on table_1 name(column(n)) ;
```

#### 前缀长度

可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的

```bash
#首先我们可以通过如下 SQL 得到全列选择性
SELECT COUNT(DISTINCT column_name) / COUNT(*) FROM table_name;

#然后再通过如下 SQL 得到某一长度前缀的选择性
SELECT COUNT(DISTINCT LEFT(column_name, prefix_length)) / COUNT(*) FROM table_name;
```

#### 前缀索引的查询流程

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251618244.jpg)

### 单列索引与联合索引

- 单列索引：即一个索引只包含单个列
- 联合索引：即一个索引包含了多个列

> 在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引

+ 如果查询使用的是联合索引，具体的结构示意图如下

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251620830.jpg)

## 索引设计原则

- 针对于数据量较大，且查询比较频繁的表建立索引
- 针对于常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引
- 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高
- 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引
- 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率
- 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率

## 索引下推（ICP）

**why**

+ 减少回表查询次数，提高查询效率

**what**

+ 索引下推(Index Condition Pushdown，简称ICP)，是MySQL5.6版本的新特性

**索引下推优化的原理**

MySQL大概的架构

![MySQL大概架构](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281611187.webp)

`索引下推`的**下推**其实就是指将部分上层（服务层）负责的事情，交给了下层（引擎层）去处理

**在没有使用ICP的情况下，MySQL的查询**

- 存储引擎读取索引记录；
- 根据索引中的主键值，定位并读取完整的行记录；
- 存储引擎把记录交给`Server`层去检测该记录是否满足`WHERE`条件

**使用ICP的情况下，查询过程**

- 存储引擎读取索引记录（不是完整的行记录）；
- 判断`WHERE`条件部分能否用索引中的列来做检查，条件不满足，则处理下一行索引记录；
- 条件满足，使用索引中的主键去定位并读取完整的行记录（就是所谓的回表）；
- 存储引擎把记录交给`Server`层，`Server`层检测该记录是否满足`WHERE`条件的其余部分

### 实例

```sql
select * from tuser where name like '张%' and age=10;
```

+ 假如你了解索引最左匹配原则，那么就知道这个语句在搜索索引树的时候，只能用 `张`，找到的第一个满足条件的记录id为1

![B+树联合索引](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281616153.webp)

**没有使用ICP**

+ 在MySQL 5.6之前，存储引擎根据通过联合索引找到`name like '张%'` 的主键id（1、4），逐一进行回表扫描，去聚簇索引找到完整的行记录，server层再对数据根据`age=10进行筛选`

![未使用ICP](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281616293.webp)

可以看到需要回表两次，把我们联合索引的另一个字段`age`浪费了。

**使用ICP**

+ 而MySQL 5.6 以后， 存储引擎根据（name，age）联合索引，找到`name like '张%'`，由于联合索引中包含`age`列，所以存储引擎直接再联合索引里按照`age=10`过滤。按照过滤后的数据再一一进行回表扫描

![使用ICP的示意图](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281617640.webp)

- 可以看到只回表了一次。
- 除此之外我们还可以看一下执行计划，看到`Extra`一列里` Using index condition`，这就是用到了索引下推

```sql
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-----------------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | tuser | NULL       | range | na_index      | na_index | 102     | NULL |    2 |    25.00 | Using index condition |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-----------------------+

```

# SQL优化

## SQL性能分析

### SQL执行频率

+ MySQL 客户端连接成功后，通过 show [session|global] status 命令可以提供服务器状态信息。通过如下指令，可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次

```bash
-- session 是查看当前会话 ;
-- global 是查询全局数据 ;
SHOW GLOBAL STATUS LIKE 'Com_______';
```

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251431089.jpg)

- Com_delete: 删除次数
- Com_insert: 插入次数
- Com_select: 查询次数
- Com_update: 更新次数

**通过上述指令，我们可以查看到当前数据库到底是以查询为主，还是以增删改为主，从而为数据库优化提供参考依据。 如果是以增删改为主，我们可以考虑不对其进行索引的优化。 如果是以查询为主，那么就要考虑对数据库的索引进行优化了**

## 慢查询日志及优化过程

- 慢查询日志记录了所有执行时间超过指定参数（long_query_time，单位：秒，默认10秒）的所有SQL语句的日志
- MySQL的慢查询日志默认没有开启，我们可以查看一下系统变量   slow_query_log

+ 如果要开启慢查询日志，需要在MySQL的配置文件（/etc/my.cnf）中配置如下信息

```bash
# 开启MySQL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
```

+ 配置完毕之后，通过以下指令重新启动MySQL服务器进行测试，查看慢日志文件中记录的信息/var/lib/mysql/localhost-slow.log

```bash
systemctl 1 restart mysqld
```

**优化过程**

利用explain关键字模拟优化器执行SQL查询语句，来分析sql慢查询语句

+ 索引没起作用的情况
  + 查看是否使用索引，是否需要增加索引

+ 优化数据库结构

  合理的数据库结构不仅可以使数据库占用更小的磁盘空间，而且能够使查询速度更快。数据库结构的设计，需要考虑数据冗余、查询和更新的速度、字段的数据类型是否合理等多方面的内容

  + 将字段很多的表分解成多个表 

    对于字段比较多的表，如果有些字段的使用频率很低，可以将这些字段分离出来形成新表。因为当一个表的数据量很大时，会由于使用频率低的字段的存在而变慢。

  + 增加中间表

    对于需要经常联合查询的表，可以建立中间表以提高查询效率。通过建立中间表，把需要经常联合查询的数据插入到中间表中，然后将原来的联合查询改为对中间表的查询，以此来提高查询效率

+ 分解关联查询

 	 将一个大的查询分解为多个小查询是很有必要的。很多高性能的应用都会对关联查询进行分解，就是可以对每一个表进行一次单表查询，然后将查询结果在应用程序中进行关联，很多场景下这样会更高效

## profile详情

+ show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。通过have_profiling参数，能够看到当前MySQL是否支持profile操作

```bash
SELECT 1 @@have_profiling ;
```

+ 通过set语句在session/global级别开启profiling

```bash
SET profiling = 1;
```

+ 直接执行如下的SQL语句：

```bash
select * from tb_user;
select * from tb_user where id = 1;
select * from tb_user where name = '白起';
select count(*) from tb_sku;
```

+ 执行一系列的业务SQL的操作，然后通过如下指令查看指令的执行耗时

```bash
-- 查看每一条SQL的耗时基本情况
show profiles;
-- 查看指定query_id的SQL语句各个阶段的耗时情况
show profile for query query_id;
-- 查看指定query_id的SQL语句CPU的使用情况
show profile cpu for query query_id;
```

+ 查看每一条SQL的耗时情况

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251440145.jpg)

+ 查看指定SQL各个阶段的耗时情况

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251441315.jpg)

### explain

+ EXPLAIN 或者 DESC命令获取 MySQL 如何执行 SELECT 语句的信息，包括在 SELECT 语句执行过程中表如何连接和连接的顺序

```bash
-- 直接在select语句之前加上关键字 explain / desc
EXPLAIN SELECT 字段列表 FROM 表名 WHERE 条件;
```

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251442028.jpg)

+ Explain 执行计划中各个字段的含义

|     字段     | 含义                                                         |
| :----------: | ------------------------------------------------------------ |
|      id      | select查询的序列号，表示查询中执行select子句或者是操作表的顺序 (id相同，执行顺序从上到下; id不同，值越大，越先执行) |
| select_type  | 表示SELECT的类型,常见的取值有SIMPLE (简单表,即不使用表连接或者子查询) 、PRIMARY (主查询,即外层的查询) UNION (UNION中的第二个或者后面的查询语句)、 SUBQUERY (SELECT /WHERE之后包含了子查询)等 |
|     type     | 表示连接类型，性能由好到差的连接类型为NULL system、 const. eq_ref、ref、 range、 index、 all |
| possible_key | 显示可能应用在这张表上的索引，一个或多个                     |
|     key      | 实际使用的索引,如果为NULL,则没有使用索引                     |
|   key len    | 表示索引中使用的字节数, 该值为索引字段最大可能长度,并非实际使用长度,在不损失精确性的前提下, 长度越短越好 |
|     rows     | Mysql认为必须要执行查询的行数,在innodb引擎的表中,是一个估计值, 可能并不总是准确的 |
|   filtered   | 表示返回结果的行数占需读取行数的百分比, filtered的值越大越好 |

##  插入数据

### insert

+ 如果我们需要一次性往数据库表中插入多条记录，可以从以下三个方面进行优化

```bash
insert into tb_test values(1,'tom');
insert into tb_test values(2,'cat');
insert into tb_test values(3,'jerry');
.....
```

#### 优化方案一

+ 批量插入数据

```bash
Insert into tb_test values(1,'Tom'),(2,'1 Cat'),(3,'Jerry');
```

+ 修改后的插入操作能够提高程序的插入效率。这里第二种SQL执行效率高的主要原因有两个，一是减少SQL语句解析的操作，只需要解析一次就能进行数据的插入操作，二是SQL语句较短，可以减少网络传输的IO

#### 优化方案二

+ 手动控制事务

```bash
start transaction;
insert into tb_test values(1,'Tom'),(2,'Cat'),(3,'Jerry');
insert into tb_test values(4,'Tom'),(5,'Cat'),(6,'Jerry');
insert into tb_test values(7,'Tom'),(8,'Cat'),(9,'Jerry');
commit;
```

+ 使用事物可以提高数据的插入效率，这是因为进行一个INSERT操作时，MySQL内部会建立一个事物，在事物内进行真正插入处理。通过使用事物可以减少创建事物的消耗，所有插入都在执行后才进行提交操作

#### 优化方案三

+ 主键顺序插入，性能要高于乱序插入

```bash
主键乱序插入 : 8 1 9 21 88 2 4 15 89 5 7 3
主键顺序插入 : 1 2 3 4 5 7 8 9 15 21 88 89
```

### 大批量插入数据

+ 如果一次性需要插入大批量数据(比如: 几百万的记录)，使用insert语句插入性能较低，此时可以使用MySQL数据库提供的load指令进行插入。操作如下

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251632147.jpg)

+ 可以执行如下指令，将数据脚本文件中的数据加载到表结构中

```bash
-- 客户端连接服务端时，加上参数 -–local-infile
mysql –-local-infile -u root -p
-- 设置全局参数local_infile为1，开启从本地加载文件导入数据的开关
set global local_infile = 1;
-- 执行load指令将准备好的数据，加载到表结构中
load data local infile '/root/sql1.log' into table tb_user fields
terminated by ',' lines terminated by '\n' ;
```

## 主键优化

### 数据组织方式

+ 在InnoDB存储引擎中，表数据都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表(index organized table IOT)

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251634415.jpg)

+ 行数据，都是存储在聚集索引的叶子节点上的。而我们之前也讲解过InnoDB的逻辑结构图

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251634808.jpg)

+ 在InnoDB引擎中，数据行是记录在逻辑结构 page 页中的，而每一个页的大小是固定的，默认16K。那也就意味着， 一个页中所存储的行也是有限的，如果插入的数据行row在该页存储不下，将会存储到下一个页中，页与页之间会通过指针连接

### 页分裂

+ 页可以为空，也可以填充一半，也可以填充100%。每个页包含了2-N行数据(如果一行数据过大，会行溢出)，根据主键排列

#### 主键顺序插入效果

+ 从磁盘中申请页， 主键顺序插入

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251636021.jpg)

+ 第一个页没有满，继续往第一页插入

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251637770.jpg)

+ 当第一个也写满之后，再写入第二个页，页与页之间会通过指针连接

#### 主键乱序插入效果

+ 加入1#,2#页都已经写满了，存放了如图所示的数据

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251638167.jpg)

+ 此时再插入id为50的记录，我们来看看会发生什么现象。会再次开启一个页，写入新的页中吗？

![image-20230125163930521](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251639584.png)

+ 不会。因为，索引结构的叶子节点是有顺序的。按照顺序，应该存储在47之后

![image-20230125164015253](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251640327.png)

+ 但是47所在的1#页，已经写满了，存储不了50对应的数据了。 那么此时会开辟一个新的页 3#

![image-20230125164047535](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251640594.png)

+ 但是并不会直接将50存入3#页，而是会将1#页后一半的数据，移动到3#页，然后在3#页，插入50

![image-20230125164110799](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251641885.png)

+ 移动数据，并插入id为50的数据之后，那么此时，这三个页之间的数据顺序是有问题的。 1#的下一个页，应该是3#， 3#的下一个页是2#。 所以，此时，需要重新设置链表指针

![image-20230125164145609](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251641671.png)

+ 上述的这种现象，称之为 "页分裂"，是比较耗费性能的操作

### 页合并

+ 目前表中已有数据的索引结构(叶子节点)如下

![image-20230125164556913](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251645978.png)

+ 当我们对已有数据进行删除时，具体的效果如下:
+ 当删除一行记录时，实际上记录并没有被物理删除，只是记录被标记（flaged）为删除并且它的空间变得允许被其他记录声明使用

![image-20230125164713704](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251647759.png)

+ 当页中删除的记录达到 MERGE_THRESHOLD（默认为页的50%），InnoDB会开始寻找最靠近的页（前或后）看看是否可以将两个页合并以优化空间使用

![image-20230125164738323](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251647381.png)

+ 删除数据，并将页合并之后，再次插入新的数据21，则直接插入3#页

![image-20230125164800554](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251648609.png)

> 知识小贴士：
> MERGE_THRESHOLD：合并页的阈值，可以自己设置，在创建表或者创建索引时指定

### 索引设计原则

- 满足业务需求的情况下，尽量**降低主键的长度**
- 插入数据时，尽量**选择顺序插入**，选择使用AUTO_INCREMENT自增主键
- 尽量不要使用UUID做主键或者是其他自然主键，如身份证号
- 业务操作时，**避免对主键的修改**

![image-20230125165015337](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301251650428.png)

## 常见优化

### order by优化

- MySQL的排序，有两种方式
- Using filesort : 通过表的索引或全表扫描，读取满足条件的数据行，然后在排序缓冲区sortbuffer中完成排序操作，所有不是通过索引直接返回排序结果的排序都叫 FileSort 排序
- Using index : 通过有序索引顺序扫描直接返回有序数据，这种情况即为 using index，不需要额外排序，操作效率高
- 对于以上的两种排序方式，Using index的性能高，而Using filesort的性能低，我们在优化排序操作时，尽量要优化为 Using index

+ 在MySQL中我们创建的索引，默认索引的叶子节点是从小到大排序的，而此时我们查询排序时，是从大到小，所以，在扫描时，就是反向扫描，就会出现 Backward index scan。 在MySQL8版本中，支持降序索引，我们也可以创建降序索引

+ 根据age, phone进行降序一个升序，一个降序

```bash
explain select id,age,phone from tb_user order by age 1 asc , phone desc ;
```

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301252118835.jpg)

+ 创建索引时，如果未指定顺序，默认都是按照升序排序的，而查询时，一个升序，一个降序，此时就会出现Using filesort

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301252119414.jpg)

+ 为了解决上述的问题，我们可以创建一个索引，这个联合索引中 age 升序排序，phone 倒序排序

```bash
create index idx_user_age_phone_ad on tb_user(age asc ,phone desc);
```

+ 然后再次执行如下SQL

```bash
explain select id,age,phone from tb_user order by age asc , phone desc ;
```

![image-20230125212114362](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301252121449.png)

+ order by优化原则

  - 根据排序字段建立合适的索引，多字段排序时，也遵循最左前缀法则

  - 尽量使用覆盖索引

  - 多字段排序, 一个升序一个降序，此时需要注意联合索引在创建时的规则（ASC/DESC）

  - 如果不可避免的出现filesort，大数据量排序时，可以适当增大排序缓冲区大小sort_buffer_size(默认256k)

### group by优化

- 在分组操作中，我们需要通过以下两点进行优化，以提升性能
  - 在分组操作时，可以通过索引来提高效率
  - 分组操作时，索引的使用也是满足最左前缀法则的


### limit优化

+ 在数据量比较大时，如果进行limit分页查询，在查询时，越往后，分页查询效率越低

- 当在进行分页查询时，如果执行 limit 2000000,10 ，此时需要MySQL排序前2000010 记录，仅仅返回 2000000 - 2000010 的记录，其他记录丢弃，查询排序的代价非常大 
- 优化思路: 一般分页查询时，通过创建覆盖索引能够比较好地提高性能，可以通过覆盖索引加子查询形式进行优化

```bash
explain select * from tb_sku t , (select id from tb_sku order by id limit 2000000,10) a where t.id = a.id;
```

### count优化

#### 概述

- 在之前的测试中，我们发现，如果数据量很大，在执行count操作时，是非常耗时的
- MyISAM 引擎把一个表的总行数存在了磁盘上，因此执行 count() 的时候会直接返回这个数，效率很高； 但是如果是带条件的count，MyISAM也慢
- InnoDB 引擎就麻烦了，它执行 count()的时候，需要把数据一行一行地从引擎里面读出来，然后累积计数
- 如果说要大幅度提升InnoDB表的count效率，主要的优化思路：自己计数(可以借助于redis这样的数据库进行,但是如果是带条件的count又比较麻烦了)

#### count用法

- count() 是一个聚合函数，对于返回的结果集，一行行地判断，如果 count 函数的参数不是NULL，累计值就加 1，否则不加，最后返回累计值
- 用法：count（*）、count（主键）、count（字段）、count（数字）

| count用 法    | 含义                                                         |
| ------------- | ------------------------------------------------------------ |
| count (主 键) | InnoDB引擎会遍历整张表,把每一行的主键id值都取出来,返回给服务层。 服务层拿到主键后,直接按行进行累加(主键不可能为null) |
| count (字 段) | 没有not null约束: InnoDB引擎会遍历整张表把每一行的字段值都取出来，返回给服务层,服务层判断是否为null,不为null,计数累加。 有not null约束: InnoDB引擎会遍历整张表字段值都取出来,返回给服务层,直接按行进行累加 |
| count(数 字   | InnoDB引擎遍历整张表,但不取值。服务层对于返回的每一行,放一个数字"1" 进去,直接按行进行累加 |
| count (*)     | InnoDB引擎并不会把全部字段取出来,而是专门做了优化,不取值,服务层直接 按行进行累加 |

> 按照效率排序的话，count(字段) < count(主键 id) < count(1) ≈ count**(*)**，所以尽量使用 count**(*)**

### update优化

+ 我们主要需要注意一下update语句执行时的注意事项
+ 但是当我们在执行如下SQL时

```bash
update course set name = 'SpringBoot' where name = 'PHP' ;
```

+ 当我们开启多个事务，在执行上述的SQL时，我们发现行锁升级为了表锁。 导致该update语句的性能大大降低

> InnoDB的行锁是针对索引加的锁，不是针对记录加的锁 ,并且该索引不能失效，否则会从行锁升级为表锁

## 实战案例（文章及内容出现慢查询）

### 背景

- 有一个article表，用于存储文章的基本信息的，有文章id，作者id等一些属性，有一个content表，主要用于存储文章的内容，主键是article_id，需求需要将一些满足条件的作者发布的文章导入到另外一个库，所以我同事就在项目中先查询出了符合条件的作者id，然后开启了多个线程，每个线程每次取一个作者id，执行查询和导入工作。查询出作者id是1111，名下的所有文章信息，文章内容相关的信息的SQL如下


```sql
SELECT
	a.*, c.*
FROM
	article a
LEFT JOIN content c ON a.id = c.article_id
WHERE
	a.author_id = 1111
AND a.create_time < '2020-04-29 00:00:00'
LIMIT 210000,100
```

### 问题

+ 因为查询的这个数据库是机械硬盘的，在offset查询到20万时，查询时间已经特别长了，运维同事那边直接收到报警，说这个库已经IO阻塞了，已经多次进行主从切换了，我们就去navicat里面试着执行了一下这个语句，也是一直在等待， 然后对数据库执行show proceesslist 命令查看了一下，发现每个查询都是处于Writing to net的状态，没办法只能先把导入的项目暂时下线，然后执行kill命令将当前的查询都杀死进程(因为只是客户端Stop的话，MySQL服务端会继续查询)

### 解决









# 锁

## why

+ **保证数据并发访问的一致性、有效性**

## 概述

+ 锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O）的争用以外，数据也是一种供许多用户共享的资源。如何**保证数据并发访问的一致性、有效性**是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂

- MySQL中的锁，按照锁的粒度分，分为以下三类
  - 全局锁：锁定数据库中的所有表
  - 表级锁：每次操作锁住整张表
  - 行级锁：每次操作锁住对应的行数据


## 全局锁

### 介绍

- 全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都将被阻塞。其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性
- 为什么全库逻辑备份，就需要加全局锁呢？

### **不加全局锁可能存在问题**

- 假设在数据库中存在这样三张表: tb_stock 库存表，tb_order 订单表，tb_orderlog 订单日志表

![image-20230126091420830](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301260914892.png)

- 在进行数据备份时，先备份了tb_stock库存表
- 然后接下来，在业务系统中，执行了下单操作，扣减库存，生成订单（更新tb_stock表，插入tb_order表）
- 然后再执行备份 tb_order表的逻辑
- 业务中执行插入订单日志操作
- 最后，又备份了tb_orderlog表
- 此时备份出来的数据，是存在问题的。因为备份出来的数据，tb_stock表与tb_order表的数据不一致(有最新操作的订单信息,但是库存数没减)
- 那如何来规避这种问题呢? 此时就可以借助于MySQL的全局锁来解决

### 加全局锁后情况

![image-20230126091737396](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301260917459.png)

+ 对数据库进行进行逻辑备份之前，先对整个数据库加上全局锁，一旦加了全局锁之后，其他的DDL、DML全部都处于阻塞状态，但是可以执行DQL语句，也就是处于只读状态，而数据备份就是查询操作。那么数据在进行逻辑备份的过程中，数据库中的数据就是不会发生变化的，这样就保证了数据的一致性和完整性

### 语法

```bash
#加全局锁
flush tables 1 with read lock ;
#数据备份
mysqldump -uroot –p1234 itcast > itcast.sql
#释放锁
unlock tables ;
```

### 特点
- 数据库中加全局锁，是一个比较重的操作，存在以下问题
  - 如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆
  - 如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志（binlog），会导致主从延迟
  - 在InnoDB引擎中，我们可以在备份时加上参数 --single-transaction 参数来完成不加锁的一致性数据备份


```bash
mysqldump --single-transaction -uroot –p123456 itcast > itcast.sql
```

## 表级锁

### 介绍

- 表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中
- 对于表级锁，主要分为以下三类
  - 表锁
  - 元数据锁（meta data lock，MDL）
  - 意向锁

### 表锁

- 对于表锁，分为两类
  - 表共享读锁（read lock）
  - 表独占写锁（write lock）

```bash
#语法
#加锁
lock tables 表名... read/write
#释放锁
unlock tables / 客户端断开连接
```

#### 读锁

![image-20230126092548936](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301260925997.png)

#### 写锁

![image-20230126092702431](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301260927495.png)

#### 结论

+ 读锁不会阻塞其他客户端的读，但是会阻塞写。写锁既会阻塞其他客户端的读，又会阻塞其他客户端的写

### 元数据锁

- meta data lock , 元数据锁，简写MDL
- MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。MDL锁主要作用是维护表元数据的数据一致性，在表上有活动事务的时候，不可以对元数据进行写入操作。为了避免DML与DDL冲突，保证读写的正确性
- **这里的元数据，大家可以简单理解为就是一张表的表结构。 也就是说，某一张表涉及到未提交的事务时，是不能够修改这张表的表结构的**
- 在MySQL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁(共享)；当对表结构进行变更操作的时候，加MDL写锁(排他)
- 常见的SQL操作时，所添加的元数据锁

| 对应SQL                                        | 锁类型                                  | 说明                                              |
| ---------------------------------------------- | --------------------------------------- | ------------------------------------------------- |
| lock tables xxx read / write                   | SHARED READ_ONLY / SHARED NO READ_WRITE |                                                   |
| select、 select... lock in share mode          | SHARED_READ                             | 与SHARED_READ、 SHARED_WRITE兼容,与 EXCLUSIVE互斥 |
| insert、update、 delete, select ... for update | SHARED_WRITE                            | 与SHARED_READ、 SHARED_WRITE兼容,与 EXCLUSIVE互斥 |
| alter table...                                 | EXCLUSIVE                               | 与其他的MDL都互斥                                 |

### 意向锁

#### 介绍

- 为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是否加锁，使用意向锁来减少表锁的检查
- 假如没有意向锁，客户端一对表加了行锁后，客户端二如何给表加表锁呢，来通过示意图简单分析一下
- 首先客户端一，开启一个事务，然后执行DML操作，在执行DML语句时，会对涉及到的行加行锁

![image-20230126100043181](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261000256.png)

+ 当客户端二，想对这张表加表锁时，会检查当前表是否有对应的行锁，如果没有，则添加表锁，此时就会从第一行数据，检查到最后一行数据，效率较低

![image-20230126100110174](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261001240.png)

- 有了意向锁之后
- 客户端一，在执行DML操作时，会对涉及的行加行锁，同时也会对该表加上意向锁

![image-20230126100201450](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261002522.png)

+ 而其他客户端，在对这张表加表锁的时候，会根据该表上所加的意向锁来判定是否可以成功加表锁，而不用逐行判断行锁情况了

![image-20230126100228397](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261002460.png)

#### 分类

- 意向共享锁(IS): 由语句select ... lock in share mode添加 。 与 表锁共享锁(read)兼容，与表锁排他锁(write)互斥
- 意向排他锁(IX): 由insert、update、delete、select...for update添加 。与表锁共享锁(read)及排他锁(write)都互斥，意向锁之间不会互斥

> 一旦事务提交了，意向共享锁、意向排他锁，都会自动释放

+ 可以通过以下SQL，查看意向锁及行锁的加锁情况

```bash
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```

## 行级锁

### 介绍

- 行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在InnoDB存储引擎中

- InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类

  - 行锁（Record Lock）：锁定单个行记录的锁，防止其他事务对此行进行update和delete。在RC、RR隔离级别下都支持

    ![image-20230126100545790](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261005844.png)

  - 间隙锁（Gap Lock）：锁定索引记录间隙（不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。在RR隔离级别下都支持

    ![image-20230126100729895](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261007955.png)

  + 临键锁（Next-Key Lock）：行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持

    ![image-20230126100743919](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261007977.png)

#### 行锁

##### 介绍

- InnoDB实现了以下两种类型的行锁
  - 共享锁（S）：允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁
  - 排他锁（X）：允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁
- 两种行锁的兼容情况如下

![image-20230126101505739](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261015799.png)

+ 常见的SQL语句，在执行时，所加的行锁如下

| SQL                           | 行锁类型    | 说明                                     |
| ----------------------------- | ----------- | ---------------------------------------- |
| INSERT                        | 排他锁      | 自动加锁                                 |
| UPDATE                        | 排他锁      | 自动加锁                                 |
| DELETE                        | 排他锁      | 自动加锁                                 |
| SELECT (正常                  | 不加任何 锁 |                                          |
| SELECT ... LOCK IN SHARE MODE | 共享锁      | 需要手动在SELECT之后加LOCK IN SHARE MODE |
| SELECT ... FOR UPDATE         | 排他锁      | 需要手动在SELECT之后加EOR UPDATE         |

- 默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描，以防止幻读
  - 针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁
  - **InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时就会升级为表锁**
- 可以通过以下SQL，查看意向锁及行锁的加锁情况

```bash
select object_schema,object_name,index_name,lock_type,lock_mode,lock_data from performance_schema.data_locks;
```

- 无索引行锁升级为表锁
- stu表中数据如下

![image-20230126101914586](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261019665.png)

+ 我们在两个客户端中执行如下操作

![image-20230126101940890](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261019021.png)

- 在客户端一中，开启事务，并执行update语句，更新name为Lily的数据，也就是id为19的记录 。然后在客户端二中更新id为3的记录，却不能直接执行，会处于阻塞状态，为什么呢？
- 原因就是因为此时，客户端一，根据name字段进行更新时，name字段是没有索引的，如果没有索引，此时行锁会升级为表锁(因为行锁是对索引项加的锁，而name没有索引)
- 接下来，我们再针对name字段建立索引，索引建立之后，再次做一个测试

![image-20230126102012166](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261020307.png)

+ 此时我们可以看到，客户端一，开启事务，然后依然是根据name进行更新。而客户端二，在更新id为3的数据时，更新成功，并未进入阻塞状态。 这样就说明，我们根据索引字段进行更新操作，就可以避免行锁升级为表锁的情况

### 间隙锁&临键锁

- **默认情况下，InnoDB在 REPEATABLE READ事务隔离级别运行，InnoDB使用 next-key 锁进行搜索和索引扫描**，以防止幻读
  - 索引上的等值查询(唯一索引)，给不存在的记录加锁时, 优化为间隙锁
  - 索引上的等值查询(非唯一普通索引)，向右遍历时最后一个值不满足查询需求时，next-key lock 退化为间隙锁
  - 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止

> 注意：间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁

+ 索引上的等值查询(唯一索引)，给不存在的记录加锁时, 优化为间隙锁

![image-20230126102332440](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261023574.png)

+ 索引上的等值查询(非唯一普通索引)，向右遍历时最后一个值不满足查询需求时，next-keylock 退化为间隙锁

+ 我们知道InnoDB的B+树索引，叶子节点是有序的双向链表。 假如，我们要根据这个二级索引查询值为18的数据，并加上共享锁，我们是只锁定18这一行就可以了吗？ 并不是，因为是非唯一索引，这个结构中可能有多个18的存在，所以，在加锁时会继续往后找，找到一个不满足条件的值（当前案例中也就是29）。此时会对18加临键锁，并对29之前的间隙加锁

![image-20230126102628590](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261026748.png)

+ 索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止

![image-20230126102649548](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202301261026668.png)

- 查询的条件为id>=19，并添加共享锁。 此时我们可以根据数据库表中现有的数据，将数据分为三个部分
  - [19]
  - (19,25]
  - (25,+∞]
- 所以数据库数据在加锁是，就是将19加了行锁，25的临键锁（包含25及25之前的间隙），正无穷的临键锁(正无穷及之前的间隙)

# InnoDB引擎

## 4大特性

### **插入缓冲 （Insert Buffer/Change Buffer）**

- 插入缓存之前版本叫insert buffer，现版本 change buffer，主要提升插入性能，change buffer是insert buffer的加强，insert buffer只针对insert有效，change buffering对insert、delete、update(delete+insert)、purge都有效。有什么用呢？
- 对于非聚集索引来说，比如存在用户购买金额这样一个字段，索引是普通索引，每个用户的购买的金额不相同的概率比较大，这样导致可能出现购买记录在数据在数据里的排序可能是1000，3，499，35…，这种不连续的数据，一会插入这个数据页，一会插入那个数据页，这样造成的IO是很耗时的，所以出现了Insert Buffer

- Insert Buffer是怎么做的呢？mysql对于非聚集索引的插入，先去判断要插入的索引页是否已经在内存中了，如果不在，暂时不着急先把索引页加载到内存中，而是把它放到了一个Insert Buffer对象中，临时先放在这，然后等待情况，等待很多和现在情况一样的非聚集索引，再和要插入的非聚集索引页合并，比如说现在Insert Buffer中有1，99，2，100，合并之前可能要4次插入，合并之后1，2可能是一个页的，99，100可能是一个页的，这样就减少到了2次插入。这样就提升了效率和插入性能，减少了随机IO带来性能损耗

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281514806.png)

+ 综合上述，Insert Buffer 只对于非聚集索引（非唯一）的插入和更新有效，对于每一次的插入不是写到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，如果在则直接插入；若不在，则先放到Insert Buffer 中，再按照一定的频率进行合并操作，再写回disk。这样通常能将多个插入合并到一个操作中，目的还是减少了随机IO带来性能损耗

**使用插入缓冲的条件**

- 非聚集索引
- 非唯一索引

**合并频率条件**

- 辅助索引页被读取到缓冲池中。正常的select先检查Insert Buffer是否有该非聚集索引页存在，若有则合并插入
- 辅助索引页没有可用空间。空间小于1/32页的大小，则会强制合并操作

- Master Thread 每秒和每10秒的合并操作

### **双写机制（Double Write）**

**why**

+ 提高innodb的可靠性，用来解决部分写失败(partial page write页断裂)
+ 一个数据页的大小是16K，假设在把内存中的脏页写到数据库的时候，写了2K突然掉电，也就是说前2K数据是新的，后14K是旧的，那么磁盘数据库这个数据页就是不完整的，是一个坏掉的数据页。redo只能加上旧、校检完整的数据页恢复一个脏块，不能修复坏掉的数据页，所以这个数据就丢失了，可能会造成数据不一致，所以需要double write

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281555058.png)

**double write工作流程**

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281558134.png)

- doublewrite由两部分组成，一部分为内存中的doublewrite buffer，其大小为2MB，另一部分是磁盘上共享表空间(ibdata x)中连续的128个页，即2个区(extent)，大小也是2M。

- 当一系列机制触发数据缓冲池中的脏页刷新时，并不直接写入磁盘数据文件中，而是先拷贝至内存中的doublewrite buffer中；

- 接着从两次写缓冲区分两次写入磁盘共享表空间中(连续存储，顺序写，性能很高)，每次写1MB；

- 待第二步完成后，再将doublewrite buffer中的脏页数据写入实际的各个表空间文件(离散写)；(脏页数据固化后，即进行标记对应doublewrite数据可覆盖)

- doublewrite的崩溃恢复

  - 如果操作系统在将页写入磁盘的过程中发生崩溃，在恢复过程中，innodb存储引擎可以从共享表空间的doublewrite中找到该页的一个最近的副本，将其复制到表空间文件，再应用redo log，就完成了恢复过程

  - 因为有副本所以也不担心表空间中数据页是否损坏


> Q：为什么log write不需要doublewrite的支持？
>
> A：因为redolog写入的单位就是512字节，也就是磁盘IO的最小单位，所以无所谓数据损坏

### 自适应哈希索引（Adaptive Hash Index，AHI）

+ 哈希算法是一种非常快的查找方法，在一般情况（没有发生hash冲突）下这种查找的时间复杂度为O(1)。InnoDB存储引擎会监控对表上辅助索引页的查询。如果观察到建立hash索引可以提升性能，就会在缓冲池建立hash索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）

**预读 （Read Ahead）**

- 预读（read-ahead)操作是一种IO操作，用于异步将磁盘的页读取到**buffer pool**中，预料这些页会马上被读取到。预读请求的所有页集中在一个范围内。InnoDB使用两种预读算法
- Linear read-ahead：**线性预读**技术预测在buffer pool中被访问到的数据它临近的页也会很快被访问到。能够通过调整被连续访问的页的数量来控制InnoDB的预读操作，使用参数 innodb_read_ahead_threshold配置，添加这个参数前，InnoDB会在读取到当前区段最后一页时才会发起异步预读请求
- Random read-ahead: **随机预读**通过buffer pool中存中的来预测哪些页可能很快会被访问，而不考虑这些页的读取顺序。如果发现buffer pool中存中一个区段的13个连续的页，InnoDB会异步发起预读请求这个区段剩余的页。通过设置 innodb_random_read_ahead 为 ON开启随机预读特性

## 逻辑存储结构

+ InnoDB的逻辑存储结构如下图所示

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031715842.jpg)

### 表空间

+ 表空间是InnoDB存储引擎逻辑结构的最高层， 如果用户启用了参数 innodb_file_per_table(在8.0版本中默认开启) ，则每张表都会有一个表空间（xxx.ibd），一个mysql实例可以对应多个表空间，用于存储记录、索引等数据

### 段

+ 段，分为数据段（Leaf node segment）、索引段（Non-leaf node segment）、回滚段（Rollback segment），InnoDB是索引组织表，数据段就是B+树的叶子节点， 索引段即为B+树的非叶子节点。段用来管理多个Extent（区）

### 区

+ 区，表空间的单元结构，每个区的大小为1M。 默认情况下， InnoDB存储引擎页大小为16K， 即一个区中一共有64个连续的页

### 页

+ 页，是InnoDB 存储引擎磁盘管理的最小单元，每个页的大小默认为 16KB。为了保证页的连续性，InnoDB 存储引擎每次从磁盘申请 4-5 个区

### 行

行，InnoDB 存储引擎数据是按行进行存放的。在行中，默认有两个隐藏字段

**Trx_id**：每次对某条记录进行改动时，都会把对应的事务id赋值给trx_id隐藏列

**Roll_pointer**：每次对某条引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，可以通过它来找到该记录修改前的信息

## 架构

+ MySQL5.5 版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。下面是InnoDB架构图，左侧为内存结构，右侧为磁盘结构

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031721556.jpg)

### 内存结构

+ 在左侧的内存结构中，主要分为这么四大块儿： Buffer Pool、Change Buffer、Adaptive Hash Index、Log Buffer。 接下来介绍一下这四个部分

#### Buffer Pool

- InnoDB存储引擎基于磁盘文件存储，**访问物理硬盘和在内存中进行访问，速度相差很大，为了尽可能弥补这两者之间的I/O效率的差值**，就需要把经常使用的数据加载到缓冲池中，避免每次访问都进行磁盘I/O
- 在InnoDB的缓冲池中不仅缓存了索引页和数据页，还包含了undo页、插入缓存、自适应哈希索引以及InnoDB的锁信息等等
- 缓冲池 Buffer Pool，是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，**在执行增删改查操作时，先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度**
- 缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型
  - free page：空闲page，未被使用
  - clean page：被使用page，数据没有被修改过
  - dirty page：脏页，被使用page，数据被修改过，缓存数据与磁盘的数据产生了不一致
- 在专用服务器上，通常将多达80％的物理内存分配给缓冲池 。参数设置： show variableslike 'innodb_buffer_pool_size';

#### Change Buffer

- Change Buffer，更改缓冲区（针对于非唯一二级索引页），**在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，而会将数据变更存在更改缓冲区 Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中**
- Change Buffer的意义是什么呢?
- 先来看一幅图，这个是二级索引的结构图

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031753046.jpg)

+ 与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更新可能会影响索引树中不相邻的二级索引页，如果每一次都操作磁盘，会造成大量的磁盘IO。有了ChangeBuffer之后，我们可以在缓冲池中进行合并处理，减少磁盘IO

> Buffer Pool和Change Buffer都是MySQL中的缓存机制，但它们的作用和使用方式略有不同。Buffer Pool主要用于缓存表数据和索引数据，而Change Buffer主要用于缓存对非唯一索引的更新操作。Buffer Pool是一个内存池，用于存储数据页，而Change Buffer则是一个内存缓存，用于缓存更新操作。Buffer Pool的作用是提高读取操作的性能，而Change Buffer的作用是提高更新操作的性能

#### Adaptive Hash Index

- 自适应hash索引，用于优化对Buffer Pool数据的查询。MySQL的innoDB引擎中虽然没有直接支持hash索引，但是给我们提供了一个功能就是这个自适应hash索引。因为前面我们讲到过，hash索引在进行等值匹配时，一般性能是要高于B+树的，因为hash索引一般只需要一次IO即可，而B+树，可能需要几次匹配，所以hash索引的效率要高，但是hash索引又不适合做范围查询、模糊匹配等
- InnoDB存储引擎会监控对表上各索引页的查询，如果观察到在特定的条件下hash索引可以提升速度，则建立hash索引，称之为自适应hash索引
- 自适应哈希索引，无需人工干预，是系统根据情况自动完成
- 参数： adaptive_hash_index

#### Log Buffer

- Log Buffer：日志缓冲区，用来保存要写入到磁盘中的log日志数据（redo log 、undo log），默认大小为 16MB，日志缓冲区的日志会定期刷新到磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘 I/O
- 参数
  - innodb_log_buffer_size：缓冲区大小
  - innodb_flush_log_at_trx_commit：日志刷新到磁盘时机，取值主要包含以下三个
    - 1: 日志在每次事务提交时写入并刷新到磁盘，默认值
    - 0: 每秒将日志写入并刷新到磁盘一次
    - 2: 日志在每次事务提交后写入，并每秒刷新到磁盘一次

### 磁盘结构

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032025219.jpg)

#### System Tablespace

- 系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和索引数据。(在MySQL5.x版本中还包含InnoDB数据字典、undolog等)

- 参数：innodb_data_file_path

  ![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032028062.jpg)

- 系统表空间，默认的文件名叫 ibdata1

#### File-Per-Table Tablespaces

- 如果开启了innodb_file_per_table开关 ，则每个表的文件表空间包含单个InnoDB表的数据和索引 ，并存储在文件系统上的单个数据文件中
- 开关参数：innodb_file_per_table ，该参数默认开启
- 也就是说，我们每创建一个表，都会产生一个表空间文件

#### General Tablespaces

- 通用表空间，需要通过 CREATE TABLESPACE 语法创建通用表空间，在创建表时，可以指定该表空间
- 创建表空间

```sql
CREATE TABLESPACE ts_name ADD DATAFILE 'file_name' ENGINE 1 = engine_name;
```

+ 创建表时指定表空间

```sql
CREATE TABLE xxx ... TABLESPACE ts_name;
```

#### Undo Tablespaces

+ 撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间（初始大小16M），用于存储undo log日志

#### Temporary Tablespaces

+ InnoDB 使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据

#### Doublewrite Buffer Files

+ 双写缓冲区，innoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，便于系统异常时恢复数据

#### Redo Log

+ 重做日志，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log）,前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中, 用于在刷新脏页到磁盘时,发生错误时, 进行数据恢复使用
+ 以循环方式写入重做日志文件，涉及两个文件

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032049833.jpg)

## 后台线程

+ 前面我们介绍了InnoDB的内存结构，以及磁盘结构，那么内存中我们所更新的数据，又是如何到磁盘中的呢？ 此时，就涉及到一组后台线程，接下来，就来介绍一些InnoDB中涉及到的后台线程

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032050040.jpg)

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032051567.jpg)

+ 在InnoDB的后台线程中，分为4类，分别是：Master Thread 、IO Thread、Purge Thread、Page Cleaner Thread

### Master Thread

+ 核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中, 保持数据的一致性，还包括脏页的刷新、合并插入缓存、undo页的回收

### IO Thread

+ 在InnoDB存储引擎中大量使用了AIO来处理IO请求, 这样可以极大地提高数据库的性能，而IOThread主要负责这些IO请求的回调

|       线程类型       | 默认个数 |             职责             |
| :------------------: | :------: | :--------------------------: |
|     Read thread      |    4     |          负责读操作          |
|     Write thread     |    4     |          负责写操作          |
|      Log thread      |    1     |  负责将日志缓冲区刷新到磁盘  |
| Insert buffer thread |    1     | 负责将写缓冲区内容刷新到磁盘 |

+ 我们可以通过以下的这条指令，查看到InnoDB的状态信息，其中就包含IO Thread信息

```sql
show engine 1 innodb status \G;
```

### Purge Thread

+ 主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回收

### Page Cleaner Thread

+ 协助 Master Thread 刷新脏页到磁盘的线程，它可以减轻 Master Thread 的工作压力，减少阻塞

## 事务原理

### 事务基础

#### 事务

+ 事务 是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败

#### 特性

- 原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败
- 一致性（Consistency）：事务完成时，必须使所有的数据都保持一致状态
- 隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行
- 持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的

那实际上，我们研究事务的原理，就是研究MySQL的InnoDB引擎是如何保证事务的这四大特性的

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032106094.jpg)

+ 而对于这四大特性，实际上分为两个部分。 其中的原子性、一致性、持久化，实际上是由InnoDB中的两份日志来保证的，一份是redo log日志，一份是undo log日志。 而隔离性是通过数据库的锁，加上MVCC来保证的

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032107065.jpg)

### redo log

- 重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。该日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file）,前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到该日志文件中, 用于在刷新脏页到磁盘,发生错误时, 进行数据恢复使用
- 如果没有redolog，可能会存在什么问题的？ 我们一起来分析一下
- 我们知道，在InnoDB引擎中的内存结构中，主要的内存区域就是缓冲池，在缓冲池中缓存了很多的数据页。 当我们在一个事务中，执行多个增删改的操作时，InnoDB引擎会先操作缓冲池中的数据，如果缓冲区没有对应的数据，会通过后台线程将磁盘中的数据加载出来，存放在缓冲区中，然后将缓冲池中的数据修改，修改后的数据页我们称为脏页。 而脏页则会在一定的时机，通过后台线程刷新到磁盘中，从而保证缓冲区与磁盘的数据一致。 而**缓冲区的脏页数据并不是实时刷新**的，而是一段时间之后将缓冲区的数据刷新到磁盘中，假如刷新到磁盘的过程出错了，而提示给用户事务提交成功，而数据却没有持久化下来，这就出现问题了，没有保证事务的持久性

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032116399.jpg)

+ 那么，如何解决上述的问题呢？ 在InnoDB中提供了一份日志 redo log，接下来我们再来分析一下，通过redolog如何解决这个问题

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032117705.jpg)

- 有了redolog之后，当对缓冲区的数据进行增删改之后，会首先将操作的数据页的变化，记录在redolog buffer中。在事务提交时，会将redo log buffer中的数据刷新到redo log磁盘文件中。过一段时间之后，如果刷新缓冲区的脏页到磁盘时，发生错误，此时就可以借助于redo log进行数据恢复，这样就保证了事务的持久性。 而如果脏页成功刷新到磁盘或者涉及到的数据已经落盘，此时redolog就没有作用了，就可以删除了，所以存在的两个redolog文件是循环写的
- **那为什么每一次提交事务，要刷新redo log到磁盘中呢，而不是直接将buffer pool中的脏页刷新到磁盘呢?**
- 因为在业务操作中，我们操作数据一般都是随机读写磁盘的，而不是顺序读写磁盘。 而redo log在往磁盘文件中写入数据，由于是日志文件，所以都是顺序写的。顺序写的效率，要远大于随机写。 这种**先写日志的方式，称之为 WAL**（Write-Ahead Logging)

### undo log

- 回滚日志，用于记录数据被修改前的信息 , 作用包含两个 : 提供回滚(保证事务的原子性) 和MVCC(多版本并发控制) 
- undo log和redo log记录物理日志不一样，它是逻辑日志。可以认为当delete一条记录时，undo log中会记录一条对应的insert记录，反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容并进行回滚
- Undo log销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于MVCC
- Undo log存储：undo log采用段的方式进行管理和记录，存放在前面介绍的 rollback segment回滚段中，内部包含1024个undo log segment

## MVCC

### 基本概念

#### 当前读

- 读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：select ... lock in share mode(共享锁)，select ...for update、update、insert、delete(排他锁)都是一种当前读
- 测试

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032146814.jpg)

+ 在测试中我们可以看到，即使是在默认的RR隔离级别下，事务A中依然可以读取到事务B最新提交的内容，因为在查询语句后面加上了 lock in share mode 共享锁，此时是当前读操作。当然，当我们加排他锁的时候，也是当前读操作

#### 快照读

- 简单的select（不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读
- Read Committed：每次select，都生成一个快照读
- **Repeatable Read：开启事务后第一个select语句才是快照读的地方**
- Serializable：快照读会退化为当前读
- 测试

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032147788.jpg)

+ 在测试中,我们看到即使事务B提交了数据,事务A中也查询不到。 原因就是因为普通的select是快照读，而在当前默认的RR隔离级别下，开启事务后第一个select语句才是快照读的地方，后面执行相同的select语句都是从快照中获取数据，可能不是当前的最新数据，这样也就保证了可重复读

#### MVCC

- 全称 Multi-Version Concurrency Control，**多版本并发控制**。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现MVCC提供了一个非阻塞读功能,**主要用于解决不可重复读和幻读问题时提高并发效率**。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、readView
- 接下来，我们再来介绍一下InnoDB引擎的表中涉及到的隐藏字段 、undolog 以及 readview，从而来介绍一下MVCC的原理

### 隐藏字段

#### 介绍

|  隐藏字段   |                             含义                             |
| :---------: | :----------------------------------------------------------: |
|  DB_TRX_ID  | 最近修改事务ID,记录插入这条记录或最后一次修改该记录的事务ID  |
| DB_ROLL_PTR | 回滚指针,指向这条记录的上一个版本,用于配合undo log,指向上一个版本 |
|  DB_ROW_ID  |      隐藏主键,如果表结构没有指定主键,将会生成该隐藏字段      |

+ 上述的前两个字段是肯定会添加的， 是否添加最后一个字段DB_ROW_ID，得看当前表有没有主键，如果有主键，则不会添加该隐藏字段

### 测试

**查看有主键的表 stu**

+ 进入服务器中的 /var/lib/mysql/itcast/ , 查看stu的表结构信息, 通过如下指令

```sql
ibd2sdi stu.ibd
```

+ 查看到的表结构信息中，有一栏 columns，在其中我们会看到处理我们建表时指定的字段以外，还有额外的两个字段 分别是：DB_TRX_ID 、 DB_ROLL_PTR ，因为该表有主键，所以没有DB_ROW_ID隐藏字段

**查看没有主键的表 employee**

+ 查看到的表结构信息中，有一栏 columns，在其中我们会看到处理我们建表时指定的字段以外，还有额外的三个字段 分别是：DB_TRX_ID 、 DB_ROLL_PTR 、DB_ROW_ID，因为employee表是没有指定主键的

### undo log

#### 介绍

- 回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志
- **当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除**
- **而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除**

#### 版本链

+ 有一张表原始数据为

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032238566.jpg)

> DB_TRX_ID : 代表最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID，是自增的
>
> DB_ROLL_PTR ： 由于这条数据是才插入的，没有被更新过，所以该字段值为null

+ 然后，有四个并发事务同时在访问这张表

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032237390.jpg)

+ 当事务2执行第一条修改语句时，会记录undo log日志，记录数据变更之前的样子; 然后更新记录，并且记录本次操作的事务ID，回滚指针，回滚指针用来指定如果发生回滚，回滚到哪一个版本

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032238703.jpg)

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032239061.jpg)

+ 当事务3执行第一条修改语句时，也会记录undo log日志，记录数据变更之前的样子; 然后更新记录，并且记录本次操作的事务ID，回滚指针，回滚指针用来指定如果发生回滚，回滚到哪一个版本

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302032242483.jpg)

> 最终我们发现，不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最早的旧记录

### ReadView

- ReadView（读视图）是 快照读 SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的）id
- ReadView中包含了四个核心字段

|      字段      |                        含义                        |
| :------------: | :------------------------------------------------: |
|     m_ids      |                当前活跃的事务ID集合                |
|   min_trx_id   |                   最小活跃事务ID                   |
|   max_trx_id   | 预分配事务1D,当前最大事务1D+1 (因为事务1D是自增的) |
| creator_trx_id |               ReadView创建者的事务ID               |

+ 而在readview中就规定了版本链数据的访问规则：trx_id 代表当前undo log版本链对应事务ID

|                条件                |               是否可以访问                |                   说明                    |
| :--------------------------------: | :---------------------------------------: | :---------------------------------------: |
|      trx_id == creator_trx_id      |              可以访问该版本               |    成立，说明数据是当前这个事务更改的     |
|        trx_id < min_trx_id         |              可以访问该版本               |         成立，说明数据已经提交了          |
|        trx id > max_trx_id         |             不可以访问该版本              | 成立，说明该事务是在 ReadView生成后才开启 |
| min_trx_id <= trx_id <= max_trx_id | 如果trx_id不在m_ids中, 是可以访问该版本的 |          成立，说明数据已经提交           |

- 不同的隔离级别，生成ReadView的时机不同
- **READ COMMITTED ：在事务中每一次执行快照读时生成ReadView**
- **REPEATABLE READ：仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView**

### 原理分析

#### RC隔离级别

+ RC隔离级别下，在事务中每一次执行快照读时生成ReadView
+ 我们就来分析事务5中，两次快照读读取数据，是如何获取数据的?
+ 在事务5中，查询了两次id为30的记录，由于隔离级别为Read Committed，所以每一次进行快照读，都会生成一个ReadView，那么两次生成的ReadView如下

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041109144.jpg)

- 那么这两次快照读在获取数据时，就需要根据所生成的ReadView以及ReadView的版本链访问规则，到undolog版本链中匹配数据，最终决定此次快照读返回的数据
- **先来看第一次快照读具体的读取过程**

![image-20230204110952977](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041109058.png)

+ 在进行匹配时，会从undo log的版本链，从上到下进行挨个匹配
+ 先匹配![image-20230204111039958](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041110033.png)

+ 这条记录，这条记录对应的trx_id为4，也就是将4带入右侧的匹配规则中。 ①不满足 ②不满足 ③不满足 ④也不满足 ，
  都不满足，则继续匹配undo log版本链的下一条

+ 再匹配第二条 ![image-20230204111137252](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041111309.png)
+ 这条记录对应的trx_id为3，也就是将3带入右侧的匹配规则中。①不满足 ②不满足 ③不满足 ④也不满足 ，都不满足，则继续匹配undo log版本链的下一条
+ 再匹配第三条 ![image-20230204111208520](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041112591.png)
+ 这条记录对应的trx_id为2，也就是将2带入右侧的匹配规则中。①不满足 ②满足 终止匹配，此次快照读，返回的数据就是版本链中记录的这条数据

**再来看第二次快照读具体的读取过程**

![image-20230204111251093](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041112173.png)

+ 在进行匹配时，会从undo log的版本链，从上到下进行挨个匹配
+ 先匹配 ![image-20230204111333788](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041113843.png)
+ 这条记录，这条记录对应的trx_id为4，也就是将4带入右侧的匹配规则中。 ①不满足 ②不满足 ③不满足 ④也不满足 ，都不满足，则继续匹配undo log版本链的下一条
+ 再匹配第二条![image-20230204111341663](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041113712.png)
+ 这条记录对应的trx_id为3，也就是将3带入右侧的匹配规则中。①不满足 ②满足 。终止匹配，此次快照读，返回的数据就是版本链中记录的这条数据。

#### RR隔离级别

+ RR隔离级别下，仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView。 而RR 是可重复读，在一个事务中，执行两次相同的select语句，查询到的结果是一样的。那MySQL是如何做到可重复读的呢? 我们简单分析一下就知道了

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041114233.jpg)

+ 我们看到，在RR隔离级别下，只是在事务中第一次快照读时生成ReadView，后续都是复用该ReadView，那么既然ReadView都一样， ReadView的版本链匹配规则也一样， 那么最终快照读返回的结果也是一样的
+ 所以呢，**MVCC的实现原理就是通过 InnoDB表的隐藏字段、UndoLog 版本链、ReadView来实现的。而MVCC + 锁，则实现了事务的隔离性。 而一致性则是由redolog 与 undolog保证**

![MySQL-进阶篇](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302041115241.jpg)

## 事务原理总结

**原子性---undo log**

**持久性---redo log**

**隔离性---MVCC+锁**

**一致性---redo log + undo log**

# 主从复制

## 工作过程

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306261402936.png)

# 数据库设计

## 三大范式

### why

- 表设计后，很可能结构不合理，出现数据冗余，这对数据的增删改查带来很多后患，所以我们需要审核是否合理
- 审核需要一些有关数据库设计的理论指导规则，这些规则业界简称数据库的范式。数据库范式为数据库的设计、开发提供了一个可参考的典范

### 范式和反范式各自优缺点

| 名称   | 优点                                                         | 缺点                                         |
| ------ | ------------------------------------------------------------ | -------------------------------------------- |
| 范式   | 范式化的表减少了数据冗余，数据表更新操作快、占用存储空间少   | 查询时通常需要多表关联查询，更难进行索引优化 |
| 反范式 | 反范式的过程就是通过冗余数据来提高查询性能，可以减少表关联和更好进行索引优化 | 存在大量冗余数据，并且数据的维护成本更高     |

### 第一范式（1NF）：原子性（存储的数据应该具有“不可再分性”）

+ 不良做法如下，“学生”一列有多项信息都合在一起了，不再具有原子性，所以应该分开

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281348584.png)

+ 修改后

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281348798.png)

### 第二范式（2NF）：唯一性 (消除非主键部分依赖联合主键中的部分字段)（一定要在第一范式已经满足的情况下）

- 需要实现每一行数据具有唯一可区分的特性，并不能有部分依赖关系。通常，给一个表加主键（也是推荐做法），就可以做到“唯一可区分”

- 但主键有这样情况

  - 设定一个字段为主键：此时，表示该一个字段的值就可以明确确定一行数据

  - 设定多个字段为主键：表示只有这多个字段的值都确定后才能确定一行数据。此时也称为“联合主键”

- 什么叫依赖

  - 如果确定一个表中的某个数据（A），则就可以确定该表中的其他另一个数据（B），则我们说：B依赖于A。实际上，一个表只要有主键，则其他非主键一定是依赖于主键的

- 什么叫“部分依赖”

  - 如果确定一个表中的某个数据组合（A，B），则就可以确定该表中的其他另一个数据（C），则我们说：C依赖于（A，B）（此时A，B通常就是做出主键）

  - 但如果某个数据D，它只依赖于数据A，或者说，A一确定，则D也可以确定，此时我们就称为“数据D部分依赖于数据A——可见部分依赖是指某个非主键字段，依赖于联合主键字段的其中部分字段

**不良做法**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281351953.png)

+ 这个表虽然满足了**第一范式**，但是也很明显的感受到它的冗余性，其中学生信息和课程信息是冗余的。以上表如果是需要确定主键，就得是学生+课程作为联合主键

**修改后，分为学生信息表、课程信息表、学生学分表**

**学生信息表：只代表学生的个人信息，主键使用id以防止重名**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281353142.png)

**课程信息表**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281353836.png)



### 第三范式（3NF）：独立性，消除传递依赖(非主键值不依赖于另一个非主键值)

- 在一个具有主键的表中，假设主键为A，其必然其他非主键都依赖于该主键，比如：B依赖于A，C依赖于A，D依赖于A；但同时：如果该表中的某个字段B的值一确定，就能够确定另一个字段的值C，则我们称为C依赖于B，那么，就出现了C依赖B，B依赖A——这就是**传递依赖**
- 则消除该传递依赖的的通常做法，就是将C依赖于B的数据，分离到另一个表中

**不良例子**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281355728.png)

+ 以上表既满足第一范式也满足第二范式，非主键字段也完全依赖于主键字段。但是，院系电话字段，其实是依赖院系字段的。也就是说，院系电话字段是非主键值，而依赖了另一个非主键值-院系。所以就不符合第三范式

**改良后**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281356998.png)

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202306281356934.png)

# 存储过程

## what

+ 存储过程是一些sql语句和控制语句组成的被封装起来的过程，它驻留在数据库中，可以被客户应用程序通过存储过程名字调用，也可以从另一个存储过程或触发器调用

## why

+ 执行速度快——存储过程只在创建时进行编译，以后每次执行存储过程都不需要重新编译，而一般SQL语句没执行一次就需编译一次，所以使用存储过程可提高数据库的执行速度
+ 减少网络通信量——当对数据库进行复杂操作时，（如对多个表进行insert、update、select、delete时）可将这些复杂操作用存储过程封装起来与数据库提供的事务处理结合一起使用。这些操作，如果用程序完成就是多条SQL语句，可能要多次连接数据库，而换成存储过程只需一次连接
+ 更强的适应性与复用性——存储过程可以重复使用，提高了可重用性，减少数据库开发人员的工作量

## 缺点

- 执行速度快——存储过程只在创建时进行编译，以后每次执行存储过程都不需要重新编译，而一般SQL语句没执行一次就需编译一次，所以使用存储过程可提高数据库的执行速度
- 减少网络通信量——当对数据库进行复杂操作时，（如对多个表进行insert、update、select、delete时）可将这些复杂操作用存储过程封装起来与数据库提供的事务处理结合一起使用。这些操作，如果用程序完成就是多条SQL语句，可能要多次连接数据库，而换成存储过程只需一次连接
- 更强的适应性与复用性——存储过程可以重复使用，提高了可重用性，减少数据库开发人员的工作量
- 可维护性差

## 存储过程(procedure)和函数(function)

**应用场景不同**

- 如果需要返回多个值和不返回值，就使用存储过程；如果只需要返回一个值，就使用函数
- 存储过程一般用于执行一个指定的动作，函数一般用于计算和返回一个值
- 可以再SQL内部调用函数来完成复杂的计算问题，但不能调用存储过程



# 常见问题

## Java项目如何防止SQL注入

### 什么是SQL注入？

SQL注入(SQL Injection)是一种代码注入技术,是通过把SQL命令插入到Web表单递交或输入域名或页面请求的查询字符串,最终达到欺骗服务器执行恶意的SQL命令。简单来说,SQL注入攻击者通过把SQL命令插入到Web表单提交或输入域名或页面请求的查询字符串,传入后端的SQL服务器执行。结果是可以执行恶意攻击者设计的任意SQL命令。例如,Web应用程序具有以下登录页面

```yaml
User Name: admin
Password: 1234
```

攻击者在密码框中输入

```
'1234 ' or '1'='1
```

之后构造的SQL查询为

```
SELECT * FROM users WHERE name='admin' AND password='1234 ' or '1'='1';
```

由于'或'1'='1 总是为真,所以可以绕过密码验证登录系统。SQL注入可以通过多种方式进行防范,如使用参数化的SQL语句、输入验证和过滤等方法

### 防止SQL注入方式

#### PreparedStatement防止SQL注入

使用PreparedStatement可以有效防止SQL注入攻击。PreparedStatement会先将SQL语句发送到数据库进行预编译,之后再将参数值单独传递,从而避免了SQL语句拼接的过程。例如,使用Statement时

```java
String sql = "SELECT * FROM users WHERE name = ? AND password = ?";
PreparedStatement stmt = connection.prepareStatement(sql);
stmt.setString(1, username); 
stmt.setString(2, password);
```

PreparedStatement会区分SQL语句字符串和参数值,用`?`作为占位符,之后调用setString()等方法设置参数,这样可以有效防止SQL注入

#### mybatis中#{}防止SQL注入

在MyBatis中,可以使用#{}来防止SQL注入。#{}是MyBatis提供的PreparedStatement的参数占位符。MyBatis会自动将#{}替换为? ,并且对用户传入的参数自动进行Escape处理,以防止SQL注入。例如

```java
<select id="findUser" parameterType="String" resultType="User">
  select * from user where name = #{name}
</select>
```

在Mapper接口中

```java
User findUser(String name);
```

在这里,传入的name参数会被直接传递给PreparedStatement作为参数,而不是拼接到SQL语句中,所以安全。如果使用${}进行拼接

#### 对请求参数的敏感词汇进行过滤

对用户请求参数中的敏感词汇进行过滤,可以防止多种注入攻击,包括SQL注入、XSS等。 常见的防范措施包括:

1. **构建敏感词汇库**,收集所有可能的敏感词汇,如delete、drop、script等。并定期更新。
2. **对用户请求参数进行遍历,判断参数值是否包含敏感词汇。** 可以用正则表达式或包含关系来判断。
3. **一旦发现参数值存在敏感词汇,可以采取以下措施**:

- 返回错误,拒绝请求
- 删除敏感词汇后,再进行后续处理
- 将敏感词汇替换为安全的占位符,如replace('delete', '***')

1. **对关键参数与业务规则进行校验**,例如长度、类型、允许范围等。
2. **考虑在边界处过滤**,如WAF、防火墙等。
3. **避免直接在SQL中拼接参数,应使用参数化查询**。
4. **输出时对敏感数据编码或替换**

#### nginx反向代理防止SQL注入

nginx作为反向代理服务器,可以实现一些防范SQL注入的措施:

**请求参数过滤** 可以使用nginx的ngx_http_rewrite_module模块,在server区域加入过滤规则,对请求参数中敏感字符进行过滤或拦截,例如

```java
if ($args ~* "select|insert|update|delete|drop|exec") {
 return 403;
}
```

**WAF功能** 启用nginx的Web应用防火墙功能,对疑似SQL注入的请求进行拦截,如检测特殊字符,语句规则等。

**访问控制** 通过nginx的access模块,禁止某些IP地址或子网段访问,限制请求频率,以防止滥用。

**隐藏数据库结构信息** nginx可以基于请求中的User-Agent等信息,显示不同的错误页面,避免泄露数据库元信息。

**连接数据库的用户权限控制** 只允许访问应用需要的最小权限集





aio是什么

为什么快照读需要undo log
