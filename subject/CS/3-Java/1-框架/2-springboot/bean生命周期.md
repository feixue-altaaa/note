```java
Spring帮助我们管理Bean分为两个部分 

一个是注册Bean(@Component , @Repository , @Controller , @Service , @Configration)， 
 
一个装配Bean(@Autowired , @Resource，可以通过byTYPE（@Autowired）、byNAME（@Resource）的方式获取Bean)。 完成这两个动作有三种方式，一种是使用自动配置的方式、一种是使用JavaConfig的方式，一种就是使用XML配置的方式。
 
```



# 什么是bean？

举个简单的例子，如果我们自己提供一个X.class类，那此时的X.class我们称之为class对象，在经过spring的一系列生命周期的处理(如后置处理器方法的处理)之后，就会变成可以放到spring容器中的bean对象，我们称此时的对象是bean对象

# Bean的生命周期

Spring Bean 的生命周期可以简单概括为 4 个阶段

1. 实例化（Instantiation）
2. 属性赋值（Populate）
3. 初始化（Initialization）
4. 销毁（Destruction）

![dc3c664a-78a3-4b1a-8328-5597a9319b0b](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408161511288.jpg)

## 阶段1：Bean元信息配置阶段

这个阶段主要是bean信息的定义阶段。

### Bean信息定义4种方式

- API的方式
- Xml文件方式
- properties文件的方式
- 注解的方式

### API的方式

先来说这种方式，因为其他几种方式最终都会采用这种方式来定义bean配置信息。

**Spring容器启动的过程中，会将Bean解析成Spring内部的BeanDefinition结构**。
不管是是通过xml配置文件的`<Bean>`标签，还是通过注解配置的`@Bean`，还是`@Compontent`标注的类，还是扫描得到的类，它最终都会被解析成一个BeanDefinition对象，最后我们的Bean工厂就会根据这份Bean的定义信息，对bean进行实例化、初始化等等操作。

你可以把BeanDefinition丢给Bean工厂，然后Bean工厂就会根据这个信息帮你生产一个Bean实例，拿去使用。

BeanDefinition里面里面包含了bean定义的各种信息，如：bean对应的class、scope、lazy信息、dependOn信息、autowireCandidate（是否是候选对象）、primary（是否是主要的候选者）等信息。

BeanDefinition是个接口，有几个实现类，看一下类图

![image-20240815095746265](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408150957418.png)

#### BeanDefinition接口：bean定义信息接口

表示bean定义信息的接口，里面定义了一些获取bean定义配置信息的各种方法，来看一下源码

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    /**
     * 设置此bean的父bean名称（对应xml中bean元素的parent属性）
     */
    void setParentName(@Nullable String parentName);
    /**
     * 返回此bean定义时指定的父bean的名称
     */
    @Nullable
    String getParentName();
    /**
     * 指定此bean定义的bean类名(对应xml中bean元素的class属性)
     */
    void setBeanClassName(@Nullable String beanClassName);
    /**
     * 返回此bean定义的当前bean类名
     * 注意，如果子定义重写/继承其父类的类名，则这不一定是运行时使用的实际类名。此外，这可能只是调用工厂方法的类，或者在调用方法的工厂bean引用的情况下，它甚至可能是空的。因此，不要认为这是运行时的最终bean类型，而只将其用于单个bean定义级别的解析目的。
     */
    @Nullable
    String getBeanClassName();
    /**
     * 设置此bean的生命周期，如：singleton、prototype（对应xml中bean元素的scope属性）
     */
    void setScope(@Nullable String scope);
    /**
     * 返回此bean的生命周期，如：singleton、prototype
     */
    @Nullable
    String getScope();
    /**
     * 设置是否应延迟初始化此bean（对应xml中bean元素的lazy属性）
     */
    void setLazyInit(boolean lazyInit);
    /**
     * 返回是否应延迟初始化此bean，只对单例bean有效
     */
    boolean isLazyInit();
    /**
     * 设置此bean依赖于初始化的bean的名称,bean工厂将保证dependsOn指定的bean会在当前bean初始化之前先初始化好
     */
    void setDependsOn(@Nullable String... dependsOn);
    /**
     * 返回此bean所依赖的bean名称
     */
    @Nullable
    String[] getDependsOn();
    /**
     * 设置此bean是否作为其他bean自动注入时的候选者
     * autowireCandidate
     */
    void setAutowireCandidate(boolean autowireCandidate);
    /**
     * 返回此bean是否作为其他bean自动注入时的候选者
     */
    boolean isAutowireCandidate();
    /**
     * 设置此bean是否为自动注入的主要候选者
     * primary：是否为主要候选者
     */
    void setPrimary(boolean primary);
    /**
     * 返回此bean是否作为自动注入的主要候选者
     */
    boolean isPrimary();
    /**
     * 指定要使用的工厂bean（如果有）。这是要对其调用指定工厂方法的bean的名称。
     * factoryBeanName：工厂bean名称
     */
    void setFactoryBeanName(@Nullable String factoryBeanName);
    /**
     * 返回工厂bean名称（如果有）（对应xml中bean元素的factory-bean属性）
     */
    @Nullable
    String getFactoryBeanName();
    /**
     * 指定工厂方法（如果有）。此方法将使用构造函数参数调用，如果未指定任何参数，则不使用任何参数调用。该方法将在指定的工厂bean（如果有的话）上调用，或者作为本地bean类上的静态方法调用。
     * factoryMethodName：工厂方法名称
     */
    void setFactoryMethodName(@Nullable String factoryMethodName);
    /**
     * 返回工厂方法名称（对应xml中bean的factory-method属性）
     */
    @Nullable
    String getFactoryMethodName();
    /**
     * 返回此bean的构造函数参数值
     */
    ConstructorArgumentValues getConstructorArgumentValues();
    /**
     * 是否有构造器参数值设置信息（对应xml中bean元素的<constructor-arg />子元素）
     */
    default boolean hasConstructorArgumentValues() {
        return !getConstructorArgumentValues().isEmpty();
    }
    /**
     * 获取bean定义是配置的属性值设置信息
     */
    MutablePropertyValues getPropertyValues();
    /**
     * 这个bean定义中是否有属性设置信息（对应xml中bean元素的<property />子元素）
     */
    default boolean hasPropertyValues() {
        return !getPropertyValues().isEmpty();
    }
    /**
     * 设置bean初始化方法名称
     */
    void setInitMethodName(@Nullable String initMethodName);
    /**
     * bean初始化方法名称
     */
    @Nullable
    String getInitMethodName();
    /**
     * 设置bean销毁方法的名称
     */
    void setDestroyMethodName(@Nullable String destroyMethodName);
    /**
     * bean销毁的方法名称
     */
    @Nullable
    String getDestroyMethodName();
    /**
     * 设置bean的role信息
     */
    void setRole(int role);
    /**
     * bean定义的role信息
     */
    int getRole();
    /**
     * 设置bean描述信息
     */
    void setDescription(@Nullable String description);
    /**
     * bean描述信息
     */
    @Nullable
    String getDescription();
    /**
     * bean类型解析器
     */
    ResolvableType getResolvableType();
    /**
     * 是否是单例的bean
     */
    boolean isSingleton();
    /**
     * 是否是多列的bean
     */
    boolean isPrototype();
    /**
     * 对应xml中bean元素的abstract属性，用来指定是否是抽象的
     */
    boolean isAbstract();
    /**
     * 返回此bean定义来自的资源的描述（以便在出现错误时显示上下文）
     */
    @Nullable
    String getResourceDescription();
    @Nullable
    BeanDefinition getOriginatingBeanDefinition();
}
```

BeanDefinition接口上面还继承了2个接口：

- AttributeAccessor
- BeanMetadataElement

##### AttributeAccessor接口：属性访问接口

```java
public interface AttributeAccessor {
    /**
     * 设置属性->值
     */
    void setAttribute(String name, @Nullable Object value);
    /**
     * 获取某个属性对应的值
     */
    @Nullable
    Object getAttribute(String name);
    /**
     * 移除某个属性
     */
    @Nullable
    Object removeAttribute(String name);
    /**
     * 是否包含某个属性
     */
    boolean hasAttribute(String name);
    /**
     * 返回所有的属性名称
     */
    String[] attributeNames();
}
```

这个接口相当于key->value数据结构的一种操作，BeanDefinition继承这个，内部实际上是使用了LinkedHashMap来实现这个接口中的所有方法，通常我们通过这些方法来保存BeanDefinition定义过程中产生的一些附加信息。

##### BeanMetadataElement接口

看一下其源码

```java
public interface BeanMetadataElement {
    @Nullable
    default Object getSource() {
        return null;
    }
}
```

BeanDefinition继承这个接口，getSource返回BeanDefinition定义的来源，比如我们通过xml定义BeanDefinition的，此时getSource就表示定义bean的xml资源；若我们通过api的方式定义BeanDefinition，我们可以将source设置为定义BeanDefinition时所在的类，出错时，可以根据这个来源方便排错。

#### RootBeanDefinition类：表示根bean定义信息

通常bean中没有父bean的就使用这种表示

#### ChildBeanDefinition类：表示子bean定义信息

如果需要指定父bean的，可以使用ChildBeanDefinition来定义子bean的配置信息，里面有个`parentName`属性，用来指定父bean的名称。

#### GenericBeanDefinition类：通用的bean定义信息

既可以表示没有父bean的bean配置信息，也可以表示有父bean的子bean配置信息，这个类里面也有parentName属性，用来指定父bean的名称。

#### ConfigurationClassBeanDefinition类：表示通过配置类中@Bean方法定义bean信息

可以通过配置类中使用@Bean来标注一些方法，通过这些方法来定义bean，这些方法配置的bean信息最后会转换为ConfigurationClassBeanDefinition类型的对象

#### AnnotatedBeanDefinition接口：表示通过注解的方式定义的bean信息

里面有个方法

```java
AnnotationMetadata getMetadata();
```

用来获取定义这个bean的类上的所有注解信息。

#### BeanDefinitionBuilder：构建BeanDefinition的工具类

spring中为了方便操作BeanDefinition，提供了一个类：`BeanDefinitionBuilder`，内部提供了很多静态方法，通过这些方法可以非常方便的组装BeanDefinition对象，下面我们通过案例来感受一下

#### 案例1：组装一个简单的bean

##### 来个简单的类

```java
package com.javacode2018.lesson002.demo1;
public class Car {
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "Car{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

##### 测试用例

```java
@Test
public void test1() {
    //指定class
    BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName());
    //获取BeanDefinition
    BeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
    System.out.println(beanDefinition);
}
```

等效于

```xml
<bean class="com.javacode2018.lesson002.demo1.Car" />
```

##### 运行输出

```java
Root bean: class [com.javacode2018.lesson002.demo1.Car]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
```

#### 案例2：组装一个有属性的bean

```java
@Test
public void test2() {
    //指定class
    BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName());
    //设置普通类型属性
    beanDefinitionBuilder.addPropertyValue("name", "奥迪"); //@1
    //获取BeanDefinition
    BeanDefinition carBeanDefinition = beanDefinitionBuilder.getBeanDefinition();
    System.out.println(carBeanDefinition);
    //创建spring容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory(); //@2
    //调用registerBeanDefinition向容器中注册bean
    factory.registerBeanDefinition("car", carBeanDefinition); //@3
    Car bean = factory.getBean("car", Car.class); //@4
    System.out.println(bean);
}
```

@1：调用addPropertyValue给Car中的name设置值

@2：创建了一个spring容器

@3：将carBeanDefinition这个bean配置信息注册到spring容器中，bean的名称为car

@4：从容器中获取car这个bean，最后进行输出

##### 运行输出

```java
Root bean: class [com.javacode2018.lesson002.demo1.Car]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
Car{name='奥迪'}
```

第二行输出了从容器中获取的car这个bean实例对象。

#### 案例3：组装一个有依赖关系的bean

##### 再来个类

下面这个类中有个car属性，我们通过spring将这个属性注入进来。

```java
package com.javacode2018.lesson002.demo1;
public class User {
    private String name;
    private Car car;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Car getCar() {
        return car;
    }
    public void setCar(Car car) {
        this.car = car;
    }
    @Override
    public String toString() {
        return "User{" +
                "name='" + name + '\'' +
                ", car=" + car +
                '}';
    }
}
```

##### 重点代码

```java
@Test
public void test3() {
    //先创建car这个BeanDefinition
    BeanDefinition carBeanDefinition = BeanDefinitionBuilder.rootBeanDefinition(Car.class.getName()).addPropertyValue("name", "奥迪").getBeanDefinition();
    //创建User这个BeanDefinition
    BeanDefinition userBeanDefinition = BeanDefinitionBuilder.rootBeanDefinition(User.class.getName()).
            addPropertyValue("name", "路人甲Java").
            addPropertyReference("car", "car"). //@1
            getBeanDefinition();
    //创建spring容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //调用registerBeanDefinition向容器中注册bean
    factory.registerBeanDefinition("car", carBeanDefinition); 
    factory.registerBeanDefinition("user", userBeanDefinition);
    System.out.println(factory.getBean("car"));
    System.out.println(factory.getBean("user"));
}
```

@1：注入依赖的bean，需要使用addPropertyReference方法，2个参数，第一个为属性的名称，第二个为需要注入的bean的名称

上面代码等效于

```java
<bean id="car" class="com.javacode2018.lesson002.demo1.Car">
    <property name="name" value="奥迪"/>
</bean>
<bean id="user" class="com.javacode2018.lesson002.demo1.User">
    <property name="name" value="路人甲Java"/>
    <property name="car" ref="car"/>
</bean>
```

##### 运行输出

```java
Car{name='奥迪'}
User{name='路人甲Java', car=Car{name='奥迪'}}
```

#### 案例4：来2个有父子关系的bean

```java
@Test
public void test4() {
    //先创建car这个BeanDefinition
    BeanDefinition carBeanDefinition1 = BeanDefinitionBuilder.
            genericBeanDefinition(Car.class).
            addPropertyValue("name", "保时捷").
            getBeanDefinition();
    BeanDefinition carBeanDefinition2 = BeanDefinitionBuilder.
            genericBeanDefinition(). //内部生成一个GenericBeanDefinition对象
            setParentName("car1"). //@1：设置父bean的名称为car1
            getBeanDefinition();
    //创建spring容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //调用registerBeanDefinition向容器中注册bean
    //注册car1->carBeanDefinition1
    factory.registerBeanDefinition("car1", carBeanDefinition1);
    //注册car2->carBeanDefinition2
    factory.registerBeanDefinition("car2", carBeanDefinition2);
    //从容器中获取car1
    System.out.println(String.format("car1->%s", factory.getBean("car1")));
    //从容器中获取car2
    System.out.println(String.format("car2->%s", factory.getBean("car2")));
}
```

等效于

```java
<bean id="car1" class="com.javacode2018.lesson002.demo1.Car">
    <property name="name" value="保时捷"/>
</bean>
<bean id="car2" parent="car1" />
```

##### 运行输出

```java
car1->Car{name='保时捷'}
car2->Car{name='保时捷'}
```

#### 案例5：通过api设置（Map、Set、List）属性

下面我们来演示注入List、Map、Set，内部元素为普通类型及其他bean元素

```java
import java.util.List;
import java.util.Map;
import java.util.Set;
public class CompositeObj {
    private String name;
    private Integer salary;
    private Car car1;
    private List<String> stringList;
    private List<Car> carList;
    private Set<String> stringSet;
    private Set<Car> carSet;
    private Map<String, String> stringMap;
    private Map<String, Car> stringCarMap;
    //此处省略了get和set方法，大家写的时候记得补上
    @Override
    public String toString() {
        return "CompositeObj{" +
                "name='" + name + '\'' +
                "\n\t\t\t, salary=" + salary +
                "\n\t\t\t, car1=" + car1 +
                "\n\t\t\t, stringList=" + stringList +
                "\n\t\t\t, carList=" + carList +
                "\n\t\t\t, stringSet=" + stringSet +
                "\n\t\t\t, carSet=" + carSet +
                "\n\t\t\t, stringMap=" + stringMap +
                "\n\t\t\t, stringCarMap=" + stringCarMap +
                '}';
    }
}
```

**注意：上面省略了get和set方法，大家写的时候记得补上**

##### 先用xml来定义一个CompositeObj的bean，如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
    <bean id="car1" class="com.javacode2018.lesson002.demo1.Car">
        <property name="name" value="奥迪"/>
    </bean>
    <bean id="car2" class="com.javacode2018.lesson002.demo1.Car">
        <property name="name" value="保时捷"/>
    </bean>
    <bean id="compositeObj" class="com.javacode2018.lesson002.demo1.CompositeObj">
        <property name="name" value="路人甲Java"/>
        <property name="salary" value="50000"/>
        <property name="car1" ref="car1"/>
        <property name="stringList">
            <list>
                <value>java高并发系列</value>
                <value>mysql系列</value>
                <value>maven高手系列</value>
            </list>
        </property>
        <property name="carList">
            <list>
                <ref bean="car1"/>
                <ref bean="car2"/>
            </list>
        </property>
        <property name="stringSet">
            <set>
                <value>java高并发系列</value>
                <value>mysql系列</value>
                <value>maven高手系列</value>
            </set>
        </property>
        <property name="carSet">
            <set>
                <ref bean="car1"/>
                <ref bean="car2"/>
            </set>
        </property>
        <property name="stringMap">
            <map>
                <entry key="系列1" value="java高并发系列"/>
                <entry key="系列2" value="Maven高手系列"/>
                <entry key="系列3" value="mysql系列"/>
            </map>
        </property>
        <property name="stringCarMap">
            <map>
                <entry key="car1" value-ref="car1"/>
                <entry key="car2" value-ref="car2"/>
            </map>
        </property>
    </bean>
</beans>
```

##### 下面我们采用纯api的方式实现，如下

```java
@Test
public void test5() {
    //定义car1
    BeanDefinition car1 = BeanDefinitionBuilder.
            genericBeanDefinition(Car.class).
            addPropertyValue("name", "奥迪").
            getBeanDefinition();
    //定义car2
    BeanDefinition car2 = BeanDefinitionBuilder.
            genericBeanDefinition(Car.class).
            addPropertyValue("name", "保时捷").
            getBeanDefinition();
    //定义CompositeObj这个bean
    //创建stringList这个属性对应的值
    ManagedList<String> stringList = new ManagedList<>();
    stringList.addAll(Arrays.asList("java高并发系列", "mysql系列", "maven高手系列"));
    //创建carList这个属性对应的值,内部引用其他两个bean的名称[car1,car2]
    ManagedList<RuntimeBeanReference> carList = new ManagedList<>();
    carList.add(new RuntimeBeanReference("car1"));
    carList.add(new RuntimeBeanReference("car2"));
    //创建stringList这个属性对应的值
    ManagedSet<String> stringSet = new ManagedSet<>();
    stringSet.addAll(Arrays.asList("java高并发系列", "mysql系列", "maven高手系列"));
    //创建carSet这个属性对应的值,内部引用其他两个bean的名称[car1,car2]
    ManagedList<RuntimeBeanReference> carSet = new ManagedList<>();
    carSet.add(new RuntimeBeanReference("car1"));
    carSet.add(new RuntimeBeanReference("car2"));
    //创建stringMap这个属性对应的值
    ManagedMap<String, String> stringMap = new ManagedMap<>();
    stringMap.put("系列1", "java高并发系列");
    stringMap.put("系列2", "Maven高手系列");
    stringMap.put("系列3", "mysql系列");
    ManagedMap<String, RuntimeBeanReference> stringCarMap = new ManagedMap<>();
    stringCarMap.put("car1", new RuntimeBeanReference("car1"));
    stringCarMap.put("car2", new RuntimeBeanReference("car2"));
    //下面我们使用原生的api来创建BeanDefinition
    GenericBeanDefinition compositeObj = new GenericBeanDefinition();
    compositeObj.setBeanClassName(CompositeObj.class.getName());
    compositeObj.getPropertyValues().add("name", "路人甲Java").
            add("salary", 50000).
            add("car1", new RuntimeBeanReference("car1")).
            add("stringList", stringList).
            add("carList", carList).
            add("stringSet", stringSet).
            add("carSet", carSet).
            add("stringMap", stringMap).
            add("stringCarMap", stringCarMap);
    //将上面bean 注册到容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerBeanDefinition("car1", car1);
    factory.registerBeanDefinition("car2", car2);
    factory.registerBeanDefinition("compositeObj", compositeObj);
    //下面我们将容器中所有的bean输出
    for (String beanName : factory.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName, factory.getBean(beanName)));
    }
}
```

> 有几点需要说一下：
>
> RuntimeBeanReference：用来表示bean引用类型，类似于xml中的ref

##### 看一下效果，运行输出

```
car1->Car{name='奥迪'}
car2->Car{name='保时捷'}
compositeObj->CompositeObj{name='路人甲Java'
            , salary=50000
            , car1=Car{name='奥迪'}
            , stringList=[java高并发系列, mysql系列, maven高手系列]
            , carList=[Car{name='奥迪'}, Car{name='保时捷'}]
            , stringSet=[java高并发系列, mysql系列, maven高手系列]
            , carSet=[Car{name='奥迪'}, Car{name='保时捷'}]
            , stringMap={系列1=java高并发系列, 系列2=Maven高手系列, 系列3=mysql系列}
            , stringCarMap={car1=Car{name='奥迪'}, car2=Car{name='保时捷'}}}
```

### Xml文件方式

这种方式已经讲过很多次了，大家也比较熟悉，即通过xml的方式来定义bean，如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
    <bean id="bean名称" class="bean完整类名"/>
</beans>
```

xml中的bean配置信息会被解析器解析为BeanDefinition对象，一会在第二阶段详解。

### properties文件的方式

这种方式估计大家比较陌生，将bean定义信息放在properties文件中，然后通过解析器将配置信息解析为BeanDefinition对象。

properties内容格式如下

```properties
employee.(class)=MyClass       // 等同于：<bean class="MyClass" />
employee.(abstract)=true       // 等同于：<bean abstract="true" />
employee.group=Insurance       // 为属性设置值，等同于：<property name="group" value="Insurance" />
employee.usesDialUp=false      // 为employee这个bean中的usesDialUp属性设置值,等同于：等同于：<property name="usesDialUp" value="false" />
salesrep.(parent)=employee     // 定义了一个id为salesrep的bean，指定父bean为employee，等同于：<bean id="salesrep" parent="employee" />
salesrep.(lazy-init)=true      // 设置延迟初始化，等同于：<bean lazy-init="true" />
salesrep.manager(ref)=tony     // 设置这个bean的manager属性值，是另外一个bean，名称为tony，等同于：<property name="manager" ref="tony" />
salesrep.department=Sales      // 等同于：<property name="department" value="Sales" />
techie.(parent)=employee       // 定义了一个id为techie的bean，指定父bean为employee，等同于：<bean id="techie" parent="employee" />
techie.(scope)=prototype       // 设置bean的作用域，等同于<bean scope="prototype" />
techie.manager(ref)=jeff       // 等同于：<property name="manager" ref="jeff" />
techie.department=Engineering  // <property name="department" value="Engineering" />
techie.usesDialUp=true         // <property name="usesDialUp" value="true" />
ceo.$0(ref)=secretary          // 设置构造函数第1个参数值，等同于：<constructor-arg index="0" ref="secretary" />
ceo.$1=1000000                 // 设置构造函数第2个参数值，等同于：<constructor-arg index="1" value="1000000" />
```

### 注解的方式

常见的2种

- 类上标注@Compontent注解来定义一个bean
- 配置类中使用@Bean注解来定义bean

### 小结

**bean注册者只识别BeanDefinition对象，不管什么方式最后都会将这些bean定义的信息转换为BeanDefinition对象，然后注册到spring容器中**

## 阶段2：Bean元信息解析阶段

Bean元信息的解析就是将各种方式定义的bean配置信息解析为BeanDefinition对象。

### Bean元信息的解析主要有3种方式

1. xml文件定义bean的解析
2. properties文件定义bean的解析
3. 注解方式定义bean的解析

### XML方式解析：XmlBeanDefinitionReader

spring中提供了一个类`XmlBeanDefinitionReader`，将xml中定义的bean解析为BeanDefinition对象。

直接来看案例代码

#### 来一个bean xml配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
    <bean id="car" class="com.javacode2018.lesson002.demo1.Car">
        <property name="name" value="奥迪"/>
    </bean>
    <bean id="car1" class="com.javacode2018.lesson002.demo1.Car">
        <property name="name" value="保时捷"/>
    </bean>
    <bean id="car2" parent="car1"/>
    <bean id="user" class="com.javacode2018.lesson002.demo1.User">
        <property name="name" value="路人甲Java"/>
        <property name="car" ref="car1"/>
    </bean>
</beans>
```

#### 将bean xml解析为BeanDefinition对象

```java
/**
 * xml方式bean配置信息解析
 */
@Test
public void test1() {
    //定义一个spring容器，这个容器默认实现了BeanDefinitionRegistry，所以本身就是一个bean注册器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //定义一个xml的BeanDefinition读取器，需要传递一个BeanDefinitionRegistry（bean注册器）对象
    XmlBeanDefinitionReader xmlBeanDefinitionReader = new XmlBeanDefinitionReader(factory);
    //指定bean xml配置文件的位置
    String location = "classpath:/com/javacode2018/lesson002/demo2/beans.xml";
    //通过XmlBeanDefinitionReader加载bean xml文件，然后将解析产生的BeanDefinition注册到容器容器中
    int countBean = xmlBeanDefinitionReader.loadBeanDefinitions(location);
    System.out.println(String.format("共注册了 %s 个bean", countBean));
    //打印出注册的bean的配置信息
    for (String beanName : factory.getBeanDefinitionNames()) {
        //通过名称从容器中获取对应的BeanDefinition信息
        BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
        //获取BeanDefinition具体使用的是哪个类
        String beanDefinitionClassName = beanDefinition.getClass().getName();
        //通过名称获取bean对象
        Object bean = factory.getBean(beanName);
        //打印输出
        System.out.println(beanName + ":");
        System.out.println("    beanDefinitionClassName：" + beanDefinitionClassName);
        System.out.println("    beanDefinition：" + beanDefinition);
        System.out.println("    bean：" + bean);
    }
}
```

> 注意一点：创建XmlBeanDefinitionReader的时候需要传递一个bean注册器(BeanDefinitionRegistry)，解析过程中生成的BeanDefinition会丢到bean注册器中

#### 运行输出

```
共注册了 4 个bean
car:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.Car]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo2/beans.xml]
    bean：Car{name='奥迪'}
car1:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.Car]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo2/beans.xml]
    bean：Car{name='保时捷'}
car2:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean with parent 'car1': class [null]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo2/beans.xml]
    bean：Car{name='保时捷'}
user:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.User]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo2/beans.xml]
    bean：User{name='路人甲Java', car=Car{name='奥迪'}}
```

上面的输出认真看一下，这几个BeanDefinition都是`GenericBeanDefinition`这种类型的，也就是说xml中定义的bean被解析之后都是通过`GenericBeanDefinition`这种类型表示的。

### properties文件定义bean的解析：PropertiesBeanDefinitionReader

spring中提供了一个类`XmlBeanDefinitionReader`，将xml中定义的bean解析为BeanDefinition对象，过程和xml的方式类似

下面通过properties文件的方式实现上面xml方式定义的bean

#### 来个properties文件：beans.properties

```properties
car.(class)=com.javacode2018.lesson002.demo1.Car
car.name=奥迪
car1.(class)=com.javacode2018.lesson002.demo1.Car
car1.name=保时捷
car2.(parent)=car1
user.(class)=com.javacode2018.lesson002.demo1.User
user.name=路人甲Java
user.car(ref)=car
```

**将bean properties文件解析为BeanDefinition对象**

```java
/**
 * properties文件方式bean配置信息解析
 */
@Test
public void test2() {
    //定义一个spring容器，这个容器默认实现了BeanDefinitionRegistry，所以本身就是一个bean注册器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //定义一个properties的BeanDefinition读取器，需要传递一个BeanDefinitionRegistry（bean注册器）对象
    PropertiesBeanDefinitionReader propertiesBeanDefinitionReader = new PropertiesBeanDefinitionReader(factory);
    //指定bean xml配置文件的位置
    String location = "classpath:/com/javacode2018/lesson002/demo2/beans.properties";
    //通过PropertiesBeanDefinitionReader加载bean properties文件，然后将解析产生的BeanDefinition注册到容器容器中
    int countBean = propertiesBeanDefinitionReader.loadBeanDefinitions(location);
    System.out.println(String.format("共注册了 %s 个bean", countBean));
    //打印出注册的bean的配置信息
    for (String beanName : factory.getBeanDefinitionNames()) {
        //通过名称从容器中获取对应的BeanDefinition信息
        BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
        //获取BeanDefinition具体使用的是哪个类
        String beanDefinitionClassName = beanDefinition.getClass().getName();
        //通过名称获取bean对象
        Object bean = factory.getBean(beanName);
        //打印输出
        System.out.println(beanName + ":");
        System.out.println("    beanDefinitionClassName：" + beanDefinitionClassName);
        System.out.println("    beanDefinition：" + beanDefinition);
        System.out.println("    bean：" + bean);
    }
}
```

#### 运行输出

```java
user:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.User]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：User{name='路人甲Java', car=Car{name='奥迪'}}
car1:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.Car]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：Car{name='保时捷'}
car:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo1.Car]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：Car{name='奥迪'}
car2:
    beanDefinitionClassName：org.springframework.beans.factory.support.GenericBeanDefinition
    beanDefinition：Generic bean with parent 'car1': class [null]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：Car{name='保时捷'}
```

输出和xml方式输出基本上一致

properties方式使用起来并不是太方便，所以平时我们很少看到有人使用

### 注解方式：AnnotatedBeanDefinitionReader

注解的方式定义的bean，需要使用AnnotatedBeanDefinitionReader这个类来进行解析，方式也和上面2种方式类似

#### 通过注解来标注2个类

##### Service1

```java
import org.springframework.beans.factory.config.ConfigurableBeanFactory;
import org.springframework.context.annotation.Lazy;
import org.springframework.context.annotation.Primary;
import org.springframework.context.annotation.Scope;
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Primary
@Lazy
public class Service1 {
}
```

> 这个类上面使用了3个注解，这些注解前面都介绍过，可以用来配置bean的信息
>
> 上面这个bean是个多例的

##### Service2

```java
import org.springframework.beans.factory.annotation.Autowired;
public class Service2 {
    @Autowired
    private Service1 service1; //@1
    @Override
    public String toString() {
        return "Service2{" +
                "service1=" + service1 +
                '}';
    }
}
```

#### 注解定义的bean解析为BeanDefinition

```java
@Test
public void test3() {
    //定义一个spring容器，这个容器默认实现了BeanDefinitionRegistry，所以本身就是一个bean注册器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //定义一个注解方式的BeanDefinition读取器，需要传递一个BeanDefinitionRegistry（bean注册器）对象
    AnnotatedBeanDefinitionReader annotatedBeanDefinitionReader = new AnnotatedBeanDefinitionReader(factory);
    //通过PropertiesBeanDefinitionReader加载bean properties文件，然后将解析产生的BeanDefinition注册到容器容器中
    annotatedBeanDefinitionReader.register(Service1.class, Service2.class);
    //打印出注册的bean的配置信息
    for (String beanName : new String[]{"service1", "service2"}) {
        //通过名称从容器中获取对应的BeanDefinition信息
        BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
        //获取BeanDefinition具体使用的是哪个类
        String beanDefinitionClassName = beanDefinition.getClass().getName();
        //通过名称获取bean对象
        Object bean = factory.getBean(beanName);
        //打印输出
        System.out.println(beanName + ":");
        System.out.println("    beanDefinitionClassName：" + beanDefinitionClassName);
        System.out.println("    beanDefinition：" + beanDefinition);
        System.out.println("    bean：" + bean);
    }
}
```

输出中可以看出service1这个bean的beanDefinition中lazyInit确实为true，primary也为true，scope为prototype，说明类Service1注解上标注3个注解信息被解析之后放在了beanDefinition中

**注意下：最后一行中的service1为什么为null，不是标注了@Autowired么？**

这个地方提前剧透一下，看不懂的没关系，这篇文章都结束之后，就明白了。

调整一下上面的代码，加上下面@1这行代码，如下

```java
@Test
public void test3() {
    //定义一个spring容器，这个容器默认实现了BeanDefinitionRegistry，所以本身就是一个bean注册器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //定义一个注解方式的BeanDefinition读取器，需要传递一个BeanDefinitionRegistry（bean注册器）对象
    AnnotatedBeanDefinitionReader annotatedBeanDefinitionReader = new AnnotatedBeanDefinitionReader(factory);
    //通过PropertiesBeanDefinitionReader加载bean properties文件，然后将解析产生的BeanDefinition注册到容器容器中
    annotatedBeanDefinitionReader.register(Service1.class, Service2.class);
    factory.getBeansOfType(BeanPostProcessor.class).values().forEach(factory::addBeanPostProcessor); // @1
    //打印出注册的bean的配置信息
    for (String beanName : new String[]{"service1", "service2"}) {
        //通过名称从容器中获取对应的BeanDefinition信息
        BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
        //获取BeanDefinition具体使用的是哪个类
        String beanDefinitionClassName = beanDefinition.getClass().getName();
        //通过名称获取bean对象
        Object bean = factory.getBean(beanName);
        //打印输出
        System.out.println(beanName + ":");
        System.out.println("    beanDefinitionClassName：" + beanDefinitionClassName);
        System.out.println("    beanDefinition：" + beanDefinition);
        System.out.println("    bean：" + bean);
    }
}
```

再次运行一下，最后一行有值了

```
service1:
    beanDefinitionClassName：org.springframework.beans.factory.annotation.AnnotatedGenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo2.Service1]; scope=prototype; abstract=false; lazyInit=true; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=true; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：com.javacode2018.lesson002.demo2.Service1@564718df
service2:
    beanDefinitionClassName：org.springframework.beans.factory.annotation.AnnotatedGenericBeanDefinition
    beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo2.Service2]; scope=singleton; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
    bean：Service2{service1=com.javacode2018.lesson002.demo2.Service1@52aa2946}
```

## 阶段3：Spring Bean注册阶段

bean注册阶段需要用到一个非常重要的接口：BeanDefinitionRegistry

### Bean注册接口：BeanDefinitionRegistry

这个接口中定义了注册bean常用到的一些方法，源码如下

```java
public interface BeanDefinitionRegistry extends AliasRegistry {
    /**
     * 注册一个新的bean定义
     * beanName：bean的名称
     * beanDefinition：bean定义信息
     */
    void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
            throws BeanDefinitionStoreException;
    /**
     * 通过bean名称移除已注册的bean
     * beanName：bean名称
     */
    void removeBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    /**
     * 通过名称获取bean的定义信息
     * beanName：bean名称
     */
    BeanDefinition getBeanDefinition(String beanName) throws NoSuchBeanDefinitionException;
    /**
     * 查看beanName是否注册过
     */
    boolean containsBeanDefinition(String beanName);
    /**
     * 获取已经定义（注册）的bean名称列表
     */
    String[] getBeanDefinitionNames();
    /**
     * 返回注册器中已注册的bean数量
     */
    int getBeanDefinitionCount();
    /**
     * 确定给定的bean名称或者别名是否已在此注册表中使用
     * beanName：可以是bean名称或者bean的别名
     */
    boolean isBeanNameInUse(String beanName);
}
```

### 别名注册接口：AliasRegistry

`BeanDefinitionRegistry`接口继承了`AliasRegistry`接口，这个接口中定义了操作bean别名的一些方法，看一下其源码

```java
public interface AliasRegistry {
    /**
     * 给name指定别名alias
     */
    void registerAlias(String name, String alias);
    /**
     * 从此注册表中删除指定的别名
     */
    void removeAlias(String alias);
    /**
     * 判断name是否作为别名已经被使用了
     */
    boolean isAlias(String name);
    /**
     * 返回name对应的所有别名
     */
    String[] getAliases(String name);
}
```

### BeanDefinitionRegistry唯一实现：DefaultListableBeanFactory

spring中BeanDefinitionRegistry接口有一个唯一的实现类：

```
org.springframework.beans.factory.support.DefaultListableBeanFactory
```

大家可能看到有很多类也实现了`BeanDefinitionRegistry`接口，比如我们经常用到的`AnnotationConfigApplicationContext`，但实际上其内部是转发给了`DefaultListableBeanFactory`进行处理的，所以真正实现这个接口的类是`DefaultListableBeanFactory`。

大家再回头看一下开头的几个案例，都使用的是`DefaultListableBeanFactory`作为bean注册器，此时你们应该可以理解为什么了。

下面我们来个案例演示一下上面常用的一些方法。

### 案例

```java
import org.junit.Test;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.beans.factory.support.GenericBeanDefinition;
import java.util.Arrays;
/**
 * BeanDefinitionRegistry 案例
 */
public class BeanDefinitionRegistryTest {
    @Test
    public void test1() {
        //创建一个bean工厂，这个默认实现了BeanDefinitionRegistry接口，所以也是一个bean注册器
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        //定义一个bean
        GenericBeanDefinition nameBdf = new GenericBeanDefinition();
        nameBdf.setBeanClass(String.class);
        nameBdf.getConstructorArgumentValues().addIndexedArgumentValue(0, "路人甲Java");
        //将bean注册到容器中
        factory.registerBeanDefinition("name", nameBdf);
        //通过名称获取BeanDefinition
        System.out.println(factory.getBeanDefinition("name"));
        //通过名称判断是否注册过BeanDefinition
        System.out.println(factory.containsBeanDefinition("name"));
        //获取所有注册的名称
        System.out.println(Arrays.asList(factory.getBeanDefinitionNames()));
        //获取已注册的BeanDefinition的数量
        System.out.println(factory.getBeanDefinitionCount());
        //判断指定的name是否使用过
        System.out.println(factory.isBeanNameInUse("name"));
        //别名相关方法
        //为name注册2个别名
        factory.registerAlias("name", "alias-name-1");
        factory.registerAlias("name", "alias-name-2");
        //判断alias-name-1是否已被作为别名使用
        System.out.println(factory.isAlias("alias-name-1"));
        //通过名称获取对应的所有别名
        System.out.println(Arrays.asList(factory.getAliases("name")));
        //最后我们再来获取一下这个bean
        System.out.println(factory.getBean("name"));
    }
}
```

#### 运行输出

```\
Generic bean: class [java.lang.String]; scope=; abstract=false; lazyInit=null; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null
true
[name]
1
true
true
[alias-name-2, alias-name-1]
路人甲Java
```

**下面要介绍的从阶段4到阶段14，也就是从：`BeanDefinition合并阶段`到`Bean初始化完成阶段`，都是在调用getBean从容器中获取bean对象的过程中发送的操作，要注意细看了，大家下去了建议去看getBean这个方法的源码，以下过程均来自于这个方法**

```java
org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
```

## 阶段4：BeanDefinition合并阶段

### 合并阶段是做什么的？

可能我们定义bean的时候有父子bean关系，此时子BeanDefinition中的信息是不完整的，比如设置属性的时候配置在父BeanDefinition中，此时子BeanDefinition中是没有这些信息的，需要将子bean的BeanDefinition和父bean的BeanDefinition进行合并，得到最终的一个`RootBeanDefinition`，合并之后得到的`RootBeanDefinition`包含bean定义的所有信息，包含了从父bean中继继承过来的所有信息，后续bean的所有创建工作就是依靠合并之后BeanDefinition来进行的

合并BeanDefinition会使用下面这个方法

```java
org.springframework.beans.factory.support.AbstractBeanFactory#getMergedBeanDefinition
```

**bean定义可能存在多级父子关系，合并的时候进进行递归合并，最终得到一个包含完整信息的RootBeanDefinition**

### 案例

#### 来一个普通的类

```java
public class LessonModel {
    //课程名称
    private String name;
    //课时
    private int lessonCount;
    //描述信息
    private String description;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public int getLessonCount() {
        return lessonCount;
    }
    public void setLessonCount(int lessonCount) {
        this.lessonCount = lessonCount;
    }
    public String getDescription() {
        return description;
    }
    public void setDescription(String description) {
        this.description = description;
    }
    @Override
    public String toString() {
        return "LessonModel{" +
                "name='" + name + '\'' +
                ", lessonCount=" + lessonCount +
                ", description='" + description + '\'' +
                '}';
    }
}
```

#### 通过xml定义3个具有父子关系的bean

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans-4.3.xsd">
    <bean id="lesson1" class="com.javacode2018.lesson002.demo4.LessonModel"/>
    <bean id="lesson2" parent="lesson1">
        <property name="name" value="spring高手系列"/>
        <property name="lessonCount" value="100"/>
    </bean>
    <bean id="lesson3" parent="lesson2">
        <property name="description" value="路人甲Java带你学spring，超越90%开发者!"/>
    </bean>
</beans>
```

> lesson2相当于lesson1的儿子，lesson3相当于lesson1的孙子

#### 解析xml注册bean

下面将解析xml，进行bean注册，然后遍历输出bean的名称，解析过程中注册的原始的BeanDefinition，合并之后的BeanDefinition，以及合并前后BeanDefinition中的属性信息

```java
/**
 * BeanDefinition 合并
 */
@Test
public void MergedBeanDefinitionTest() {
    //创建bean容器
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //创建一个bean xml解析器
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(factory);
    //解析bean xml，将解析过程中产生的BeanDefinition注册到DefaultListableBeanFactory中
    beanDefinitionReader.loadBeanDefinitions("beans.xml");
    //遍历容器中注册的所有bean信息
    for (String beanName : factory.getBeanDefinitionNames()) {
        //通过bean名称获取原始的注册的BeanDefinition信息
        BeanDefinition beanDefinition = factory.getBeanDefinition(beanName);
        //获取合并之后的BeanDefinition信息
        BeanDefinition mergedBeanDefinition = factory.getMergedBeanDefinition(beanName);
        System.out.println(beanName);
        System.out.println("解析xml过程中注册的beanDefinition：" + beanDefinition);
        System.out.println("beanDefinition中的属性信息" + beanDefinition.getPropertyValues());
        System.out.println("合并之后得到的mergedBeanDefinition：" + mergedBeanDefinition);
        System.out.println("mergedBeanDefinition中的属性信息" + mergedBeanDefinition.getPropertyValues());
        System.out.println("---------------------------");
    }
}
```

#### 运行输出

```
lesson1
解析xml过程中注册的beanDefinition：Generic bean: class [com.javacode2018.lesson002.demo4.LessonModel]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
beanDefinition中的属性信息PropertyValues: length=0
合并之后得到的mergedBeanDefinition：Root bean: class [com.javacode2018.lesson002.demo4.LessonModel]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
mergedBeanDefinition中的属性信息PropertyValues: length=0
---------------------------
lesson2
解析xml过程中注册的beanDefinition：Generic bean with parent 'lesson1': class [null]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
beanDefinition中的属性信息PropertyValues: length=2; bean property 'name'; bean property 'lessonCount'
合并之后得到的mergedBeanDefinition：Root bean: class [com.javacode2018.lesson002.demo4.LessonModel]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
mergedBeanDefinition中的属性信息PropertyValues: length=2; bean property 'name'; bean property 'lessonCount'
---------------------------
lesson3
解析xml过程中注册的beanDefinition：Generic bean with parent 'lesson2': class [null]; scope=; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
beanDefinition中的属性信息PropertyValues: length=1; bean property 'description'
合并之后得到的mergedBeanDefinition：Root bean: class [com.javacode2018.lesson002.demo4.LessonModel]; scope=singleton; abstract=false; lazyInit=false; autowireMode=0; dependencyCheck=0; autowireCandidate=true; primary=false; factoryBeanName=null; factoryMethodName=null; initMethodName=null; destroyMethodName=null; defined in class path resource [com/javacode2018/lesson002/demo4/beans.xml]
mergedBeanDefinition中的属性信息PropertyValues: length=3; bean property 'name'; bean property 'lessonCount'; bean property 'description'
---------------------------
```

从输出的结果中可以看到，合并之前，BeanDefinition是不完整的，比如lesson2和lesson3中的class是null，属性信息也不完整，但是合并之后这些信息都完整了

合并之前是`GenericBeanDefinition`类型的，合并之后得到的是`RootBeanDefinition`类型的

获取lesson3合并的BeanDefinition时，内部会递归进行合并，先将lesson1和lesson2合并，然后将lesson2再和lesson3合并，最后得到合并之后的BeanDefinition

**后面的阶段将使用合并产生的RootBeanDefinition**

## 阶段5：Bean Class加载阶段

**这个阶段就是将bean的class名称转换为Class类型的对象**

BeanDefinition中有个Object类型的字段：beanClass

```java
private volatile Object beanClass;
```

用来表示bean的class对象，通常这个字段的值有2种类型，一种是bean对应的Class类型的对象，另一种是bean对应的Class的完整类名，第一种情况不需要解析，第二种情况：即这个字段是bean的类名的时候，就需要通过类加载器将其转换为一个Class对象

此时会对阶段4中合并产生的`RootBeanDefinition`中的`beanClass`进行解析，将bean的类名转换为`Class对象`，然后赋值给`beanClass`字段

源码位置

```java
org.springframework.beans.factory.support.AbstractBeanFactory#resolveBeanClass
```

上面得到了Bean Class对象以及合并之后的BeanDefinition，下面就开始进入实例化这个对象的阶段了

**Bean实例化分为3个阶段：前阶段、实例化阶段、后阶段；下面详解介绍**

## 阶段6：Bean实例化阶段

### 分2个小的阶段

- Bean实例化前操作
- Bean实例化操作

### Bean实例化前操作

先来看一下`DefaultListableBeanFactory`，这个类中有个非常非常重要的字段

```java
private final List<BeanPostProcessor> beanPostProcessors = new CopyOnWriteArrayList<>();
```

是一个`BeanPostProcessor`类型的集合

**BeanPostProcessor是一个接口，还有很多子接口，这些接口中提供了很多方法，spring在bean生命周期的不同阶段，会调用上面这个列表中的BeanPostProcessor中的一些方法，来对生命周期进行扩展，bean生命周期中的所有扩展点都是依靠这个集合中的BeanPostProcessor来实现的，所以如果大家想对bean的生命周期进行干预，这块一定要掌握好。**

**注意：本文中很多以BeanPostProcessor结尾的，都实现了BeanPostProcessor接口，有些是直接实现的，有些是实现了它的子接口。**

Bean实例化之前会调用一段代码

```java
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
    for (BeanPostProcessor bp : getBeanPostProcessors()) {
        if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
            if (result != null) {
                return result;
            }
        }
    }
    return null;
}
```

这段代码在bean实例化之前给开发者留了个口子，开发者自己可以在这个地方直接去创建一个对象作为bean实例，而跳过spring内部实例化bean的过程。

上面代码中轮询`beanPostProcessors`列表，如果类型是`InstantiationAwareBeanPostProcessor`， 尝试调用`InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation`获取bean的实例对象，如果能够获取到，那么将返回值作为当前bean的实例，那么spring自带的实例化bean的过程就被跳过了。

`postProcessBeforeInstantiation`方法如下

```java
default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
    return null;
}
```

> 这个地方给开发者提供了一个扩展点，允许开发者在这个方法中直接返回bean的一个实例

#### 案例

```java
package com.javacode2018.lesson002.demo5;
import com.javacode2018.lesson002.demo1.Car;
import org.junit.Test;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.lang.Nullable;
/**
 * bean初始化前阶段，会调用：{@link org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessBeforeInitialization(Object, String)}
 */
public class InstantiationAwareBeanPostProcessorTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        //添加一个BeanPostProcessor：InstantiationAwareBeanPostProcessor
        factory.addBeanPostProcessor(new InstantiationAwareBeanPostProcessor() { //@1
            @Nullable
            @Override
            public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
                System.out.println("调用postProcessBeforeInstantiation()");
                //发现类型是Car类型的时候，硬编码创建一个Car对象返回
                if (beanClass == Car.class) {
                    Car car = new Car();
                    car.setName("保时捷");
                    return car;
                }
                return null;
            }
        });
        //定义一个car bean,车名为：奥迪
        AbstractBeanDefinition carBeanDefinition = BeanDefinitionBuilder.
                genericBeanDefinition(Car.class).
                addPropertyValue("name", "奥迪").  //@2
                getBeanDefinition();
        factory.registerBeanDefinition("car", carBeanDefinition);
        //从容器中获取car这个bean的实例，输出
        System.out.println(factory.getBean("car"));
    }
}
```

> @1：创建了一个InstantiationAwareBeanPostProcessor，丢到了容器中的BeanPostProcessor列表中
>
> @2：创建了一个car bean，name为奥迪

#### 运行输出

```
调用postProcessBeforeInstantiation()
Car{name='保时捷'}
```

> bean定义的时候，名称为：奥迪，最后输出的为：保时捷
>
> 定义和输出不一致的原因是因为我们在`InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation`方法中手动创建了一个实例直接返回了，而不是依靠spring内部去创建这个实例

#### 小结

实际上，在实例化前阶段对bean的创建进行干预的情况，用的非常少，所以大部分bean的创建还会继续走下面的阶段

### Bean实例化操作

#### 这个过程可以干什么？

这个过程会通过反射来调用bean的构造器来创建bean的实例

具体需要使用哪个构造器，spring为开发者提供了一个接口，允许开发者自己来判断用哪个构造器

看一下这块的代码逻辑

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
        SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
        Constructor<?>[] ctors = ibp.determineCandidateConstructors(beanClass, beanName);
        if (ctors != null) {
            return ctors;
        }
    }
}
```

会调用`SmartInstantiationAwareBeanPostProcessor接口的determineCandidateConstructors`方法，这个方法会返回候选的构造器列表，也可以返回空，看一下这个方法的源码

```java
@Nullable
default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
throws BeansException {
    return null;
}
```

这个方法有个比较重要的实现类

```java
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
```

可以将`@Autowired`标注的方法作为候选构造器返回，有兴趣的可以去看一下代码

#### 案例

**下面我们来个案例，自定义一个注解，当构造器被这个注解标注的时候，让spring自动选择使用这个构造器创建对象**

##### 自定义一个注解

下面这个注解可以标注在构造器上面，使用这个标注之后，创建bean的时候将使用这个构造器

```java
package com.javacode2018.lesson002.demo6;
import java.lang.annotation.*;
@Target(ElementType.CONSTRUCTOR)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface MyAutowried {
}
```

##### 来个普通的类

下面这个类3个构造器，其中一个使用`@MyAutowried`，让其作为bean实例化的方法

```java
public class Person {
    private String name;
    private Integer age;
    public Person() {
        System.out.println("调用 Person()");
    }
    @MyAutowried
    public Person(String name) {
        System.out.println("调用 Person(String name)");
        this.name = name;
    }
    public Person(String name, Integer age) {
        System.out.println("调用 Person(String name, int age)");
        this.name = name;
        this.age = age;
    }
    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

##### 自定义一个SmartInstantiationAwareBeanPostProcessor

代码的逻辑：将`@MyAutowried`标注的构造器列表返回

```java
package com.javacode2018.lesson002.demo6;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor;
import org.springframework.lang.Nullable;
import java.lang.reflect.Constructor;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;
public class MySmartInstantiationAwareBeanPostProcessor implements SmartInstantiationAwareBeanPostProcessor {
    @Nullable
    @Override
    public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) throws BeansException {
        System.out.println(beanClass);
        System.out.println("调用 MySmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors 方法");
        Constructor<?>[] declaredConstructors = beanClass.getDeclaredConstructors();
        if (declaredConstructors != null) {
            //获取有@MyAutowried注解的构造器列表
            List<Constructor<?>> collect = Arrays.stream(declaredConstructors).
                    filter(constructor -> constructor.isAnnotationPresent(MyAutowried.class)).
                    collect(Collectors.toList());
            Constructor[] constructors = collect.toArray(new Constructor[collect.size()]);
            return constructors.length != 0 ? constructors : null;
        } else {
            return null;
        }
    }
}
```

##### 来个测试用例

```java
package com.javacode2018.lesson002.demo6;
import org.junit.Test;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
/**
 * 通过{@link org.springframework.beans.factory.config.SmartInstantiationAwareBeanPostProcessor#determineCandidateConstructors(Class, String)}来确定使用哪个构造器来创建bean实例
 */
public class SmartInstantiationAwareBeanPostProcessorTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        //创建一个SmartInstantiationAwareBeanPostProcessor,将其添加到容器中
        factory.addBeanPostProcessor(new MySmartInstantiationAwareBeanPostProcessor());
        factory.registerBeanDefinition("name",
                BeanDefinitionBuilder.
                        genericBeanDefinition(String.class).
                        addConstructorArgValue("路人甲Java").
                        getBeanDefinition());
        factory.registerBeanDefinition("age",
                BeanDefinitionBuilder.
                        genericBeanDefinition(Integer.class).
                        addConstructorArgValue(30).
                        getBeanDefinition());
        factory.registerBeanDefinition("person",
                BeanDefinitionBuilder.
                        genericBeanDefinition(Person.class).
                        getBeanDefinition());
        Person person = factory.getBean("person", Person.class);
        System.out.println(person);
    }
}
```

##### 运行输出

```bash
class com.javacode2018.lesson002.demo6.Person
调用 MySmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors 方法
class java.lang.String
调用 MySmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors 方法
调用 Person(String name)
Person{name='路人甲Java', age=null}
```

> 从输出中可以看出调用了Person中标注@MyAutowired标注的构造器

到目前为止bean实例化阶段结束了，继续进入后面的阶段

## 阶段7：合并后的BeanDefinition处理

这块的源码如下

```java
protected void applyMergedBeanDefinitionPostProcessors(RootBeanDefinition mbd, Class<?> beanType, String beanName) {
        for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof MergedBeanDefinitionPostProcessor) {
                MergedBeanDefinitionPostProcessor bdp = (MergedBeanDefinitionPostProcessor) bp;
                bdp.postProcessMergedBeanDefinition(mbd, beanType, beanName);
            }
        }
    }
```

会调用`MergedBeanDefinitionPostProcessor接口的postProcessMergedBeanDefinition`方法，看一下这个方法的源码

```java
void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName);
```

spring会轮询`BeanPostProcessor`，依次调用`MergedBeanDefinitionPostProcessor#postProcessMergedBeanDefinition`

第一个参数为beanDefinition，表示合并之后的RootBeanDefinition，我们可以在这个方法内部对合并之后的`BeanDefinition`进行再次处理

**postProcessMergedBeanDefinition有2个实现类，前面我们介绍过，用的也比较多，面试的时候也会经常问的**

```
org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Autowired、@Value 标注的方法、字段进行缓存
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
在 postProcessMergedBeanDefinition 方法中对 @Resource 标注的字段、@Resource 标注的方法、 @PostConstruct 标注的字段、 @PreDestroy标注的方法进行缓存
```

## 阶段8：Bean属性设置阶段

### 属性设置阶段分为3个小的阶段

- 实例化后阶段
- Bean属性赋值前处理
- Bean属性赋值

### 实例化后阶段

会调用`InstantiationAwareBeanPostProcessor`接口的`postProcessAfterInstantiation`这个方法，调用逻辑如下：

看一下具体的调用逻辑如下

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
            return;
        }
    }
}
```

> `postProcessAfterInstantiation`方法返回false的时候，后续的**Bean属性赋值前处理、Bean属性赋值**都会被跳过了

来看一下`postProcessAfterInstantiation`这个方法的定义

```java
default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
    return true;
}
```

**来看个案例，案例中返回false，跳过属性的赋值操作。**

#### 案例

##### 来个类

```java
public class UserModel {
    private String name;
    private Integer age;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
    @Override
    public String toString() {
        return "UserModel{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

##### 测试用例

下面很简单，来注册一个UserModel的bean

```java
import org.junit.Test;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
/**
 * {@link InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation(java.lang.Object, java.lang.String)}
 * 返回false，可以阻止bean属性的赋值
 */
public class InstantiationAwareBeanPostProcessoryTest1 {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        factory.registerBeanDefinition("user1", BeanDefinitionBuilder.
                genericBeanDefinition(UserModel.class).
                addPropertyValue("name", "路人甲Java").
                addPropertyValue("age", 30).
                getBeanDefinition());
        factory.registerBeanDefinition("user2", BeanDefinitionBuilder.
                genericBeanDefinition(UserModel.class).
                addPropertyValue("name", "刘德华").
                addPropertyValue("age", 50).
                getBeanDefinition());
        for (String beanName : factory.getBeanDefinitionNames()) {
            System.out.println(String.format("%s->%s", beanName, factory.getBean(beanName)));
        }
    }
}
```

> 上面定义了2个bean：[user1,user2]，获取之后输出

##### 运行输出

```java
user1->UserModel{name='路人甲Java', age=30}
user2->UserModel{name='刘德华', age=50}
```

此时UserModel中2个属性都是有值的。

下面来阻止user1的赋值，对代码进行改造，加入下面代码

```java
factory.addBeanPostProcessor(new InstantiationAwareBeanPostProcessor() {
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if ("user1".equals(beanName)) {
            return false;
        } else {
            return true;
        }
    }
});
```

再次运行测试输出

```
user1->UserModel{name='null', age=null}
user2->UserModel{name='刘德华', age=50}
```

user1的属性赋值被跳过了

### Bean属性赋值前阶段

这个阶段会调用`InstantiationAwareBeanPostProcessor`接口的`postProcessProperties`方法，调用逻辑

```java
for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
        InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
        PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
        if (pvsToUse == null) {
            if (filteredPds == null) {
                filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
            }
            pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
        }
        pvs = pvsToUse;
    }
}
```

> 从上面可以看出，如果`InstantiationAwareBeanPostProcessor`中的`postProcessProperties`和`postProcessPropertyValues`都返回空的时候，表示这个bean不需要设置属性，直接返回了，直接进入下一个阶段。

来看一下`postProcessProperties`这个方法的定义

```java
@Nullable
default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
    throws BeansException {
    return null;
}
```

> PropertyValues中保存了bean实例对象中所有属性值的设置，所以我们可以在这个这个方法中对PropertyValues值进行修改。

#### 这个方法有2个比较重要的实现类

##### AutowiredAnnotationBeanPostProcessor在这个方法中对@Autowired、@Value标注的字段、方法注入值

##### CommonAnnotationBeanPostProcessor在这个方法中对@Resource标注的字段和方法注入值

**来个案例，我们在案例中对pvs进行修改**

#### 案例

```java
@Test
public void test3() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.addBeanPostProcessor(new InstantiationAwareBeanPostProcessor() { // @0
        @Nullable
        @Override
        public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
            if ("user1".equals(beanName)) {
                if (pvs == null) {
                    pvs = new MutablePropertyValues();
                }
                if (pvs instanceof MutablePropertyValues) {
                    MutablePropertyValues mpvs = (MutablePropertyValues) pvs;
                    //将姓名设置为：路人
                    mpvs.add("name", "路人");
                    //将年龄属性的值修改为18
                    mpvs.add("age", 18);
                }
            }
            return null;
        }
    });
    //注意 user1 这个没有给属性设置值
    factory.registerBeanDefinition("user1", BeanDefinitionBuilder.
            genericBeanDefinition(UserModel.class).
            getBeanDefinition()); //@1
    factory.registerBeanDefinition("user2", BeanDefinitionBuilder.
            genericBeanDefinition(UserModel.class).
            addPropertyValue("name", "刘德华").
            addPropertyValue("age", 50).
            getBeanDefinition());
    for (String beanName : factory.getBeanDefinitionNames()) {
        System.out.println(String.format("%s->%s", beanName, factory.getBean(beanName)));
    }
}
```

> @1：user1这个bean没有设置属性的值
>
> @0：这个实现 org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor#postProcessProperties 方法，在其内部对没有属性赋值（通过properties或者MutablePropertyValues中add方法）user   bean进行属性值信息进行修改

##### 运行输出

```
user1->UserModel{name='路人', age=18}
user2->UserModel{name='刘德华', age=50}
```

上面过程都ok，进入bean赋值操作

### Bean属性赋值阶段

这个过程比较简单了，循环处理`PropertyValues`中的属性值信息，通过反射调用set方法将属性的值设置到bean实例中

PropertyValues中的值是通过bean xml中property元素配置的，或者调用MutablePropertyValues中add方法设置的值

## 阶段9：Bean初始化阶段

### 这个阶段分为5个小的阶段

- Bean Aware接口回调
- Bean初始化前操作
- Bean初始化操作
- Bean初始化后操作
- Bean初始化完成操作

### Bean Aware接口回调

这块的源码

```java
private void invokeAwareMethods(final String beanName, final Object bean) {
        if (bean instanceof Aware) {
            if (bean instanceof BeanNameAware) {
                ((BeanNameAware) bean).setBeanName(beanName);
            }
            if (bean instanceof BeanClassLoaderAware) {
                ClassLoader bcl = getBeanClassLoader();
                if (bcl != null) {
                    ((BeanClassLoaderAware) bean).setBeanClassLoader(bcl);
                }
            }
            if (bean instanceof BeanFactoryAware) {
                ((BeanFactoryAware) bean).setBeanFactory(AbstractAutowireCapableBeanFactory.this);
            }
        }
    }
```

如果我们的bean实例实现了上面的接口，会按照下面的顺序依次进行调用

```
BeanNameAware：将bean的名称注入进去
BeanClassLoaderAware：将BeanClassLoader注入进去
BeanFactoryAware：将BeanFactory注入进去
```

来个案例感受一下

来个类，实现上面3个接口

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.BeanClassLoaderAware;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.BeanFactoryAware;
import org.springframework.beans.factory.BeanNameAware;
public class AwareBean implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware {
    @Override
    public void setBeanName(String name) {
        System.out.println("setBeanName：" + name);
    }
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("setBeanFactory：" + beanFactory);
    }
    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
        System.out.println("setBeanClassLoader：" + classLoader);
    }
}
```

来个测试类，创建上面这个对象的的bean

```java
package com.javacode2018.lesson002.demo8;
import org.junit.Test;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
public class InvokeAwareTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        factory.registerBeanDefinition("awareBean", BeanDefinitionBuilder.genericBeanDefinition(AwareBean.class).getBeanDefinition());
        //调用getBean方法获取bean，将触发bean的初始化
        factory.getBean("awareBean");
    }
}
```

运行输出

```
setBeanName：awareBean
setBeanClassLoader：sun.misc.Launcher$AppClassLoader@18b4aac2
setBeanFactory：org.springframework.beans.factory.support.DefaultListableBeanFactory@5bb21b69: defining beans [awareBean]; root of factory hierarchy
```

### Bean初始化前操作

这个阶段的源码

```java
@Override
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
    throws BeansException {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

会调用`BeanPostProcessor的postProcessBeforeInitialization`方法，若返回null，当前方法将结束

**通常称postProcessBeforeInitialization这个方法为：bean初始化前操作。**

这个接口有2个实现类，比较重要

```java
org.springframework.context.support.ApplicationContextAwareProcessor
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```

#### ApplicationContextAwareProcessor注入6个Aware接口对象

如果bean实现了下面的接口，在`ApplicationContextAwareProcessor#postProcessBeforeInitialization`中会依次调用下面接口中的方法，将`Aware`前缀对应的对象注入到bean实例中

```
EnvironmentAware：注入Environment对象
EmbeddedValueResolverAware：注入EmbeddedValueResolver对象
ResourceLoaderAware：注入ResourceLoader对象
ApplicationEventPublisherAware：注入ApplicationEventPublisher对象
MessageSourceAware：注入MessageSource对象
ApplicationContextAware：注入ApplicationContext对象
```

名称上可以看出这个类以`ApplicationContext`开头的，说明这个类只能在`ApplicationContext`环境中使用。

#### CommonAnnotationBeanPostProcessor调用@PostConstruct标注的方法

`CommonAnnotationBeanPostProcessor#postProcessBeforeInitialization`中会调用bean中所有标注@PostConstruct注解的方法

#### 案例

下面的类有2个方法标注了`@PostConstruct`，并且实现了上面说的那6个Aware接口

```java
import org.springframework.beans.BeansException;
import org.springframework.context.*;
import org.springframework.core.env.Environment;
import org.springframework.core.io.ResourceLoader;
import org.springframework.util.StringValueResolver;
import javax.annotation.PostConstruct;
public class Bean1 implements EnvironmentAware, EmbeddedValueResolverAware, ResourceLoaderAware, ApplicationEventPublisherAware, MessageSourceAware, ApplicationContextAware {
    @PostConstruct
    public void postConstruct1() { //@1
        System.out.println("postConstruct1()");
    }
    @PostConstruct
    public void postConstruct2() { //@2
        System.out.println("postConstruct2()");
    }
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("setApplicationContext:" + applicationContext);
    }
    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        System.out.println("setApplicationEventPublisher:" + applicationEventPublisher);
    }
    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        System.out.println("setEmbeddedValueResolver:" + resolver);
    }
    @Override
    public void setEnvironment(Environment environment) {
        System.out.println("setEnvironment:" + environment.getClass());
    }
    @Override
    public void setMessageSource(MessageSource messageSource) {
        System.out.println("setMessageSource:" + messageSource);
    }
    @Override
    public void setResourceLoader(ResourceLoader resourceLoader) {
        System.out.println("setResourceLoader:" + resourceLoader);
    }
}
```

##### 来个测试案例

```java
package com.javacode2018.lesson002.demo9;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
public class PostProcessBeforeInitializationTest {
    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(Bean1.class);
        context.refresh();
    }
}
```

##### 运行输出

```
setEmbeddedValueResolver:org.springframework.beans.factory.config.EmbeddedValueResolver@15b204a1
setResourceLoader:org.springframework.context.annotation.AnnotationConfigApplicationContext@64bf3bbf, started on Sun Apr 05 21:16:00 CST 2020
setApplicationEventPublisher:org.springframework.context.annotation.AnnotationConfigApplicationContext@64bf3bbf, started on Sun Apr 05 21:16:00 CST 2020
setMessageSource:org.springframework.context.annotation.AnnotationConfigApplicationContext@64bf3bbf, started on Sun Apr 05 21:16:00 CST 2020
setApplicationContext:org.springframework.context.annotation.AnnotationConfigApplicationContext@64bf3bbf, started on Sun Apr 05 21:16:00 CST 2020
postConstruct1()
postConstruct2()
```

大家可以去看一下AnnotationConfigApplicationContext的源码，其内部会添加很多`BeanPostProcessor`到`DefaultListableBeanFactory`中

### Bean初始化阶段

#### 2个步骤

1. 调用InitializingBean接口的afterPropertiesSet方法
2. 调用定义bean的时候指定的初始化方法。

#### 调用InitializingBean接口的afterPropertiesSet方法

来看一下InitializingBean这个接口

```java
public interface InitializingBean {
    void afterPropertiesSet() throws Exception;
}
```

> 当我们的bean实现了这个接口的时候，会在这个阶段被调用

#### 调用bean定义的时候指定的初始化方法

**先来看一下如何指定bean的初始化方法，3种方式**

##### 方式1：xml方式指定初始化方法

```xml
<bean init-method="bean中方法名称"/>
```

##### 方式2：@Bean的方式指定初始化方法

```java
@Bean(initMethod = "初始化的方法")
```

##### 方式3：api的方式指定初始化方法

```java
this.beanDefinition.setInitMethodName(methodName);
```

初始化方法最终会赋值给下面这个字段

```java
org.springframework.beans.factory.support.AbstractBeanDefinition#initMethodName
```

#### 案例

##### 来个类

```java
package com.javacode2018.lesson002.demo10;
import org.springframework.beans.factory.InitializingBean;
public class Service implements InitializingBean{
    public void init() {
        System.out.println("调用init()方法");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用afterPropertiesSet()");
    }
}
```

##### 下面我们定义Service这个bean，指定init方法为初始化方法

```java
package com.javacode2018.lesson002.demo10;
import org.junit.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
/**
 * 初始化方法测试
 */
public class InitMethodTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        BeanDefinition service = BeanDefinitionBuilder.genericBeanDefinition(Service.class).
                setInitMethodName("init"). //@1：指定初始化方法
                getBeanDefinition();
        factory.registerBeanDefinition("service", service);
        System.out.println(factory.getBean("service"));
    }
}
```

##### 运行输出

```
调用afterPropertiesSet()
调用init()方法
com.javacode2018.lesson002.demo10.Service@12f41634
```

调用顺序：InitializingBean中的afterPropertiesSet、然后再调用自定义的初始化方法

### Bean初始化后阶段

这块的源码

```java
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {
    Object result = existingBean;
    for (BeanPostProcessor processor : getBeanPostProcessors()) {
        Object current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
        result = current;
    }
    return result;
}
```

调用`BeanPostProcessor接口的postProcessAfterInitialization方法`，返回null的时候，会中断上面的操作。

**通常称postProcessAfterInitialization这个方法为：bean初始化后置操作。**

来个案例

```java
package com.javacode2018.lesson002.demo11;
import org.junit.Test;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
import org.springframework.lang.Nullable;
/**
 * {@link BeanPostProcessor#postProcessAfterInitialization(java.lang.Object, java.lang.String)}
 * bean初始化后置处理
 */
public class PostProcessAfterInitializationTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        //加入bean初始化后置处理器方法实现
        factory.addBeanPostProcessor(new BeanPostProcessor() {
            @Nullable
            @Override
            public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                System.out.println("postProcessAfterInitialization：" + beanName);
                return bean;
            }
        });
        //下面注册2个String类型的bean
        factory.registerBeanDefinition("name",
                BeanDefinitionBuilder.
                        genericBeanDefinition(String.class).
                        addConstructorArgValue("公众号：【路人甲Java】").
                        getBeanDefinition());
        factory.registerBeanDefinition("personInformation",
                BeanDefinitionBuilder.genericBeanDefinition(String.class).
                        addConstructorArgValue("带领大家成为java高手！").
                        getBeanDefinition());
        System.out.println("-------输出bean信息---------");
        for (String beanName : factory.getBeanDefinitionNames()) {
            System.out.println(String.format("%s->%s", beanName, factory.getBean(beanName)));
        }
    }
}
```

运行输出

```
-------输出bean信息---------
postProcessAfterInitialization：name
name->公众号：【路人甲Java】
postProcessAfterInitialization：personInformation
personInformation->带领大家成为java高手！
```

## 阶段10：所有单例bean初始化完成后阶段

所有单例bean实例化完成之后，spring会回调下面这个接口

```java
public interface SmartInitializingSingleton {
    void afterSingletonsInstantiated();
}
```

调用逻辑在下面这个方法中

```java
/**
 * 确保所有非lazy的单例都被实例化，同时考虑到FactoryBeans。如果需要，通常在工厂设置结束时调用。
 */
org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons
```

> 这个方法内部会先触发所有非延迟加载的单例bean初始化，然后从容器中找到类型是`SmartInitializingSingleton`的bean，调用他们的`afterSingletonsInstantiated`方法

有兴趣的可以去看一下带有ApplicationContext的容器，内部最终都会调用上面这个方法触发所有单例bean的初始化。

来个2个案例演示一下SmartInitializingSingleton的使用。

### 案例1：ApplicationContext自动回调SmartInitializingSingleton接口

**Service3**

```java
package com.dailycodebuffer.beanlifecycle.service;

import org.springframework.stereotype.Component;
@Component
public class Service3 {
    public Service3() {
        System.out.println("create " + this.getClass());
    }
}

```

**Service4**

```java
package com.dailycodebuffer.beanlifecycle.service;

import org.springframework.stereotype.Component;
@Component
public class Service4 {
    public Service4() {
        System.out.println("create " + this.getClass());
    }
}

```

**自定义一个SmartInitializingSingleton**

```java
package com.dailycodebuffer.beanlifecycle.service;

import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.stereotype.Component;
@Component
public class MySmartInitializingSingleton implements SmartInitializingSingleton {
    @Override
    public void afterSingletonsInstantiated() {
        System.out.println("所有bean初始化完毕！");
    }
}
```

来个测试类，通过包扫描的方式注册上面3个bean

```java
package com.dailycodebuffer.beanlifecycle.service;

import org.junit.Test;
import org.springframework.beans.factory.SmartInitializingSingleton;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.ComponentScan;
/**
 * 所有bean初始化完毕，容器会回调{@link SmartInitializingSingleton#afterSingletonsInstantiated()}
 */
@ComponentScan
public class MySmartInitializingSingletonTest {
    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(MySmartInitializingSingletonTest.class);
        System.out.println("开始启动容器!");
        context.refresh();
        System.out.println("容器启动完毕!");
    }
}
```

**运行输出**

```
开始启动容器!
create class com.javacode2018.lesson002.demo12.Service1
create class com.javacode2018.lesson002.demo12.Service2
所有bean初始化完毕！
容器启动完毕!
```

### 案例2：通过api的方式让DefaultListableBeanFactory去回调SmartInitializingSingleton

```java
@Test
public void test2() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    factory.registerBeanDefinition("service1", BeanDefinitionBuilder.genericBeanDefinition(Service1.class).getBeanDefinition());
    factory.registerBeanDefinition("service2", BeanDefinitionBuilder.genericBeanDefinition(Service2.class).getBeanDefinition());
    factory.registerBeanDefinition("mySmartInitializingSingleton", BeanDefinitionBuilder.genericBeanDefinition(MySmartInitializingSingleton.class).getBeanDefinition());
    System.out.println("准备触发所有单例bean初始化");
    //触发所有bean初始化，并且回调 SmartInitializingSingleton#afterSingletonsInstantiated 方法
    factory.preInstantiateSingletons();
}
```

> 上面通过api的方式注册bean
>
> 最后调用`factory.preInstantiateSingletons`触发所有非lazy单例bean初始化，所有bean装配完毕之后，会回调SmartInitializingSingleton接口

## 阶段11：Bean使用阶段

这个阶段就不说了，调用getBean方法得到了bean之后，大家可以随意使用，任意发挥。

## 阶段12：Bean销毁阶段

### 触发bean销毁的几种方式

1. 调用org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#destroyBean
2. 调用org.springframework.beans.factory.config.ConfigurableBeanFactory#destroySingletons
3. 调用ApplicationContext中的close方法

### Bean销毁阶段会依次执行

1. 轮询beanPostProcessors列表，如果是DestructionAwareBeanPostProcessor这种类型的，会调用其内部的postProcessBeforeDestruction方法
2. 如果bean实现了org.springframework.beans.factory.DisposableBean接口，会调用这个接口中的destroy方法
3. 调用bean自定义的销毁方法

### DestructionAwareBeanPostProcessor接口

看一下源码

```java
public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
    /**
     * bean销毁前调用的方法
     */
    void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
    /**
     * 用来判断bean是否需要触发postProcessBeforeDestruction方法
     */
    default boolean requiresDestruction(Object bean) {
        return true;
    }
}
```

这个接口有个关键的实现类

```java
org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
```

**CommonAnnotationBeanPostProcessor#postProcessBeforeDestruction方法中会调用bean中所有标注了@PreDestroy的方法。**

### 再来说一下自定义销毁方法有3种方式

#### 方式1：xml中指定销毁方法

```java
<bean destroy-method="bean中方法名称"/>
```

#### 方式2：@Bean中指定销毁方法

```java
@Bean(destroyMethod = "初始化的方法")
```

#### 方式3：api的方式指定销毁方法

```java
this.beanDefinition.setDestroyMethodName(methodName);
```

初始化方法最终会赋值给下面这个字段

```java
org.springframework.beans.factory.support.AbstractBeanDefinition#destroyMethodName
```

下面来看销毁的案例

### 案例1：自定义DestructionAwareBeanPostProcessor

#### 来个类

```java
package com.javacode2018.lesson002.demo13;
public class ServiceA {
    public ServiceA() {
        System.out.println("create " + this.getClass());
    }
}
```

#### 自定义一个DestructionAwareBeanPostProcessor

```java
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor;
public class MyDestructionAwareBeanPostProcessor implements DestructionAwareBeanPostProcessor {
    @Override
    public void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException {
        System.out.println("准备销毁bean：" + beanName);
    }
}
```

#### 来个测试类

```java
import org.junit.Test;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.DefaultListableBeanFactory;
/**
 * 自定义 {@link org.springframework.beans.factory.config.DestructionAwareBeanPostProcessor}
 */
public class DestructionAwareBeanPostProcessorTest {
    @Test
    public void test1() {
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        //添加自定义的DestructionAwareBeanPostProcessor
        factory.addBeanPostProcessor(new MyDestructionAwareBeanPostProcessor());
        //向容器中注入3个单例bean
        factory.registerBeanDefinition("serviceA1", BeanDefinitionBuilder.genericBeanDefinition(ServiceA.class).getBeanDefinition());
        factory.registerBeanDefinition("serviceA2", BeanDefinitionBuilder.genericBeanDefinition(ServiceA.class).getBeanDefinition());
        factory.registerBeanDefinition("serviceA3", BeanDefinitionBuilder.genericBeanDefinition(ServiceA.class).getBeanDefinition());
        //触发所有单例bean初始化
        factory.preInstantiateSingletons(); //@1
        System.out.println("销毁serviceA1"); 
        //销毁指定的bean
        factory.destroySingleton("serviceA1");//@2
        System.out.println("触发所有单例bean的销毁");
        factory.destroySingletons();
    }
}
```

> 上面使用了2种方式来触发bean的销毁[@1和@2]

#### 运行输出

```
create class com.javacode2018.lesson002.demo13.ServiceA
create class com.javacode2018.lesson002.demo13.ServiceA
create class com.javacode2018.lesson002.demo13.ServiceA
销毁serviceA1
准备要销毁bean：serviceA1
触发所有单例bean的销毁
准备要销毁bean：serviceA3
准备要销毁bean：serviceA2
```

> 可以看到postProcessBeforeDestruction被调用了3次，依次销毁3个自定义的bean

### 案例2：触发@PreDestroy标注的方法被调用

上面说了这个注解是在`CommonAnnotationBeanPostProcessor#postProcessBeforeDestruction`中被处理的，所以只需要将这个加入BeanPostProcessor列表就可以了。

#### 再来个类

```java
import javax.annotation.PreDestroy;

public class ServiceB {
    public ServiceB() {
        System.out.println("create " + this.getClass());
    }
    @PreDestroy
    public void preDestroy() { //@1
        System.out.println("preDestroy()");
    }
}
```

> @1：标注了@PreDestroy注解

#### 测试用例

```java
@Test
public void testPreDestroy() {
    DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
    //添加自定义的DestructionAwareBeanPostProcessor
    factory.addBeanPostProcessor(new MyDestructionAwareBeanPostProcessor()); //@1
    //将CommonAnnotationBeanPostProcessor加入
    factory.addBeanPostProcessor(new CommonAnnotationBeanPostProcessor()); //@2
    //向容器中注入bean
    factory.registerBeanDefinition("serviceB", BeanDefinitionBuilder.genericBeanDefinition(ServiceB.class).getBeanDefinition());
    //触发所有单例bean初始化
    factory.preInstantiateSingletons();
    System.out.println("销毁serviceB");
    //销毁指定的bean
    factory.destroySingleton("serviceB");
}
```

> @1：放入了一个自定义的DestructionAwareBeanPostProcessor
>
> @2：放入了CommonAnnotationBeanPostProcessor，这个会处理bean中标注@PreDestroy注解的方法

#### 看效果运行输出

```
create class com.javacode2018.lesson002.demo13.ServiceB
销毁serviceB
准备销毁bean：serviceB
preDestroy()
```

### 案例3：看一下销毁阶段的执行顺序

实际上ApplicationContext内部已经将spring内部一些常见的必须的`BeannPostProcessor`自动装配到`beanPostProcessors列表中`，比如我们熟悉的下面的几个

```
1.org.springframework.context.annotation.CommonAnnotationBeanPostProcessor
  用来处理@Resource、@PostConstruct、@PreDestroy的
2.org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor
  用来处理@Autowired、@Value注解
3.org.springframework.context.support.ApplicationContextAwareProcessor
  用来回调Bean实现的各种Aware接口
```

所以通过ApplicationContext来销毁bean，会触发3中方式的执行。

下面我们就以AnnotationConfigApplicationContext来演示一下销毁操作。

来一个类

```java
package com.javacode2018.lesson002.demo14;

import org.springframework.beans.factory.DisposableBean;
import javax.annotation.PreDestroy;
public class ServiceA implements DisposableBean {
    public ServiceA() {
        System.out.println("创建ServiceA实例");
    }
    @PreDestroy
    public void preDestroy1() {
        System.out.println("preDestroy1()");
    }
    @PreDestroy
    public void preDestroy2() {
        System.out.println("preDestroy2()");
    }
    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean接口中的destroy()");
    }
    //自定义的销毁方法
    public void customDestroyMethod() { //@1
        System.out.println("我是自定义的销毁方法:customDestroyMethod()");
    }
}
```

> 上面的类中有2个方法标注了@PreDestroy
>
> 这个类实现了DisposableBean接口，重写了接口的中的destroy方法
>
> @1：这个destroyMethod我们一会通过@Bean注解的方式，将其指定为自定义方法。

来看测试用例

```java
package com.javacode2018.lesson002.demo14;
import org.junit.Test;
import org.springframework.beans.factory.annotation.Configurable;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
@Configurable
public class DestroyTest {
    @Bean(destroyMethod = "customDestroyMethod") //@1
    public ServiceA serviceA() {
        return new ServiceA();
    }
    @Test
    public void test1() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(DestroyTest.class);
        //启动容器
        System.out.println("准备启动容器");
        context.refresh();
        System.out.println("容器启动完毕");
        System.out.println("serviceA：" + context.getBean(ServiceA.class));
        //关闭容器
        System.out.println("准备关闭容器");
        //调用容器的close方法，会触发bean的销毁操作
        context.close(); //@2
        System.out.println("容器关闭完毕");
    }
}
```

> 上面这个类标注了@Configuration，表示是一个配置类，内部有个@Bean标注的方法，表示使用这个方法来定义一个bean。
>
> @1：通过destroyMethod属性将customDestroyMethod指定为自定义销毁方法
>
> @2：关闭容器，触发bean销毁操作

来运行test1，输出

```
准备启动容器
创建ServiceA实例
容器启动完毕
serviceA：com.javacode2018.lesson002.demo14.ServiceA@243c4f91
准备关闭容器
preDestroy1()
preDestroy2()
DisposableBean接口中的destroy()
我是自定义的销毁方法:customDestroyMethod()
容器关闭完毕
```

> 可以看出销毁方法调用的顺序：
>
> 1. @PreDestroy标注的所有方法
> 2. DisposableBean接口中的destroy()
> 3. 自定义的销毁方法

下面来说一个非常非常重要的类，打起精神，一定要注意看。

## AbstractApplicationContext类（非常重要的类）

来看一下UML图

![0b109318-092c-49a6-8047-eb4db9983865](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408161512128.png)

### BeanFactory接口

这个我们已经很熟悉了，Bean工厂的顶层接口

### DefaultListableBeanFactory类

实现了BeanFactory接口，可以说这个可以是BeanFactory接口真正的唯一实现，内部真正实现了bean生命周期中的所有代码。

其他的一些类都是依赖于DefaultListableBeanFactory类，将请求转发给DefaultListableBeanFactory进行bean的处理的。

### 其他3个类

我们经常用到的就是这3个类：AnnotationConfigApplicationContext/ClassPathXmlApplicationContext/FileSystemXmlApplicationContext这3个类，他们的主要内部的功能是依赖他的父类AbstractApplicationContext来实现的，所以大家主要看`AbstractApplicationContext`这个类。

### AbstractApplicationContext类

这个类中有2个比较重要的方法

```java
public abstract ConfigurableListableBeanFactory getBeanFactory() throws IllegalStateException;
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory)
```

大家是否注意过我们使用`AnnotationConfigApplicationContext`的时候，经常调用`reflush方法`，这个方法内部就会调用上面这2个方法。

#### 第一个方法：getBeanFactory()

返回当前应用上下文中的`ConfigurableListableBeanFactory`，这也是个接口类型的，这个接口有一个唯一的实现类：`DefaultListableBeanFactory`。

有没有很熟悉，上面说过：DefaultListableBeanFactory是BeanFactory真正的唯一实现。

应用上线文中就会使用这个`ConfigurableListableBeanFactory`来操作spring容器。

#### 第二个方法：registerBeanPostProcessors

**说的通俗点：这个方法就是向ConfigurableListableBeanFactory中注册BeanPostProcessor，内容会从spring容器中获取所有类型的BeanPostProcessor，将其添加到DefaultListableBeanFactory#beanPostProcessors列表中**

看一下这个方法的源码

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}
```

会将请求转发给`PostProcessorRegistrationDelegate#registerBeanPostProcessors`

内部比较长，大家可以去看一下源码，这个方法内部主要用到了4个`BeanPostProcessor`类型的List集合

```java
List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
List<BeanPostProcessor> orderedPostProcessors
List<BeanPostProcessor> nonOrderedPostProcessors;
List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
```

**先说一下：当到方法的时候，spring容器中已经完成了所有Bean的注册**

spring会从容器中找出所有类型的BeanPostProcessor列表，然后按照下面的规则将其分别放到上面的4个集合中，上面4个集合中的`BeanPostProcessor`会被依次添加到DefaultListableBeanFactory#beanPostProcessors列表中，来看一下4个集合的分别放的是那些BeanPostProcessor

##### priorityOrderedPostProcessors（指定优先级的BeanPostProcessor）

实现org.springframework.core.PriorityOrdered接口的BeanPostProcessor，但是不包含MergedBeanDefinitionPostProcessor类型的

##### orderedPostProcessors（指定了顺序的BeanPostProcessor）

实现了org.springframework.core.annotation.Order接口的BeanPostProcessor，但是不包含MergedBeanDefinitionPostProcessor类型的

##### nonOrderedPostProcessors（未指定顺序的BeanPostProcessor）

上面2种类型之外以及MergedBeanDefinitionPostProcessor之外的

##### internalPostProcessors

MergedBeanDefinitionPostProcessor类型的BeanPostProcessor列表

> 大家可以去看一下`CommonAnnotationBeanPostProcessor`和`AutowiredAnnotationBeanPostProcessor`，这两个类都实现了`PriorityOrdered`接口，但是他们也实现了`MergedBeanDefinitionPostProcessor`接口，所以最终他们会被丢到`internalPostProcessors`这个集合中，会被放入BeanPostProcessor的最后面

# Spring中BeanFactory与ApplicationContext的本质区别和作用

BeanFactory 是Bean工厂，是Spring 框架最核心的接口，它提供了高级IoC 的配置机制。如果说BeanFactory是Spring的心脏，那么应用上下文ApplicationContext就是完整的躯体了，ApplicationContext继承了BeanFactory接口，拥有BeanFactory的全部功能，扩展了更多面向实际应用的、企业级的高级特性

它们都可以视作Spring的容器，Spring容器负责对象（Spring中我们都称之为bean）整个生命周期的管理——实例化、装配和销毁 Bean。在基于Spring的Java EE应用中，所有的组件都被当成Bean处理，包括数据源，Hibernate的SessionFactory、事务管理器等。我们一般称BeanFactory为IoC容器，而称ApplicationContext为应用上下文

**本质区别**：BeanFactory是懒加载，ApplicationContext则在初始化应用上下文时就实例化所有单实例的Bean，可以指定为延迟加载

BeanFactory在初始化容器时，并未实例化Bean，如果Bean的某一个属性没有注入，BeanFacotry加载后，直至首次调用getBean方法才会抛出异常；而ApplicationContext则在初始化应用上下文时初始化所有单实例的Bean，这样有利于检查所依赖属性是否已经注入。综上所述，通常情况下我们首选ApplicationContext

由于ApplicationContext会预先初始化所有的Singleton Bean，于是在系统创建前期会有较大的系统开销，但一旦ApplicationContext初始化完成，程序后面获取Singleton Bean实例时候将有较好的性能

# BeanPostProcessor的方法何时调用

Bean的创建在doCreateBean中是怎么完成的

- 通过createBeanInstance对Bean进行实例化
- 通过populateBean方法完成依赖注入

而BeanPostProcessor也就是Bean的后置处理，是在完成依赖注入后，通过initializeBean方法调用进行的

后置：指的是在Bean实例化后，BeanPostProcessor都是在createBeanInstance对Bean进行实例化后执行的

```java
try {
    this.populateBean(beanName, mbd, instanceWrapper);
    exposedObject = this.initializeBean(beanName, exposedObject, mbd);
} catch (Throwable var18) {
    if (var18 instanceof BeanCreationException && beanName.equals(((BeanCreationException)var18).getBeanName())) {
        throw (BeanCreationException)var18;
    }

    throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", var18);
}
```

initializeBean方法中

- 调用了applyBeanPostProcessorsBeforeInitialization处理Bean初始化前的处理
- 调用了invokeInitMethods执行初始化方法
- 调用了applyBeanPostProcessorsAfterInitialization处理Bean初始化后的处理

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    if (System.getSecurityManager() != null) {
        AccessController.doPrivileged(() -> {
            this.invokeAwareMethods(beanName, bean);
            return null;
        }, this.getAccessControlContext());
    } else {
        this.invokeAwareMethods(beanName, bean);
    }

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    }

    try {
        this.invokeInitMethods(beanName, wrappedBean, mbd);
    } catch (Throwable var6) {
        throw new BeanCreationException(mbd != null ? mbd.getResourceDescription() : null, beanName, "Invocation of init method failed", var6);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

applyBeanPostProcessorsBeforeInitialization方法中，调用了BeanPostProcessor的postProcessBeforeInitialization方法，该方法在Bean的初始化前执行

```java
public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName) throws BeansException {
    Object result = existingBean;

    Object current;
    for(Iterator var4 = this.getBeanPostProcessors().iterator(); var4.hasNext(); result = current) {
        BeanPostProcessor processor = (BeanPostProcessor)var4.next();
        current = processor.postProcessBeforeInitialization(result, beanName);
        if (current == null) {
            return result;
        }
    }

    return result;
}
```

applyBeanPostProcessorsAfterInitialization方法中，调用了BeanPostProcessor的postProcessAfterInitialization方法，该方法在Bean的初始化后执行

```java
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName) throws BeansException {
    Object result = existingBean;

    Object current;
    for(Iterator var4 = this.getBeanPostProcessors().iterator(); var4.hasNext(); result = current) {
        BeanPostProcessor processor = (BeanPostProcessor)var4.next();
        current = processor.postProcessAfterInitialization(result, beanName);
        if (current == null) {
            return result;
        }
    }

    return result;
}
```

# 循环依赖

## 什么是循环依赖

```java
public class A {
    @Autowired
    private B b;
}

public class B {
    @Autowired
    private A a;
}
```

**如上代码所示，即 A 里面注入 B，B 里面又注入 A。此时，就发生了「循环依赖」**

## 三级缓存

> Spring 中，单例 Bean 在创建后会被放入 IoC 容器的缓存池中，并触发 Spring 对该 Bean 的生命周期管理

单例模式下，在第一次使用 Bean 时，会创建一个 Bean 对象，并放入 IoC 容器的缓存池中。后续再使用该 Bean 对象时，会直接从缓存池中获取

保存单例模式 Bean 的缓存池，采用了三级缓存设计，如下代码所示

```java
/** Cache of singleton objects: bean name --> bean instance */
/** 一级缓存：用于存放完全初始化好的 bean **/
private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object>(256);
 
/** Cache of early singleton objects: bean name --> bean instance */
/** 二级缓存：存放原始的 bean 对象（尚未填充属性），用于解决循环依赖 */
private final Map<String, Object> earlySingletonObjects = new HashMap<String, Object>(16);

/** Cache of singleton factories: bean name --> ObjectFactory */
/** 三级级缓存：存放 bean 工厂对象，用于解决循环依赖 */
private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<String, ObjectFactory<?>>(16);
```

| 缓存层级   | 名称                    | 描述                                                         |
| ---------- | ----------------------- | ------------------------------------------------------------ |
| 第一层缓存 | `singletonObjects`      | 单例对象缓存池，存放的 Bean 已经实例化、属性赋值、完全初始化好（成品） |
| 第二层缓存 | `earlySingletonObjects` | 早期单例对象缓存池，存放的 Bean 已经实例化但尚未属性赋值、未执行 `init` 方法（半成品） |
| 第三层缓存 | `singletonFactories`    | 单例工厂的缓存                                               |

## 使用三级缓存解决循环依赖

### getSingleton方法中三级缓存的使用

```java
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // Spring首先从singletonObjects（一级缓存）中尝试获取
  Object singletonObject = this.singletonObjects.get(beanName);
  // 若是获取不到而且对象在建立中，则尝试从earlySingletonObjects(二级缓存)中获取
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    synchronized (this.singletonObjects) {
        singletonObject = this.earlySingletonObjects.get(beanName);
        if (singletonObject == null && allowEarlyReference) {
          ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
          if (singletonFactory != null) {
            //若是仍是获取不到而且允许从singletonFactories经过getObject获取，则经过singletonFactory.getObject()(三级缓存)获取
              singletonObject = singletonFactory.getObject();
              //若是获取到了则将singletonObject放入到earlySingletonObjects,也就是将三级缓存提高到二级缓存中
              this.earlySingletonObjects.put(beanName, singletonObject);
              this.singletonFactories.remove(beanName);
          }
        }
    }
  }
  return (singletonObject != NULL_OBJECT ? singletonObject : null);
}
```

> `getSingleton()` 方法中
>
> - `isSingletonCurrentlyInCreation()` 方法用于判断当前单例 Bean 是否正在创建中，即「还没有执行初始化方法」。比如，A 的构造器依赖了 B 对象因此要先去创建 B 对象，或者在 A 的属性装配过程中依赖了 B 对象因此要先创建 B 对象，这时 A 就是处于创建中的状态
> - `allowEarlyReference` 变量表示是否允许从三级缓存 `singletonFactories` 中经过 `singletonFactory` 的 `getObject()` 方法获取 Bean 对象

分析 `getSingleton()` 的整个过程，可知三级缓存的使用过程如下

- Spring 会先从一级缓存 `singletonObjects` 中尝试获取 Bean。
- 若是获取不到，而且对象正在建立中，就会尝试从二级缓存 `earlySingletonObjects` 中获取 Bean。
- 若还是获取不到，且允许从三级缓存 `singletonFactories` 中经过 `singletonFactory` 的 `getObject()` 方法获取 Bean 对象，就会尝试从三级缓存 `singletonFactories` 中获取 Bean。
- 若是在三级缓存中获取到了 Bean，会将该 Bean 存放到二级缓存中

### 第三级缓存为什么可以解决循环依赖

**Spring 解决循环依赖的诀窍就在于 `singletonFactories` 这个三级缓存。** 三级缓存中使用到了`ObjectFactory` 接口，定义如下

```java
public interface ObjectFactory<T> {
    T getObject() throws BeansException;
}
```

在 Bean 建立过程当中，有两处比较重要的匿名内部类实现了该接口。一处是 Spring 利用其建立 Bean 的时候，另外一处就是在 `addSingletonFactory` 方法中，如下代码所示

```java
addSingletonFactory(beanName, new ObjectFactory<Object>() {
   @Override   
   public Object getObject() throws BeansException {
       return getEarlyBeanReference(beanName, mbd, bean);
   }
});
```

**此处就是解决循环依赖的关键，这段代码发生在 `createBeanInstance` 以后**

1. 此时，单例 Bean 对象已经实例化（可以通过对象引用定位到堆中的对象），但尚未属性赋值和初始化。
2. **Spring 会将该状态下的 Bean 存放到三级缓存中，提早曝光给 IoC 容器（“提早”指的是不必等对象完成属性赋值和初始化后再交给 IoC 容器）。也就是说，可以在三级缓存 `singletonFactories` 中找到该状态下的 Bean 对象。**

### 解决循环依赖示例分析

```java
public class A {
    @Autowired
    private B b;
}

public class B {
    @Autowired
    private A a;
}
```

在上文章节铺垫的基础上，此处结合一个循环依赖的案例，分析下如何使用三级缓存解决单例 Bean 的循环依赖。

1. **创建对象 A，完成生命周期的第一步，即实例化（Instantiation），在调用 `createBeanInstance` 方法后，会调用 `addSingletonFactory` 方法，将已实例化但未属性赋值未初始化的对象 A 放入三级缓存 `singletonFactories` 中。即将对象 A 提早曝光给 IoC 容器。**
2. 继续，执行对象 A 生命周期的第二步，即属性赋值（Populate）。此时，发现对象 A 依赖对象，所以就会尝试去获取对象 B。
3. 继续，发现 B 尚未创建，所以会执行创建对象 B 的过程。
4. 在创建对象 B 的过程中，执行实例化（Instantiation）和属性赋值（Populate）操作。此时发现，对象 B 依赖对象 A。
5. 继续，尝试在缓存中查找对象 A。先查找一级缓存，发现一级缓存中没有对象 A（因为对象 A 还未初始化完成）；转而查找二级缓存，二级缓存中也没有对象 A（因为对象 A 还未属性赋值）；转而查找三级缓存 `singletonFactories`，对象 B 可以通过 `ObjectFactory.getObject` 拿到对象 A。
6. 继续，对象 B 在获取到对象 A 后，继续执行属性赋值（Populate）和初始化（Initialization）操作。对象 B 完成初始化操作后，会被存放到一级缓存中。
7. 继续，转到「对象 A 执行属性赋值过程并发现依赖了对象 B」的场景。此时，对象 A 可以从一级缓存中获取到对象 B，所以可以顺利执行属性赋值操作。
8. 继续，对象 A 执行初始化（Initialization）操作，完成后，会被存放到一级缓存中。

## Spring为何不能解决非单例Bean的循环依赖

Spring 为何不能解决非单例 Bean 的循环依赖？ 这个问题可以细分为下面几个问题

1. Spring 为什么不能解决构造器的循环依赖？
2. Spring 为什么不能解决 `prototype` 作用域循环依赖？
3. Spring 为什么不能解决多例的循环依赖？

### Spring 为什么不能解决构造器的循环依赖

> **对象的构造函数是在实例化阶段调用的。**

上文中提到，在对象已实例化后，会将对象存入三级缓存中。在调用对象的构造函数时，对象还未完成初始化，所以也就无法将对象存放到三级缓存中。

在构造函数注入中，对象 A 需要在对象 B 的构造函数中完成初始化，对象 B 也需要在对象 A的构造函数中完成初始化。此时两个对象都不在三级缓存中，最终结果就是两个 Bean 都无法完成初始化，无法解决循环依赖问题。

### Spring 为什么不能解决prototype作用域循环依赖

Spring IoC 容器只会管理单例 Bean 的生命周期，并将单例 Bean 存放到缓存池中（三级缓存）。Spring 并不会管理 `prototype` 作用域的 Bean，也不会缓存该作用域的 Bean，而 Spring 中循环依赖的解决正是通过缓存来实现的。

### Spring 为什么不能解决多例的循环依赖

多实例 Bean 是每次调用 `getBean` 都会创建一个新的 Bean 对象，该 Bean 对象并不能缓存。而 Spring 中循环依赖的解决正是通过缓存来实现的。

### 非单例Bean的循环依赖如何解决

- 对于构造器注入产生的循环依赖，可以使用 `@Lazy` 注解，延迟加载。
- 对于多例 Bean 和 `prototype` 作用域产生的循环依赖，可以尝试改为单例 Bean。

## 为什么一定要三级缓存

### 如果只有一级缓存

- 实例化A对象
- 填充A的属性阶段时需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象；
- 实例化B对象
- 执行到B对象的填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，导致又需要去创建A对象。

这样，就会循环往复，一直创建下去，只到堆栈溢出

**为什么不能在实例化A之后就放入Map？**

因为此时A尚未创建完整，所有属性都是默认值，并不是一个完整的对象，在执行业务时可能会抛出未知的异常。所以必须要在A创建完成之后才能放入Map

### 如果只有二级缓存

此时我们引入二级缓存用另外一个`Map2 {k:name; v:earlybean}` 来存储尚未已经开始创建但是尚未完整创建的对象

- 实例化A对象之后，将A对象放入Map2中。
- 在填充A的属性阶段需要去填充B对象，而此时B对象还没有创建，所以这里为了完成A的填充就必须要先去创建B对象
- 创建B对象的过程中，实例化B对象之后，将B对象放入Map2中。
- 执行到B对象填充属性阶段，又会需要去获取A对象，而此时Map中没有A，因为A还没有创建完成，但是我们继续从Map2中拿到尚未创建完毕的A的引用赋值给a字段。这样B对象其实就已经创建完整了，尽管B.a对象是一个还未创建完成的对象。
- 此时将B放入Map并且从Map2中删除。
- 这时候B创建完成，A继续执行b的属性填充可以拿到B对象，这样A也完成了创建。
- 此时将A对象放入Map并从Map2中删除

```java
protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
    Assert.notNull(singletonFactory, "Singleton factory must not be null");

    synchronized (this.singletonObjects) {
        // 判断一级缓存中不存在此对象
        if (!this.singletonObjects.containsKey(beanName)) { 
            // 直接从工厂中获取 Bean
            Object o = singletonFactory.getObject();

            // 添加至二级缓存中
            this.earlySingletonObjects.put(beanName, o);
            this.registeredSingletons.add(beanName);
        }
    }
}
```

这样的话，每次实例化完 Bean 之后就直接去创建代理对象，并添加到二级缓存中

测试结果是完全正常的，Spring 的初始化时间应该也是不会有太大的影响，因为如果 Bean 本身不需要代理的话，是直接返回原始 Bean 的，并不需要走复杂的创建代理 Bean 的流程

![图片](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408172011683.webp)

从上图中我们可以看到，虽然在创建B时会提前给B注入了一个还未初始化的A对象，但是在创建A的流程中一直使用的是注入到B中的A对象的引用，之后会根据这个引用对A进行初始化，所以这是没有问题的

### 三级缓存的意义

测试证明，二级缓存也是可以解决循环依赖的

**如果 Spring 选择二级缓存来解决循环依赖的话，那么就意味着所有 Bean 都需要在实例化完成之后就立马为其创建代理，而 Spring 的设计原则是在 Bean 初始化完成之后才为其创建代理**

第三级缓存的目的是为了延迟代理对象的创建，因为如果没有依赖循环的话，那么就不需要为其提前创建代理

> 使用三级而非二级缓存并非出于 IOC 的考虑，而是出于 AOP 的考虑，即若使用二级缓存，在 AOP 情形注入到其他Bean的，不是最终的代理对象，而是原始对象

# bean的存储

bean对象最终存储在spring容器中，我们简单的、狭义上的spring容器，在spring源码底层就是一个map集合，这个map集合存储的key是当前bean的name，如果不指定，默认的是class类型首字母小写作为key，value是bean对象。存储bean的map在DefaultListableBeanFactory类中

```java
/** Map of bean definition objects, keyed by bean name. */
private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);
```

当Spring容器扫描到Bean类时 , 会把这个类的描述信息, 以包名加类名的方式存到beanDefinitionMap 中，Map<String,BeanDefinition> , 其中 **String是Key , 默认是类名首字母小写** , BeanDefinition , 存的是类的定义(描述信息) , 我们通常叫BeanDefinition接口为 : bean的定义对象

**问题思考---spring为什么不在扫描到class文件之后，立即执行生命周期方法进行初始化、实例化？而是要先放到beanDefinitionMap集合中？**

因为spring所提供的容器管理功能中，某些class类并不一定是立马需要初始化的，比如：原型bean，就是在使用的时候，再去初始化。

如果是原型bean，并且假设我们没有beanDefinition这一层，每初始化一次，就需要去扫描一次，就很浪费时间。spring容器在启动的过程中，对bean进行初始化实例化的时候，大致是分为了两步，第一是将class文件转换为beanDefinition对象，第二步是根据beanDefinition对象，按照配置的要求，去进行初始化、实例化。有了beanDefinition之后，beanDefinitionMap相当于一个中转站，所有要初始化的bean，都是以beanDefinitionMap中的beanDefinition的属性信息为准

- spring在将class文件转换为beanDefinition的时候，是没有额外的要求的，不管这个bean是单例的，还是原型的，还是懒加载的，都会扫描出来，统一放到beanDefinitionMap集合中
- 在第二步去初始化bean的时候，才会判断当前bean是否是原型的、是否是懒加载的、是否是有依赖关系的，等

```java
A.class --> beanDefinition --> bean
```

> 中间加了一层beanDefinition之后，那很有可能就会出现，我们提供的是X.class，但是在将class存入到beanDefinition之后，将对应的beanDefinition属性的beanClass设置为了Y.class，那这时候初始化的就是Y对象，而不是X对象。这只是举个例子，想要说明的是：我们提供了一个X.class，正常情况下，初始化的就是x这个bean，但是在特殊情况下，我们可以将X的beanDefinition的beanClass修改为Y.class,这样就偷天换日了，实际初始化的是Y对象；这个过程就依赖于Spring所提供的N多个扩展点

**如果我们想在Spring对class初始化的过程中，进行干预怎么办？**

Spring提供了一系列的扩展点，让我们根据自己的业务需求，去处理class文件从class变成bean对象过程中的关键点。这也是Spring强大的地方，不仅仅帮我们去管理bean，也提供了相应的扩展点，让我们根据自己的需求去扩展Spring，比如：beanFactoryPostProcessor可以让我们去干预beanDefinitionMap中的beanDefinition对象；spring的后置处理器，可以让我们根据自己的需求，去干预各个后置处理器方法执行的逻辑









