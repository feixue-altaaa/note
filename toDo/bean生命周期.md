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

从输出的结果中可以看到，合并之前，BeanDefinition是不完整的，比lesson2和lesson3中的class是null，属性信息也不完整，但是合并之后这些信息都完整了。

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

会调用`BeanPostProcessor的postProcessBeforeInitialization`方法，若返回null，当前方法将结束。

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

调用顺序：InitializingBean中的afterPropertiesSet、然后在调用自定义的初始化方法

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

