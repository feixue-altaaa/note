# 简介

+ MyBatis-Plus（简称 MP）是一个 MyBatis的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生

  > 愿景
  > 我们的愿景是成为 MyBatis 最好的搭档，就像魂斗罗中的 1P、2P，基友搭配，效率翻倍

# Mybatis 和 Mybatis Plus 的区别

- MyBatis
  - 所有SQL语句全部自己写
  - 手动解析实体关系映射转换为MyBatis内部对象注入容器
  - 不支持Lambda形式调用
- Mybatis Plus
  - 强大的条件构造器,满足各类使用需求
  - 内置的Mapper,通用的Service,少量配置即可实现单表大部分CRUD操作
  - 支持Lambda形式调用
  - 提供了基本的CRUD功能,连SQL语句都不需要编写
  - 自动解析实体关系映射转换为MyBatis内部对象注入容器

# 特性

- 无侵入：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- 损耗小：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- 强大的 CRUD 操作：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分CRUD 操作，更有强大的条件构造器，满足各类使用需求
- 支持 Lambda 形式调用：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错支持
- 主键自动生成：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- 支持 ActiveRecord 模式：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- 支持自定义全局通用操作：支持全局通用方法注入（ Write once, use anywhere ）
- 内置代码生成器：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- 内置分页插件：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- 分页插件支持多种数据库：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- 内置性能分析插件：可输出 SQL 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- 内置全局拦截插件：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

# 支持数据库

- 任何能使用MyBatis进行 CRUD, 并且支持标准 SQL 的数据库，具体支持情况如下
- MySQL，Oracle，DB2，H2，HSQL，SQLite，PostgreSQL，SQLServer，Phoenix，Gauss ，ClickHouse，Sybase，OceanBase，Firebird，Cubrid，Goldilocks，csiidb
- 达梦数据库，虚谷数据库，人大金仓数据库，南大通用(华库)数据库，南大通用数据库，神通数据库，瀚高数据库

# 框架结构

![1](D:\课件\java教程\mybatisplus\资料\1.jpg)

# 常用注解

## @TableName

> 经过以上的测试，在使用MyBatis-Plus实现基本的CRUD时，我们并没有指定要操作的表，只是在Mapper接口继承BaseMapper时，设置了泛型User，而操作的表为user表
> 由此得出结论，MyBatis-Plus在确定操作的表时，由BaseMapper的泛型决定，即实体类型决定，且默认操作的表名和实体类型的类名一致

+ 通过@TableName解决实体类类型的类名和要操作的表的表名不一致问题

![1673955340378](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1673955340378.png)

+ 通过全局配置解决通用前缀问题
+ 在开发的过程中，我们经常遇到以上的问题，即实体类所对应的表都有固定的前缀，例如t_或tbl_此时，可以使用MyBatis-Plus提供的全局配置，为实体类所对应的表名设置默认的前缀，那么就不需要在每个实体类上通过@TableName标识实体类对应的表

```java
mybatis-plus:
configuration:
# 配置MyBatis日志
log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
global-config:
db-config:
# 配置MyBatis-Plus操作表的默认前缀
table-prefix: t_
```

## @TableId

> 经过以上的测试，MyBatis-Plus在实现CRUD时，会默认将id作为主键列，并在插入数据时，默认
> 基于雪花算法的策略生成id

### 问题
- 若实体类和表中表示主键的不是id，而是其他字段，例如uid，MyBatis-Plus会自动识别uid为主键列吗？
- 我们实体类中的属性id改为uid，将表中的字段id也改为uid，测试添加功能程序抛出异常，Field 'uid' doesn't have a default value，说明MyBatis-Plus没有将uid作为主键赋值

### 通过@TableId解决问题

+ 在实体类中uid属性上通过@TableId将其标识为主键，即可成功执行SQL语句

![1673956311717](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1673956311717.png)

### @TableId的value属性

- 若实体类中主键对应的属性为id，而表中表示主键的字段为uid，此时若只在属性id上添加注解@TableId，则抛出异常Unknown column 'id' in 'field list'，即MyBatis-Plus仍然会将id作为表的主键操作，而表中表示主键的是字段uid
- 此时需要通过@TableId注解的value属性，**指定表中的主键字段**，@TableId("uid")或@TableId(value="uid")

### @TableId的type属性

+ type属性用来**定义主键策略**

|            值            |                             描述                             |
| :----------------------: | :----------------------------------------------------------: |
| IdType.ASSIGN_ID（默认） |   基于雪花算法的策略生成数据id，与数据库id是否设置自增无关   |
|       IdType.AUTO        | 使用数据库的自增策略，注意，该类型请确保数据库设置了id自增，否则无效 |

### 配置全局主键策略

```java
mybatis-plus:
configuration:
# 配置MyBatis日志
log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
global-config:
db-config:
# 配置MyBatis-Plus操作表的默认前缀
table-prefix: t_
# 配置MyBatis-Plus的主键策略
id-type: auto
```

### 雪花算法

#### 背景

- 需要选择合适的方案去应对数据规模的增长，以应对逐渐增长的访问压力和数据量
- 数据库的扩展方式主要包括：业务分库、主从复制，数据库分表

#### 数据库分表

+ 将不同业务数据分散存储到不同的数据库服务器，能够支撑百万甚至千万用户规模的业务，但如果业务继续发展，同一业务的单表数据也会达到单台数据库服务器的处理瓶颈。例如，淘宝的几亿用户数据，如果全部存放在一台数据库服务器的一张表中，肯定是无法满足性能要求的，此时就需要对单表数据进行拆分。

+ 单表数据拆分有两种方式：垂直分表和水平分表。示意图如下：

![1673956638534](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1673956638534.png)

#### 垂直分表

+ 垂直分表适合将表中某些不常用且占了大量空间的列拆分出去。例如，前面示意图中的 nickname 和 description 字段，假设我们是一个婚恋网站，用户在筛选其他用户的时候，主要是用 age 和 sex 两个字段进行查询，而 nickname 和 description 两个字段主要用于展示，一般不会在业务查询中用到。description 本身又比较长，因此我们可以将这两个字段独立到另外一张表中，这样在查询 age 和 sex 时，就能带来一定的性能提升
#### 水平分表

- 水平分表适合表行数特别大的表，有的公司要求单表行数超过 **5000 万**就必须进行分表，这个数字可以作为参考，但并不是绝对标准，关键还是要看表的访问性能。对于一些比较复杂的表，可能超过 1000万就要分表了；而对于一些简单的表，即使存储数据超过 1 亿行，也可以不分表。但不管怎样，当看到表的数据量达到千万级别时，作为架构师就要警觉起来，因为这很可能是架构的性能瓶颈或者隐患
- 水平分表相比垂直分表，会引入更多的复杂性，例如要求全局唯一的数据id该如何处理

#### 主键自增

- 以最常见的用户 ID 为例，可以按照 1000000 的范围大小进行分段，1 ~ 999999 放到表 1中，1000000 ~ 1999999 放到表2中，以此类推
- **复杂点**：**分段大小的选取**。分段太小会导致切分后子表数量过多，增加维护复杂度；分段太大可能会导致单表依然存在性能问题，一般建议分段大小在 100 万至 2000 万之间，具体需要根据业务选取合适的分段大小
- **优点**：可以随着数据的增加平滑地扩充新的表。例如，现在的用户是 100 万，如果增加到 1000 万，只需要增加新的表就可以了，**原有的数据不需要动**
- **缺点**：**分布不均匀**。假如按照 1000 万来进行分表，有可能某个分段实际存储的数据量只有 1 条，而另外一个分段实际存储的数据量有 1000 万条

#### 取模

- 同样以用户 ID 为例，假如我们一开始就规划了 10 个数据库表，可以简单地用 user_id % 10 的值来表示数据属的数据库表编号，ID 为 985 的用户放到编号为 5 的子表中，ID 为 10086 的用户放到编号为 6 的子表中
- **复杂点：初始表数量的确定**。表数量太多维护比较麻烦，表数量太少又可能导致单表性能存在问题
- **优点：表分布比较均匀**
- **缺点：扩充新的表很麻烦，所有数据都要重分布**

#### 雪花算法

- 雪花算法是由Twitter公布的分布式主键生成算法，它能够保证不同表的主键的不重复性，以及相同表的主键的有序性
- **核心思想**：长度共64bit（一个long型）。
  - 首先是一个符号位，1bit标识，由于long基本类型在Java中是带符号的，最高位是符号位，正数是0，负数是1，所以id一般是正数，最高位是0。
  - 41bit时间截(毫秒级)，存储的是时间截的差值（当前时间截 - 开始时间截)，结果约等于69.73年。
  - 10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID，可以部署在1024个节点）。
  - 12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID）

![1673956903603](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1673956903603.png)

+ **优点：整体上按照时间自增排序，并且整个分布式系统内不会产生ID碰撞，并且效率较高**

## @TableField

+ 经过以上的测试，我们可以发现，MyBatis-Plus在执行SQL语句时，要保证实体类中的属性名和表中的字段名一致，如果实体类中的属性名和字段名不一致的情况，会出现什么问题呢？

+ 情况1

  若实体类中的属性使用的是**驼峰命名**风格，而表中的字段使用的是下划线命名风格

  例如实体类属性userName，表中字0段user_name

  此时MyBatis-Plus会自动将下划线命名风格转化为驼峰命名风格

+ 情况2

  若实体类中的属性和表中的字段不满足情况1

  例如实体类属性name，表中字段username

  此时需要在实体类属性上使用@TableField("username")设置属性所对应的字段名

## @TableLogic

- 物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除的数据
- 逻辑删除：假删除，将对应数据中代表是否被删除字段的状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录
- 使用场景：可以进行数据恢复

### 实现逻辑删除
- step1：数据库中创建逻辑删除状态列，设置默认值为0
- step2：实体类中添加逻辑删除属性

![1673963775858](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1673963775858.png)

- step3：测试
- 测试删除功能，真正执行的是修改
- UPDATE t_user SET is_deleted=1 WHERE id=? AND is_deleted=0
- 测试查询功能，被逻辑删除的数据默认不会被查询
- SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0

# 条件构造器和常用接口

## wapper介绍

![1674020265009](C:\Users\qls\AppData\Roaming\Typora\typora-user-images\1674020265009.png)

- Wrapper ： 条件构造抽象类，最顶端父类
- AbstractWrapper ： 用于查询条件封装，生成 sql 的 where 条件
- QueryWrapper ： 查询条件封装
- UpdateWrapper ： Update 条件封装
- AbstractLambdaWrapper ： 使用Lambda 语法
- LambdaQueryWrapper ：用于Lambda语法使用的查询Wrapper
- LambdaUpdateWrapper ： Lambda 更新封装Wrapper

## QueryWrapper

### 组装查询条件

```java
public void test01(){
    //查询用户名包含a，年龄在20到30之间，并且邮箱不为null的用户信息
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE
    is_deleted=0 AND (username LIKE ? AND age BETWEEN ? AND ? AND email IS NOT NULL)
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("username", "a")
        .between("age", 20, 30)
        .isNotNull("email");
    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}
```

### 组装排序条件

```java
@Test
public void test02(){
    //按年龄降序查询用户，如果年龄相同则按id升序排列
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE
    is_deleted=0 ORDER BY age DESC,id ASC
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper
        .orderByDesc("age")
        .orderByAsc("id");
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

### 组装删除条件

```java
@Test
public void test03(){
    //删除email为空的用户
    //DELETE FROM t_user WHERE (email IS NULL)
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.isNull("email");
    //条件构造器也可以构建删除语句的条件
    int result = userMapper.delete(queryWrapper);
    System.out.println("受影响的行数：" + result);
}
```

### 组装select子句

```java
@Test
public void test05() {
    //查询用户信息的username和age字段
    //SELECT username,age FROM t_user
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.select("username", "age");
    //selectMaps()返回Map集合列表，通常配合select()使用，避免User对象中没有被查询到的列值
    为null
        List<Map<String, Object>> maps = userMapper.selectMaps(queryWrapper);
    maps.forEach(System.out::println);
}
```

### 实现子查询

```java
@Test
public void test06() {
    //查询id小于等于3的用户信息
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE (id IN
    (select id from t_user where id <= 3))
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.inSql("id", "select id from t_user where id <= 3");
    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}
```

## UpdateWrapper

```java
@Test
public void test07() {
    //将（年龄大于20或邮箱为null）并且用户名中包含有a的用户信息修改
    //组装set子句以及修改条件
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    //lambda表达式内的逻辑优先运算
    updateWrapper
        .set("age", 18)
        .set("email", "user@atguigu.com")
        .like("username", "a")
        .and(i -> i.gt("age", 20).or().isNull("email"));
    //这里必须要创建User对象，否则无法应用自动填充。如果没有自动填充，可以设置为null
    //UPDATE t_user SET username=?, age=?,email=? WHERE (username LIKE ? AND
    (age > ? OR email IS NULL))
        //User user = new User();
        //user.setName("张三");
        //int result = userMapper.update(user, updateWrapper);
        //UPDATE t_user SET age=?,email=? WHERE (username LIKE ? AND (age > ? OR
        email IS NULL))
        int result = userMapper.update(null, updateWrapper);
    System.out.println(result);
}
```

## condition

> 在真正开发的过程中，组装条件是常见的功能，而这些条件数据来源于用户输入，是可选的，因此我们在组装这些条件时，必须先判断用户是否选择了这些条件，若选择则需要组装该条件，若没有选择则一定不能组装，以免影响SQL执行的结果思

### 思路一

```java
@Test
public void test08() {
    //定义查询条件，有可能为null（用户未输入或未选择）
    String username = null;
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    //StringUtils.isNotBlank()判断某字符串是否不为空且长度不为0且不由空白符(whitespace)
    构成
        if(StringUtils.isNotBlank(username)){
            queryWrapper.like("username","a");
        }
    if(ageBegin != null){
        queryWrapper.ge("age", ageBegin);
    }
    if(ageEnd != null){
        queryWrapper.le("age", ageEnd);
    }
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE (age >=
    ? AND age <= ?)
        List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

### 思路二

> 上面的实现方案没有问题，但是代码比较复杂，我们可以使用带condition参数的重载方法构建查询条件，简化代码的编写

```java
@Test
public void test08UseCondition() {
    //定义查询条件，有可能为null（用户未输入或未选择）
    String username = null;
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    //StringUtils.isNotBlank()判断某字符串是否不为空且长度不为0且不由空白符(whitespace)构成
    queryWrapper
        .like(StringUtils.isNotBlank(username), "username", "a")
        .ge(ageBegin != null, "age", ageBegin)
        .le(ageEnd != null, "age", ageEnd);
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE (age >=
    ? AND age <= ?)
        List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

## LambdaQueryWrapper

```java
@Test
public void test09() {
    //定义查询条件，有可能为null（用户未输入）
    String username = "a";
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    //避免使用字符串表示字段，防止运行时错误
    queryWrapper
        .like(StringUtils.isNotBlank(username), User::getName, username)
        .ge(ageBegin != null, User::getAge, ageBegin)
        .le(ageEnd != null, User::getAge, ageEnd);
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
```

## LambdaUpdateWrapper

```java
@Test
public void test10() {
    //组装set子句
    LambdaUpdateWrapper<User> updateWrapper = new LambdaUpdateWrapper<>();
    updateWrapper
        .set(User::getAge, 18)
        .set(User::getEmail, "user@atguigu.com")
        .like(User::getName, "a")
        .and(i -> i.lt(User::getAge, 24).or().isNull(User::getEmail)); //lambda
    表达式内的逻辑优先运算
        User user = new User();
    int result = userMapper.update(user, updateWrapper);
    System.out.println("受影响的行数：" + result);
}
```

# 插件

## 分页插件

> MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

### 添加配置类

```java
@Configuration
@MapperScan("com.atguigu.mybatisplus.mapper") //可以将主类中的注解移到此处
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

### 测试

```java
@Test
public void testPage(){
    //设置分页参数
    Page<User> page = new Page<>(1, 5);
    userMapper.selectPage(page, null);
    //获取分页数据
    List<User> list = page.getRecords();
    list.forEach(System.out::println);
    System.out.println("当前页："+page.getCurrent());
    System.out.println("每页显示的条数："+page.getSize());
    System.out.println("总记录数："+page.getTotal());
    System.out.println("总页数："+page.getPages());
    System.out.println("是否有上一页："+page.hasPrevious());
    System.out.println("是否有下一页："+page.hasNext());
}
```

## xml自定义分页

### UserMapper中定义接口方法

```java
/**
* 根据年龄查询用户列表，分页显示
* @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位
* @param age 年龄
* @return
*/
Page<User> selectPageVo(@Param("page") Page<User> page, @Param("age") Integer age);
```

### UserMapper.xml中编写SQL

```java
<!--SQL片段，记录基础字段-->
    <sql id="BaseColumns">id,username,age,email</sql>
<!--IPage<User> selectPageVo(Page<User> page, Integer age);-->
    <select id="selectPageVo" resultType="User">
SELECT <include refid="BaseColumns"></include> FROM t_user WHERE age> #
{age}
</select>
```

## 乐观锁

### 场景

> 一件商品，成本价是80元，售价是100元。老板先是通知小李，说你去把商品价格增加50元。小李正在玩游戏，耽搁了一个小时。正好一个小时后，老板觉得商品价格增加到150元，价格太高，可能会影响销量。又通知小王，你把商品价格降低30元。此时，小李和小王同时操作商品后台系统。小李操作的时候，系统先取出商品价格100元；小王也在操作，取出的商品价格也是100元。小李将价格加了50元，并将100+50=150元存入了数据库；小王将商品减了30元，并将100-30=70元存入了数据库。是的，如果没有锁，小李的操作就
> 完全被小王的覆盖了。
>
> 现在商品价格是70元，比成本价低10元。几分钟后，这个商品很快出售了1千多件商品，老板亏1万多。

### 乐观锁实现流程

- 数据库中添加version字段
- 取出记录时，获取当前version

```sql
SELECT id,`name`,price,`version` FROM product WHERE id=1
```

+ 更新时，version + 1，如果where语句中的version版本不对，则更新失败

```sql
UPDATE product SET price=price+50, `version`=`version` + 1 WHERE id=1 AND `version`=1
```

### Mybatis-Plus实现乐观锁

#### 实体类

```java
package com.atguigu.mybatisplus.entity;
import com.baomidou.mybatisplus.annotation.Version;
import lombok.Data;
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    @Version
    private Integer version;
}
```

### 添加乐观锁插件配置

```java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    //添加分页插件
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    //添加乐观锁插件
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}
```

### 测试修改冲突

> 小李查询商品信息：
> SELECT id,name,price,version FROM t_product WHERE id=?
> 小王查询商品信息：
> SELECT id,name,price,version FROM t_product WHERE id=?
> 小李修改商品价格，自动将version+1
> UPDATE t_product SET name=?, price=?, version=? WHERE id=? AND version=?
> Parameters: 外星人笔记本(String), 150(Integer), 1(Integer), 1(Long), 0(Integer)
> 小王修改商品价格，此时version已更新，条件不成立，修改失败
> UPDATE t_product SET name=?, price=?, version=? WHERE id=? AND version=?
> Parameters: 外星人笔记本(String), 70(Integer), 1(Integer), 1(Long), 0(Integer)
> 最终，小王修改失败，查询价格：150
> SELECT id,name,price,version FROM t_product WHERE id=?

### 优化流程

```java
@Test
public void testConcurrentVersionUpdate() {
    //小李取数据
    Product p1 = productMapper.selectById(1L);
    //小王取数据
    Product p2 = productMapper.selectById(1L);
    //小李修改 + 50
    p1.setPrice(p1.getPrice() + 50);
    int result1 = productMapper.updateById(p1);
    System.out.println("小李修改的结果：" + result1);
    //小王修改 - 30
    p2.setPrice(p2.getPrice() - 30);
    int result2 = productMapper.updateById(p2);
    System.out.println("小王修改的结果：" + result2);
    if(result2 == 0){
        //失败重试，重新获取version并更新
        p2 = productMapper.selectById(1L);
        p2.setPrice(p2.getPrice() - 30);
        result2 = productMapper.updateById(p2);
    }
    System.out.println("小王修改重试的结果：" + result2);
    //老板看价格
    Product p3 = productMapper.selectById(1L);
    System.out.println("老板看价格：" + p3.getPrice());
}
```

# 通用枚举

> 表中的有些字段值是固定的，例如性别（男或女），此时我们可以使用MyBatis-Plus的通用枚举来实现

## 创建通用枚举类型

```java
package com.atguigu.mp.enums;
import com.baomidou.mybatisplus.annotation.EnumValue;
import lombok.Getter;
@Getter
public enum SexEnum {
    MALE(1, "男"),
    FEMALE(2, "女");
    @EnumValue
    private Integer sex;
    private String sexName;
    SexEnum(Integer sex, String sexName) {
        this.sex = sex;
        this.sexName = sexName;
    }
}
```

## 配置扫描通用枚举

```java
mybatis-plus:
configuration:
# 配置MyBatis日志
log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
global-config:
db-config:
# 配置MyBatis-Plus操作表的默认前缀
table-prefix: t_
# 配置MyBatis-Plus的主键策略
id-type: auto
# 配置扫描通用枚举
type-enums-package: com.atguigu.mybatisplus.enums
```

## 测试

```java
@Test
public void testSexEnum(){
    User user = new User();
    user.setName("Enum");
    user.setAge(20);
    //设置性别信息为枚举项，会将@EnumValue注解所标识的属性值存储到数据库
    user.setSex(SexEnum.MALE);
    //INSERT INTO t_user ( username, age, sex ) VALUES ( ?, ?, ? )
    //Parameters: Enum(String), 20(Integer), 1(Integer)
    userMapper.insert(user);
}
```

# 代码生成器

## 引入依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
```

## 代码生成器

```java
package com.catalystplus.admin.config;

import com.baomidou.mybatisplus.core.toolkit.StringPool;
import com.baomidou.mybatisplus.generator.AutoGenerator;
import com.baomidou.mybatisplus.generator.InjectionConfig;
import com.baomidou.mybatisplus.generator.config.*;
import com.baomidou.mybatisplus.generator.config.po.TableInfo;
import com.baomidou.mybatisplus.generator.config.rules.NamingStrategy;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.ArrayList;
import java.util.List;

/**
 *
 * 代码生成器 ，先修改下面的常量配置参数，然后执行 main方法
 */
public class CodeGenerator {

    // 数据库连接配置
    private static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
    private static final String JDBC_URL = "jdbc:mysql://192.168.3.100:3306/catalystplus-library?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull&useSSL=false&allowMultiQueries=true&serverTimezone=Asia/Shanghai";
    private static final String JDBC_USER_NAME = "root";
    private static final String JDBC_PASSOWRD = "123456";

    // 包名和模块名
    private static final String PACKAGE_NAME = "com.catalystplus";
    private static final String MODULE_NAME = "admin";

    // 表名，多个表使用英文逗号分割
    private static final String TBL_NAMES = "paper_count";

    // 表名的前缀，从表生成代码时会去掉前缀
    //    private static final String TABLE_PREFIX = "paper_";

    // 生成代码入口main方法
    public static void main(String[] args) {
        // 0.代码生成器
        AutoGenerator mpg = new AutoGenerator();

        // 1.全局配置
        GlobalConfig gc = getGlobalConfig();
        mpg.setGlobalConfig(gc);

        // 2.数据源配置
        DataSourceConfig dsc = getDataSourceConfig();
        mpg.setDataSource(dsc);

        // 3.包配置
        PackageConfig pc = getPackageConfig();
        mpg.setPackageInfo(pc);

        // 4.自定义配置
        InjectionConfig cfg = getInjectionConfig();
        mpg.setCfg(cfg);

        // 5.模板配置
        TemplateConfig templateConfig = getTemplateConfig();
        mpg.setTemplate(templateConfig);
        // 使用Freemarker模板引擎
        mpg.setTemplateEngine(new FreemarkerTemplateEngine());

        // 6.策略配置
        StrategyConfig strategy = getStrategyConfig();
        mpg.setStrategy(strategy);

        // 7.开始生成代码
        mpg.execute();
    }

    /** 1.全局配置 */
    private static GlobalConfig getGlobalConfig() {
        GlobalConfig gc = new GlobalConfig();
        String projectPath = System.getProperty("user.dir");
        gc.setOutputDir(projectPath + "/src/main/java");
        gc.setAuthor("qls");
        gc.setOpen(false);
        // 自定义生成的ServiceName，去掉默认的ServiceName前面的I
        gc.setServiceName("%s" + ConstVal.SERVICE);
        // gc.setSwagger2(true); 实体属性 Swagger2 注解
        return gc;
    }

    /** 2.数据源配置 */
    private static DataSourceConfig getDataSourceConfig() {
        DataSourceConfig dsc = new DataSourceConfig();
        dsc.setDriverName(JDBC_DRIVER);
        dsc.setUrl(JDBC_URL);
        // dsc.setSchemaName("public");
        dsc.setUsername(JDBC_USER_NAME);
        dsc.setPassword(JDBC_PASSOWRD);
        return dsc;
    }

    /** 3.包配置 */
    private static PackageConfig getPackageConfig() {
        PackageConfig pc = new PackageConfig();
        // 生成PACKAGE_NAME.MODULE_NAME的包路径
        pc.setParent(PACKAGE_NAME);
        pc.setModuleName(MODULE_NAME);
        return pc;
    }

    /** 4.自定义配置 */
    private static InjectionConfig getInjectionConfig() {

        // 这里模板引擎使用的是freemarker
        String templatePath = "/templates/mapper.xml.ftl";

        // 自定义输出配置
        List<FileOutConfig> focList = new ArrayList<>();
        String projectPath = System.getProperty("user.dir");
        // 自定义配置会被优先输出
        focList.add(new FileOutConfig(templatePath) {
            @Override
            public String outputFile(TableInfo tableInfo) {
                // 自定义输出文件名 ， 如果你 Entity 设置了前后缀、此处注意 xml 的名称会跟着发生变化
                return projectPath + "/src/main/resources/mapper/" + tableInfo.getEntityName() + "Mapper"
                    + StringPool.DOT_XML;
            }
        });

        InjectionConfig cfg = new InjectionConfig() {
            @Override
            public void initMap() {
                // to do nothing
            }
        };
        cfg.setFileOutConfigList(focList);
        return cfg;
    }

    /** 5.模板配置 */
    private static TemplateConfig getTemplateConfig() {
        TemplateConfig templateConfig = new TemplateConfig();

        // 配置自定义输出模板
        // 指定自定义模板路径，注意不要带上.ftl/.vm, 会根据使用的模板引擎自动识别
        // templateConfig.setEntity("templates/entity2.java");
        // templateConfig.setService();
        // templateConfig.setController();

        templateConfig.setXml(null);
        return templateConfig;
    }

    /** 6.策略配置 */
    private static StrategyConfig getStrategyConfig() {
        StrategyConfig strategy = new StrategyConfig();
        // 下划线转驼峰命名
        strategy.setNaming(NamingStrategy.underline_to_camel);
        strategy.setColumnNaming(NamingStrategy.underline_to_camel);
        strategy.setEntityLombokModel(false);
        strategy.setRestControllerStyle(true);
        strategy.setInclude(TBL_NAMES.split(","));
        strategy.setControllerMappingHyphenStyle(true);
        //        strategy.setTablePrefix(TABLE_PREFIX);
        return strategy;
    }

}
```

# 多数据源

> 适用于多种场景：纯粹多库、 读写分离、 一主多从、 混合模式等
>
> 目前我们就来模拟一个纯粹多库的一个场景，其他场景类似
>
> 场景说明：
>
> 我们创建两个库，分别为：mybatis_plus（以前的库不动）与mybatis_plus_1（新建），将mybatis_plus库的product表移动到mybatis_plus_1库，这样每个库一张表，通过一个测试用例分别获取用户数据与商品数据，如果获取到说明多库模拟成功

## 引入依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>
```

## 配置多数据源

```yaml
spring:
# 配置数据源信息
datasource:
dynamic:
# 设置默认的数据源或者数据源组,默认值即为master
primary: master
# 严格匹配数据源,默认false.true未匹配到指定数据源时抛异常,false使用默认数据源
strict: false
datasource:
master:
url: jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-
8&useSSL=false
driver-class-name: com.mysql.cj.jdbc.Driver
username: root
password: 123456
slave_1:
url: jdbc:mysql://localhost:3306/mybatis_plus_1?characterEncoding=utf-
8&useSSL=false
driver-class-name: com.mysql.cj.jdbc.Driver
username: root
password: 123456
```

## 创建用户service

```java
@DS("master") //指定所操作的数据源
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements
    UserService {
}
```

## 创建商品service

```java
@DS("slave_1")
@Service
public class ProductServiceImpl extends ServiceImpl<ProductMapper, Product>
implements ProductService {
}
```

# MyBatisX插件

> MyBatis-Plus为我们提供了强大的mapper和service模板，能够大大的提高开发效率,但是在真正开发过程中，MyBatis-Plus并不能为我们解决所有问题，例如一些复杂的SQL，多表联查，我们就需要自己去编写代码和SQL语句，我们该如何快速的解决这个问题呢，这个时候可以使用MyBatisX插件
>
> MyBatisX一款基于 IDEA 的快速开发插件，为效率而生

[官网地址][https://baomidou.com/pages/ba5b24/#%E5%8A%9F%E8%83%BD]

