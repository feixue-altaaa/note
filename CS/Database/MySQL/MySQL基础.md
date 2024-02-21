# mysql和Oracle对比

## MySQL

+ **中小型数据库**
+ 开源
+ 安装简单（200M以内）
+ 一张表最多能存500万行数据
+ mysql+innodb 两亿数据量没问题

## Oracle

+ **大型数据库**
+ 收费，较为昂贵
+ 安装复杂（3G左右）
+ 一张表推荐存500万行数据

# 数据库基础

## 好处

| 存储方式 |                             优点                             |                     缺点                      |
| :------: | :----------------------------------------------------------: | :-------------------------------------------: |
|   内存   |                            速度快                            |        不能够永久保存,数据是临时状态的        |
|   文件   |                        实现数据持久化                        |           使用IO流操作文件, 不方便            |
|  数据库  | 实现数据持久化、方便存储和管理数据、使用统一的方式操作数据库 (SQL) | 占用资源,有些数据库需要付费(比如Oracle数据库) |

#### 持久化(persistence)

+ 把数据保存到可掉电式存储设备中以供之后使用。持久化的大多数时候是将内存中的数据存储在数据库中，当然也可以存储在磁盘文件、XML数据文件中

#### 方便管理数据、检索速度快（例如：快速的检索等）

#### 数据共享

+ 通过使用一个数据库，多个用户能够共享数据。每个用户可根据需要查询和更新这些数据，增强了数据的有效性和准确性

#### 数据保护

+ 数据库可以提供一系列功能来保护数据的安全性和完整性。这些功能包括访问控制、数据备份、数据加密等。这些功能可以帮助保护重要数据免遭非法访问和误操作

#### 数据一致性

+ 使用数据库可以提高数据一致性，即不同用户使用同一数据，可以避免数据的冲突和重复操作。如果不使用数据库，每个用户会独立地处理数据，这可能导致数据的不一致性

#### 数据可靠性

+ 通过数据库管理系统，数据存储在数据文件中，可以定期备份，使得数据在意外出现系统故障或硬件损坏时，不会出现数据丢失问题，从而保证数据的可靠性

#### 数据操作的规范性和可扩展性

+ 数据库支持SQL等规范化的数据操作语言，可以帮助控制数据操作的规范性。并且，数据库提供了容量和性能的可扩展性，可以根据需求进行扩展和升级，使其更适合于大型、高并发的应用场景

## 概念 

### DB

- 数据库（database）：存储数据的“仓库”。它保存了一系列有组织的数据

### DBMS

- 数据库管理系统（Database Management System）。数据库是通过DBMS创建和操作的容器

  ![img](https://img.mubu.com/document_image/32529e89-8f14-45cb-9add-32fe9a96ce48-4093910.jpg)

### SQL

- 结构化查询语言（Structure Query Language）：专门用来与数据库通信的语言

- 优点

  快速：使用SQL查询，用户可以快速有效地从数据库中检索大量记录

  无需编码：在标准SQL中，管理数据库系统非常容易。它不需要大量代码来管理数据库系统

  明确界定标准：ISO和ANSI是长期建立使用的SQL数据库标准

  可移植性：SQL可用于笔记本电脑，PC，服务器甚至某些手机

  互动语言：SQL是用于与数据库通信的域语言。它还用于在几秒钟内接收复杂问题的答案

  多个数据视图：使用SQL语言，用户可以对数据库结构进行不同的视图

- 关系

  ![img](https://img.mubu.com/document_image/fe4a2b68-d104-4b3f-a135-7d8975546567-4093910.jpg)

## MySQL的启动和停止

+ 通过计算机管理方式

  + 计算机--管理--服务--启动或停止MySQL服务

+ 命令行

  ```bash
  #启动
  net start mysql
  #停止
  net stop mysql
  ```

# 数据类型

## MySQL有哪些数据库类型？

### 数值类型

- 有包括 TINYINT、SMALLINT、MEDIUMINT、INT、BIGINT，分别表示 1 字节、2 字节、3 字节、4 字节、8 字节的整数类型
- 任何整数类型都可以加上 UNSIGNED 属性，表示无符号整数

- 任何整数类型都可以指定长度，但它不会限制数据的合法长度，仅仅限制了显示长度

- 还有包括 FLOAT、DOUBLE、DECIMAL 在内的小数类型


### 字符串类型

- 包括 VARCHAR、CHAR、TEXT、BLOB
- 注意：VARCHAR(n) 和 CHAR(n) 中的 n 并不代表字节个数，而是代表字符的个数


### 日期和时间类型

- 常用于表示日期和时间类型为 DATETIME、DATE 和 TIMESTAMP
- 尽量使用 TIMESTAMP，空间效率高于 DATETIME

## CHAR 和 VARCHAR 区别？

- 首先可以明确的是 CHAR 是定长的，而 VARCHAR 是可以变长
  - CHAR 会根据声明的字符串长度分配空间，并会使用空格对字符串右边进行尾部填充。所以在检索 CHAR 类型数据时尾部空格会被删除，如保存的是字符串 'char '，但最后查询到的是 'char'。又因为长度固定，所以存储效率高于 VARCHAR 类型

  - VARCHAR 在 MySQL 5.0 之后长度支持到 65535 字节，但会在数据开头使用额外 1~2 个字节存储字符串长度（列长度小于 255 字节时使用 1 字节表示，否则 2 字节），在结尾使用 1 字节表示字符串结束

- 再者，在存储方式上，CHAR 对英文字符（ASCII）占用 1 字节，对一个汉字使用用 2 字节。而 VARCHAR 对每个字符均使用 2 字节

  - 虽然 VARCHAR 是根据字符串长度分配存储空间的，但在内存中依旧使用声明长度进行排序等作业，故在使用时仍需综合考量字段长度


## CHAR 和 VARCHAR 如何选择？

- 对于经常变更的数据来说，CHAR 比 VARCHAR更好，因为 CHAR 不容易产生碎片
- 对于非常短的列或固定长度的数据（如 MD5），CHAR 比 VARCHAR 在存储空间上更有效率

- 使用时要注意只分配需要的空间，更长的列排序时会消耗更多内存

- 尽量避免使用 TEXT/BLOB 类型，查询时会使用临时表，导致严重的性能开销


## CHAR，VARCHAR 和 Text 的区别？

**长度区别**

- Char 范围是 0～255

- Varchar 最长是 64k（注意这里的 64k 是整个 row 的长度，要考虑到其它的 column，还有如果存在 not null 的时候也会占用一位，对不同的字符集，有效长度还不一样，比如 utf-8 的，最多 21845，还要除去别的column），但 Varchar 在一般情况下存储都够用了

- 如果遇到了大文本，考虑使用 Text，最大能到 4G（其中 TEXT 长度 65,535 bytes，约 64kb；MEDIUMTEXT 长度 16,777,215 bytes，约 16 Mb；而 LONGTEXT 长度 4,294,967,295 bytes，约 4Gb）


**效率区别**

+ 效率来说基本是 Char > Varchar > Text，但是如果使用的是 Innodb 引擎的话，推荐使用 Varchar 代替 Char

**默认值区别**

+ Char 和 Varchar 支持设置默认值，而 Text 不能指定默认值

# SQL通用语法

## 注释方式

```bash
-- 单行注释
--
#
-- 多行注释
/* 

*/
```

## SQL的分类

|     分类     |                             说明                             |
| :----------: | :----------------------------------------------------------: |
| 数据定义语言 | 简称DDL(Data Definition Language)，用来定义数据库对象：数据库，表，列等 |
| 数据操作语言 | 简称DML(Data Manipulation Language)，用来对数据库中表的记录进行更新 |
| 数据查询语言 | 简称DQL(Data Definition Language)，用来定义数据库对象：数据库，表，列等 |
| 数据控制语言 | 简称DCL(Date Control Language)，用来定义数据库的访问权限和安全级别，及创建用户 |

## DDL操作 数据库(CRUD)

### 创建数据库

```bash
/*
方式1 直接指定数据库名进行创建
默认数据库字符集为：latin1
*/
CREATE DATABASE db1;
/*
方式2 指定数据库名称，指定数据库的字符集
一般都指定为 utf8,与Java中的编码保持一致
*/
CREATE DATABASE db1_1 CHARACTER SET utf8;
```

### 查看/选择数据库

```bash
-- 切换数据库 从db1 切换到 db1_1
USE db1_1;

-- 查看当前正在使用的数据库
SELECT DATABASE();

-- 查看Mysql中有哪些数据库
SHOW DATABASES;

-- 查看一个数据库的定义信息
SHOW CREATE DATABASE db1_1;
```

### 修改数据库

```bash
-- 将数据库db1 的字符集 修改为 utf8
ALTER DATABASE db1 CHARACTER SET utf8;

-- 查看当前数据库的基本信息，发现编码已更改
SHOW CREATE DATABASE db1;
```

### 删除数据库

```bash
-- 删除某个数据库
-- 语法格式 drop database 数据库名称 将数据库从MySql中永久删除
DROP DATABASE db1_1;  -- 慎用
```

## DDL操作 数据表

### MySQL与Java数据类型对应

|        MySql 类型名         | GetColumnClassName 返回值 |                        返回的 Java 类                        |
| :-------------------------: | :-----------------------: | :----------------------------------------------------------: |
|     bit(1) (MySQL-5.0)      |            BIT            |                      java.lang.Boolean                       |
|   bit(大于1) (MySQL-5.0)    |            BIT            |                            byte[]                            |
|           tinyint           |          TINYINT          | 如果 tinyInt1isBit 配置设置为 true(默认为 true)，是java.lang.Boolean，存储空间为 1；否则是为 java.lang.Integer |
|        bool boolean         |          TINYINT          |          参见 TINYINT。这些是 TINYINT(1) 另一种写法          |
|   smallint(M) [unsigned]    |    SMALLINT [UNSIGNED]    |               java.lang.Integer(不管是否无符)                |
|   mediumint(M) [unsigned]   |   MEDIUMINT [UNSIGNED]    |                      java.lang.Integer                       |
|  int integer(M) [unsigned]  |    INTEGER [UNSIGNED]     |         java.lang.Integer；无符的话是 java.lang.Long         |
|    bigint(M) [unsigned]     |     BIGINT [UNSIGNED]     |       java.lang.Long；无符的话是 java.math.BigInteger        |
|         float(M,D)          |           FLOAT           |                       java.lang.Float                        |
|         double(M,B)         |          DOUBLE           |                       java.lang.Double                       |
|        decimal(M,D)         |          DECIMAL          |                     java.math.BigDecimal                     |
|            date             |           DATE            |                        java.sql.Date                         |
|          datetime           |         DATETIME          |                      java.sql.Timestamp                      |
|        timestamp(M)         |         TIMESTAMP         |                      java.sql.Timestamp                      |
|            time             |           TIME            |                        java.sql.Time                         |
|          year(2/4)          |           YEAR            | 如果 yearIsDateType 配置设置为 false，返回的对象类型为 java.sql.Short；如果设置为 true(默认为 true)，返回的对象类型是 java.sql.Date，其具体时间是为一月一日零时零分 |
|           char(M)           |           CHAR            | java.lang.String(除非该列字符集设置为 BINARY，那样返回 byte[]) |
|     varchar(M) [binary]     |          VARCHAR          | java.lang.String(除非该列字符集设置为 BINARY，那样返回 byte[]) |
|          binary(M)          |          BINARY           |                            byte[]                            |
|        varbinary(M)         |         VARBINARY         |                            byte[]                            |
|          tinyblob           |         TINYBLOB          |                            byte[]                            |
|          tinytext           |          VARCHAR          |                       java.lang.String                       |
|            blob             |           BLOB            |                            byte[]                            |
|            text             |          VARCHAR          |                       java.lang.String                       |
|         mediumblob          |        MEDIUMBLOB         |                            byte[]                            |
|         mediumtext          |          VARCHAR          |                       java.lang.String                       |
|          longblob           |         LONGBLOB          |                            byte[]                            |
|          longtext           |          VARCHAR          |                       java.lang.String                       |
| enum('value1','value2',...) |           CHAR            |                       java.lang.String                       |
| set('value1','value2',...)  |           CHAR            |                       java.lang.String                       |

> 注意：MySQL中的 char类型与 varchar类型，都对应了 Java中的字符串类型，区别在于： - char类型是固定长度的： 根据定义的字符串长度分配足够的空间。 - varchar类型是可变长度的： 只使用字符串长度所需的空间
>
> 适用场景：
>
> - char类型适合存储 固定长度的字符串，比如 密码 ，性别一类
> - varchar类型适合存储 在一定范围内，有长度变化的字符串

### 创建表

```bash
/*
创建表的语法格式
create table 表名(
   字段名称1 字段类型(长度),
   字段名称2 字段类型,
   字段名称3 字段类型 最后一个列不要添加逗号
);
*/

-- 实例
-- 创建表
CREATE TABLE category(
    cid INT,
    cname VARCHAR(20)
);

-- 创建一个表结构与 test1 相同的 test2表
CREATE TABLE test2 LIKE test1;
```

### 查看表

```bash
-- 查看当前数据库中的所有表名
SHOW TABLES;

-- 显示当前数据表的结构
DESC category;

-- 查看创建表的SQL语句
SHOW CREATE TABLE category;
```

### 删除表

```bash
-- 直接删除 test1 表
DROP TABLE test1;

-- 先判断 再删除test2表
DROP TABLE IF EXISTS test2;
```

### 修改表

#### **修改表名**

```bash
-- 修改表名称 语法格式: rename table 旧表名 to 新表名
RENAME TABLE category TO category1;
```

#### **修改表的字符集**

```bash
-- 修改表的字符集为 gbk 
-- 语法格式: alter table 表名 character set 字符集
ALTER TABLE category1 CHARACTER SET gbk; 
```

#### **向表中添加列， 关键字 ADD**

```bash
-- 语法格式: alter table 表名 add 字段名称 字段类型(长度)
-- 添加分类描述字段
ALTER TABLE category1 ADD cdesc VARCHAR(20);
```

#### **修改表中列的 数据类型或长度 ， 关键字 MODIFY**

```bash
-- alter table 表名 modify 字段名称 字段类型
-- 修改cdesc 字段的长度为 50
ALTER TABLE category1 MODIFY cdesc VARCHAR(50); -- 修改字段长度
ALTER TABLE category1 MODIFY cdesc CHAR(20); -- 修改字段类型
```

#### **修改列名称 , 关键字 CHANGE**

```bash
-- 语法格式: alter table 表名 change 旧列名 新列名 类型(长度)
-- 修改cdesc字段 名称改为 description varchar(30)
ALTER TABLE category1 CHANGE cdesc description VARCHAR(30);
```

#### **删除列 ，关键字 DROP**

```bash
-- 语法格式: alter table 表名 drop 列名
ALTER TABLE category1 DROP description;
```

## DML 操作表中数据

### 插入数据

```bash
-- insert into 表名 （字段名1，字段名2...） values(字段值1，字段值2...);

-- 方式1： 插入全部字段， 将所有字段名都写出来
INSERT INTO student (sid,sname,age,sex,address) VALUES(1,'孙悟空',18,'男','花果山');

-- 方式2 插入全部字段 不写字段名
INSERT INTO student VALUES(2,'孙悟饭',5,'男','地球');

-- 方式3 插入指定字段
INSERT INTO student (sid,sname) VALUES(3,'蜘蛛精');

---- 注意事项
-- 1.值与字段必须对应 个数&数据类型&长度 都必须一致
INSERT INTO student (sid,sname,age,sex,address) VALUES(4,'孙悟空',18,'男','花果山');

-- 2.在插入 varchar char date 类型的时候,必须要使用 单引号 或者双引号进行包裹
INSERT INTO student (sid,sname,age,sex,address) VALUES(4,'孙悟空',18,'男','花果山');

-- 3.如果插入空值 可以忽略不写 或者写 null
INSERT INTO student (sid,sname) VALUES(5,'唐僧');
INSERT INTO student (sid,sname,age,sex,address) VALUES(6,'八戒',NULL,NULL,NULL);
```

### 更改数据

```bash
-- 语法格式1: 不带条件的修改
update 表名 set 列名 = 值

-- 语法格式2: 带条件的修改
update 表名 set 列名 = 值 [where 条件表达式：字段名 = 值 ]

-- 例 一次修改多个列， 将sid为 2 的学员，年龄改为 20，地址改为 北京
UPDATE student SET age = 20 , address = '大唐' WHERE sid = 5;
```

### 删除数据

```bash
-- 语法格式1: 删除所有数据
delete from 表名

-- 语法格式2： 指定条件 删除数据
delete from 表名 [where 字段名 = 值]

-- 如果要删除表中的所有数据,有两种做法
-- delete from 表名; 不推荐. 有多少条记录 就执行多少次删除操作. 效率低
-- truncate table 表名: 推荐. 先删除整张表, 然后再重新创建一张一模一样的表. 效率高
```

## DQL 查询表中数据

## having和where的区别

having子句与where都是设定条件筛选的语句，有相似之处也有区别。

- having是在分组后对数据进行过滤
- where是在分组前对数据进行过滤
- having后面可以使用聚合函数
- where后面不可以使用聚合

在查询过程中执行顺序：**from>where>group（含聚合）>having>order>select。**

聚合语句(sum,min,max,avg,count)要比having子句优先执行，所有having后面可以使用聚合函数。而where子句在查询过程中执行优先级别优先于聚合语句(sum,min,max,avg,count)，所有where条件中不能使用聚合函数

```sql
select sum(num) as rmb from order where id>10;
```

//先查询出id大于10的数据，再执行聚合语句sum(num)

//执行以下语句会报错，因为where子句先于sum(num)执行，执行where子句的时候还没有sum(num)，所以会报错

```sql
select sum(num) as rmb from order where sum(num)>10;
```

对分组数据再次判断时要用having

```sql
select reports，count(*) from employees group by reports having count(*) > 4;
```

//首先查询了select reports，count(*) from employees group by reports，在此基础上查找count(*) > 4的数据

- 聚合函数：例如SUM, COUNT, MAX, AVG等，这些函数和其它函数的根本区别就是它们一般作用在多条记录上
- HAVING子句可以让我们直接筛选成组后的各组数据，也可以在聚合后对组记录进行筛选，而WHERE子句在聚合前先筛选记录，也就是说作用在GROUP BY 子句和HAVING子句前

### 简单查询

```bash
-- 语法格式
select 列名 from 表名

-- 别名查询，使用关键字 as
SELECT
eid AS '编号',
ename AS '姓名' ,
sex AS '性别',
salary AS '薪资',
hire_date '入职时间', -- AS 可以省略
dept_name '部门名称'
FROM emp;

-- 去重操作 关键字 distinct
SELECT DISTINCT dept_name FROM emp;

-- 将我们的员工薪资数据 +1000 进行展示
SELECT ename, salary+1000 AS salary FROM emp;
```

### 条件查询

#### **运算符**

##### **比较运算符**

|      运算符       |                             说明                             |
| :---------------: | :----------------------------------------------------------: |
| > < <= >= = <> != |              大于、小于、大于(小于)等于、不等于              |
| BETWEEN ...AND... | 显示在某一区间的值 例如: 2000-10000之间： Between 2000 and 10000 |
|     IN(集合)      | 集合表示多个值,使用逗号分隔,例如: name in (悟空，八戒) in中的每个数据都会作为一次条件,只要满足条件就会显示 |
|    LIKE '%张%'    |                           模糊查询                           |
|      IS NULL      |           查询某一列为NULL的值, 注: 不能写 = NULL            |

> MySQL 使用三值逻辑 —— TRUE, FALSE 和 UNKNOWN。任何与 NULL 值进行的比较都会与第三种值 UNKNOWN 做比较。这个“任何值”包括 NULL 本身！这就是为什么 MySQL 提供 IS NULL 和 IS NOT NULL 两种操作来对 NULL 特殊判断
>
> https://leetcode.cn/problems/find-customer-referee/solutions/20888/xun-zhao-yong-hu-tui-jian-ren-by-leetcode/?envType=study-plan-v2&envId=sql-free-50

##### **逻辑运算符**

| 运算符 |       说明       |
| :----: | :--------------: |
| And && | 多个条件同时成立 |
|   Or   | 多个条件任一成立 |
|  Not   |   不成立，取反   |

```bash
-- 查询薪水价格是3600或7200或者20000的所有员工信息
-- 方式1 使用 or
SELECT * FROM emp WHERE salary = 3600 OR salary = 7200 OR salary = 20000;


-- 方式2 使用 in() 匹配括号中的参数
SELECT * FROM emp WHERE salary IN(3600,7200,20000);
```

##### 模糊查询 通配符

| 通配符 |          说明          |
| :----: | :--------------------: |
|   %    | 表示匹配任意多个字符串 |
|   _    |   表示匹配 一个字符    |

```bash
-- 查询以'孙'开头的所有员工信息
SELECT * FROM emp WHERE ename LIKE '孙%';

-- 查询第二个字为'兔'的所有员工信息
SELECT * FROM emp WHERE ename LIKE '_兔%';
```



# 数据处理

- 查询

  - 基础查询

    - 查询字段

      - 

        ![img](https://img.mubu.com/document_image/342ec7d7-91e0-4e96-8a1d-64bcd06a3bc9-4093910.jpg)

    - 查询常量值

      - 

        ![img](https://img.mubu.com/document_image/efc01f66-d860-4773-8818-2a02d8dd9a1f-4093910.jpg)

    - 查询表达式

      - 

        ![img](https://img.mubu.com/document_image/c9d19423-e804-4b45-b539-7a31e7cea102-4093910.jpg)

    - 查询函数

      - 

        ![img](https://img.mubu.com/document_image/419af115-ff41-46ad-bd38-2ee9e38e51cf-4093910.jpg)

    - 

  - 重命名

    - 空格

    - AS

      - 

        ![img](https://img.mubu.com/document_image/9f013899-59eb-4150-94d3-20263ec35bf1-4093910.jpg)

  - 字符串

    - 

      ![img](https://img.mubu.com/document_image/6df1a0fd-b4ba-478f-8d9c-87fd7e15d94e-4093910.jpg)

  - 过滤

    - 

      ![img](https://img.mubu.com/document_image/f9efffcf-2022-4ddc-8033-833e668b427b-4093910.jpg)

  - 条件查询

    - 按条件表达式筛选

      - 

        ![img](https://img.mubu.com/document_image/40a72c97-2d5e-4c80-8f7a-5071a5a7f9c1-4093910.jpg)

    - 按逻辑表达式筛选

      - 

        ![img](https://img.mubu.com/document_image/d1283c19-025b-47ad-a39d-0efc066c8925-4093910.jpg)

      - 

        ![img](https://img.mubu.com/document_image/f52fb0cb-73b3-41c7-a6cf-6661bb5ab57e-4093910.jpg)

    - 模糊查询

      - like

        - 

          ![img](https://img.mubu.com/document_image/e4e6696f-4f5d-4b5e-adad-2bffc1f3e75e-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/b81371c8-74b5-4b3f-b8cc-9b3213605d62-4093910.jpg)

      - between and

        - 

          ![img](https://img.mubu.com/document_image/45ffa656-fdf1-44c6-be5f-c4cf753cfc8f-4093910.jpg)

      - in

        - 

          ![img](https://img.mubu.com/document_image/9064d615-64c4-49aa-a78c-fe7e87acd1c8-4093910.jpg)

      - is null is not null

        - 

          ![img](https://img.mubu.com/document_image/d9f2ce8f-3b26-4114-b4a5-e7da7003ef1d-4093910.jpg)

      - 安全等于 <=>

        - 

          ![img](https://img.mubu.com/document_image/38d91026-eff2-44c2-9752-4faddd5d8015-4093910.jpg)

  - 排序查询

    - 按表达式排名

      - 

        ![img](https://img.mubu.com/document_image/cf211874-22ec-4b07-8a7b-13ba28ed03ac-4093910.jpg)

    - 按别名排名

      - 

        ![img](https://img.mubu.com/document_image/281e6fe1-65d7-4ea0-ab0a-b38fe93f421a-4093910.jpg)

    - 按函数排序

      - 

        ![img](https://img.mubu.com/document_image/2558645d-67b4-469d-addc-b171634d9a80-4093910.jpg)

    - 按多个字段排序

      - 

        ![img](https://img.mubu.com/document_image/6b5d7ac3-b524-4eb6-84b4-523c2d369d54-4093910.jpg)

  - 分组查询

    - 语法

      - 

        ![img](https://img.mubu.com/document_image/ac2345c8-0898-43fd-a363-decee5ed8362-4093910.jpg)

    - 特点

      - 

        ![img](https://img.mubu.com/document_image/5e59c3f3-b908-495e-b35c-2b6475541d63-4093910.jpg)

    - 代码

      - 简单筛选

        - 

          ![img](https://img.mubu.com/document_image/b5a40729-aaf0-41cb-b80a-29a5e07a4b8c-4093910.jpg)

      - 有条件的筛选

        - 

          ![img](https://img.mubu.com/document_image/210faa79-d1a2-4276-9dd8-45ff55c85cac-4093910.jpg)

      - 添加分组的筛选条件

        - 

          ![img](https://img.mubu.com/document_image/96586793-a92d-4837-9ffd-efcba8d23c98-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/69dd2652-9234-46db-bd98-9361277da471-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/40b6408a-4690-4a7a-8362-6790fa64899f-4093910.jpg)

      - 按表达式或函数筛选

        - 

          ![img](https://img.mubu.com/document_image/dbb0b241-d310-47ec-a0ef-ad11cb950af6-4093910.jpg)

      - 按多个字段分组

        - 

          ![img](https://img.mubu.com/document_image/36b3d4da-6c71-4dd4-b8a2-2d4130d01624-4093910.jpg)

      - 添加排序

        - 

          ![img](https://img.mubu.com/document_image/ca0b67ea-a30a-4a6e-b54a-fdb95fbb39ba-4093910.jpg)

  - 笛卡尔乘积

    - 如果连接条件省略或无效则会出现

  - 连接查询

    - 定义

      - 

        ![img](https://img.mubu.com/document_image/00087133-bfd2-4dc5-b8b0-f53e83d47361-4093910.jpg)

    - 分类

      - sql92

        - 定义

          - 

            ![img](https://img.mubu.com/document_image/73c5d4a9-819b-4e53-b8f3-e0373fc9c497-4093910.jpg)

        - 语法

          - 等值连接

            - 语法

              - 

                ![img](https://img.mubu.com/document_image/06401734-9fbf-47e7-a375-86978cdafdff-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/d0f738fa-a59e-4106-ae9b-b4f23bb77875-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/498f6d34-47fb-4641-a902-9a2ae0be03c1-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/b2b84600-4dfa-4074-be7e-a2ba0765c413-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/e5315eee-936e-4e31-81a1-d71c64e14104-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/8dc75e53-f098-44b3-857b-d4f796884ce5-4093910.jpg)

            - 特点

              - 

                ![img](https://img.mubu.com/document_image/5f8dee47-e060-467e-8cdb-82f8f5fd7a56-4093910.jpg)

          - 非等值连接

            - 语法

              - 

                ![img](https://img.mubu.com/document_image/46edd463-abad-42f1-b577-09ea757f9596-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/5a24b465-2018-4e09-b278-a9bd95289b62-4093910.jpg)

              - 

          - 自连接

            - 语法

              - 

                ![img](https://img.mubu.com/document_image/7329489f-7315-4c09-9126-22669005639f-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/8d3ce561-9920-4732-b623-e10a94dc59bd-4093910.jpg)

              - 

      - sql99【推荐使用】

        - 定义

          - 

            ![img](https://img.mubu.com/document_image/d0c4e963-146f-4613-9e96-f3ca091fae7f-4093910.jpg)

        - 内连接

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/bfad82f2-6a1f-40ec-817d-8a8d04f2e05c-4093910.jpg)

          - 特点

            - 

              ![img](https://img.mubu.com/document_image/048c1763-559a-4ecc-b5b3-486eca06f1e4-4093910.jpg)

          - 等值连接

            - 

              ![img](https://img.mubu.com/document_image/50a9d7bd-de9b-4522-a3a4-a389c22af73b-4093910.jpg)

            - 三表连接

              ![img](https://img.mubu.com/document_image/4cc5b00c-b600-46bb-b441-cd2aee3219df-4093910.jpg)

            - 

          - 非等值连接

            - 

              ![img](https://img.mubu.com/document_image/1e7d81d0-b2f9-4364-bb20-69d34661f804-4093910.jpg)

            - 

          - 自连接

            - 

              ![img](https://img.mubu.com/document_image/aeff595a-e930-4290-a985-7e4e3bcb96ec-4093910.jpg)

        - 外连接

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/6c5f9ac7-e1e9-418e-9a08-12d7a560480c-4093910.jpg)

            - 

              ![img](https://img.mubu.com/document_image/8b94b82a-2b2c-4bea-aa4e-15dafaadd911-4093910.jpg)

            - 

          - 特点

            - 

              ![img](https://img.mubu.com/document_image/7115d23c-f22c-4a47-a168-b356e7e55097-4093910.jpg)

        - 交叉连接

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/8d35f357-b086-4eca-897c-b7065c73b537-4093910.jpg)

            - 

              ![img](https://img.mubu.com/document_image/5fc895b3-a757-455e-a9ba-ccb2a39d1817-4093910.jpg)

  - 子查询

    - 定义

      - 

        ![img](https://img.mubu.com/document_image/3ef50fab-7e46-47e4-8c43-032b7e2ae2e4-4093910.jpg)

    - 分类

      - 按位置

        - select后面

          - 仅仅支持标量子查询

          - 代码

            - 

              ![img](https://img.mubu.com/document_image/c730ea43-eb62-4271-95ae-887ba6da1117-4093910.jpg)

        - from后面

          - 表子查询

          - 代码

            - 

              ![img](https://img.mubu.com/document_image/a0816455-fa65-4a85-9686-51ade1ed18c3-4093910.jpg)

        - where或having后面

          - 

          - 代码

            - 标量子查询

              - 

                ![img](https://img.mubu.com/document_image/ba5e7791-1e80-477f-8490-575c423e2d3e-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/973e0422-9bc3-471a-9978-ac7da4c0e0a9-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/eebed9d3-4f65-4d73-8001-d6632a6894bf-4093910.jpg)

            - 列子查询

              - 

                ![img](https://img.mubu.com/document_image/45560a86-8176-43ac-a465-4feb2964eb1e-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/9d4e384c-2759-4b74-9fc2-87ffcbab9e5e-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/0e54689f-3b50-49d0-b955-be07856f8c0a-4093910.jpg)

              - 

            - 行子查询

              - 

                ![img](https://img.mubu.com/document_image/14d69777-1849-41d9-9e97-cdefc759da81-4093910.jpg)

        - exists后面

          - 标量子查询

          - 列子查询

          - 行子查询

          - 表子查询

          - 代码

            - 

              ![img](https://img.mubu.com/document_image/9560dd8a-f089-43b7-9095-2bf54e102274-4093910.jpg)

            - 

              ![img](https://img.mubu.com/document_image/40416c6a-514b-4bd5-a78d-a633ab515734-4093910.jpg)

      - 按结果集的行列

        - 标量子查询（单行子查询）
          - 结果集为一行一列
        - 列子查询（多行子查询）
          - 结果集为多行一列
        - 行子查询
          - 结果集为多行多列
        - 表子查询
          - 结果集为多行多列

  - 联合查询

    - 定义

      - 合并、联合，将多次查询结果合并成一个结果

    - 语法

      - 查询语句1
      - union 【all】
      - 查询语句2
      - union 【all】
      - ...

    - 意义

      - 1、将一条比较复杂的查询语句拆分成多条语句
      - 2、适用于查询多个表的时候，查询的列基本是一致

    - 特点

      - 1、要求多条查询语句的查询列数必须一致
      - 2、要求多条查询语句的查询的各列类型、顺序最好一致
      - 3、union 去重，union all包含重复项

    - 案例

      - 

        ![img](https://img.mubu.com/document_image/6e364922-1e9a-46f5-a47b-16a9f82bf3e4-4093910.jpg)

  - 分页查询

    - 应用场景

      - 当要查询的条目数太多，一页显示不全

    - 语法

      - select 查询列表
      - from 表
      - limit 【offset，】size;

    - 注意

      - offset代表的是起始的条目索引，默认从0卡死
      - size代表的是显示的条目数

    - 公式

      - 假如要显示的页数为page，每一页条目数为size
      - select 查询列表
      - from 表
      - limit (page-1)*size,size;

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/5014d7cb-470a-4b8b-a3d9-bf0a8000bbaf-4093910.jpg)

  - 执行顺序

    - 

      ![img](https://img.mubu.com/document_image/0d639d35-3ade-4849-8013-1c2350a1a73e-4093910.jpg)

  - 例题

    - 查询每个国家下的部门个数大于 2 的国家编号

      - 

        ![img](https://img.mubu.com/document_image/395316e1-358d-4969-8046-743810cad085-4093910.jpg)

    - 查询和姓名中包含u的员工在相同部门的员工的员工号和姓名

      - 

        ![img](https://img.mubu.com/document_image/bcd9e5ac-6a35-4ca4-aa41-784ba72a93d0-4093910.jpg)

    - 查询在部门的location_id 为1700的部门工作的员工的员工号

      - 

        ![img](https://img.mubu.com/document_image/158b39d2-1bbc-4195-9991-fcb36bd11e40-4093910.jpg)

    - 查询平均工资最高的部门的manager 的详细信息

      - 

        ![img](https://img.mubu.com/document_image/57696f50-7bca-4e7d-800e-e8a1aeae90cc-4093910.jpg)

    - 各部门中。最高工资中最低的哪个部门中的最低工资是多少

      - 

        ![img](https://img.mubu.com/document_image/7e0ebd42-bb9d-48b2-8794-3f4cf01f3b7f-4093910.jpg)

- 插入

  - 方式一

    - 语法

      - insert into 表名(字段名,...) values(值,...);

    - 特点

      - 1、要求值的类型和字段的类型要一致或兼容
      - 2、字段的个数和顺序不一定与原始表中的字段个数和顺序一致，但必须保证值和字段一一对应
      - 3、假如表中有可以为null的字段，注意可以通过以下两种方式插入null值
        - ①字段和值都省略
        - ②字段写上，值使用null
      - 4、字段和值的个数必须一致
      - 5、字段名可以省略，默认所有列

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/2198a64a-8c2e-4147-844f-2c3fb1ba4c26-4093910.jpg)

      - 

        ![img](https://img.mubu.com/document_image/ca0a9ddd-3af8-4d0d-ba9b-f145afec20b3-4093910.jpg)

  - 方式二

    - 语法
      - insert into 表名 set 字段=值,字段=值,...;

  - 两种方式 的区别

    - 1.方式一支持一次插入多行，语法如下：

      - insert into 表名【(字段名,..)】 values(值，..),(值，...),...;

      - 

        ![img](https://img.mubu.com/document_image/f643d183-74de-4985-8c98-6aaa414edb6b-4093910.jpg)

    - 2.方式一支持子查询，语法如下

      - insert into 表名

      - 查询语句;

      - 

        ![img](https://img.mubu.com/document_image/2c9d815b-a4d8-4ec4-a012-5e998bed0f97-4093910.jpg)

- 修改

  - 修改单表的记录 ★

    - 语法

      - 语法：update 表名 set 字段=值,字段=值 【where 筛选条件】;

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/6b3e10c6-be43-4b95-9d08-9d03ca6da155-4093910.jpg)

  - 修改多表的记录【补充】

    - 语法

      - update 表1 别名
      - left|right|inner join 表2 别名
      - on 连接条件
      - set 字段=值,字段=值
      - 【where 筛选条件】;

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/4d38eaba-d7e5-4cbf-b419-63ea4c80d59b-4093910.jpg)

- 删除

  - 方式一——delete

    - 删除单表的记录★

      - 语法

        - delete from 表名 【where 筛选条件】【limit 条目数】

      - 代码

        - 

          ![img](https://img.mubu.com/document_image/e39c4301-a0c9-4331-be76-2534fff36a9e-4093910.jpg)

    - 级联删除[补充]

      - 语法

        - delete 别名1,别名2 from 表1 别名
        - inner|left|right join 表2 别名
        - on 连接条件
        - 【where 筛选条件】

      - 代码

        - 

          ![img](https://img.mubu.com/document_image/0315d47b-f72a-46e1-8b94-222703ccac6c-4093910.jpg)

  - 方式二：使用truncate

    - 语法
      - truncate table 表名

  - 两种方式的区别【面试题】★

    - 1.truncate删除后，如果再插入，标识列从1开始，delete删除后，如果再插入，标识列从断点开始
    - 2.delete可以添加筛选条件，truncate不可以添加筛选条件
    - 3.truncate效率较高
    - 4.truncate没有返回值，delete可以返回受影响的行数
    - 5.truncate不可以回滚，delete可以回滚

- DDL语言（数据定义语言）

  - 库的管理

    - 创建库

      - 

        ![img](https://img.mubu.com/document_image/ea9af520-0aad-453d-a160-3c2c433672d2-4093910.jpg)

    - 库的修改

      - 更改库的字符集

        - 

          ![img](https://img.mubu.com/document_image/5da037c6-618a-4962-b7d3-34d4864a6213-4093910.jpg)

    - 库的删除

      - 

        ![img](https://img.mubu.com/document_image/d0cdad8f-9711-4e80-ac85-286eb1f45ff0-4093910.jpg)

  - 表的管理

    - 表的创建

      - 语法

        - create table 【if not exists】 表名(

        - 字段名 字段类型 【约束】,

        - 字段名 字段类型 【约束】,

        - 。。。

        - 字段名 字段类型 【约束】

        - )

        - 

          ![img](https://img.mubu.com/document_image/459865d1-c0b8-4ec9-b80c-92509f96809a-4093910.jpg)

    - 表的修改

      - 1.添加列

        - alter table 表名 add column 列名 类型 【first|after 字段名】;

        - 

          ![img](https://img.mubu.com/document_image/97ce9909-600e-4989-b670-bd058bc0de51-4093910.jpg)

      - 2.修改列的类型或约束

        - alter table 表名 modify column 列名 新类型 【新约束】;

        - 

          ![img](https://img.mubu.com/document_image/240858ed-b794-4c55-9520-a4c3026023c8-4093910.jpg)

      - 3.修改列名

        - alter table 表名 change column 旧列名 新列名 类型;

        - 

          ![img](https://img.mubu.com/document_image/ff176dce-7f6d-46e6-8297-3bf5b112de32-4093910.jpg)

      - 4 .删除列

        - alter table 表名 drop column 列名;

        - 

          ![img](https://img.mubu.com/document_image/2c1249f2-0db9-4877-bffd-fb58677df3d6-4093910.jpg)

      - 5.修改表名

        - alter table 表名 rename 【to】 新表名;

        - 

          ![img](https://img.mubu.com/document_image/7cd6d7d0-42a5-4b9d-b9c3-07e4159f771b-4093910.jpg)

    - 表的删除

      - 语法

        - drop table【if exists】 表名;

        - 

          ![img](https://img.mubu.com/document_image/99721c8c-9ab1-42c4-bedb-8509a2444214-4093910.jpg)

    - 表的复制

      - 仅仅复制表的结构

        - 

          ![img](https://img.mubu.com/document_image/6fc0c81f-38b9-4741-a6e1-9d1bbecaea2b-4093910.jpg)

      - 复制结构+数据

        - 

          ![img](https://img.mubu.com/document_image/7783002a-81db-4e5a-a0b0-956d840db51d-4093910.jpg)

      - 只复制部分内容

        - 

          ![img](https://img.mubu.com/document_image/63f2d1dc-e73e-4f85-bc6a-287c8693d823-4093910.jpg)

      - 只复制某些字段

        - 

          ![img](https://img.mubu.com/document_image/4aaff090-35a3-4cde-bd03-34c67c6bdea4-4093910.jpg)

  - 数据类型

    - 数值型

      - 整型
        - 内容
          - tinyint、smallint、mediumint、int/integer、bigint
          - 1          2        3          4            8
        - 特点
          - ①都可以设置无符号和有符号，默认有符号，通过unsigned设置无符号
          - ②如果超出了范围，会报out of range异常，插入临界值
          - ③长度可以不指定，默认会有一个长度
          - 长度代表显示的最大宽度，如果不够则左边用0填充，但需要搭配zerofill，并且默认变为无符号整型
      - 浮点型
        - 定点数
          - decimal(M,D)
        - 浮点数
          - float(M,D)   4
          - double(M,D)  8
        - 特点
          - ①M代表整数部位+小数部位的个数，D代表小数部位
          - ②如果超出范围，则报out or range异常，并且插入临界值
          - ③M和D都可以省略，但对于定点数，M默认为10，D默认为0
          - ④如果精度要求较高，则优先考虑使用定点数

    - 字符型

      - 内容

        - char、varchar、binary、varbinary、enum、set、text、blob
        - char：固定长度的字符，写法为char(M)，最大长度不能超过M，其中M可以省略，默认为1
        - varchar：可变长度的字符，写法为varchar(M)，最大长度不能超过M，其中M不可以省略

      - enum

        - 

          ![img](https://img.mubu.com/document_image/01b35652-5c9c-4ad2-a9e7-dda2e9c55043-4093910.jpg)

      - set

        - 

          ![img](https://img.mubu.com/document_image/994947a8-149d-4992-9c67-b177f315f2ec-4093910.jpg)

    - 日期型

      - year年
      - date日期
      - time时间
      - datetime 日期+时间          8
      - timestamp 日期+时间         4   比较容易受时区、语法模式、版本的影响，更能反映当前时区的真实时间

  - 常见约束

    - 常见的约束

      - 内容
        - NOT NULL：非空，该字段的值必填
        - UNIQUE：唯一，该字段的值不可重复
        - DEFAULT：默认，该字段的值不用手动插入有默认值
        - CHECK：检查，mysql不支持
        - PRIMARY KEY：主键，该字段的值不可重复并且非空  unique+not null
        - FOREIGN KEY：外键，该字段的值引用了另外的表的字段
      - 主键和唯一
        - 区别
          - ①、一个表至多有一个主键，但可以有多个唯一
          - ②、主键不允许为空，唯一可以为空
        - 相同点
          - 都具有唯一性
          - 都支持组合键，但不推荐
      - 外键
        - 1、用于限制两个表的关系，从表的字段值引用了主表的某字段值
        - 2、外键列和主表的被引用列要求类型一致，意义一样，名称无要求
        - 3、主表的被引用列要求是一个key（一般就是主键）
        - 4、插入数据，先插入主表，删除数据，先删除从表
        - 可以通过以下两种方式来删除主表的记录
          - \#方式一：级联删除
          - ALTER TABLE stuinfo ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE CASCADE;
          - \#方式二：级联置空
          - ALTER TABLE stuinfo ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE SET NULL;

    - 创建表时添加约束

      - 语法

        - create table 表名(
        - 字段名 字段类型 not null,#非空
        - 字段名 字段类型 primary key,#主键
        - 字段名 字段类型 unique,#唯一
        - 字段名 字段类型 default 值,#默认
        - constraint 约束名 foreign key(字段名) references 主表（被引用列）
        - )

      - 注意

        - 

          ![img](https://img.mubu.com/document_image/b62295a1-8f62-4343-8894-c5c88ee16a65-4093910.jpg)

      - 代码

        - 

          ![img](https://img.mubu.com/document_image/a18e1189-17c6-4178-830f-8a8860871529-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/b17656fb-5a37-49a5-aa0a-ae1c5c10c997-4093910.jpg)

    - 修改表时添加或删除约束

      - 1、非空
        - 添加非空
          - alter table 表名 modify column 字段名 字段类型 not null;
        - 删除非空
          - alter table 表名 modify column 字段名 字段类型 ;
      - 2、默认
        - 添加默认
          - alter table 表名 modify column 字段名 字段类型 default 值;
        - 删除默认
          - alter table 表名 modify column 字段名 字段类型 ;
      - 3、主键
        - 添加主键
          - 表级约束
            - alter table 表名 add【 constraint 约束名】 primary key(字段名);
          - 列级约束
            - alter table 表名 modify column 字段名 字段类型 primary key;
        - 删除主键
          - alter table 表名 drop primary key;
      - 4、唯一
        - 添加唯一
          - 表级约束
            - alter table 表名 add【 constraint 约束名】 unique(字段名);
          - 列级约束
            - alter table 表名 modify column 字段名 字段类型 unique;
        - 删除唯一
          - alter table 表名 drop index 索引名;
      - 5、外键
        - 添加外键
          - alter table 表名 add【 constraint 约束名】 foreign key(字段名) references 主表（被引用列）;
        - 删除外键
          - alter table 表名 drop foreign key 约束名;

    - 自增长列

      - 特点

        - 1、不用手动插入值，可以自动提供序列值，默认从1开始，步长为1，auto_increment_increment，如果要更改起始值：手动插入值，如果要更改步长：更改系统变量，set auto_increment_increment=值;
        - 2、一个表至多有一个自增长列
        - 3、自增长列只能支持数值型
        - 4、自增长列必须为一个key

      - 创建表时设置自增长列

        - 语法

          - create table 表(

          - 字段名 字段类型 约束 auto_increment

          - )

          - 

            ![img](https://img.mubu.com/document_image/e9c28dfa-4bae-4cac-b69d-842f55b03cfe-4093910.jpg)

      - 修改表时设置自增长列

        - 语法

          - alter table 表 modify column 字段名 字段类型 约束 auto_increment

          - 

            ![img](https://img.mubu.com/document_image/46ff2931-8a63-4b7e-a708-70440134311e-4093910.jpg)

      - 删除自增长列

        - 语法

          - alter table 表 modify column 字段名 字段类型 约束 

          - 

            ![img](https://img.mubu.com/document_image/05bbe375-7af2-4c81-a8da-c4feaa41a4d8-4093910.jpg)

- TCL语言（事务控制语言）

  - 事务

    - 含义

      - 事务：一条或多条sql语句组成一个执行单位，一组sql语句要么都执行要么都不执行

    - 特点（ACID）

      - A 原子性：一个事务是不可再分割的整体，要么都执行要么都不执行
      - C 一致性：一个事务可以使数据从一个一致状态切换到另外一个一致的状态
      - I 隔离性：一个事务不受其他事务的干扰，多个事务互相隔离的
      - D 持久性：一个事务一旦提交了，则永久的持久化到本地

    - 事务的使用步骤 ★

      - 分类
        - 了解：
        - 隐式（自动）事务：没有明显的开启和结束，本身就是一条事务可以自动提交，比如insert、update、delete
        - 显式事务：具有明显的开启和结束
      - 使用显式事务
        - ①开启事务
        - set autocommit=0;
        - start transaction;#可以省略
        - ②编写一组逻辑sql语句
        - 注意：sql语句支持的是insert、update、delete
        - 设置回滚点：
        - savepoint 回滚点名;
        - ③结束事务
        - 提交：commit;
        - 回滚：rollback;
        - 回滚到指定的地方：rollback to 回滚点名;

    - 并发事务

      - 1、事务的并发问题是如何发生的？

        - 多个事务 同时 操作 同一个数据库的相同数据时

      - 2、并发问题都有哪些？

        - 脏读：一个事务读取了其他事务还没有提交的数据，读到的是其他事务“更新”的数据
        - 不可重复读：一个事务多次读取，结果不一样
        - 幻读：一个事务读取了其他事务还没有提交的数据，只是读到的是 其他事务“插入”的数据

      - 3、如何解决并发问题

        - 通过设置隔离级别来解决并发问题

      - 4、隔离级别

        - 

          ![img](https://img.mubu.com/document_image/2e669de6-fca0-41c1-9769-718e80574440-4093910.jpg)

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/9e72f4c6-9fb0-4751-a23b-f8069ad4c654-4093910.jpg)

      - 

        ![img](https://img.mubu.com/document_image/77fc6e40-86a7-4bba-8c6d-d6bd19d7192e-4093910.jpg)

- 其他

  - 视图

    - 含义

      - mysql5.1版本出现的新特性，本身是一个虚拟表，它的数据来自于表，通过执行时动态生成。

    - 好处

      - 好处：
      - 1、简化sql语句
      - 2、提高了sql的重用性
      - 3、保护基表的数据，提高了安全性

    - 创建

      - 语法
        - create view 视图名
        - as
        - 查询语句;

    - 修改

      - 方式一

        - create or replace view 视图名

        - as

        - 查询语句;

        - 

          ![img](https://img.mubu.com/document_image/67960f2d-dc8e-4129-b06d-ddda4cf0b06d-4093910.jpg)

      - 方式二

        - alter view 视图名

        - as

        - 查询语句

        - 

          ![img](https://img.mubu.com/document_image/6e53a260-c40a-4881-a636-175e474b6185-4093910.jpg)

    - 删除

      - 语法
        - drop view 视图1，视图2,...;

    - 查看

      - desc 视图名;
      - show create view 视图名;

    - 使用

      - 1.插入

        - insert

        - 

          ![img](https://img.mubu.com/document_image/b396a533-ddf9-410b-ad74-3dc2974b7740-4093910.jpg)

      - 2.修改

        - update

        - 

          ![img](https://img.mubu.com/document_image/06a484ff-eff7-411d-a67f-27269a0c89e0-4093910.jpg)

      - 3.删除

        - delete

        - 

          ![img](https://img.mubu.com/document_image/ca1ddb21-4eb4-4dd2-aff0-c9c641080582-4093910.jpg)

      - 4.查看

        - select

      - 注意

        - 视图一般用于查询的，而不是更新的，所以具备以下特点的视图都不允许更新
        - ①包含分组函数、group by、distinct、having、union、
        - ②join
        - ③常量视图
        - ④where后的子查询用到了from中的表
        - ⑤用到了不可更新的视图

    - 视图和表的对比

      - 

        ![img](https://img.mubu.com/document_image/c1285beb-9480-4412-bb63-686f9d1db8bf-4093910.jpg)

    - 代码

      - 

        ![img](https://img.mubu.com/document_image/0a27340c-1aae-4ded-b067-7afffb467db2-4093910.jpg)

      - 

        ![img](https://img.mubu.com/document_image/d5d23e7e-6d3d-4aac-8369-15fa2bbaf2e9-4093910.jpg)

      - 

        ![img](https://img.mubu.com/document_image/a6006aba-7703-43a0-a9f9-dbafd50e247d-4093910.jpg)

  - 变量

    - 系统变量

      - 定义

        - 变量由系统提供的，不用自定义

      - 分类

        - 全局变量
          - 服务器层面上的，必须拥有super权限才能为系统变量赋值，作用域为整个服务器，也就是针对于所有连接（会话）有效
        - 会话变量
          - 服务器为每一个连接的客户端都提供了系统变量，作用域为当前的连接（会话）

      - 语法

        - ①查看系统变量
          - show 【global|session 】variables like ''; 如果没有显式声明global还是session，则默认是session
        - ②查看指定的系统变量的值
          - select @@【global|session】.变量名; 如果没有显式声明global还是session，则默认是session
        - ③为系统变量赋值
          - 方式一
            - set 【global|session 】 变量名=值; 如果没有显式声明global还是session，则默认是session
          - 方式二
            - set @@global.变量名=值;
            - set @@变量名=值；

      - 代码

        - 

          ![img](https://img.mubu.com/document_image/a6623950-c383-4b1c-9835-61325fa1c2d6-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/47e308fb-1c44-4101-a095-991225f6b1c5-4093910.jpg)

    - 自定义变量

      - 用户变量
        - 内容
          - 作用域
            - 针对于当前连接（会话）生效
          - 位置
            - begin end里面，也可以放在外面
        - 使用
          - ①声明并赋值
            - set @变量名=值;
            - set @变量名:=值;
            - select @变量名:=值;
          - ②更新值
            - 方式一
              - set @变量名=值;
              - set @变量名:=值;
              - select @变量名:=值;
            - 方式二
              - select xx into @变量名 from 表;
          - ③使用
            - select @变量名;
      - 局部变量
        - 内容
          - 作用域
            - 仅仅在定义它的begin end中有效
          - 位置
            - 只能放在begin end中，而且只能放在第一句
        - 使用
          - ①声明
            - declare 变量名 类型 【default 值】;
          - ②赋值或更新
            - 方式一
              - set 变量名=值;
              - set 变量名:=值;
              - select @变量名:=值;
              - 方式二
                - select xx into 变量名 from 表;
          - ③使用
            - select 变量名;

  - 存储过程和函数

    - 存储过程

      - 内容

        - 一组预先编译好的SQL语句的集合，理解成批处理语句

      - 优点

        - 1、提高代码的重用性
        - 2、简化操作
        - 3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

      - 创建

        - 语法

          - 

            ![img](https://img.mubu.com/document_image/eb7b2aea-d111-4792-8bf1-efb4296d2888-4093910.jpg)

        - 注意

          - 

            ![img](https://img.mubu.com/document_image/eea2cbc7-384a-4b2e-97da-92c1af695354-4093910.jpg)

      - 调用

        - 语法

          - 

            ![img](https://img.mubu.com/document_image/7a8b053e-5b60-44bf-bcbf-87272beb758b-4093910.jpg)

        - 案例演示

          - 空参列表

            - 

              ![img](https://img.mubu.com/document_image/d909b5e2-20e9-48f8-be8a-38de08c6e28e-4093910.jpg)

          - 创建带in模式参数的存储过程

            - 

              ![img](https://img.mubu.com/document_image/c788246e-e128-4a7d-9dff-89d0dea6889f-4093910.jpg)

            - 

              ![img](https://img.mubu.com/document_image/c3270255-c118-4107-b075-d1052d8e030e-4093910.jpg)

            - 

          - 创建out 模式参数的存储过程

            - 

              ![img](https://img.mubu.com/document_image/e41d66f3-d936-4f67-af5c-4a9dd598ae42-4093910.jpg)

            - 

              ![img](https://img.mubu.com/document_image/af7e62f5-d782-4443-8a0e-5fc57671909c-4093910.jpg)

            - 

          - 创建带inout模式参数的存储过程

            - 

              ![img](https://img.mubu.com/document_image/be2e9a93-c5e4-43fa-a217-42b92f567239-4093910.jpg)

      - 删除存储过程

        - 

          ![img](https://img.mubu.com/document_image/49a5f95f-1714-47ef-9b44-1874c4716b11-4093910.jpg)

      - 查看存储过程的信息

        - 

          ![img](https://img.mubu.com/document_image/e35e8aae-5b1e-4a10-9229-7e28d91cfcff-4093910.jpg)

    - 函数

      - 自定义函数

        - 定义

          - 一组预先编译好的SQL语句的集合，理解成批处理语句

        - 优点

          - 1、提高代码的重用性
          - 2、简化操作
          - 3、减少了编译次数并且减少了和数据库服务器的连接次数，提高了效率

        - 创建

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/5c9ca4b7-3d07-45c3-9eb4-ff40377ff59a-4093910.jpg)

          - 注意

            - 

              ![img](https://img.mubu.com/document_image/70393171-921b-4e3e-b4ed-71c8aa28f701-4093910.jpg)

        - 调用

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/601f2262-86f5-4205-bf96-259e4c1d9709-4093910.jpg)

          - 案例演示

            - 无参有返回

              - 

                ![img](https://img.mubu.com/document_image/10fffeb5-9434-4ddf-b113-2daff723fe81-4093910.jpg)

            - 有参有返回

              - 

                ![img](https://img.mubu.com/document_image/9e5f1edf-328c-4406-9f89-ecb4d1cc3d9c-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/14ef6d13-194b-49ee-bf75-43cf669afd28-4093910.jpg)

        - 查看函数

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/8e86884b-770b-4899-895a-9a866c06d22e-4093910.jpg)

        - 删除函数

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/76ca3071-f7b1-4294-830a-c95f9f82c01a-4093910.jpg)

        - 案例

          - 

            ![img](https://img.mubu.com/document_image/2f360b1e-fa70-4547-9241-191710c03b34-4093910.jpg)

      - 系统函数

        - ISNULL

          - 判断某字段或表达式是否为null，如果是，则返回1，否则返回0

        - IFNULL

          - 功能：判断某字段或表达式是否为null，如果为null 返回指定的值，否则返回原本的值

          - 

            ![img](https://img.mubu.com/document_image/72e9909d-cdd8-406e-9a99-4c57579abbfe-4093910.jpg)

        - 单行函数

          - 字符函数

            - concat拼接

              - 

                ![img](https://img.mubu.com/document_image/646fabd8-163e-45d1-aeb7-6264d213f72f-4093910.jpg)

            - substr截取子串

              - 

                ![img](https://img.mubu.com/document_image/2c62e747-89a2-40b1-bb65-2954c90348f3-4093910.jpg)

            - upper转换成大写

              - 

                ![img](https://img.mubu.com/document_image/fe49d4fa-085c-4fe2-a1bf-4b793de248c1-4093910.jpg)

            - lower转换成小写

              - 

                ![img](https://img.mubu.com/document_image/9eb9e82b-7639-401d-a223-9025c6467fe3-4093910.jpg)

            - trim去前后指定的空格和字符

              - 

                ![img](https://img.mubu.com/document_image/ce3fd622-2584-4865-ad21-16ac80fe01e9-4093910.jpg)

            - ltrim去左边空格

            - rtrim去右边空格

            - replace替换

              - 

                ![img](https://img.mubu.com/document_image/26151008-e0dc-4b43-b549-f9710c9f6eda-4093910.jpg)

            - lpad左填充

              - 

                ![img](https://img.mubu.com/document_image/49b5ee7a-f998-426e-8ebf-ff2f16aaa82c-4093910.jpg)

            - rpad右填充

            - instr返回子串第一次出现的索引

              - 

                ![img](https://img.mubu.com/document_image/7e2b7260-c008-4ca9-8ddd-6a19685224ca-4093910.jpg)

            - length 获取字节个数

              - 

                ![img](https://img.mubu.com/document_image/40fe3d44-6428-4607-81f0-eb116814ed9d-4093910.jpg)

          - 数学函数

            - round 四舍五入

              - 

                ![img](https://img.mubu.com/document_image/d9d213c7-6094-4d9a-9aae-e4bbb469fff9-4093910.jpg)

            - rand 随机数

            - floor向下取整

              - 

                ![img](https://img.mubu.com/document_image/ebd7dfc6-3dc7-4836-8aee-0a483bb3743a-4093910.jpg)

            - ceil向上取整

              - 

                ![img](https://img.mubu.com/document_image/0e1a3504-1819-447b-9fe8-bd04b52c9625-4093910.jpg)

            - mod取余

              - 

                ![img](https://img.mubu.com/document_image/2d6a21d2-eb1a-4438-bc97-bbd662df5ea0-4093910.jpg)

            - truncate截断

              - 

                ![img](https://img.mubu.com/document_image/ef6875ab-8043-49a8-8c44-de03e8a8e331-4093910.jpg)

          - 日期函数

            - now当前系统日期+时间

              - 

                ![img](https://img.mubu.com/document_image/84a96381-ee64-44ac-b880-1dbe23b4b93d-4093910.jpg)

              - 

                ![img](https://img.mubu.com/document_image/7d1b2b7d-a4b7-492c-aac6-4c2a183f5263-4093910.jpg)

            - curdate当前系统日期

              - 

                ![img](https://img.mubu.com/document_image/a4bf38a1-82f1-4e59-8701-b2acb432e23d-4093910.jpg)

            - curtime当前系统时间

              - 

                ![img](https://img.mubu.com/document_image/ac35a46e-d8d7-4cbf-9eac-13795d8b1147-4093910.jpg)

            - str_to_date 将字符转换成日期

              - 

                ![img](https://img.mubu.com/document_image/f36dd339-bf53-4ff7-9ecd-87f7125266dd-4093910.jpg)

            - date_format将日期转换成字符

              - 

                ![img](https://img.mubu.com/document_image/ad97e2e2-2f40-4e7f-bd9b-35b4232a2f74-4093910.jpg)

            - year

            - month

            - day

            - hour

            - monthname

              - 返回英文月份

            - datediff

              - 返回两个日期相差的天数

          - 流程控制函数

            - if 处理双分支

              - 

                ![img](https://img.mubu.com/document_image/f5786e0a-0b76-4800-8fdd-fdd6124a2dbb-4093910.jpg)

            - case语句 处理多分支

              - 情况1：处理等值判断

                - 

                  ![img](https://img.mubu.com/document_image/bb3ffbfc-1d66-40a4-ae09-cab62eb28534-4093910.jpg)

              - 情况2：处理条件判断

                - 

                  ![img](https://img.mubu.com/document_image/99dbf6f0-c7ad-4b67-94ef-17a30c6a30f6-4093910.jpg)

          - 其他函数

            - version版本

              - 

                ![img](https://img.mubu.com/document_image/66de9c73-c66e-4d41-bff3-524c8631af1c-4093910.jpg)

            - database当前库

            - user当前连接用户

            - password返回该字符的密码形式

            - md5 用于加密

        - 分组函数

          - sum 求和

            - 

              ![img](https://img.mubu.com/document_image/f09670f9-2eb3-43d9-bbd5-7a5b525b7fd5-4093910.jpg)

          - max 最大值

          - min 最小值

          - avg 平均值

          - count 计数

          - 特点

            - 1、以上五个分组函数都忽略null值，除了count(*)

            - 2、sum和avg一般用于处理数值型，max、min、count可以处理任何数据类型

            - 3、都可以搭配distinct使用，用于统计去重后的结果

              - 

                ![img](https://img.mubu.com/document_image/532cd5a7-1e56-4645-beac-62442d3cd28c-4093910.jpg)

            - 4、count的参数可以支持：字段、*、常量值，一般放1

            - 

            - 5、count函数：count(字段)：统计该字段非空值的个数，count(*):统计结果集的行数

              - 案例：查询每个部门的员工个数

              - 1 xx    10

              - 2 dd    20

              - 3 mm    20

              - 4 aa    40

              - 5 hh    40

              - count(1):统计结果集的行数

              - 

                ![img](https://img.mubu.com/document_image/0aa269c1-0784-4190-9643-941029e348d9-4093910.jpg)

            - 效率上：

              - MyISAM存储引擎，count(*)最高
              - InnoDB存储引擎，count(*)和count(1)效率>count(字段)

            - 6、 和分组函数一同查询的字段，要求是group by后出现的字段

    - 区别

      - 存储过程：可以有0个返回，也可以有多个返回，适合做批量插入、批量更新
      - 函数：有且仅有1 个返回，适合做处理数据后返回一个结果

  - 流程控制结构

    - 分支结构

      - 分类

        - if函数

          - 

            ![img](https://img.mubu.com/document_image/0d434382-0dab-44bc-b3c7-fb87421f846b-4093910.jpg)

        - case结构

          - 

            ![img](https://img.mubu.com/document_image/631e2870-35b4-4ea4-aa42-aec03f326c9c-4093910.jpg)

        - if结构

          - 

            ![img](https://img.mubu.com/document_image/f6483d16-47cd-4875-b8a8-03f6aa54a9cb-4093910.jpg)

      - 案例

        - 

          ![img](https://img.mubu.com/document_image/442885a2-123f-46e9-a1b5-e59948b9b9ac-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/0921217d-e737-45f5-8db7-66af1c0ed12e-4093910.jpg)

        - 

          ![img](https://img.mubu.com/document_image/f479e231-ec9c-47fb-9970-f55645a7576a-4093910.jpg)

        - 

    - 循环结构

      - 分类

        - while

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/b77ceae5-0172-4a69-8f81-d50fb86d31b7-4093910.jpg)

        - loop

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/1e41c5a4-92af-4df2-86f1-de3f6a67cf78-4093910.jpg)

        - repeat

          - 语法

            - 

              ![img](https://img.mubu.com/document_image/297f854c-8594-4d0c-aa01-f1ad1c381272-4093910.jpg)

      - 循环控制

        - iterate类似于 continue，继续，结束本次循环，继续下一次
        - leave 类似于  break，跳出，结束当前所在的循环

      - 案例

        - 没有添加循环控制语句

          - 

            ![img](https://img.mubu.com/document_image/e2b31adf-42b4-4ad2-bc1e-eade58b83cf7-4093910.jpg)

        - 添加leave语句

          - 

            ![img](https://img.mubu.com/document_image/73570355-b4e2-49bb-b10f-72d9f1f21776-4093910.jpg)

        - 添加iterate语句

          - 

            ![img](https://img.mubu.com/document_image/90a080c9-7750-4e12-92ff-b567d65080ad-4093910.jpg)

- 关键字

  - AS

    - 重命名

  - DESCRIBE

    - 

      ![img](https://img.mubu.com/document_image/48f20d0a-9e95-4fb7-a327-a3db1e67adb9-4093910.jpg)

  - DISTINCT

    - 去重

      - 

        ![img](https://img.mubu.com/document_image/9d073878-4ff9-422c-8aed-1f80aa97e10b-4093910.jpg)