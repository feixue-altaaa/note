# 框架概述

## 软件开发常用结构

### 三层架构

- 三层架构为界面层（User Interface layer）、业务逻辑层（Business Logic Layer）、数据访问层（Data access layer）三层的职责
- 界面层（表示层，视图层）:主要功能是接受用户的数据，显示请求的处理结果。使用 web 页面和用户交互，手机 app 也就是表示层的，用户在 app 中操作，业务逻辑在服务器端处理。
- 业务逻辑层：接收表示传递过来的数据，检查数据，计算业务逻辑，调用数据访问层获取数据。
- 数据访问层：与数据库打交道。主要实现对数据的增、删、改、查。将存储在数据库中的数据提交给业务层，同时将业务层处理的数据保存到数据库.
- 三层的处理请求的交互
- 客户端<--->界面层<--->业务逻辑层<--->数据访问层<--->数据库

![1](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302262046375.jpg)

**为什么要使用三层？**

- 结构清晰、耦合度低, 各层分工明确
- 可维护性高，可扩展性高

- 有利于标准化
- 开发人员可以只关注整个结构中的其中某一层的功能实现
- 有利于各层逻辑的复用

### 常用框架

+ 常见的 J2EE 中开发框架

> **MyBatis 框架**
>
> **MyBatis** **是一个优秀的基于** **java** **的持久层框架，内部封装了** **jdbc**，开发者只需要关注 **sql** **语句**本身，而不需要处理加载驱动、创建连接、创建 statement、关闭连接，资源等繁杂的过程
>
> MyBatis 通过 xml 或注解两种方式将要执行的各种 sql 语句配置起来，并通过 java 对象和 sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回

> **Spring 框架**
>
> Spring 框架为了解决软件开发的复杂性而创建的。Spring 使用的是基本的 JavaBean 来完成以前非常复杂的企业级开发。Spring 解决了业务对象，功能模块之间的耦合，不仅在 javase,web 中使用，大部分 Java 应用都可以从 Spring 中受益
>
> Spring 是一个轻量级控制反转(IoC)和面向切面(AOP)的容器

> **SpringMVC 框架**
>
> Spring MVC 属于 SpringFrameWork 3.0 版本加入的一个模块，为 Spring 框架提供了构建 Web 应用程序的能力。现在可以 Spring 框架提供的 SpringMVC 模块实现 web 应用开发，在 web 项目中可以无缝使用 Spring 和 Spring MVC 框架

## 框架是什么

### 框架定义

+ 框架（Framework）是整个或部分系统的可重用设计，表现为一组抽象构件及构件实例间交互的方法;另一种认为，框架是可被应用开发者定制的应用骨架、模板
+ 简单的说，框架其实是半成品软件，就是一组组件，供你使用完成你自己的系统。从另一个角度来说框架一个舞台，你在舞台上做表演。在框架基础上加入你要完成的功能

### 框架解决的问题

- 框架要解决的最重要的一个问题是**技术整合**，在 J2EE 的框架中，有着各种各样的技术，不同的应用，系统使用不同的技术解决问题。需要从 J2EE 中选择不同的技术，而技术自身的复杂性，有导致更大的风险。企业在开发软件项目时，主要目的是解决业务问题。 即要求企业负责技术本身，又要求解决业务问题。这是大多数企业不能完成的。框架把相关的技术融合在一起，企业开发可以集中在业务领域方面
-  另一个方面可以**提高开发的效率**

## JDBC编程

```java
public void findStudent() {
    Connection conn = null; 
    Statement stmt = null;
    ResultSet rs = null;
    try {
        //注册 mysql 驱动 
        Class.forName("com.mysql.jdbc.Driver");
        //连接数据的基本信息 url ，username，password
        String url = "jdbc:mysql://localhost:3306/springdb"; 
        String username = "root";
        String password = "123456";
        //创建连接对象 
        conn = DriverManager.getConnection(url, username, password);
        //保存查询结果 
        List<Student> stuList = new ArrayList<>();
        //创建 Statement, 用来执行 sql 语句 
        stmt = conn.createStatement();
        //执行查询，创建记录集， 
        rs = stmt.executeQuery("select * from student");
        while (rs.next()) {
            Student stu = new Student(); 
            stu.setId(rs.getInt("id"));
            stu.setName(rs.getString("name"));
            stu.setAge(rs.getInt("age"));
            //从数据库取出数据转为 Student 对象，封装到 List 集合 
            stuList.add(stu);}
    }catch(Exception e){
        e.printStackTrace();
    }finally{
        try{
            if(rs != null)
                rs.lose();
            if(pstm != null)
                pstm.close();
            if(con != null)
                con.close();

        }catch(Exception e){
            e.printStackTrace();
        }
    }
```

### 使用JDBC的缺陷

- 代码比较多，开发效率低
- 需要关注 Connection ,Statement, ResultSet 对象创建和销毁
- 对 ResultSet 查询的结果，需要自己封装为 List
- **重复的代码比较多些**
- **业务代码和数据库的操作混在一起**

## MyBatis框架概述

> MyBatis 框架：
>
> MyBatis 本是 apache 的一个开源项目 iBatis, 2010 年这个项目由 apache software foundation 迁移到了 google code，并且改名为 MyBatis 。2013 年 11 月迁移到 Github
>
> iBATIS 一词来源于“internet”和“abatis”的组合，是一个基于 Java 的持久层框架。iBATIS 提供的持久层框架包括 SQL Maps 和 Data Access Objects（DAOs）

### MyBatis框架解决的主要问题

**减轻使用 JDBC 的复杂性，不用编写重复的创建 Connetion , Statement ; 不用编写关闭资源代码。直接使用 java 对象，表示结果数据。让开发者专注 SQL的处理。 其他分心的工作由 MyBatis 代劳**

**MyBatis 可以完成**

- 注册数据库的驱动，例如 Class.forName(“com.mysql.jdbc.Driver”)
- 创建 JDBC 中必须使用的 Connection ， Statement， ResultSet 对象
- 从 xml 中获取 sql，并执行 sql 语句，把 ResultSet 结果转换 java 对象
- 关闭资源

### MyBatis框架的结构

![image-20230226205442661](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302262054723.png)

- mybatis配置  SqlMapConfig.xml，此文件作为mybatis的全局配置文件，配置了mybatis的运行环境等信息。  mapper.xml文件即sql映射文件，文件中配置了操作数据库的sql语句。此文件需要在SqlMapConfig.xml中加载
- 通过mybatis环境等配置信息构造SqlSessionFactory即会话工厂  
- 由会话工厂创建sqlSession即会话，操作数据库需要通过sqlSession进行
- mybatis底层自定义了Executor执行器接口操作数据库，Executor接口有两个实现，一个是基本执行器、一个是缓存执行器
- Mapped Statement也是mybatis一个底层封装对象，它包装了mybatis配置信息及sql映射信息等。mapper.xml文件中一个sql对应一个Mapped Statement对象，sql的id即是Mapped  statement的id
- Mapped Statement对sql执行输入参数进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql前将输入的java对象映射至sql中，输入参数映射就是jdbc编程中对preparedStatement设置参数
- Mapped Statement对sql执行输出结果进行定义，包括HashMap、基本类型、pojo，Executor通过Mapped Statement在执行sql后将输出结果映射至java对象中，输出结果映射过程相当于jdbc编程中对结果的解析处理过程

# MyBatis 框架快速入门

## 搭建 MyBatis 开发环境

### 创建 mysql 数据库和表

```mysql
CREATE DATABASE ssm DEFAULT CHARSET utf8;

use ssm;

CREATE TABLE `student` (
    `id` int(11)  AUTO_INCREMENT primary key ,
    `name` varchar(255) DEFAULT NULL,
    `email` varchar(255) DEFAULT NULL,
    `age` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
insert into student(name,email,age) values('张三','zhangsan@126.com',22);
insert into student(name,email,age) values('李四','lisi@126.com',21);
insert into student(name,email,age) values('王五','wangwu@163.com',22);
insert into student(name,email,age) values('赵六','zhaoliun@qq.com',24);
select * from student;
```

### 创建工程，添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.11</version>
        <scope>test</scope>
    </dependency>
    <!--    添加mybatis框架的依赖-->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.1</version>
    </dependency>
    <!--    添加mysql8.0驱动-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.22</version>
    </dependency>
```

### 编写Student实体类

```java
public class Student {
    private int id;
    private String name;
    private String email;
    private int age;
}
```

### 添加db.properties文件

### 创建 MyBatis 主配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--mybatis-3-config.dtd:它规定了当前的xml文件中可以出现哪些标签
                         这些标签的顺序和位置,这些标签的属性
                         这些标签的子标签--!>
```

### 创建 MyBatis 主配置文件参数详解

```xml
<configuration>
    <!--    将文件流指向db.properties,最终是为了从中读取数据库的一堆配置信息-->
    <properties resource="db.properties"></properties>
    <!--    要进行数据库的访问，所以要进行数据库访问的配置，驱动，url,username,password-->
    <!--    environments:进行环境变量(连接数据库)的配置
                     可以进行多个数据库连接配置,可以上线一套,开发一套,可以mysql一套,oracle一套等
        default:本次配置中使用的环境变量的名称,多套配置,default决定哪套配置生效
-->
    <environments default="development">
        <!--environment：进行具体环境变量的配置
    id: 为当前的配置的环境变量起个名称,为了在environments的default中使用
-->
        <environment id="development">
            <!--            transactionManager:事务管理
                type: JDBC:就是程序员自己来管理事务的提交和回滚
                      MANAGED:由容器来进行事务的管理,例如:spring
-->
            <transactionManager type="JDBC"></transactionManager>
            <!--            dataSource:数据源的配置
                      type:指定数据源的配置方式,是否是连接池
                           "POOLED":表明使用数据库连接池
                           "UNPOOLED":不使用连接池
                           "JNDI":java命名目录接口,由服务器端负责连接池的管理
                      property: driver:数据库驱动
                                url:数据库的路径
                                username:访问数据库的用户名
                                password:访问数据库的用户的密码
-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!--    注册StudentDaoImpl.xml文件,注意.xml后缀要带上-->
    <mappers>
        <mapper resource="StudentDaoImpl.xml"></mapper>
    </mappers>
</configuration>
```

### 创建StudentDaoImpl.xml文件

- 该文件完成数据库中student表的所有增删改查的操作.
- <mapper namespace=”自定义的路径名称”>,在简单访问中namespace中的内容可以自定义,目的是为了区别不同<mapper>中相同id的语句.
- 该文件中提供<select><update><insert><delete>等数据库中的基本操作标签

```xml
<?xml version="1.0" encoding="UTF-8" ?> <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
```

![1](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302262110120.png)

### 创建测试类

- 使用Junit单元测试完成各种功能测试
- 使用@Before注解来进行所有测试前的SqlSession的创建工作
- 使用@After注解来进行所有测试方法执行后的关闭SqlSession的工作
- 使用@Test注解来验证每一个功能的实现

```java
public class MyTest {
    SqlSession session;
    @Before
    public void openSession()throws Exception{
        //创建文件流,读取SqlMapConfig.xml
        InputStream inputStream = Resources.getResourceAsStream("SqlMapConfig.xml");
        //创建SqlSessionFactory
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        //取得SqlSession
        session = factory.openSession();
    }
    
    @Test
    public void testSelectStudentAll()throws Exception{
        List<Student> list = session.selectList("com.bjpowernode.selectStudentAll");
        list.forEach(stu-> System.out.println(stu));
    }
    @Test
    public void testStudentGetById(){
        Student stu = session.selectOne("com.bjpowernode.selectStudentById",2);
        System.out.println(stu);
    }
    @Test
    public void testSelectStudentByEmail(){
        List<Student> list = session.selectList("com.bjpowernode.selectStudentByEmail","l");
        list.forEach(stu-> System.out.println(stu));
    }
    @Test
    public void testInsertStudent(){
        Student stu = new Student("李四四","23234@qq.com",22);
        int num = session.insert("com.bjpowernode.insertStudent",stu);
        //切记切记切记:因为我们的事务管理机制使用的是JDBC,所以所有的增删改查后,要手工提交事务
        //session.commit();
        session.commit();//手工提交事务
        System.out.println(num);
    }
    
   
    @After
    public void closeSession(){
        session.close();
    }
}
```

## MyBatis对象分析

### Resources 类

+ Resources 类，顾名思义就是资源，用于读取资源文件。其有很多方法通过加载并解析资源文件，返回不同类型的 IO 流对象

### SqlSessionFactoryBuilder 类

+ SqlSessionFactory 的 创 建 ， 需 要 使 用 SqlSessionFactoryBuilder 对 象 的 build() 方 法 。 由 于SqlSessionFactoryBuilder对象在创建完工厂对象后，就完成了其历史使命，即可被销毁。所以，一般会将该 对象创建为一个方法内的局部对象，方法结束，对象销毁

### SqlSessionFactory 接口

+ SqlSessionFactory 接口对象是一个重量级对象（系统开销大的对象），是线程安全的，所以一个应用只需要一个该对象即可。创建 SqlSession 需要使用 SqlSessionFactory 接口的的 openSession()方法

  - openSession(true)：创建一个有自动提交功能的 SqlSession

  - openSession(false)：创建一个非自动提交功能的 SqlSession，需手动提交

  - openSession()：同 openSession(false)

### SqlSession 接口

- SqlSession 接口对象用于执行持久化操作。一个 SqlSession 对应着一次数据库会话，一次会话以SqlSession 对象的创建开始，以 SqlSession 对象的关闭结束
- SqlSession 接口对象是线程不安全的，所以每次数据库会话结束前，需要马上调用其 close()方法，将其关闭。再次需要会话，再次创建。 SqlSession 在方法内部创建，使用完毕后关闭

## SqlMapConfig.xml文件的优化

### 添加日志打印输出

+ 文件加入日志配置，可以在控制台输出执行的 sql 语句和参数

```xml
<!--    设置查看mybatis生成的sql语句的日志配置-->
<settings>
    <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

### 为实体类起别名

+ 由于XXXmapper.xml文件中入参和返回值都要使用实体类的对象,而我们在使用的时候必须书写实体类的完全限定类名,这样比较麻烦,我们可以通过起别名的方式来简化此操作.起别名的方式有两种.

#### 为单个实体类起别名

```xml
<typeAlias type="com.bjpowernode.pojo.Users" alias="users"></typeAlias>
```

- type:实体类的完全限定名称
- alias: 为实体类起的别名,在以后所有使用实体类类型的地方,写此别名即可

#### 批量别名注册

+ 可以通过<package>来批量的为实体类起别名,只要指定实体类所在的包的名称即可.MyBatis框架会自动为每个实体类起别名为类型的全小写或类名的大小写混合.推荐使用类名的全小写.

```xml
<typeAliases>
    <package name="com.bjpowernode.pojo"/>
</typeAliases>
```

### 在SqlMapConfig.xml文件中注册XXXMapper.xml

 **注册XXXMapper.xml文件的方式有四种**

#### 使用resource注册

```xml
<mapper resource="com/bjpowernode/mapper/UsersMapper.xml"></mapper>
```

+ 注意: UserMapper.xml是带后缀的,分隔符使用/

#### 使用class注册

+ 动态代理的方式,使用此种注册

```xml
<mapper class="com.bjpowernode.mapper.UsersMapper"></mapper>
```

注意: class的值是接口的完全限定名称.

#### 使用url注册

```xml
<mapper url="file:///E:/UserMapper.xml"></mapper>
```

+ 指定绝对路径注册,注意file后面是双杠

#### 使用<package>注册

```xml
<package name="com.bjpowernode.mapper"/>
```

# 常用注解

+ 这些常用注解分为三大类：SQL语句映射，结果集映射和关系映射

## SQL语句映射

+ **增删改查四个注解**

```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM user WHERE id = #{id}")
    User getById(@Param("id") Long id);

    @Insert("INSERT INTO user(name, age) VALUES(#{name}, #{age})")
    void insert(User user);

    @Update("UPDATE user SET name=#{name}, age=#{age} WHERE id=#{id}")
    void update(User user);

    @Delete("DELETE FROM user WHERE id=#{id}")
    void deleteById(@Param("id") Long id);
}
```

+ @SelectKey注解：插入后，获取id的值
+ 以mysql为例，mysql在插入一条数据后，如何能获得到这个自增id的值呢？使用select last_insert_id() 可以取到最后生成的主键

```java
@Insert("insert into user(id,name) values(#{id},#{name})")
@Options(useGeneratedKeys = true, keyColumn = "id", keyProperty = "id")
@SelectKey(statement = "select last_insert_id()" ,keyProperty = "id",keyColumn = "id",resultType = int.class,before = false) 
public int insert(User user);
```

## 结果集映射

- @Result，@Results，@ResultMap是结果集映射的三大注解
- 首先说明一下@Results各个属性的含义，id为当前结果集声明唯一标识，value值为结果集映射关系，@Result代表一个字段的映射关系，column指定数据库字段的名称，property指定实体类属性的名称，jdbcType数据库字段类型，@Result里的id值为true表明主键，默认false；使用@ResultMap来引用映射结果集，其中value可省略
- 声明结果集映射关系代码

```java
@Select({"select id, name, class_id from student"})
@Results(id="studentMap", value={
    @Result(column="id", property="id", jdbcType=JdbcType.INTEGER, id=true),
    @Result(column="name", property="name", jdbcType=JdbcType.VARCHAR),
    @Result(column="class_id ", property="classId", jdbcType=JdbcType.INTEGER)
})
List<Student> selectAll();
```

+ 引用结果集代码

```java
@Select({"select id, name, class_id from student where id = #{id}"})
@ResultMap(value="studentMap")
Student selectById(integer id);
```

+ 这样就不用每次需要声明结果集映射的时候都复制冗余代码，简化开发，提高了代码的复用性

## 关系映射

### @one注解：用于一对一关系映射

```java
@Select("select * from student")  
@Results({  
    @Result(id=true,property="id",column="id"),  
    @Result(property="name",column="name"),  
    @Result(property="age",column="age"),  
    @Result(property="address",column="address_id",one=@One(select="cn.mybatis.mydemo.mappers.AddressMapper.getAddress"))  
})  
public List<Student> getAllStudents();  
```

### @many注解：用于一对多关系映射

```java
@Select("select * from t_class where id=#{id}")  
@Results({  
    @Result(id=true,column="id",property="id"),  
    @Result(column="class_name",property="className"),  
    @Result(property="students", column="id", many=@Many(select="cn.mybatis.mydemo.mappers.StudentMapper.getStudentsByClassId"))  
    })  
public Class getClass(int id);
```

# 动态代理

## 动态代理开发规范

-  MyBatis框架使用动态代理的方式来进行数据库的访问
- Mapper接口的开发相当于是过去的Dao接口的开发。由MyBatis框架根据接口定义创建动态代理对象，代理对象的方法体同Dao接口实现类的方法。在设计时要遵守以下规范
  -  Mapper接口与Mapper.xml文件在同一个目录下
  - Mapper接口的完全限定名与Mapper.xml文件中的namespace的值相同
  - Mapper接口方法名称与Mapper.xml中的标签的statement 的ID完全相同
  - Mapper接口方法的输入参数类型与Mapper.xml的每个sql的parameterType的类    型相同
  - Mapper接口方法的输出参数与Mapper.xml的每个sql的resultType的类型相同
  - Mapper文件中的namespace的值是接口的完全限定名称
  - 在SqlMapConfig.xml文件中注册时,使用class属性=接口的完全限定名

## 开发步骤

- 新建项目添加依赖
- 新建属性文件db.properties
- 新建环境配置文件(SqlMapConfig.xml)
- ![image-20230228152219884](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281522806.png)
  - 注册实体类的别名，简化开发。注意：在注册 mapper.xml文件时，使用class=”接口的完全限定名称”

+ 新建实体类

+ 新建接口

+ 新建（接口的实现类）与接口同名的xml文件，完成数据库中的所有操作

+ 优化测试

  ![image-20230228152428683](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281524749.png)

**总结**

**UsersMapper.java和UsersMapper.xml文件必须在同一个目录下，且必须同名**

**在UsersMapper.xml文件中添加namespace属性为接口的完全路径名**

## #{}和${}

- **#{}**是对非字符串拼接的参数的占位符，如果入参是简单数据类型，#{}里可以任意写，但是如果入参是对象类型，则#{}里必须是对象的成员变量的名称，#{}可以有效防止sql注入
- **${}主要是针对字符串拼接替换**，如果入参是基本数据类型，${}里必须是value,但是如果入参是对象类型，则${}里必须是对象的成员变量的名称。${}还可以替换列名和表名，存在sql注入风险，尽量少用

### \#｛｝演示

![image-20230228152529963](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281525025.png)

### **${}演示**

![image-20230228152624064](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281526125.png)

## 返回主键标签

+ 在完成插入操作后，将生成的主键信息通过实体类对象返回，在进行后继关联插入操作时，不用再次访问数据库

```xml
<insert id="insertUser" parameterType="com.oracle.demo.pojo.Users">
    <!--
           Order:指定生成返回主键的时机，AFTER：先插入再返回主键
                                        BEFORE: 先生成再完成插入
           keyProperty: 生成的主键放入到对象的哪个属性中
           resultType: 返回的主键的类型
       -->
    <selectKey order="AFTER" keyProperty="uid" resultType="int">
        select LAST_INSERT_ID()
    </selectKey>
    <!--  先生成随机字符串，再返回
        <selectKey order="BEFORE" keyProperty="uid" resultType="string">
            select uuid()
        </selectKey>
        -->
```

# 动态sql

+ 动态 SQL 是 MyBatis 的强大特性之一。如果你使用过 JDBC 或其它类似的框架，你应该能理解根据不同条件拼接 SQL 语句有多痛苦，例如拼接时要确保不能忘记添加必要的空格，还要注意去掉列表最后一个列名的逗号。利用动态 SQL，可以彻底摆脱这种痛苦。使用动态 SQL 并非一件易事，但借助可用于任何 SQL 映射语句中的强大的动态 SQL 语言，MyBatis 显著地提升了这一特性的易用性

##  <sql>标签

+ 当多种类型的查询语句的查询字段或者查询条件相同时，可以将其定义为常量，方便调用

```xml
<!--定义列名-->
<sql id="columns">
    id,username,birthday,sex,address
</sql>
<!--定义条件-->
<sql id="Example_Where_Clause" >
    <where >
        <foreach collection="oredCriteria" item="criteria" separator="or" >
            <if test="criteria.valid" >
                <trim prefix="(" suffix=")" prefixOverrides="and" >
                    <foreach collection="criteria.criteria" item="criterion" >
                        <choose >
                            <when test="criterion.noValue" >
                                and ${criterion.condition}
                            </when>
                            <when test="criterion.singleValue" >
                                and ${criterion.condition} #{criterion.value}
                            </when>
                            <when test="criterion.betweenValue" >
                                and ${criterion.condition} #{criterion.value} and #{criterion.secondValue}
                            </when>
                            <when test="criterion.listValue" >
                                and ${criterion.condition}
                                <foreach collection="criterion.value" item="listItem" open="(" close=")" separator="," >
                                    #{listItem}
                                </foreach>
                            </when>
                        </choose>
                    </foreach>
                </trim>
            </if>
        </foreach>
    </where>
</sql>
```

## <include>标签

+  用于引用定义的常量

```xml
<!--引用定义的列名-->
select <include refid="columns"></include>
from users
<!--引用定义的条件-->
<select id="selectByExample" resultMap="BaseResultMap" parameterType="com.bjpowernode.pojo.ProductInfoExample" >
    select
    <if test="distinct" >
        distinct
    </if>
    <include refid="Base_Column_List" />
    from product_info
    <if test="_parameter != null" >
        <include refid="Example_Where_Clause" />
    </if>
    <if test="orderByClause != null" >
        order by ${orderByClause}
    </if>
</select>
```

## <if>标签

+ 进行条件判断

```xml
<if test="userName != null and userName != '' ">
    and username like concat('%',#{userName},'%')
</if>
```

## <where>标签

+ 一般开发复杂业务的查询条件时，如果有多个查询条件，通常会使用<where>标签来进行控制。 标签可以自动的将第一个条件前面的逻辑运算符 (or ,and) 去掉，正如代码中写的，id 查询条件前面是有“and”关键字的，但是在打印出来的 SQL 中却没有，这就是<where> 的作用

```
<select id="getByCondition" resultType="users" parameterType="users">
select <include refid="allColumns"></include>
from users
<where>
<if test="userName != null and userName != '' ">
and username like concat('%',#{userName},'%')
</if>
<if test="birthday != null ">
and birthday = #{birthday}
</if>
<if test="sex != null and sex !=''">
and sex = #{sex}
</if>
<if test="address != null and address !=''">
and address like concat('%',#{address},'%')
</if>
</where>
</select>
```

```java
#测试
    @Test
    public void testGetByCondition()throws Exception{
    Users u = new Users();
    u.setUserName("小");
    u.setSex("1");
    u.setAddress("市");
    u.setBirthday(sf.parse("2002-01-19"));
    List<Users> list = usersMapper.getByCondition(u);
    list.forEach(users -> System.out.println(users));
}
```

## <set>标签

+ 使用set标签可以动态的配置 SET 关键字，并剔除追加到条件末尾的任何不相关的逗号。使用 if+set 标签修改后，在进行表单更新的操作中，哪个字段中有值才去更新，如果某项为 null 则不进行更新，而是保持数据库原值。**切记：至少更新一列**

```xml
<update id="updateBySet" parameterType="users">
    update users
    <set>
        <if test="userName != null and userName != ''">
            username = #{userName},
        </if>
        <if test="birthday != null">
            birthday = #{birthday},
        </if>
        <if test="sex != null and sex != ''">
            sex =#{sex},
        </if>
        <if test="address != null and address !=''">
            address = #{address}
        </if>
    </set>
    where id = #{id}
</update>
```

**测试**

```java
@Test
public void testUpdateSet()throws Exception{
    // Users u = new Users("哈哈",new Date(),"1","北京亦庄大兴");
    //Users u = new Users(3,"不知道",sf.parse("1998-08-08"),"2","北京亦庄大兴888");
    Users u = new Users();
    u.setId(2);
    u.setUserName("认识张三不");
    //u.setSex("2");
    //u.setBirthday(sf.parse("2000-01-01"));
    int num = usersMapper.updateBySet(u);
    //切记切记:必须提交事务
    sqlSession.commit();
    System.out.println(num);
}
```

## <foreach>标签

- <foreach>主要用来进行集合或数组的遍历，主要有以下参数
  - collection：collection 属性的值有三个分别是 list、array、map 三种，分别对应的参数类型为：List、数组、map 集合。
  -  item ：循环体中的具体对象。支持属性的点路径访问，如item.age,item.info.details，在list和数组中是其中的对象，在map中value
  -  index ：在list和数组中,index是元素的序号，在map中，index是元素的key，该参数可选
  -  open ：表示该语句以什么开始
  -  close ：表示该语句以什么结束
  -  separator ：表示元素之间的分隔符，例如在in()的时候，separator=","会自动在元素中间用“,“隔开，避免手动输入逗号导致sql错误，如in(1,2,)这样。该参数可选

### 循环遍历参数集合

```xml
<select id="getByIds" resultType="users">
    select <include refid="columns"></include>
    from users
    where id in
        <foreach collection="list" item="id" separator="," open="(" close=")">
           #{id}
        </foreach>
</select>
```

+ 测试

![image-20230228164644137](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281646189.png)

### 批量删除

```xml
<!--    批量删除-->
<delete id="deleteBatch" >
    delete from users where id in
    <foreach collection="array" item="id" close=")" open="(" separator=",">
        #{id}
    </foreach>
</delete>
```

+ 测试

![image-20230228164726246](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281647296.png)

### 批量增加

```xml
<!--    批量增加-->
    <insert id="insertBatch" >
        insert into users(username,birthday,sex,address) values
        <foreach collection="list" separator="," item="u">
            (#{u.userName},#{u.birthday},#{u.sex},#{u.address})
        </foreach>
    </insert>
```

+ 测试

![image-20230228164802496](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281648556.png)

### 批量更新

```xml
<!--    有选择的批量更新,至少更新一列-->
<update id="updateSet"  >
    <foreach collection="list" item="u" separator=";">
        update users
        <set>
            <if test="u.userName != null  and u.userName != ''">
                username=#{u.userName},
            </if>
            <if test="u.birthday != null">
                birthday = #{u.birthday},
            </if>
            <if test="u.sex != null  and u.sex != ''">
                sex = #{u.sex},
            </if>
            <if test="u.address != null  and u.address != ''">
                address = #{u.address}
            </if>
        </set>
        where id = #{u.id}
    </foreach>
</update>
```

+ 测试

```java
@Test
public void testUpdateBatch()throws Exception{
    List<Users> list = new ArrayList<>();
    Users u1 = new Users(3,"张1167",new SimpleDateFormat("yyyy-MM-dd").parse("1997-02-03"),"2","北京大兴亦庄1111");
    Users u2 = new Users(4,"李2267",new SimpleDateFormat("yyyy-MM-dd").parse("1998-02-03"),"1","北京大兴亦庄2333");
    Users u3 = new Users(5,"王3367",new SimpleDateFormat("yyyy-MM-dd").parse("1999-02-03"),"2","北京大兴亦庄122");
    list.add(u1);
    list.add(u2);
    list.add(u3);
    int num = mapper.updateSet(list);
    session.commit();
    System.out.println(num);
}
```

> **注意：要使用批量更新，必须在jdbc.properties属性文件中的url中添加&allowMultiQueries=true，允许多行操作**

## 指定参数位置

- 可以不使用对象的属性名进行参数值绑定，使用下标值。 mybatis-3.3 版本和之前的版本使用#{0},#{1}方式， 从 mybatis3.4 开始使用#{arg0}方式
- UsersMapper.java接口中

```java
//查询指定日期范围内的用户信息,实体类的成员变量无法包含条件了,所以散给条件
List<Users> getByBirthday(Date begin, Date end);
```

+ UsersMapper.xml文件中

```xml
<select id="getByBirthday" resultType="users">
   select * from users where birthday between #{arg0} and #{arg1}
</select>
```

![image-20230228165009283](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281650352.png)

## @Param指定参数名称

+ UsersMapper.java接口中

```java
//切换列名进行模糊查询
//@Param("columnName"):这里定义的columnName的名称是要在xml文件中的${引用定义的名称}
List<Users> getByColunm(@Param("columnName") String columnName,
                     @Param("columnValue") String columnValue);
```

+ UsersMapper.xml文件中

```xml
<select id="getByColunm" resultType="users">
    select <include refid="columns"></include>
    from users
    where ${columnName} =#{columnValue}
</select>
```

## 入参是map

- 入参是map,是因为当传递的数据有多个,不适合使用指定下标或指定名称的方式来进行传参,又加上参数不一定与对象的成员变量一致,考虑使用map集合来进行传递.map使用的是键值对的方式.当在sql语句中使用的时候#{键名},${键名},用的是键的名称
- UsersMapper.java接口中

```java
//入参是map
List<Users> getByMap(Map<String,Date> map);
```

+ UsersMapper.xml文件中

```xml
<select id="getByMap" parameterType="map" resultType="users">
    select <include refid="columns"></include>
    from users
    where birthday between #{zarbegin} and #{zarend}
</select>
```

+ 测试

```java
@Test
public void testGetByMap()throws Exception{
    Date begin = new SimpleDateFormat("yyyy-MM-dd").parse("1996-01-01");
    Date end = new SimpleDateFormat("yyyy-MM-dd").parse("1998-12-31");
    Map<String,Date> map = new HashMap<>();
    //放入map集合中的数据是键值对
    map.put("zarbegin",begin);
    map.put("zarend",end);
    List<Users> list = mapper.getByMap(map);
    list.forEach(u-> System.out.println(u));
}
```

## 返回值是map

+ 返回值是map的适用场景，如果数据不能使用对象来进行封装，可能查询的数据来自多张表中的某些列，这种情况下，使用map，但是map的返回方式破坏了对象的封装，返回来的数据是一个一个单独的数据,，之间不相关。**map**使用表中的列名或别名做为键名进行返回数据

### map封装返回值是一行

+ UsersMapper.java接口中

```java
/返回值是一个值,是map类型,根据主键查用户对象
Map<String,Object> getReturnMapOne(int id);
```

+ UsersMapper.xml文件中

```xml
<select id="getReturnMapOne" resultType="map" parameterType="int">
    select id myid,username myusername,sex mysex,address myaddress,birthday mybirthday
    from users
    where id=#{id}
</select>
```

+ 测试类中

```java
@Test
public void testGetReturnMapOne(){
    Map<String,Object> map = mapper.getReturnMapOne(7);
    System.out.println(map);
    System.out.println(map.get("username"));
}
```

### 返回值是map多行

+ UsersMapper.java接口中

```java
//使用map封装返回多个map的集合--->List<Map<String,Object>>
List<Map<String,Object>> getReturnMap();
```

+ UsersMapper.xml文件中

```xml
<select id="getReturnMap" resultType="map" >
    select <include refid="columns"></include>
    from users
</select>
```

+ 测试类中

```java
@Test
public void testGetReturnMap(){
    List<Map<String,Object>> list = mapper.getReturnMap();
    list.forEach(map-> System.out.println(map));
}
```

## 列名与类中成员变量名称不一致

- 解决方案一：使用列的别名，别名与类中的成员变量名一样，即可完成注入

```xml
<select id="getAll" resultType="book">
    select bookid id,bookname name
    from book
</select>
```

+ 解决方案二：使用<resultMap>标签进行映射

![image-20230228165521113](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281655185.png)

![image-20230228165529146](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281655213.png)

# 表的关联关系

-  我们通常说的关联关系有以下四种，一对多关联，多对一关联，一对一关联，多对多关联。**关联关系是有方向的**。如果是高并发的场景中，不适合做表的关联
- 在多对一和一对多的关联关系中，我们使用订单表和客户表

## 一对多关联

**在一对多关联关系中，一方(客户)中有多方(订单)的集合 ，所以要使用<collection>标签来映射多方的属性**

**每个mapper里只有自己的增删改查**

![image-20230228190813717](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281908789.png)

**调用OrdersMapper.xml中的按用户ID查该用户名下的所有订单**

![image-20230228190833389](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281908447.png)

## 多对一关联

**在多对一关联关系中，多方(订单)中持有一方(客户)的对象，要使用标签<association>标签来映射一方的属性**

![image-20230228190857526](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281908599.png)

**调用CustomerMapper.xml里的按用户主键查询用户信息的方法**

![image-20230228190916846](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281909900.png)

## 一对一关联

+  一个班级只有一个授课老师，一个老师也只为一个班级授课

![image-20230228190935005](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281909085.png)

+ 实体类

![image-20230228191006179](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281910233.png)

+ 接口

![image-20230228191016951](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281910005.png)

+ mapper.xml

![image-20230228191034155](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281910229.png)

## 多对多关联

+ 多对多关联中，需要通过中间表化解关联关系。中间表描述两张主键表的关联。中间表没有对应的实体类。Mapper.xml文件中也没有中间表的对应标签描述，只是在查询语句中使用中间表来进行关联

![image-20230228191054018](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281910086.png)

+ 实体类

![image-20230228191116623](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281911682.png)

+ 接口

![image-20230228191127736](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281911795.png)

+ Mapper.xml

![image-20230228191139463](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281911539.png)

**总结：无论是什么关联关系，如果某方持有另一方的集合，则使用<collection>标签完成映射，如果某方持有另一方的对象，则使用<association>标签完成映射**

# 事务

- Mybatis 框架是对 JDBC 的封装，所以 Mybatis 框架的事务控制方式有两种，一种是容器进行事务管理的，一种是程序员手工决定事务的提交与回滚
- SqlMapConfig.xml文件中设定的事务处理的方式

![image-20230228191524079](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281915147.png)

- Connection 对象的 setAutoCommit()方法来设置事务提交方式的。自动提交和手工提交。该标签用于指定MyBatis 所使用的事务管理器
- MyBatis 支持两种事务管理器类型：**JDBC** **与 MANAGED**
- JDBC：使用 JDBC 的事务管理机制。即，通过 Connection 的 commit()方法提交，通过 rollback()方法回滚。但默认情况下，MyBatis 将自动提交功能关闭了，改为了手动提交。即程序中需要显式的对事务进行提交或回滚。从日志的输出信息中可以看到

![image-20230228191546599](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302281915658.png)

+ 在获得SqlSession的时候，如果openSession()是无参或者是false，则必须手工提交事务，如果openSession(true)，则为自动提交事务，在执行完增删改后无须commit()，事务自动提交

**session = factory.openSession(true);**

# 缓存

+ 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上(关系型数据库数据文件)查询，从缓存中查询，从而**提高查询效率**，解决了高并发系统的性能问题。mybatis提供查询缓存，用于减轻数据库压力，提高数据库性能

## 缓存执行机制

+ 在进行数据库访问时，首先去访问缓存，如果缓存中有要访问的数据，则直接返回客户端，如果没有则去访问数据库，在库中得到数据后，先在缓存放一份，再返回客户端。如下图

![image-20230228200124908](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282001975.png)

+ **mybaits提供一级缓存和二级缓存。默认开启一级缓存**

![image-20230228200136291](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282001350.png)

## 一级缓存

- 第一次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，如果没有，从数据库查询用户信息。得到用户信息，将用户信息存储到一级缓存中。如果sqlSession去执行commit操作（执行插入、更新、删除），则清空SqlSession中的一级缓存，这样做的目的为了让缓存中存储的是最新的信息，避免脏读
- 第二次发起查询用户id为1的用户信息，先去找缓存中是否有id为1的用户信息，缓存中有，直接从缓存中获取用户信息。如果没有重复第一次查询操作。如下图

![image-20230228200154739](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282001800.png)

+ 测试代码

```java
@Test
public void testCacheOne() {
    //一级缓存基于SqlSession的
    //取出用户ID为7的用户
    Users u1 = mapper.getById(7);
    System.out.println("第一次访问数据库得到的u=" + u1);
    //再取一次,看看有没有访问数据库,就知道开没有开启一级缓存
    Users u2 = mapper.getById(7);
    System.out.println("第二次访问数据库得到的u=" + u2);
}
@Test
public void testCacheOneClose() {
    //一级缓存基于SqlSession的
    //取出用户ID为7的用户
    Users u1 = mapper.getById(7);
    System.out.println("第一次访问数据库得到的u=" + u1);
    //关闭session,也会清除缓存
    session.close();
    //再取一次,看看有没有访问数据库,就知道开没有开启一级缓存
    session = factory.openSession();
    mapper = session.getMapper(UsersMapper.class);
    Users u2 = mapper.getById(7);
    System.out.println("第二次访问数据库得到的u=" + u2);
}
@Test
public void testCacheOneCommit() {
    //一级缓存基于SqlSession的
    //取出用户ID为7的用户
    Users u1 = mapper.getById(7);
    System.out.println("第一次访问数据库得到的u=" + u1);

    //完成数据更新操作
    int num = mapper.update(new Users(38, "abc", new Date(), "1", "朝阳"));
    session.commit();
    System.out.println(num + "==================");

    Users u2 = mapper.getById(7);
    System.out.println("第二次访问数据库得到的u=" + u2);

}
```

## 二级缓存

- mybaits的二级缓存是mapper范围级别，除了在SqlMapConfig.xml设置二级缓存的总开关，还要在具体的mapper.xml中开启二级缓存，并且要让实体类实现serializable接口
- 在核心配置文件  SqlMapConfig.xml中加入设置

![image-20230228200237382](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282002460.png)

+ 在UsersMapper.xml文件中二启二级缓存，使用<cache></cache>

<img src="https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282002159.png" alt="image-20230228200247077" style="zoom:150%;" />

+ 实体类必须实现java.io.serializable接口，保证实体可序列化

![image-20230228200301020](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282003083.png)

+ 测试代码

![image-20230228200312142](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282003220.png)

> MyBatis是一种流行的Java持久化框架，它支持一级缓存和二级缓存来提高性能，下面是它们的异同点：
>
> 一级缓存：
>
> - 一级缓存是在SqlSession级别的缓存，也就是说只在一个SqlSession中有效；
> - 当执行一条SQL语句时，MyBatis会将查询结果缓存在SqlSession的缓存中，如果再次执行相同的SQL语句，MyBatis会先从缓存中获取数据，而不是再次发送SQL语句；
> - 一级缓存默认是开启的，可以通过在Mapper中配置`<select>`标签的`flushCache`属性来控制是否清空缓存。
>
> 二级缓存：
>
> - 二级缓存是在Mapper级别的缓存，也就是说多个SqlSession共享同一个缓存；
> - 当执行一条SQL语句时，MyBatis会将查询结果缓存在缓存中，如果再次执行相同的SQL语句，MyBatis会先从缓存中获取数据，而不是再次发送SQL语句；
> - 二级缓存需要在Mapper.xml文件中进行配置，并且需要启用才能使用；
> - 二级缓存默认是关闭的，可以通过在Mapper.xml文件中配置`<cache>`标签来启用二级缓存，并配置缓存的策略、失效时间等参数。
>
> 总的来说，一级缓存和二级缓存都是用来提高查询性能的，但是它们的作用范围和使用方式有所不同。一级缓存是SqlSession级别的缓存，非常适合于频繁查询的场景，而二级缓存是Mapper级别的缓存，适合于多个SqlSession之间共享缓存的场景。在实际开发中，可以根据具体需求来选择使用哪种缓存

# 什么是ORM

- **ORM(Object Relational Mapping):**对象关系映射
- **编写程序的时候，以面向对象的方式处理数据，保存数据的时候，却以关系型数据库的方式存储**

![image-20230228200336820](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282003898.png)

![image-20230228200344593](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302282003669.png)

+ 持久化的操作：将对象保存到关系型数据库中 ,将关系型数据库中的数据读取出来以对象的形式封装
+ O: java中的对象

```java
public class Users implements java.io.Serializable {
	private Long id;
	private String name;
	private String password;
	private String telephone;
	private String username;
	private String isadmin;
	private Set houses = new HashSet(0);
	set()….get()……
}
```

+ **数据库中的表**

```sql
create table USERS
(
  ID        NUMBER(10) primary key,
  NAME      VARCHAR2(50),
  PASSWORD  VARCHAR2(50),
  TELEPHONE VARCHAR2(15),
  USERNAME  VARCHAR2(50),
  ISADMIN   VARCHAR2(5)
);
```

+ **映射**

```xml
<collection property="orders" ofType="order">
     <id property="id" column="oid"></id>
     <result property="orderNumber" column="ordernumber"></result>
     <result property="orderPrice" column="orderprice"></result>
</collection>
```

