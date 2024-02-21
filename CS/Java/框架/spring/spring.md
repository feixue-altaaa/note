# 什么是spring

- Spring是一个开源免费的框架 , 容器
- Spring是一个针对bean的生命周期进行管理的轻量级的框架 , 非侵入式的
- 控制反转 IoC , 面向切面 Aop
- 对事务的支持 , 对框架的支持
- 解决企业应用开发的复杂性

一句话概括

**Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器（框架）**

## applicationContext.xml	

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        https://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解的支持-->
    <context:annotation-config />
</beans>
```

## 注解说明

- [@Autowried](https://github.com/Autowried) : 自动装配通过类型，名字。
  如果Autowired不能唯一自动装配上属性，则需要通过[@Qualifier](https://github.com/Qualifier)(value = “XXX”)装配。
- [@Nullable](https://github.com/Nullable) : 字段标记了这个注解,说明这个字段可以为null。
- [@Resource](https://github.com/Resource) : 自动装配通过名字，类型。
- [@Component](https://github.com/Component) : 组件，放在类上说明这个类被Spring管理了，就是Bean。
- [@Repository](https://github.com/Repository)(dao层)、[@Service](https://github.com/Service)(Service层)、[@Controller](https://github.com/Controller) (Controller层)与 [@Component](https://github.com/Component)(组件)四个注解功能一样，都是代表将某个类注册到Spring容器中，装备Bean。
- [@Scope](https://github.com/Scope) : 设置作用域。[@Scope](https://github.com/Scope)(“singleton”):单例模式,[@Scope](https://github.com/Scope)(“prototype”):原型模式。
- [@Configuration](https://github.com/Configuration) : 代表这是一个配置类，就像我们之前看的beans.xml。配合[@Bean](https://github.com/Bean)使用将实体类注册。
- [@Bean](https://github.com/Bean) : 注册一个bean，就相当于我们之前写的一个bean标签;这个方法的名字，就相当于bean标签中的id属性;这个方法的返回值，就相当于bean标签中的class属性。
- [@Import](https://github.com/Import) : [@Import](https://github.com/Import)(XXX.class)可以引入其他配置类，使其合并为一个总配置类，使用时通过AnnotationConfig上下文来获取总配置类即可
- [@Data](https://github.com/Data) : Lombok中的注解,放在实体类上会自己创建set和get方法，toString方法等。实体类中只需要写属性即可。
- [@AllArgsConstructor](https://github.com/AllArgsConstructor),[@NoArgsConstructor](https://github.com/NoArgsConstructor) : 创建全部参数的有参构造，创建无参构造

> @Bean和@[Component](https://so.csdn.net/so/search?q=Component&spm=1001.2101.3001.7020) 注解的区别和作用
>
> 两个注解的作用
>
> 1、@Component： 作用于类上，告知Spring，为这个类创建Bean
>
> 2、@Bean：主要作用于方法上，告知Spring，这个方法会返回一个对象，且要注册在Spring的上下文中。通常方法体中包含产生Bean的逻辑。 相当于 xml文件的中<bean>标签

> Spring帮助我们管理Bean分为两个部分
>
> 一个是注册Bean(@Component , @Repository , @Controller , @Service , @Configration)
>
> 一个装配Bean(@Autowired , @Resource，可以通过byTYPE（@Autowired）、byNAME（@Resource）的方式获取Bean)。 完成这两个动作有三种方式，一种是使用自动配置的方式、一种是使用JavaConfig的方式，一种就是使用XML配置的方式

# spring、springmvc、springboot关系

## spring

+ Spring是一个三层框架，可以接管web层，业务层，dao层的组件，并且可以配置各种bean,和维护bean与bean之间的关系。其核心就是控制反转(IOC),和面向切面([AOP](https://so.csdn.net/so/search?q=AOP&spm=1001.2101.3001.7020)),简单的说就是一个分层的轻量级开源框架

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200920135841872.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzQ1MjcwNjY3,size_16,color_FFFFFF,t_70#pic_center)

## springmvc

+ Spring MVC属于[SpringFrameWork](https://so.csdn.net/so/search?q=SpringFrameWork&spm=1001.2101.3001.7020)的后续产品，已经融合在Spring Web Flow里面。SpringMVC是一种web层mvc框架，用于替代servlet处理响应请求，获取表单参数，表单校验等

## **SpringBoot**

+ 基于 Spring 的项目有很多的配置。当使用 Spring MVC 时，我们需要配置组件扫描，前段控制器 Servlet，视图解析器。当使用 Hibernate 时，我们需要配置数据源，实体工厂，事务管理等
+ Spring Boot 提供了使用这些框架应用程序所需的基本配置。这就是所谓的**自动配置**

# IOC

## 什么是IOC

### what

- 控制反转，把对象创建和对象之间的调用过程，交给Spring进行管理

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302021914119.webp)

- 在讨论控制反转之前，我们先来看看软件系统中耦合的对象。从图中可以看到，软件中的对象就像齿轮一样，协同工作，但是互相耦合，一个零件不能正常工作，整个系统就崩溃了。这是一个强耦合的系统。齿轮组中齿轮之间的啮合关系,与软件系统中对象之间的耦合关系非常相似。对象之间的耦合关系是无法避免的，也是必要的，这是协同工作的基础。现在，伴随着工业级应用的规模越来越庞大，对象之间的依赖关系也越来越复杂，经常会出现对象之间的多重依赖性关系，因此，架构师和设计师对于系统的分析和设计，将面临更大的挑战。对象之间耦合度过高的系统，必然会出现牵一发而动全身的情形。
- 为了解决对象间耦合度过高的问题，软件专家Michael Mattson提出了IoC理论，用来实现对象之间的“解耦”。
- **控制反转（Inversion of Control）**是一种是面向对象编程中的一种设计原则，用来减低计算机代码之间的耦合度。其基本思想是：**借助于“第三方”实现具有依赖关系的对象之间的解耦**

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302021915936.webp)

- 由于引进了中间位置的“第三方”，也就是IOC容器，使得A、B、C、D这4个对象没有了耦合关系，齿轮之间的传动全部依靠“第三方”了，全部对象的控制权全部上缴给“第三方”IOC容器，所以，IOC容器成了整个系统的关键核心，它起到了一种类似“粘合剂”的作用，把系统中的所有对象粘合在一起发挥作用，如果没有这个“粘合剂”，对象与对象之间会彼此失去联系，这就是有人把IOC容器比喻成“粘合剂”的由来

-  我们再来看看，控制反转(IOC)到底为什么要起这么个名字？我们来对比一下

  - 软件系统在没有引入IOC容器之前，如图1所示，对象A依赖于对象B，那么对象A在初始化或者运行到某一点的时候，自己必须主动去创建对象B或者使用已经创建的对象B。无论是创建还是使用对象B，控制权都在自己手上

  - 软件系统在引入IOC容器之后，这种情形就完全改变了，如图2所示，由于IOC容器的加入，对象A与对象B之间失去了直接联系，所以，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方

  -  通过前后的对比，我们不难看出来：对象A获得依赖对象B的过程,由主动行为变为了被动行为，控制权颠倒过来了，这就是“控制反转”这个名称的由来

## 为什么使用IOC

- 使用IOC最大的好处就是**降低了代码的耦合度，降低了程序的维护成本**。可能很多人都知道这个道理，就是不太明白它到底是怎么降低的，别慌下面让我来给大家讲解一下
- 假设现在有一道菜：宫保鸡丁

```java
// 伪代码
public class KungPaoChicken {
    
    public static KungPaoChicken getKungPaoChicken(各种食材) {
        // 加工各种食材最终得到一份美味的宫爆鸡丁。
        return KungPaoChicken;
    }
}
```

### **传统做法**

+ 如果现在不使用IOC，我们想要吃到宫保鸡丁，那么就需要如下操作

```java
// 伪代码
public class Person() {
    // 采购各种食材
    // 准备好各种食材通过KungPaoChicken获取到一份宫保鸡丁。
    KungPaoChicken kungPaoChicken = KungPaoChicken.getKungPaoChicken(各种食材);
}
```

+ 代码之间的耦合关系图

![image.png](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031021864.png)

- 看起来也不难，也不麻烦对吧？
- 别着急下定论，现在只是一个人想要宫保鸡丁，假如现在有10个人都想要那？是不是有十份相同的代码？这10个人都和KungPaoChicken有耦合。又比如现在需要的食材有所改变，那这样的话是不是这10个人都需要调整代码？这么一来是不是发现这种实现方式一点也不友好

### **使用IOC的做法**

+ 现在我们转变一下思路，不再自己动手做了，我们把这道菜的做法告诉饭店，让饭店来做

```java
// 伪代码
public class Restaurant {
    
    public static KungPaoChicken getKungPaoChicken() {
        // 处理食材，返回宫保鸡丁
        retur KungPaoChicken;
    }
}
```

+ 转变之后的耦合关系图

![image.png](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031022541.png)

+ 经过这样处理，就可以很大程度上解决上面的这些问题。

> 1、我们将KungPaoChicken交给Restaurant（饭店）来进行管理，它的创建由Restaurant进行。 2、现在不论是1个人还是10个人我们只需要从Restaurant中进行获取就可以了。这样耦合就改变了，Person只需要和Restaurant发生耦合就可以了。 3、当KungPaoChicken有变动时，也不需要每个人都变动，只需要Restaurant随之改变就可以了。

+ Spring的IOC容器就充当了上面案例中的Restaurant角色，我们只需要告诉Spring哪些Bean需要Spring进行管理，然后通过指定的方式从IOC容器中获取即可

## Spring提供的IOC容器

+ Spring提供了一个接口BeanFactory。这个接口是Spring实现IOC容器的顶级接口，这个接口是Spring内部使用的，并不是专门为框架的使用者提供的

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031023694.png)

- 我们一般使用的是BeanFactory的子接口ApplicationContext接口，这个接口提供了更多并且更加强大的功能
- 在ApplicationContext接口中有三个常用的实现类分别是：AnnotationConfigApplicationContext、FileSystemXmlApplicationContext、ClassPathXmlApplicationContext
- 容器的创建需要读取配置文件或配置类，通过这些配置告诉Spring哪些bean是需要Spring来进行管理的
- 注意：读取配置文件时，如果读取绝对路径时入参需要添加前缀“file:”，读取相对路径时入参需要添加“classpath:”

### **AnnotationConfigApplicationContext**

> 作用：用于在全注解开发时，读取配置类的相关配置信息。 注意：通过@Configuration注解标注当前类为Spring的配置类

**示例代码**

```java
ApplicationContext context = new AnnotationConfigApplicationContext(自定义的配置类.class);
```

### **ClassPathXmlApplicationContext**

> 作用：默认加载classPath下的配置文件，也就是代码编译之后的classes文件夹下。 注意：使用ClassPathXmlApplicationContext读取相对路径时入参的“classpath:”是可以省略的。读取绝对路径时，需要在入参添加前缀“file:”。

**示例代码**

```java
// 相对路径
ApplicationContext context = new ClassPathXmlApplicationContext("classpath:配置文件名称.xml");
ApplicationContext context = new ClassPathXmlApplicationContext("配置文件名称.xml");

// 绝对路径
ApplicationContext context = new ClassPathXmlApplicationContext("file:绝对路径下的配置文件路径")；
```

### **FileSystemXmlApplicationContext**

> 作用：默认加载的是项目的所在路径下的配置文件。注意：对FileSystemXmlApplicationContext来说读取绝对路径时的入参前缀“file:”是可以省略的，但是读取相对路径的入参“classpath:”是必须的。

**示例代码**

```java
// 相对路径
ApplicationContext context = new FileSystemXmlApplicationContext("classpath:beans.xml");

// 绝对路径
ApplicationContext context = new FileSystemXmlApplicationContext("file:绝对路径下的配置文件路径");
ApplicationContext context = new FileSystemXmlApplicationContext("绝对路径下的配置文件路径");
// 直接从项目的路径下
ApplicationContext context = new FileSystemXmlApplicationContext("src\main\resources\配置文件名");
```

## 依赖注入

- **依赖注入**就是将实例变量传入到一个对象中去
- 如果在 Class A 中，有 Class B 的实例，则称 Class A 对 Class B 有一个依赖。例如下面类 Human 中用到一个 Father 对象，我们就说类 Human 对类 Father 有一个依赖

```java
public class Human {
    ...
        Father father;
    ...
        public Human() {
        father = new Father();
    }
}
```

- 仔细看这段代码我们会发现存在一些问题
  - 如果现在要改变 father 生成方式，如需要用new Father(String name)初始化 father，需要修改 Human 代码
  - 如果想测试不同 Father 对象对 Human 的影响很困难，因为 father 的初始化被写死在了 Human 的构造函数中
  - 如果new Father()过程非常缓慢，单测时我们希望用已经初始化好的 father 对象 Mock 掉这个过程也很困难

+ 上面将依赖在构造函数中直接初始化是一种 Hard init 方式，弊端在于两个类不够独立，不方便测试。我们还有另外一种 Init 方式，如下

```java
public class Human {
    ...
        Father father;
    ...
        public Human(Father father) {
        this.father = father;
    }
}
```

- 上面代码中，我们将 father 对象作为构造函数的一个参数传入。在调用 Human 的构造方法之前外部就已经初始化好了 Father 对象。像这种非自己主动初始化依赖，而通过外部来传入依赖的方式，我们就称为依赖注入
- 现在我们发现上面 1 中存在的两个问题都很好解决了，简单的说依赖注入主要有两个好处
  - 解耦，将依赖之间解耦。
  - 因为已经解耦，所以方便做单元测试，尤其是 Mock 测试

## 控制反转和依赖注入的关系

我们已经分别解释了控制反转和依赖注入的概念。有些人会把控制反转和依赖注入等同，但实际上它们有着本质上的不同

- **控制反转**是一种思想
- **依赖注入**是一种设计模式

IoC框架使用依赖注入作为实现控制反转的方式，但是控制反转还有其他的实现方式，例如说[ServiceLocator](https://link.jianshu.com?t=http://martinfowler.com/articles/injection.html#UsingAServiceLocator)，所以不能将控制反转和依赖注入等同

## Spring中的依赖注入

```java
class MovieLister...
    private MovieFinder finder;
    public void setFinder(MovieFinder finder) {
        this.finder = finder;
    }

class ColonMovieFinder...
    public void setFilename(String filename) {
        this.filename = filename;
    }
```

- 我们先定义两个类，可以看到都使用了依赖注入的方式，通过外部传入依赖，而不是自己创建依赖。那么问题来了，谁把依赖传给他们，也就是说谁负责创建finder，并且把finder传给MovieLister。答案是Spring的IoC容器
- 要使用IoC容器，首先要进行配置。这里我们使用xml的配置，也可以通过代码注解方式配置。下面是spring.xml的内容

```xml
<beans>
    <bean id="MovieLister" class="spring.MovieLister">
        <property name="finder">
            <ref local="MovieFinder"/>
        </property>
    </bean>
    <bean id="MovieFinder" class="spring.ColonMovieFinder">
        <property name="filename">
            <value>movies1.txt</value>
        </property>
    </bean>
</beans>
```

+ 在Spring中，每个bean代表一个对象的实例，默认是单例模式，即在程序的生命周期内，所有的对象都只有一个实例，进行重复使用。通过配置bean，IoC容器在启动的时候会根据配置生成bean实例。具体的配置语法参考Spring文档。这里只要知道IoC容器会根据配置创建MovieFinder，在运行的时候把MovieFinder赋值给MovieLister的finder属性，完成依赖注入的过程

```java
public void testWithSpring() throws Exception {
    ApplicationContext ctx = new FileSystemXmlApplicationContext("spring.xml");//1
    MovieLister lister = (MovieLister) ctx.getBean("MovieLister");//2
    Movie[] movies = lister.moviesDirectedBy("Sergio Leone");
    assertEquals("Once Upon a Time in the West", movies[0].getTitle());
}
```

+ 根据配置生成ApplicationContext，即IoC容器

## IOC底层原理

+ **xml解析、工厂模式、反射**

![Spring5框架课堂笔记](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302021939441.jpg)

+ Spring 启动时读取应用程序提供的 Bean 配置信息，并在 Spring 容器中生成一份相应的 Bean 配置注册表，然后根据这张注册表实例化 Bean，装配好 Bean 之间的依赖关系，为上层应用提供准备就绪的运行环境。 其中 Bean 缓存池为 HashMap 实现

![image](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031039030.png)

### BeanFactory 接口

**IOC思想基于IOC容器完成，IOC容器底层就是对象工厂**

**Spring提供IOC容器实现两种方式：（两个接口）**

+ BeanFactory：IOC容器基本实现，是Spring内部的使用接口，不提供开发人员进行使用
  * 加载配置文件时候不会创建对象，在获取对象（使用）才去创建对象

* ApplicationContext：BeanFactory接口的子接口，提供更多更强大的功能，一般由开发人员进行使用
  * 加载配置文件时候就会把在配置文件对象进行创建

## IOC操作 Bean 管理

### 什么是Bean管理

- Spring创建对象
- Spirng注入属性

### 基于xml配置文件方式实现

#### 基于xml方式创建对象

- 在spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象创建

- 在bean标签有很多属性，介绍常用的属性

  * id属性：唯一标识

  * class属性：类全路径（包类路径）

  * 创建对象时候，默认也是执行无参数构造方法完成对象创建

#### 基于xml方式注入属性

##### 使用set方法进行注入

+ 创建类，定义属性和对应的set方法

```java
/** * 演示使用set方法进行注入属性 */ 
public class Book { 
    //创建属性 
    private String bname; 
    private String bauthor; 
    //创建属性对应的set方法 
    public void setBname(String bname) { 
        this.bname = bname; } 
    public void setBauthor(String bauthor) { 
        this.bauthor = bauthor; } 
}
```

+ 在spring配置文件配置对象创建，配置属性注入

```xml
<bean id="book" class="com.atguigu.spring5.Book"> 
    <!--使用property完成属性注入 name：类里面属性名称 value：向属性注入的值 --> 
    <property name="bname" value="易筋经"></property> 
    <property name="bauthor" value="达摩老祖"></property> 
</bean>
```

##### 使用有参数构造进行注入

+ 创建类，定义属性，创建属性对应有参数构造方法

```java
/** * 使用有参数构造注入
*/ 
public class Orders { 
    //属性 
    private String oname; 
    private String address; 
    //有参数构造 
    public Orders(String oname,String address) { 
        this.oname = oname; this.address = address; 
    } 
}
```

+ 在spring配置文件中进行配置

```xml
<!--3 有参数构造注入属性--> 
<bean id="orders" class="com.atguigu.spring5.Orders"> 
    <constructor-arg name="oname" value="电脑"></constructor-arg> 
    <constructor-arg name="address" value="China"></constructor-arg> 
</bean>
```

##### xml 注入其他类型属性

#### FactoryBean

- Spring有两种类型bean，一种普通bean，另外一种工厂bean（FactoryBean）
- 普通bean：在配置文件中定义bean类型就是返回类型
- 工厂bean：在配置文件定义bean类型可以和返回类型不一样
  - 第一步 创建类，让这个类作为工厂bean，实现接口 FactoryBean
  - 第二步 实现接口里面的方法，在实现的方法中定义返回的bean类型

```java
public class MyBean implements FactoryBean<Course> { 
    //定义返回bean 
    @Override public Course getObject() throws Exception { 
        Course course = new Course(); 
        course.setCname("abc"); 
        return course; 
    }
}
```

```xml
<bean id="myBean" class="com.atguigu.spring5.factorybean.MyBean"> 
</bean>
```

> # Spring中BeanFactory与FactoryBean的区别
>
> ## BeanFactory
>
> `BeanFactory`是一个接口，它是Spring中工厂的顶层规范，是SpringIoc容器的核心接口，它定义了`getBean()`、`containsBean()`等管理Bean的通用方法。Spring的容器都是它的具体实现如：
>
> - DefaultListableBeanFactory
> - XmlBeanFactory
> - ApplicationContext
>
> ```java
> public interface BeanFactory {
> 
> 	//对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
> 	//如果需要得到工厂本身，需要转义
> 	String FACTORY_BEAN_PREFIX = "&";
> 
> 	//根据bean的名字，获取在IOC容器中得到bean实例
> 	Object getBean(String name) throws BeansException;
> 
> 	//根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。
> 	<T> T getBean(String name, @Nullable Class<T> requiredType) throws BeansException;
> 
> 	Object getBean(String name, Object... args) throws BeansException;
> 
> 	<T> T getBean(Class<T> requiredType) throws BeansException;
> 
> 	<T> T getBean(Class<T> requiredType, Object... args) throws BeansException;
> 
> 	//提供对bean的检索，看看是否在IOC容器有这个名字的bean
> 	boolean containsBean(String name);
> 
> 	//根据bean名字得到bean实例，并同时判断这个bean是不是单例
> 	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
> 
> 	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
> 
> 	boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;
> 
> 	boolean isTypeMatch(String name, @Nullable Class<?> typeToMatch) throws NoSuchBeanDefinitionException;
> 
> 	//得到bean实例的Class类型
> 	@Nullable
> 	Class<?> getType(String name) throws NoSuchBeanDefinitionException;
> 
> 	//得到bean的别名，如果根据别名检索，那么其原名也会被检索出来
> 	String[] getAliases(String name);
> }
> ```
>
> ### 使用场景
>
> - 从Ioc容器中获取Bean(byName or byType)
>
> - 检索Ioc容器中是否包含指定的Bean
>
> - 判断Bean是否为单例
>
> ## FactoryBean
>
> 首先它是一个Bean，但又不仅仅是一个Bean。它是一个能生产或修饰对象生成的工厂Bean，类似于设计模式中的工厂模式和装饰器模式。它能在需要的时候生产一个对象，且不仅仅限于它自身，它能返回任何Bean的实例
>
> ```java
> public interface FactoryBean<T> {
> 
> 	//从工厂中获取bean
> 	@Nullable
> 	T getObject() throws Exception;
> 
> 	//获取Bean工厂创建的对象的类型
> 	@Nullable
> 	Class<?> getObjectType();
> 
> 	//Bean工厂创建的对象是否是单例模式
> 	default boolean isSingleton() {
> 		return true;
> 	}
> }
> ```
>
> 从它定义的接口可以看出，`FactoryBean`表现的是一个工厂的职责。   **即一个Bean A如果实现了FactoryBean接口，那么A就变成了一个工厂，根据A的名称获取到的实际上是工厂调用`getObject()`返回的对象，而不是A本身，如果要获取工厂A自身的实例，那么需要在名称前面加上'`&`'符号**
>
> - getObject('name')返回工厂中的实例
> - getObject('&name')返回工厂本身的实例
>
> ### 使用场景
>
> 说了这么多，为什么要有`FactoryBean`这个东西呢，有什么具体的作用吗？
>
>  FactoryBean在Spring中最为典型的一个应用就是用来**创建AOP的代理对象**
>
> 我们知道AOP实际上是Spring在运行时创建了一个代理对象，也就是说这个对象，是我们在运行时创建的，而不是一开始就定义好的，这很符合工厂方法模式。更形象地说，AOP代理对象通过Java的反射机制，在运行时创建了一个代理对象，在代理对象的目标方法中根据业务要求织入了相应的方法。这个对象在Spring中就是——`ProxyFactoryBean`
>
> ## 区别
>
> - 他们两个都是个工厂，但`FactoryBean`本质上还是一个Bean，也归`BeanFactory`管理
> - `BeanFactory`是Spring容器的顶层接口，`FactoryBean`更类似于用户自定义的工厂接口

#### bean 作用域

+ 在Spring里面，默认情况下，bean是单实例对象
+ 在spring配置文件bean标签里面有属性（scope）用于设置单实例还是多实例
+ 第一个值 默认值，singleton，表示是单实例对象
+ 第二个值 prototype，表示是多实例对象

![Spring5框架课堂笔记](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302022046502.jpg)

+ singleton和prototype区别
  + 设置scope值是singleton时候，加载spring配置文件时候就会创建单实例对象
  + 设置scope值是prototype时候，不是在加载spring配置文件时候创建对象，在调用getBean方法时候创建多实例对象

+ 在Spring中，那些组成应用程序的主体及由Spring IoC容器所管理的对象，被称之为bean。简单地讲，bean就是由IoC容器初始化、装配及管理的对象

##### **Singleton**

Singleton是单例类型，就是在创建起容器时就同时自动创建了一个bean的对象，不管你是否使用，他都存在了，每次获取到的对象都是同一个对象。注意，Singleton作用域是Spring中的缺省作用域。要在XML中将bean定义成singleton，可以这样配置

```xml
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">
```

##### **Prototype**

+ Prototype是原型类型，它在我们创建容器的时候并没有实例化，而是当我们获取bean的时候才会去创建一个对象，而且我们每次获取到的对象都不是同一个对象。根据经验，对有状态的bean应该使用prototype作用域，而对无状态的bean则应该使用singleton作用域。在XML中将bean定义成prototype，可以这样配置

```xml
 <bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
  或
 <bean id="account" class="com.foo.DefaultAccount" singleton="false"/>
```

##### **Request**

+ 当一个bean的作用域为Request，表示在一次HTTP请求中，一个bean定义对应一个实例；即每个HTTP请求都会有各自的bean实例，它们依据某个bean定义创建而成。该作用域仅在基于web的Spring ApplicationContext情形下有效
+ 针对每次HTTP请求，Spring容器会根据loginAction bean的定义创建一个全新的LoginAction bean实例，且该loginAction bean实例仅在当前HTTP request内有效，因此可以根据需要放心的更改所建实例的内部状态，而其他请求中根据loginAction bean定义创建的实例，将不会看到这些特定于某个请求的状态变化。当处理请求结束，request作用域的bean实例将被销毁

##### **Session**

+ 当一个bean的作用域为Session，表示在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效
+ 针对某个HTTP Session，Spring容器会根据userPreferences bean定义创建一个全新的userPreferences bean实例，且该userPreferences bean仅在当前HTTP Session内有效。与request作用域一样，可以根据需要放心的更改所创建实例的内部状态，而别的HTTP Session中根据userPreferences创建的实例，将不会看到这些特定于某个HTTP Session的状态变化。当HTTP Session最终被废弃的时候，在该HTTP Session作用域内的bean也会被废弃掉

#### bean 生命周期

- 通过构造器创建bean实例（无参数构造）
- 为bean的属性设置值和对其他bean引用（调用set方法）
- 调用bean的初始化的方法（需要进行配置初始化的方法）
- bean可以使用了（对象获取到了）
- 当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

```java
public class Orders {
//无参数构造 
    public Orders() { 
        System.out.println("第一步 执行无参数构造创建bean实例"); 
    } 
    private String oname; 
    public void setOname(String oname) { 
        this.oname = oname; 
        System.out.println("第二步 调用set方法设置属性值"); 
    } 
    //创建执行的初始化的方法 
    public void initMethod() { 
        System.out.println("第三步 执行初始化的方法"); 
    } 
    //创建执行的销毁的方法 
    public void destroyMethod() { 
        System.out.println("第五步 执行销毁的方法"); 
    } 
}
<bean id="orders" class="com.atguigu.spring5.bean.Orders" init-method="initMethod" destroy-method="destroyMethod"> <property name="oname" value="手机"></property> </bean>
    
    
@Test public void testBean3() { 
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml"); 
    Orders orders = context.getBean("orders", Orders.class); 
    System.out.println("第四步 获取创建bean实例对象"); 
    System.out.println(orders); 
    //手动让bean实例销毁 
    context.close(); 
}
```

**bean的后置处理器，bean生命周期有七步**

- 通过构造器创建bean实例（无参数构造）
- 为bean的属性设置值和对其他bean引用（调用set方法）
- 把bean实例传递bean后置处理器的方法postProcessBeforeInitialization
- 调用bean的初始化的方法（需要进行配置初始化的方法）
- 把bean实例传递bean后置处理器的方法 postProcessAfterInitialization
- bean可以使用了（对象获取到了）
- 当容器关闭时候，调用bean的销毁的方法（需要进行配置销毁的方法）

**演示添加后置处理器效果**

+ 创建类，实现接口BeanPostProcessor，创建后置处理器

```java
public class MyBeanPost implements BeanPostProcessor { 
    @Override public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException { 
        System.out.println("在初始化之前执行的方法"); 
        return bean; 
    } 
    @Override public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException { 
        System.out.println("在初始化之后执行的方法"); 
        return bean; 
    } 
}
```

```xml
<!--配置后置处理器--> 
<bean id="myBeanPost" class="com.atguigu.spring5.bean.MyBeanPost"></bean>
```

#### xml 自动装配

**什么是自动装配**

+ 根据指定装配规则（属性名称或者属性类型），Spring自动将匹配的属性值进行注入

**演示自动装配过程**

+ 根据属性名称自动注入

```xml
<!--实现自动装配 bean标签属性autowire，
配置自动装配 autowire属性常用两个值： 
byName根据属性名称注入 ，注入值bean的id值和类属性名称一样 
byType根据属性类型注入 --> 
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byName"> <!--<property name="dept" ref="dept"></property>-->
</bean> 
<bean id="dept" class="com.atguigu.spring5.autowire.Dept"></bean>
```

+ 根据属性类型自动注入

```xml
<bean id="emp" class="com.atguigu.spring5.autowire.Emp" autowire="byType"> 
</bean>
```

### 基于注解方式

#### 什么是注解

- 注解是代码特殊标记，格式：@注解名称(属性名称=属性值, 属性名称=属性值..)
- 使用注解，注解作用在类上面，方法上面，属性上面
- 使用注解目的：简化xml配置

**Spring针对Bean管理中创建对象提供注解**

- @Component
- @Service
- @Controller
- @Repository

上面四个注解功能是一样的，都可以用来创建bean实例

#### **基于注解方式实现对象创建**

**引入依赖**

**开启组件扫描**

+ 方法一

```xml
<!--开启组件扫描 1 如果扫描多个包，多个包使用逗号隔开 2 扫描包上层目录 --> 
<!--示例1 use-default-filters="false" 表示现在不使用默认filter，自己配置filter context:include-filter ，设置扫描哪些内容 --> 
<context:component-scan base-package="com.atguigu" use-default-filters="false"> 
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/> </context:component-scan>
```

+ 方法二
+ 创建配置类，替代xml配置文件 

```java
@Configuration //作为配置类，替代xml配置文件 
@ComponentScan(basePackages = {"com.atguigu"}) 
public class SpringConfig { 
}
```

**创建类，在类上面添加创建对象注解**

```java
//在注解里面value属性值可以省略不写， 
//默认值是类名称，首字母小写
//UserService -- userService
@Component(value = "userService") //<bean id="userService" class=".."/> 
public class UserService { public void add() { 
    System.out.println("service add......."); 
} 
```

#### @Autowired

- Spring 2.5 引入了 @Autowired 注释，它可以对类成员变量、方法及构造函数、参数等进行标注【主要还是用在**变量**和**方法**上】，完成自动装配的工作。 通过 @Autowired的使用来消除 set ，get方法，**也就是说，使用@Autowired注解注入属性数据不需要这个类提供set方法，方便快捷**。`@Autowired`作用就和在xml配置文件中的bean标签中写一个`< property >`标签的作用是一样的
- 在之前的文章[Spring中如何使用工厂模式实现程序解耦？](https://link.juejin.cn?target=https%3A%2F%2Fblog.csdn.net%2Fqq_44543508%2Farticle%2Fdetails%2F103680068)中，我们多多少少知道spring的IOC底层实际上就是一个**Map**结构容器，所谓**key 就是 bean标签 中的 id，value 则是对应 bean标签 中的 class**
- @Autowired自动装配首先会在IOC容器中跳过key直接去容器中找到对应的属性！也就是说与key无关
- @Autowired自动装配的三种情况

> 1、容器中有唯一的一个bean对象类型和被@Autowired修饰的变量类型匹配，就可以注入成功！ 2、容器中没有一个bean对象类型和被@Autowired修饰的变量类型匹配，则注入失败运行报错。 3、容器中有多个bean对象类型和被@Autowired修饰的变量类型匹配，则根据被@Autowired修饰的变量名寻找，找到则注入成功【**重点**】(但@Autowired无法匹配变量名)

#### @Qualifier

+ 根据上面@Autowired的第三种情况，需要更改变量名来对应注入，这样就对程序不是很灵活，于是有了@Qualifier这个注解。@Qualifier的作用是在按照类中注入的基础之上再按照名称注入。它在给类成员注入时不能单独使用（但是在给方法参数注入时可以单独使用），因此@Qualifier注解很受限制，因此用的不是很多。**@Qualifier常常组合@Autowired一起使用，用来指明具体名字的自动装配**

```java

    @Autowired //如果单纯一个@Autowired 注解则表示找类型为IAccuntDao的，如果有两个类型为IAccuntDao的，则接着匹配类型为IAccuntDao而且名字为accountDao的【缺点：要改变量名指定】
    @Qualifier("accountDao2") //加上这个注解直接找类型为IAccuntDao而且名字为accountDao2的
    private IAccuntDao accountDao;
    
    //所以这段代码注解的意思就是直接找类型为IAccuntDao而且名字为accountDao的组件
```

#### @Resource

+ @Resource由J2EE提供，默认是按照byName自动注入（通过名字自动注入），@Resource有两个重要的属性，name和type，当然默认是通过name，这里type属性就没必要讲了，用type属性多此一举，还不如用@Autowired，因此对于@Resource记住通过名字自动注入就好了

#### @Autowired、@Resource的区别

不得不说这两个注解非常相似，而且很容易混淆。

@Autowired、@Resource的主要区别主要有下面几点

|              | @Autowired | @Resource |
| ------------ | ---------- | --------- |
| 注解提供者   | Spring     | J2EE      |
| 自动装配方式 | 属性       | 名字      |

> 比较重要的一点就：**@Resource 相当于 @Autowired + @Qualifier**

#### @Value

- 由于@Autowired、@Qualifier、@Resource三者自动装配只能针对于注入其他bean类型的数据，而基本类型和String类型无法使用上述注解实现。因此有了@Value这个注解，@Value专门用来服务基本类型和String类型。
- 另外@Value注解有一个value 属性：用于指定数据的值。它可以使用spring中SpEL(也就是spring的EL表达式）。SpEL的写法：${表达式}，当然也可以类似mybatis中的 #{表达式} 的写法

```java
@Value("#{2*3}")  //#写法 表示6
private int age;

@Value("178")    //普遍写法 178
private int height;

@Value("${man.weight}")  //SpEL的写法一般操作配置文件中数据
private int weight;
```

> 注意：集合类型的注入只能通过XML来实现

# AOP

- AOP : 面向切面编程，通过预编译方式和运行期动态代理实现程序功能统一维护的一种技术

## why

- **通过切面，可以实现不修改源代码，在主干功能中添加新功能**
- 利用AOP可以对业务逻辑的各个部分进行隔离，从而使得**业务逻辑各部分之间的耦合度降低**，提高程序的可重用性，同时提高了开发的效率

![Spring5框架课堂笔记](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202302031202483.jpg)

> AOP和OOP的区别：AOP是OOP的补充，当我们需要为多个对象引入一个公共行为，比如日志，操作记录等，就需要在每个对象中引用公共行为，这样程序就产生了大量的重复代码，使用AOP可以完美解决这个问题

## 基本知识点

- 切面：拦截器类，其中会定义切点以及通知
- **Joinpoint（连接点）：**程序执行时的某个特定的点，在Spring中就是某一个方法的执行 
- 切点：具体拦截的某个业务点。
- 通知：切面当中的方法，声明通知方法在目标业务层的执行位置，通知类型如下：
  - 前置通知：@Before 在目标业务方法执行之前执行
  - 后置通知：@After 在目标业务方法执行之后执行
  - 返回通知：@AfterReturning 在目标业务方法返回结果之后执行
  - 异常通知：@AfterThrowing 在目标业务方法抛出异常之后
  - 环绕通知：@Around 功能强大，可代替以上四种通知，还可以控制目标业务方法是否执行以及何时执行

## 样例

+ 我们现在有个学校管理系统，已经实现了对老师和学生的增删改，又新来个需求，说是对老师和学生的每次增删改做一个记录，到时候校长可以查看记录的列表。那么问题来了，怎么样处理是最好的解决办法呢？这里我罗列了三种解决办法，我们来看下他的优缺点
+ 最简单的就是第一种方法，我们直接在每次的增删改的函数当中直接实现这个记录的方法，这样代码的重复度太高，耦合性太强，不建议使用
+ 其次就是我们最长使用的，将记录这个方法抽离出来，其他的增删改调用这个记录函数即可，显然代码重复度降低，但是这样的调用还是没有降低耦合性
+ 这个时候我们想一下AOP的定义，再想想我们的场景，其实我们就是要在不改变原来增删改的方法，给这个系统增加记录的方法，而且作用的也是一个层面的方法。这个时候我们就可以采用AOP来实现了

**具体实现**

+ 首先我定义了一个自定义注解作为切点

```java
@Target(AnnotationTarget.FUNCTION)  //注解作用的范围，这里声明为函数
@Order(Ordered.HIGHEST_PRECEDENCE)  //声明注解的优先级为最高，假设有多个注解，先执行这个
annotation class Hanler(val handler: HandlerType)  //自定义注解类，HandlerType是一个枚举类型，里面定义的就是学生和老师的增删改操作，在这里就不展示具体内容了
```

+ 接下来就是要定义切面类

```java
@Aspect   //该注解声明这个类为一个切面类
@Component
class HandlerAspect{
 
 @Autowired
 private lateinit var handlerService: HandlerService
 
@AfterReturning("@annotation(handler)")   //当有函数注释了注解，将会在函数正常返回后在执行我们定义的方法
fun hanler(hanler: Hanler) {
    handlerService.add(handler.operate.value)   //这里是真正执行记录的方法
}
}
```

+ 最后就是我们本来的业务方法

```java
/**
* 删除学生方法
*/
@Handler(operate= Handler.STUDENT_DELETE)   //当执行到删除学生方法时，切面类就会起作用了,当学生正常删除后就会执行记录方法，我们就可以看到记录方法生成的数据
fun delete(id：String) {
   studentService.delete(id)
}
```

## 底层原理

+ `Spring`的`AOP`实现原理其实很简单，就是通过**动态代理**实现的。如果我们为`Spring`的某个`bean`配置了切面，那么`Spring`在创建这个`bean`的时候，实际上创建的是这个`bean`的一个代理对象，我们后续对`bean`中方法的调用，实际上调用的是代理类重写的代理方法。而`Spring`的`AOP`使用了两种动态代理，分别是**JDK的动态代理**，以及**CGLib的动态代理**

> ## 代理是什么
>
> 代理模式，就是为其他的对象提供一种代理，以控制对这个对象的访问。Proxy代理对象与被代理对象对于调用方来说，完全一致，并且Proxy代理对调用方隐藏了被代理对象的实现细节。流程如下
>
> ![img](https://img-blog.csdnimg.cn/20200604210138666.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ld19jb20=,size_16,color_FFFFFF,t_70)
>
> ## 为什么要使用代理模式
>
> 没错，代理模式就是这么简单，可以这么理解，Proxy代理对象向调用方统一了对被代理对象的所有方法。有时，在调用被代理对象的正在执行的方法前，可能需要增加参数的校验逻辑，或者打印日志的逻辑；在执行完方法后，可能需要统计执行的时间，触发结束的事件等等逻辑。此时，如果在Proxy代理对象里动态地添加此类逻辑，就避免了在委托对象中硬编码。此时的执行流程1如下
> ![img](https://img-blog.csdnimg.cn/20200604211302558.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L25ld19jb20=,size_16,color_FFFFFF,t_70)

## 示例

### 静态代理

+ 首先，我们创建一个Person接口。这个接口就是学生（被代理类），和班长（代理类）的公共接口，他们都有交作业的行为。这样，学生交作业就可以让班长来代理执行

```java
/**
 * Created by Mapei on 2018/11/7
 * 创建person接口
 */
public interface Person {
    //交作业
    void giveTask();
}
```

+ Student类实现Person接口，Student可以具体实施交作业这个行为

```java
/**
 * Created by Mapei on 2018/11/7
 */
public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
 
    public void giveTask() {
        System.out.println(name + "交语文作业");
    }
}
```

+ StudentsProxy类，这个类也实现了Person接口，但是还另外持有一个学生类对象，那么他可以代理学生类对象执行交作业的行为

```java
/**
 * Created by Mapei on 2018/11/7
 * 学生代理类，也实现了Person接口，保存一个学生实体，这样就可以代理学生产生行为
 */
public class StudentsProxy implements Person{
    //被代理的学生
    Student stu;
 
    public StudentsProxy(Person stu) {
        // 只代理学生对象
        if(stu.getClass() == Student.class) {
            this.stu = (Student)stu;
        }
    }
 
    //代理交作业，调用被代理学生的交作业的行为
    public void giveTask() {
        stu.giveTask();
    }
}
```

+ 下面测试一下，看代理模式如何使用

```java
/**
 * Created by Mapei on 2018/11/7
 */
public class StaticProxyTest {
    public static void main(String[] args) {
        //被代理的学生林浅，他的作业上交有代理对象monitor完成
        Person linqian = new Student("林浅");
 
        //生成代理对象，并将林浅传给代理对象
        Person monitor = new StudentsProxy(linqian);
 
        //班长代理交作业
        monitor.giveTask();
    }
}
```

### 动态代理

**优点**

1. 动态代理可以在运行时动态地创建代理对象，而静态代理需要在编译时就确定代理对象
2. 动态代理可以代理多个类，而静态代理只能代理一个类
3. 动态代理不需要手动编写代理类，而静态代理需要手动编写代理类



+ 定义一个Person接口

```java
/**
 * Created by Mapei on 2018/11/7
 * 创建person接口
 */
public interface Person {
    //交作业
    void giveTask();
}
```

+ 创建需要被代理的实际类，也就是学生类

```java
/**
 * Created by Mapei on 2018/11/7
 */
public class Student implements Person {
    private String name;
    public Student(String name) {
        this.name = name;
    }
 
    public void giveTask() {
        System.out.println(name + "交语文作业");
    }
}
```

+ 创建StuInvocationHandler类，实现InvocationHandler接口，这个类中持有一个被代理对象的实例target。InvocationHandler中有一个invoke方法，所有执行代理对象的方法都会被替换成执行invoke方法

```java
/**
 * Created by Mapei on 2018/11/7
 */
public class StuInvocationHandler<T> implements InvocationHandler {
    //invocationHandler持有的被代理对象
    T target;
 
    public StuInvocationHandler(T target) {
        this.target = target;
    }
 
    /**
     * proxy:代表动态代理对象
     * method：代表正在执行的方法
     * args：代表调用目标方法时传入的实参
     */
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("代理执行" +method.getName() + "方法");
        Object result = method.invoke(target, args);
        return result;
    }
}
```

+ 具体的创建代理对象

```java
/**
 * Created by Mapei on 2018/11/7
 * 代理类
 */
public class ProxyTest {
    public static void main(String[] args) {
 
        //创建一个实例对象，这个对象是被代理的对象
        Person linqian = new Student("林浅");
 
        //创建一个与代理对象相关联的InvocationHandler
        InvocationHandler stuHandler = new StuInvocationHandler<Person>(linqian);
 
        //创建一个代理对象stuProxy来代理linqian，代理对象的每个执行方法都会替换执行Invocation中的invoke方法
        Person stuProxy = (Person) Proxy.newProxyInstance(Person.class.getClassLoader(), new Class<?>[]{Person.class}, stuHandler);
 
        //代理执行交作业的方法
        stuProxy.giveTask();
    }
}
```

