




### Spring Boot 2.7开始spring.factories不推荐使用

当我们要引入某个功能的时候，只需要在maven或gradle的配置中直接引入对应的Starter，马上就可以使用了，而不需要像传统Spring应用那样写个xml或java配置类来初始化各种Bean

如果你有探索过这些Starter的原理，那你一定知道Spring Boot并没有消灭这些原本你要配置的Bean，而是将这些Bean做成了一些默认的配置类，同时利用`/META-INF/spring.factories`这个文件来指定要加载的默认配置

这样当Spring Boot应用启动的时候，就会根据引入的各种Starter中的`/META-INF/spring.factories`文件所指定的配置类去加载Bean

而这次刚发布的Spring Boot 2.7中，有一个不推荐使用的内容就是关于这个`/META-INF/spring.factories`文件的，所以对于有自定义Starter的开发者来说，有时间要抓紧把这一变化改起来了，因为在Spring Boot 3开始将移除对`/META-INF/spring.factories`的支持

那么具体怎么改呢？下面以之前我们编写的一个swagger的starter为例，它的`/META-INF/spring.factories`内容是这样的

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.spring4all.swagger.SwaggerAutoConfiguration
```

我们只需要创建一个新的文件：`/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`，内容的话只需要直接放配置类就可以了，比如这样

```properties
com.spring4all.swagger.SwaggerAutoConfiguration
```

注意：这里多了一级spring目录。

如果你觉得维护这个太麻烦的话，还可以使用mica-auto来让他自动生成

## **Hook钩子函数**

### why

在Spring中,如果JVM异常终止,Spring是如何保证会释放掉占用的资源,比如说数据库连接等资源呢?

> ### **何为优雅关机**
>
> 就是为确保应用关闭时，通知应用进程释放所占用的资源
>
> - 线程池，shutdown（不接受新任务等待处理完）还是shutdownNow（调用`Thread.interrupt`进行中断）
> - socket 链接，比如：netty、mq
> - 告知注册中心快速下线（靠心跳机制客服早都跳起来了），比如：eureka
> - 清理临时文件，比如：poi
> - 各种堆内堆外内存释放
>
> 总之，进程强行终止会带来数据丢失或者终端无法恢复到正常状态，在分布式环境下还可能导致数据不一致的情况

Spring 容器中 Bean 在什么时候执行销毁方法?

我们知道在Spring中定义销毁方法有两种方式

- 实现 DisposableBean 的 destroy 方法
- 使用 @PreDestroy 注解修饰方法

```java
@Component
public class DataCollectBean implements DisposableBean {

    /**
     * 第一种方法实现 DisposableBean#destroy方法
     *
     * @throws Exception 异常
     */
    @Override
    public void destroy() throws Exception {
        System.err.println("执行销毁方法");
    }

    /**
     * 第二种方法使用PreDestroy注解声明销毁方法
     */
    @PreDestroy
    public void customerDestroy() {
        System.err.println("执行自定义销毁方法");
    }


}
```

那么在什么时候执行销毁方法?

![高级Java开发者都知道的Hook钩子函数，你还不知道吗？_Java_02](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408132125205.webp)

**主动执行销毁bean**

```java
public static void main(String[] args) {
    ConfigurableApplicationContext run = SpringApplication.run(DemoApplication.class, args);
    DataCollectBean bean = run.getBean(DataCollectBean.class);
    //1. 主动销毁bean
    run.getBeanFactory().destroyBean(bean);
}
```

**JVM关闭时候自动执行销毁方法**

这里就要用到钩子函数了, Spring 的钩子函数在 AbstractApplicationContext#shutdownHook属性

如果我们是SpringBoot项目我们看到在SpringApplication启动时候会注册一个钩子函数

```java
if (this.registerShutdownHook) {
    try {
       context.registerShutdownHook();
    }
    catch (AccessControlException ex) {
       // Not allowed in some environments.
    }
}

@Override
public void registerShutdownHook() {
    if (this.shutdownHook == null) {
       // No shutdown hook registered yet.
       this.shutdownHook = new Thread(SHUTDOWN_HOOK_THREAD_NAME) {
          @Override
          public void run() {
             synchronized (startupShutdownMonitor) {
                doClose();
             }
          }
       };
       Runtime.getRuntime().addShutdownHook(this.shutdownHook);
    }
}
/**
    其内部销毁了所有的bean（调用了bean的destory方法）（实现了接口 DisposableBean）
    其内部关闭了了所有的beanFactory
**/
protected void doClose() {
    // Check whether an actual close attempt is necessary...
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
       if (logger.isDebugEnabled()) {
          logger.debug("Closing " + this);
       }

       LiveBeansView.unregisterApplicationContext(this);

       try {
          // Publish shutdown event.
          publishEvent(new ContextClosedEvent(this));
       }
       catch (Throwable ex) {
          logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
       }

       // Stop all Lifecycle beans, to avoid delays during individual destruction.
       if (this.lifecycleProcessor != null) {
          try {
             this.lifecycleProcessor.onClose();
          }
          catch (Throwable ex) {
             logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
          }
       }

       // Destroy all cached singletons in the context's BeanFactory.
       destroyBeans();

       // Close the state of this context itself.
       closeBeanFactory();

       // Let subclasses do some final clean-up if they wish...
       onClose();

       // Reset local application listeners to pre-refresh state.
       if (this.earlyApplicationListeners != null) {
          this.applicationListeners.clear();
          this.applicationListeners.addAll(this.earlyApplicationListeners);
       }

       // Switch to inactive.
       this.active.set(false);
    }
}
```

**触发钩子函数的场景**

![高级Java开发者都知道的Hook钩子函数，你还不知道吗？_Java_05](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408132128184.webp)

### what

JVM 退出的钩子函数是指在 JVM 进程即将退出时，自动执行用户指定的代码段

### addShutdownHook 的使用场景？

通过代码试验，能够感知 addShutdownHook(new Thread(){}) 是 JVM 销毁前要执行的一个线程，那么只要是涉及到资源回收的场景，应该都可以满足，下面简单列举几个

> 数据同步神器 Canal 借助它，来进行关闭 socket 链接、释放 canal 的工作节点、清理缓存信息等
>
> 海量日志收集 Flume 借助它，来实现线程池资源关闭、工作线程停止等
>
> 在应用正常退出时，执行特定的业务逻辑、关闭资源等操作
>
> 在 OOM 宕机、 CTRL+C、或执行 kill pid，导致 JVM 非正常退出时，加入必要的挽救措施成为可能

### how

```java
public class Main {
  public static void main(String[] args) {
    // 注册ShutdownHook
    Runtime.getRuntime().addShutdownHook(new Thread() {
      public void run() {
        // 添加自定义逻辑
        System.out.println("ShutdownHook executed.");
      }
    });

    // 触发Hook函数
    Runtime.getRuntime().exit(0);
  }
}
```

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408141613040.png)

## 读取配置文件

### @Value()

通过在属性上使用@Value注解，直接将文件中的属性值注入到对应的属性中

```java
@Component
@Data
public class Host {

    @Value("${host01.host}")
    private String host;

    @Value("${host01.port}")
    private String port;

    @Value("${host01.user}")
    private String user;

    @Value("${host01.password}")
    private String password;

}
```

### @ConfigurationProperties(prefix = "前缀")

Spring源码中大量使用了ConfigurationProperties注解，比如server.port就是由该注解获取到的，通过与其他注解配合使用，能够实现Bean的按需配置。 该注解有一个prefix属性，通过指定的前缀，绑定配置文件中的配置，该注解可以放在类上，也可以放在方法上

@ConfigurationProperties注解将配置文件中的值映射到bean的属性上,通过在配置类上使用@ConfigurationProperties注解，将配置文件中的属性值映射到配置类的属性上。这种方式适用于需要将配置文件中的多个属性值映射到一个配置类中的情况

```java
@Component
@ConfigurationProperties(prefix = "host01")
@Data
public class Host {

    //@Value("${host01.host}")
    private String host;

    //@Value("${host01.port}")
    private String port;

    //@Value("${host01.user}")
    private String user;

    //@Value("${host01.password}")
    private String password;

}
```







# 执行启动方法

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication.run(Main.class, args);
    }
}
```

进去之后会先new一个SpringApplication对象，然后执行这个对象的run方法

```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
    // primarySources就是main方法所在类的类对象 Main.class
    return new SpringApplication(primarySources).run(args);
}
```

# new一个SpringApplication对象

```java
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    // 资源加载器，这里是null
    this.resourceLoader = resourceLoader;
    Assert.notNull(primarySources, "PrimarySources must not be null");
    // main方法所在类
    this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    // 根据依赖推断服务类型
    this.webApplicationType = WebApplicationType.deduceFromClasspath();
    // 加载spring.factory文件中的ApplicationContextInitializer.class对象
    setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    // 加载spring.factory文件中的ApplicationListener.class对象
    setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    // 检测main方法所在类
    this.mainApplicationClass = deduceMainApplicationClass();
}
```

## 根据依赖推断服务类型

```java
static WebApplicationType deduceFromClasspath() {
    /**
    WEBFLUX_INDICATOR_CLASS:org.springframework.web.reactive.DispatcherHandler
    WEBMVC_INDICATOR_CLASS:org.springframework.web.servlet.DispatcherServlet
    JERSEY_INDICATOR_CLASS:org.glassfish.jersey.servlet.ServletContainer
    **/
    if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
          && !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
       return WebApplicationType.REACTIVE;
    }
    for (String className : SERVLET_INDICATOR_CLASSES) {
       if (!ClassUtils.isPresent(className, null)) {
          return WebApplicationType.NONE;
       }
    }
    return WebApplicationType.SERVLET;
}
```

## 加载spring.factories文件中的ApplicationContextInitializer.class对象

### 作用

ApplicationContextInitializer是Spring给我们提供的一个在容器刷新之前对容器进行初始化操作的能力，如果有具体的需要在容器刷新之前处理的业务，可以通过ApplicationContextInitializer接口来实现

### META-INF/spring.factories文件

+ 加载resources文件下META-INF/spring.factories

```properties
org.springframework.context.ApplicationContextInitializer=\
com.qiha.testquartz.initializer.FirstInitializer,\
com.qiha.testquartz.initializer.TestApplicationContextInitializer,\
com.qiha.testquartz.initializer.PropertyLoadApplicationContextInitializer


org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener,\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener,\
com.qiha.testquartz.listener.PreparedEventApplicationListener
```

+ 加载springboot自带初始化器，自带有2个，分别在源码的jar包的 spring-boot-autoconfigure 项目 和 spring-boot 项目里面各有一个

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408141618458.png)

### getSpringFactoriesInstances源码

```java
// type = ApplicationContextInitializer.class
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
    ClassLoader classLoader = getClassLoader();
    // Use names and ensure unique to protect against duplicates
    // 返回了注册在META-INF/spring.factories文件中的ApplicationContextInitializer.class接口的所有实现类，
    Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
    // 对每一个实现类进行实例化
    List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
    // 对实现的实例进行排序，如果存在多个ApplicationContextInitializer.class实例，那么实例的执行顺序是什么样的？
    // 在这里通过Ordered接口或者@Order注解进行实例排序
    // 优先从Ordered接口获取顺序，如果没有实现Ordered接口就通过@Order注解获取顺序，如果都没有，那就是顺序无所谓
    // 数值越小越优先
    // 实现PriorityOrdered接口的对象的优先级高于注解方式的和实现@Ordered注解方式的
    AnnotationAwareOrderComparator.sort(instances);
    return instances;
}
// factoryType = ApplicationContextInitializer.class
// SpringFactoriesLoader.loadFactoryNames(type, classLoader)
public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
    // factoryTypeName = org.springframework.context.ApplicationContextInitializer
    String factoryTypeName = factoryType.getName();
    return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
}

// loadSpringFactories(classLoader)

private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
    Map<String, List<String>> result = (Map)cache.get(classLoader);
    if (result != null) {
       return result;
    }

    try {
       // FACTORIES_RESOURCE_LOCATION = META_INF/spring.factories
       // 加载出了所有META_INF/spring.factories文件
       Enumeration<URL> urls = (classLoader != null ?
             classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
             ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
       result = new LinkedMultiValueMap<>();
       // 遍历全部的META_INF/spring.factories文件
       while (urls.hasMoreElements()) {
          URL url = urls.nextElement();
          UrlResource resource = new UrlResource(url);
          // 每一个META_INF/spring.factories文件相当于是一个properties文件
          Properties properties = PropertiesLoaderUtils.loadProperties(resource);
          // 转成Map格式，key是接口名，value是实现类的列表
          for (Map.Entry<?, ?> entry : properties.entrySet()) {
             String factoryTypeName = ((String) entry.getKey()).trim();
             for (String factoryImplementationName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
                result.add(factoryTypeName, factoryImplementationName.trim());
             }
          }
       }
       cache.put(classLoader, result);
       return result;
    }
    catch (IOException ex) {
       throw new IllegalArgumentException("Unable to load factories from location [" +
             FACTORIES_RESOURCE_LOCATION + "]", ex);
    }
}

// createSpringFactoriesInstances
private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
       ClassLoader classLoader, Object[] args, Set<String> names) {
    List<T> instances = new ArrayList<>(names.size());
    for (String name : names) {
       try {
          Class<?> instanceClass = ClassUtils.forName(name, classLoader);
          Assert.isAssignable(type, instanceClass);
          Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
          T instance = (T) BeanUtils.instantiateClass(constructor, args);
          instances.add(instance);
       }
       catch (Throwable ex) {
          throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
       }
    }
    return instances;
}

// 优先给予Ordered接口获取顺序，如果没有实现Ordered接口就通过@Order注解获取顺序
public class AnnotationAwareOrderComparator extends OrderComparator {
    .....
    @Override
    @Nullable
    protected Integer findOrder(Object obj) {
        Integer order = super.findOrder(obj);
        if (order != null) {
           return order;
        }
        return findOrderFromAnnotation(obj);
    }
    .....
    
    @Nullable
    protected Integer findOrder(Object obj) {
        return (obj instanceof Ordered ? ((Ordered) obj).getOrder() : null);
    }
}

public class OrderComparator{
    @Nullable
    protected Integer findOrder(Object obj) {
        return (obj instanceof Ordered ? ((Ordered) obj).getOrder() : null);
    }
    
    @Override
    public int compare(@Nullable Object o1, @Nullable Object o2) {
        return doCompare(o1, o2, null);
    }
    
    private int doCompare(@Nullable Object o1, @Nullable Object o2, @Nullable OrderSourceProvider sourceProvider) {
        boolean p1 = (o1 instanceof PriorityOrdered);
        boolean p2 = (o2 instanceof PriorityOrdered);
        if (p1 && !p2) {
           return -1;
        }
        else if (p2 && !p1) {
           return 1;
        }
    
        int i1 = getOrder(o1, sourceProvider);
        int i2 = getOrder(o2, sourceProvider);
        return Integer.compare(i1, i2);
    }
}
```

### ApplicationContextInitializer.class的注册方式和执行顺序

- 注册方式1---通过`application.properties`注册（最先执行）
```Java
    context.initializer.classes=com.qiha.testquartz.initializer.ThirdInitializer
```
- 注册方式2---通过spring.factories文件（中间执行，内部可以通过下面方式调整顺序）
  - 可以通过`Ordered`接口、`@Order`、`PriorityOrdered`接口来调整顺序**（这种调整顺序的方式只限于注册在spring.factories的文件中）**`ApplicationContextInitializer.class`，通过别的方式注册的无法通过这种方式调整顺序）
  - 实现`PriorityOrdered`接口的`ApplicationContextInitializer.class`的优先级高于实现`Ordered`接口的或者被`@Order`注解的
  - 如果某个`ApplicationContextInitializer.class`注解了`@Order`且实现了`Ordered`接口，那么优先从`Ordered`接口获取顺序，如果没有实现`Ordered`接口才会去注解获取顺序
  - 顺序数越小优先级越高
- 注册方式3----通过启动方法直接添加（最后执行）
  - 通过加入顺序控制执行顺序

```java
public static void main(String[] args) {
    SpringApplication application = new SpringApplication(Main.class);
    // 可以通过加入的顺序调整ApplicationContextInitializer.class的执行顺序
    application.addInitializers(new SecondInitializer());
    application.run(args);
}
// 可以看到加入到了initializers列表的后面，在此之前initializers列表中已经存在了加载自spring.factories中的ApplicationContextInitializer.class
// 因此通过这种方式添加的ApplicationContextInitializer.class的执行顺序在spring.factories注册的方式的之后
public void addInitializers(ApplicationContextInitializer<?>... initializers) {
    this.initializers.addAll(Arrays.asList(initializers));
}
```

### ApplicationContextInitializer.class执行顺序示例

#### 定义三个`ApplicationContextInitializer.class`

```java
public class FirstInitializer implements ApplicationContextInitializer {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();

        Map<String, Object> map = new HashMap<>();
        map.put("key1", "First");

        MapPropertySource mapPropertySource = new MapPropertySource("firstInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);

        System.out.println("run firstInitializer");
    }

}

public class SecondInitializer implements ApplicationContextInitializer {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();

        Map<String, Object> map = new HashMap<>();
        map.put("key1", "Second");

        MapPropertySource mapPropertySource = new MapPropertySource("secondInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);

        System.out.println("run secondInitializer");
    }

}

public class ThirdInitializer implements ApplicationContextInitializer {

    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();

        Map<String, Object> map = new HashMap<>();
        map.put("key1", "Third");

        MapPropertySource mapPropertySource = new MapPropertySource("thirdInitializer", map);
        environment.getPropertySources().addLast(mapPropertySource);

        System.out.println("run thirdInitializer");
    }

}
```

#### 使用三种方式将这三个ApplicationContextInitializer

- 方法一：在main方法中

```java
@SpringBootApplication
public class Main {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(Main.class);
        application.addInitializers(new SecondInitializer());
        application.run(args);
    }
}
```

- 方法二：在spring.factories中

```xml
org.springframework.context.ApplicationContextInitializer=com.qiha.testquartz.initializer.FirstInitializer
```

- 方法三：在application.properties中

```properties
context.initializer.classes=com.qiha.testquartz.initializer.ThirdInitializer
```

#### 结果

```Java
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.8.RELEASE)

run thirdInitializer #properties优先级最高
run firstInitializer #spring.factories中其次
run secondInitializer #main方法优先级最差
```

#### 可以使用@Order修改优先级

order值越低优先级越高

```Java
@Order(1)
public class SecondInitializer implements ApplicationContextInitializer {
    ......
}

@Order(2)
public class FirstInitializer implements ApplicationContextInitializer {
    ......
}

@Order(3)
public class ThirdInitializer implements ApplicationContextInitializer {
    ......
}
```

## 加载spring.factories文件中的ApplicationListener.class对象

- 同加载spring.factory文件中的ApplicationContextInitializer.class对象

## 设置程序运行的主类

# 执行SpringApplication对象的run方法

![流程图](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408131347308.jpg)

```java
public ConfigurableApplicationContext run(String... args) {
    // 创建一个秒表用于记录容器的启动时间
    StopWatch stopWatch = new StopWatch();
    stopWatch.start();
    // applicationContext的上下文对象
    ConfigurableApplicationContext context = null;
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
    configureHeadlessProperty();
    /**
        第一步：
        加载spring.factories文件中的所有SpringApplicationRunListener对象
        这个对象是事件发布者，他在springBoot启动的不同阶段会发布不同的事件
        SpringApplicationRunListener接口只有一个实现类EventPublishingRunListener
    **/
    SpringApplicationRunListeners listeners = getRunListeners(args);
    /**
        第二步：
        run方法刚开始执行的事件广播
        即，触发事件ApplicationStartingEvent，所有监听了事件ApplicationStartingEvent的ApplicationListener都会在此时执行
    **/
    listeners.starting();
    try {
       /**
           第三步：创建并配置当前应用将要使用的环境
           1. 加载命令行
           2. 命令行中可能存在指定环境变量的参数--spring.active.profile=dev，所以需要传入参数applicationArguments
           3. 在创建环境完成后需要发布事件environmentPrepared，所以需要传入参数listeners
       **/
       ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
       ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
       
       configureIgnoreBeanInfo(environment);
       
       /**
           第四步：打印banner信息
       **/
       Banner printedBanner = printBanner(environment);
       /**
           第五步：创建applicationContext
           基于反射技术，根据应用类型创建了applicationContext对象，但是并没有设置applicationContext的属性
           到了这一步中context中有一个environment属性，但是其不是springApplication对象中的environment属性
           // environment.getProperty("test.string") = test
           // context.getEnvironment.getProperty("test.string") = null
       **/
       context = createApplicationContext();
       /**
           第六步：创建异常报告器
           1. 获取定义在spring.factories中的SpringBootExceptionReporter类的对象
           2. SpringBootExceptionReporter接口只有一个实现类FailureAnalyzers
           3. FailureAnalyzers会获取spring.factories定义的FailureAnalyzer类的对象  
       **/
       exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
             new Class[] { ConfigurableApplicationContext.class }, context);
       /**
           第七步：初始化容器对象（初始化applicationContext）
           1.将第三步创建的环境设置到applicationContext里面去
               在这一步之后applicationContext中的environment等于springApplication对象中的environment
           2.调用ApplicationContextInitailizer接口
               applyInitializers(context);
           3.发布ApplicationContextInializer执行完毕事件 ApplicationContextInitializedEvent
               listeners.contextPrepared(context);
           4.输出容器启动的前两条日志
                // 2023-08-26 20:59:38.282  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : Starting Main on MBYP46525326.local with PID 76061 (/Users/cheng/IdeaProjects/qiha/test-spring-boot-start/build/classes/java/main started by  cheng in /Users/ cheng/IdeaProjects/qiha)
                // 2023-08-26 20:59:38.350  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : No active profile set, falling back to default profiles: default
                // 可以通过配置参数 spring.main.log-startup-info=false 设置是否输出
                if (this.logStartupInfo) {
                   logStartupInfo(context.getParent() == null);
                   logStartupProfileInfo(context);
                }
          5.将启动参数applicationArguments保存到容器中
          6.将printedBanner保存到容器中
          7.设置是否允许相同beanName的bean覆盖
               可以通过参数spring.main.allow-bean-definition-overriding=true设置（spring-boot中默认是false）
               if (beanFactory instanceof DefaultListableBeanFactory) {
                    ((DefaultListableBeanFactory) beanFactory)
                          .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
               }
         8.是否开启懒加载，可以通过参数spring.main.lazy-initialization=true设置
               在dev环境中可以开启，加快调试速度
                 可以在JVM参数上加-Dspring.main.lazy-initialization=true启动
                 可以在JVM参数加参数，然后参数会被解析之后存储到environment中
                 比如-Dtest.c=c 可以environment.getProperty("test.c")
          9.将启动类加入到applicationContext容器中
          10.发布事件ApplicationPreparedEvent，表示容器已经准备好了
       **/
       prepareContext(context, environment, listeners, applicationArguments, printedBanner);
       /**
           第八步：刷新容器
           1. 注册应用程序关闭的钩子函数
               spring.main.register-shutdown-hook=false,默认是开启的，一般开启就好了
           2. 刷新容器，详见3.8
       **/
       refreshContext(context);
       /**
          第九步：刷新容器后操作
       **/
       afterRefresh(context, applicationArguments);
       // SpringBoot启动结束，关闭秒表，并且日志打印出启动时长
       // 可以通过配置参数 spring.main.log-startup-info=false 设置是否输出
       stopWatch.stop();
       if (this.logStartupInfo) {
          new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
       }
       listeners.started(context);
       callRunners(context, applicationArguments);
    }
    catch (Throwable ex) {
        /**
            异常报告器的用处！！！
            发布事件 ApplicationFailedEvent
        **/
       handleRunFailure(context, ex, exceptionReporters, listeners);
       throw new IllegalStateException(ex);
    }

    try {
       listeners.running(context);
    }
    catch (Throwable ex) {
       handleRunFailure(context, ex, exceptionReporters, null);
       throw new IllegalStateException(ex);
    }
    return context;
}
```

## 第一步：加载SpringApplicationRunListener对象

### SpringApplicationRunListener源码

```java
public interface SpringApplicationRunListener {

    /**
     * run方法刚执行时的通知  ApplicationStartingEvent
     */
    default void starting() {
    }

    /**
     * 环境准备好之后的通知 ApplicationEnvironmentPreparedEvent
     */
    default void environmentPrepared(ConfigurableEnvironment environment) {
    }

    /**
     * applicationContext准备好之后的通知
     */
    default void contextPrepared(ConfigurableApplicationContext context) {
    }

    /**
     * Called once the application context has been loaded but before it has been
     * refreshed.
     * @param context the application context
     */
    default void contextLoaded(ConfigurableApplicationContext context) {
    }

    /**
     * The context has been refreshed and the application has started but
     * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
     * ApplicationRunners} have not been called.
     * @param context the application context.
     * @since 2.0.0
     */
    default void started(ConfigurableApplicationContext context) {
    }

    /**
     * Called immediately before the run method finishes, when the application context has
     * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
     * {@link ApplicationRunner ApplicationRunners} have been called.
     * @param context the application context.
     * @since 2.0.0
     */
    default void running(ConfigurableApplicationContext context) {
    }

    /**
     * 启动失败时的通 ApplicationFailedEvent
     */
    default void failed(ConfigurableApplicationContext context, Throwable exception) {
    }

}
```

`SpringApplicationRunListener`监听器在SpringBoot启动的不同阶段会发布相应的监听通知。通知贯穿应用启动的全过程

- 以`environmentPrepared`通知为例，`SpringApplicationRunListener`是如何发布该事件的。在`SpringApplicationRunListener`的实现类`EventPublishingRunListener`中有一个属性为`initialMulticaster`的`SimpleApplicationEventMulticaster`实例。在`environmentPrepared`方法中调用了`initialMulticaster`的`multicastEvent`方法。
- 在multicastEvent方法中又获取了所有的`ApplicationListener`对象，将事件广播给了事件监听器`Listener`

```Java
@Override
public void environmentPrepared(ConfigurableEnvironment environment) {
    this.initialMulticaster
          .multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
}

@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
       if (executor != null) {
          executor.execute(() -> invokeListener(listener, event));
       }
       else {
          invokeListener(listener, event);
       }
    }
}
```

## 第二步：SpringApplicationRunListener调用方法startring触发事件`ApplicationStartingEvent`

```java
Override
public void starting() {
    this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
}

@Override
public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
    ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
    Executor executor = getTaskExecutor();
    for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
       if (executor != null) {
          executor.execute(() -> invokeListener(listener, event));
       }
       else {
          invokeListener(listener, event);
       }
    }
}
```

### 测试

#### 事件`ApplicationStartingEvent`

- 开始启动时监听，对应`SpringApplicationRunListener.running`
- 写一个监听事件`ApplicationStartingEvent`的监听器
```Java
public class StartingEventApplicationListener implements ApplicationListener<ApplicationStartingEvent> {

    @Override
    public void onApplicationEvent(ApplicationStartingEvent event) {
        System.out.println("run方法刚开始执行");
        System.out.println(event.getSource().toString());
        System.out.println(event.getSpringApplication().toString());
    }
}
```
- 在spring.factories中注册该监听器
```Java
org.springframework.context.ApplicationListener=com.qiha.testquartz.listener.StartingEventApplicationListener
```
- 运行结果如下
```Java
run方法刚开始执行
org.springframework.boot.SpringApplication@27c20538
org.springframework.boot.SpringApplication@27c20538

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.8.RELEASE)
```

#### 事件`ApplicationEnvironmentPreparedEvent`

- 环境准备好时监听，对应`SpringApplicationRunListener.environmentPrepared`

- 写一个监听事件`ApplicationEnvironmentPreparedEvent`的监听器

```Java
public class EnvironmentPreparedEventApplicationListener implements ApplicationListener {

    @Override
    public void onApplicationEvent(ApplicationEvent event) {
        if (event instanceof ApplicationEnvironmentPreparedEvent) {
            ApplicationEnvironmentPreparedEvent environmentPreparedEvent = (ApplicationEnvironmentPreparedEvent) event;
            System.out.println("应用环境准备好了");
            Environment environment = environmentPreparedEvent.getEnvironment();
            System.out.println(environment.getProperty("test.string", String.class, ""));
            System.out.println(environment.getProperty("test.boolean", Boolean.class, Boolean.TRUE));
        }
    }
}
```

- 在`application.properties`中存在配置

```Properties
test.env=test
```

- 在`spring.factories`中注册该监听器

```Properties
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener
```

- 运行结果如下

```Java
run方法刚开始执行
org.springframework.boot.SpringApplication@59494225
org.springframework.boot.SpringApplication@59494225
应用环境准备好了
test
true

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.8.RELEASE)
```

**结果分析**

-  可以发现`System.out.println(environment.getProperty("test.string"))`输出了`applicaiton.properties`中定义的配置`test.env`，由此可见`environment`中包含了`application.properties`中的全部配置。

- - 在`springboot`的启动过程中，可以通过`environment.getProperty`获取配置
  - 在`ApplicationEnvironmentPreparedEvent`事件的监听器中也可以通过`environment.getProperty`获取配置

#### 事件`ApplicationFailedEvent`

- 启动失败时监听，对应`SpringApplicationRunListener.failed`
- 写一个监听事件`ApplicationFailedEvent`的监听器
```Java
public class FailedEventApplicationListener implements ApplicationListener<ApplicationFailedEvent> {

    @Override
    public void onApplicationEvent(ApplicationFailedEvent event) {
        System.out.println("启动失败：" + event.getException().getMessage());
    }
}
```
- 在`spring.factories`中注册该监听器
```Java
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener\
```
- 启动类
```Java
@SpringBootApplication
public class Main {
    // 这个依赖不满足
    @Autowired
    private Test test;

    public static void main(String[] args) {
        SpringApplication.run(Main.class,args);
    }
}
```
- 运行结果如下
```Java
启动失败：Error creating bean with name 'main': Unsatisfied dependency expressed through field 'test'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.qiha.testquartz.Test' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

#### 事件`ApplicationContextInitializedEvent`

- ApplicationContextInitailizer接口全部被调用后被监听，对应`SpringApplicationRunListener.contextPrepared`
- 写一个监听事件`ApplicationContextInitializedEvent`的接口
```Java
public class ContextInitializedEventApplicationListener implements ApplicationListener<ApplicationContextInitializedEvent> {

    @Override
    public void onApplicationEvent(ApplicationContextInitializedEvent event) {
        ApplicationContext context = event.getApplicationContext();
        Environment environment = context.getEnvironment();
        System.out.println("ApplicationContextInializer已经被调用完成了");
        System.out.println(environment.getProperty("test.string"));
        System.out.println(environment.getProperty("test.boolean"));
    }
}
```
- 在`spring.factories`中注册该监听器
```Java
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener,\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener
```
- 运行结果如下
```Java
###############################
#                             #
#                             #
#         开始启动应用          #
#                             #
#                             #
###############################
执行了TestApplicationContextInitializer # applicationContextInializer中输出的
test # applicationContextInializer中输出的
true # applicationContextInializer中输出的
ApplicationContextInializer已经被调用完成了
test
true
```

#### 事件`ApplicationPreparedEvent`

- 在容器准备好，但是容器刷新前被触发，对应`SpringApplicationRunListener.contextLoaded`
- 在这个事件被触发的时候，所有实现了ApplicationContextAware接口的ApplicationContext类被注入了applicationContext
```Java
@Override
public void contextLoaded(ConfigurableApplicationContext context) {
    for (ApplicationListener<?> listener : this.application.getListeners()) {
       if (listener instanceof ApplicationContextAware) {
          ((ApplicationContextAware) listener).setApplicationContext(context);
       }
       context.addApplicationListener(listener);
    }
    this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
}
```
- 写一个监听事件`ApplicationPreparedEvent`的接口
```Java
public class PreparedEventApplicationListener implements ApplicationListener<ApplicationPreparedEvent>, ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(ApplicationPreparedEvent event) {
        System.out.println("容器applicationContext已经准备好了");
        System.out.println(applicationContext == event.getApplicationContext()); // 输出了true，可见完全可以直接通过event获取application
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```
- 在`spring.factories`中注册该监听器
```Java
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener,\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener,\
com.qiha.testquartz.listener.PreparedEventApplicationListener
```
- 运行结果如下
```Java
容器applicationContext已经准备好了
true
```

## 第三步：创建和配置环境Environment

准备环境变量，包含系统属性和用户配置的属性

```java
private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
       ApplicationArguments applicationArguments) {
    // Create and configure the environment
    ConfigurableEnvironment environment = getOrCreateEnvironment();
    configureEnvironment(environment, applicationArguments.getSourceArgs());
    ConfigurationPropertySources.attach(environment);
    // 发布了事件ApplicationEnvironmentPreparedEvent
    listeners.environmentPrepared(environment);
    bindToSpringApplication(environment);
    if (!this.isCustomEnvironment) {
       environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
             deduceEnvironmentClass());
    }
    ConfigurationPropertySources.attach(environment);
    return environment;
}
```

## 第四步：打印banner

### 源码

```properties
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.3.8.RELEASE)
```

```java
private Banner printBanner(ConfigurableEnvironment environment) {
    // 关闭banner的输出 spring.main.banner-mode=off
    if (this.bannerMode == Banner.Mode.OFF) {
       return null;
    }
    ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
          : new DefaultResourceLoader(null);
    SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
    // 输出到日志 spring.main.banner-mode=log # 在日志输出banner
    if (this.bannerMode == Mode.LOG) {
       return bannerPrinter.print(environment, this.mainApplicationClass, logger);
    }
    // 输出到控制台 spring.main.banner-mode=console默认的
    return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}

// 进入bannerPrinter.print方法
Banner print(Environment environment, Class<?> sourceClass, PrintStream out) {
    Banner banner = getBanner(environment);
    banner.printBanner(environment, sourceClass, out);
    return new PrintedBanner(banner, sourceClass);
}

// 进入getBanner方法
private Banner getBanner(Environment environment) {
    Banners banners = new Banners();
    banners.addIfNotNull(getImageBanner(environment));
    banners.addIfNotNull(getTextBanner(environment));
    if (banners.hasAtLeastOneBanner()) {
       return banners;
    }
    if (this.fallbackBanner != null) {
       return this.fallbackBanner;
    }
    // 如果textBanner和imageBanner都是空，那就使用默认的banner
    return DEFAULT_BANNER;
}

private Banner getTextBanner(Environment environment) {
    // spring.banner.location,"banner.txt"
    // 可以发现，他会根据配置spring.banner.location指定的位置去找banner文件，如果spring.banner.location为空，就去banner.txt文件找
    String location = environment.getProperty(BANNER_LOCATION_PROPERTY, DEFAULT_BANNER_LOCATION);
    Resource resource = this.resourceLoader.getResource(location);
    if (resource.exists()) {
       return new ResourceBanner(resource);
    }
    return null;
}

// 如果textBanner和imageBanner都是空，那就使用默认的banner
class SpringBootBanner implements Banner {

    private static final String[] BANNER = { "", "  .   ____          _            __ _ _",
          " /\\\\ / ___'_ __ _ _(_)_ __  __ _ \\ \\ \\ \\", "( ( )\\___ | '_ | '_| | '_ \\/ _` | \\ \\ \\ \\",
          " \\\\/  ___)| |_)| | | | | || (_| |  ) ) ) )", "  '  |____| .__|_| |_|_| |_\\__, | / / / /",
          " =========|_|==============|___/=/_/_/_/" };

    private static final String SPRING_BOOT = " :: Spring Boot :: ";

    private static final int STRAP_LINE_SIZE = 42;

    @Override
    public void printBanner(Environment environment, Class<?> sourceClass, PrintStream printStream) {
       for (String line : BANNER) {
          printStream.println(line);
       }
       String version = SpringBootVersion.getVersion();
       version = (version != null) ? " (v" + version + ")" : "";
       StringBuilder padding = new StringBuilder();
       while (padding.length() < STRAP_LINE_SIZE - (version.length() + SPRING_BOOT.length())) {
          padding.append(" ");
       }

       printStream.println(AnsiOutput.toString(AnsiColor.GREEN, SPRING_BOOT, AnsiColor.DEFAULT, padding.toString(),
             AnsiStyle.FAINT, version));
       printStream.println();
    }

}
```

- 根据源码可知，可以通过配置文件来控制banner的输出

```Properties
spring.main.banner-mode=off # 不打印banner
spring.main.banner-mode=console # 在控制台输出banner
spring.main.banner-mode=log # 在日志输出banner
```

### 测试

- 在`resource`目录下写上`banner.txt`
```Java
    ###############################
    #                             #
    #                             #
    #         开始启动应用          #
    #                             #
    #                             #
    ###############################
```
- 运行结果
```Java
    run方法刚开始执行
    org.springframework.boot.SpringApplication@59494225
    org.springframework.boot.SpringApplication@59494225
    应用环境准备好了
    test
    true
    ###############################
    #                             #
    #                             #
    #         开始启动应用          #
    #                             #
    #                             #
    ###############################
```
- 如果不希望在`banner.txt`中自定义`banner`，可以在配置文件中指定路径
```Java
    spring.banner.location=banner/banner.txt
```

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408131550873.png)

## 第五步：创建applicationContext

根据应用的类型创建不同类型的`applicationContext`

```Java
protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = this.applicationContextClass;
    if (contextClass == null) {
       try {
          switch (this.webApplicationType) {
          case SERVLET:
             // AnnotationConfigServletWebServerApplicationContext
             contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
             break;
          case REACTIVE:
             // AnnotationConfigReactiveWebServerApplicationContext
             contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
             break;
          default:
             // AnnotationConfigApplicationContext
             contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
          }
       }
       catch (ClassNotFoundException ex) {
          throw new IllegalStateException(
                "Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
       }
    }
    // 通过反射，根据类型名实例化对应的applicaitonContext
    /**
        获取构造器
        根据构造器newInstance
        public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
            Class<?> clazz = AnnotationConfigServletWebServerApplicationContext.class;
            Constructor<?> ctor = clazz.getConstructor();
            ConfigurableApplicationContext applicatonContext = (ConfigurableApplicationContext) ctor.newInstance();
            System.out.println(applicatonContext);
        }
    **/
    return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
}
```

## 第六步：创建异常报告器

 需要注意的是，这个异常报告器只会捕获启动过程抛出的异常，如果是在启动完成后，在用户请求时报错，异常报告器不会捕获请求中出现的异常

### 源码

```Java
exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
       new Class[] { ConfigurableApplicationContext.class }, context);
```

- `SpringBootExceptionReporter`只有一个实现类`FailureAnalyzers`，在`FailureAnalyzers`的构造函数中可以发现，其加载了spring.factories中的所有`FailureAnalyzer`对象
```Java
FailureAnalyzers(ConfigurableApplicationContext context, ClassLoader classLoader) {
    Assert.notNull(context, "Context must not be null");
    this.classLoader = (classLoader != null) ? classLoader : context.getClassLoader();
    // 加载所有的FailureAnalyzer类，并实例化
    this.analyzers = loadFailureAnalyzers(this.classLoader);
    // 根据FailureAnalyzer的类型，向其内注入beanFactory或者environment
    prepareFailureAnalyzers(this.analyzers, context);
}


private List<FailureAnalyzer> loadFailureAnalyzers(ClassLoader classLoader) {
    List<String> analyzerNames = SpringFactoriesLoader.loadFactoryNames(FailureAnalyzer.class, classLoader);
    List<FailureAnalyzer> analyzers = new ArrayList<>();
    for (String analyzerName : analyzerNames) {
       try {
          Constructor<?> constructor = ClassUtils.forName(analyzerName, classLoader).getDeclaredConstructor();
          ReflectionUtils.makeAccessible(constructor);
          analyzers.add((FailureAnalyzer) constructor.newInstance());
       }
       catch (Throwable ex) {
          logger.trace(LogMessage.format("Failed to load %s", analyzerName), ex);
       }
    }
    AnnotationAwareOrderComparator.sort(analyzers);
    return analyzers;
}


private void prepareFailureAnalyzers(List<FailureAnalyzer> analyzers, ConfigurableApplicationContext context) {
    for (FailureAnalyzer analyzer : analyzers) {
       prepareAnalyzer(context, analyzer);
    }
}

private void prepareAnalyzer(ConfigurableApplicationContext context, FailureAnalyzer analyzer) {
    if (analyzer instanceof BeanFactoryAware) {
       ((BeanFactoryAware) analyzer).setBeanFactory(context.getBeanFactory());
    }
    if (analyzer instanceof EnvironmentAware) {
       ((EnvironmentAware) analyzer).setEnvironment(context.getEnvironment());
    }
}
```

### 异常报告器的用处

```Java
private void handleRunFailure(ConfigurableApplicationContext context, Throwable exception,
       Collection<SpringBootExceptionReporter> exceptionReporters, SpringApplicationRunListeners listeners) {
    try {
       try {
          handleExitCode(context, exception);
          if (listeners != null) {
             // 发布事件 ApplicationFailedEvent
             listeners.failed(context, exception);
          }
       }
       finally {
          reportFailure(exceptionReporters, exception);
          if (context != null) {
             context.close();
          }
       }
    }
    catch (Exception ex) {
       logger.warn("Unable to close ApplicationContext", ex);
    }
    ReflectionUtils.rethrowRuntimeException(exception);
}


private void reportFailure(Collection<SpringBootExceptionReporter> exceptionReporters, Throwable failure) {
    try {
       for (SpringBootExceptionReporter reporter : exceptionReporters) {
          if (reporter.reportException(failure)) {
             registerLoggedException(failure);
             return;
          }
       }
    }
    catch (Throwable ex) {
       // Continue with normal handling of the original failure
    }
    if (logger.isErrorEnabled()) {
       logger.error("Application run failed", failure);
       registerLoggedException(failure);
    }
}
```

## 第七步：初始化容器（初始化applicationContext）

### 源码

```Java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
       SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
    // 1.将第三步创建和配置的环境设置到applicationContext中，因此后面可以直接根据applicationContext使用环境
    context.setEnvironment(environment);
    postProcessApplicationContext(context);
    // 2.执行ApplicationContextInitializer接口实现类的方法
    applyInitializers(context);
    // 3.发布ApplicationContextInializer执行完毕事件 ApplicationContextInitializedEvent
    listeners.contextPrepared(context);
    // 4.输出容器启动的前两条日志
    // 2023-08-26 20:59:38.282  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : Starting Main on MBYP46525326.local with PID 76061 (/Users/ cheng/IdeaProjects/qiha/test-spring-boot-start/build/classes/java/main started by  cheng in /Users/ cheng/IdeaProjects/qiha)
    // 2023-08-26 20:59:38.350  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : No active profile set, falling back to default profiles: default
    // 可以通过配置参数 spring.main.log-startup-info=false 设置是否输出
    if (this.logStartupInfo) {
       logStartupInfo(context.getParent() == null);
       logStartupProfileInfo(context);
    }
    // Add boot specific singleton beans
    // 5.将启动参数保存到容器中
    ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
    beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
    // 6.将printedBanner保存到容器中
    if (printedBanner != null) {
       beanFactory.registerSingleton("springBootBanner", printedBanner);
    }
    // 7.设置是否允许相同beanName的bean进行覆盖
    // 可以通过参数spring.main.allow-bean-definition-overriding=true设置（spring-boot中默认是false）
    if (beanFactory instanceof DefaultListableBeanFactory) {
       ((DefaultListableBeanFactory) beanFactory)
             .setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
    }
    // 8.是否开启懒加载，懒加载可以提高容器的启动速度
    // 在dev环境中可以开启懒加载，提高调试速度
    // spring.main.lazy-initialization=true
    if (this.lazyInitialization) {
       context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
    }
    // 9.将启动类加载到bean容器中
    // Load the sources
    Set<Object> sources = getAllSources();
    Assert.notEmpty(sources, "Sources must not be empty");
    load(context, sources.toArray(new Object[0]));
    // 10.发布事件ApplicationPreparedEvent，表示容器已经准备好了
    // 在这一步中，将applicationListener存储到了容器里，并且实现了ApplicationContextAware的接口被注入了applicationContext，在这一步之前实现了ApplicationContextAware接口的ApplicationListener类中的applicationContext是空的
    listeners.contextLoaded(context);
}

2. 执行ApplicationContextInitializer接口实现类的方法
protected void applyInitializers(ConfigurableApplicationContext context) {
    // getInitializers方法内部根据@Order注解进行了排序 new AnnotationAwareOrderComparator();
    for (ApplicationContextInitializer initializer : getInitializers()) {
       Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
             ApplicationContextInitializer.class);
       Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
       // 调用了ApplicationContextInitailizer接口的initialize方法
       initializer.initialize(context);
    }
}

@Override
public void contextLoaded(ConfigurableApplicationContext context) {
    for (ApplicationListener<?> listener : this.application.getListeners()) {
       if (listener instanceof ApplicationContextAware) {
          ((ApplicationContextAware) listener).setApplicationContext(context);
       }
       context.addApplicationListener(listener);
    }
    this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
}
```

这里准备的上下文环境是为了下一步刷新做准备的，里面还做了一些额外的事情

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408141641175.png)

**实例化单例的beanName生成器**

在 postProcessApplicationContext(context); 方法里面。使用单例模式创建 了BeanNameGenerator 对象，其实就是beanName生成器，用来生成bean对象的名称

**执行初始化方法**

初始化方法有哪些呢？还记得第3步里面加载的初始化器嘛？其实是执行第3步加载出来的所有初始化器，实现了ApplicationContextInitializer 接口的类

**将启动参数注册到容器中**

这里将启动参数以单例的模式注册到容器中，是为了以后方便拿来使用，参数的beanName 为 ：springApplicationArguments

### 测试通过ApplicationContextInitializer更新environment中的配置参数

**比如可以从配置中心拉取配置，注入到environment中去**

- **apollo的配置加载就是类似于这个实现机制**
- 实现一个用于配置加载的`ApplicationContextInitializer`
```Java
public class PropertyLoadApplicationContextInitializer implements ApplicationContextInitializer {
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext) {
        System.out.println("开始通过PropertyLoadApplicationContextInitializer注入配置");
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        MutablePropertySources propertySources = environment.getPropertySources();
        Map<String, Object> firstMap = new HashMap<>();
        firstMap.put("test.string", "test1"); // application.properties中值是test
        firstMap.put("test.a", "a");
        Map<String, Object> lastMap = new HashMap<>();
        lastMap.put("test.boolean", false); // application.properties中值是true
        lastMap.put("test.b", "b");
        PropertySource<Map<String, Object>> firstPropertySource = new MapPropertySource("first", firstMap);
        PropertySource<Map<String, Object>> lastPropertySource = new MapPropertySource("last", lastMap);
        // 优先级高
        propertySources.addFirst(firstPropertySource);
        // 优先级低
        propertySources.addLast(lastPropertySource);
        System.out.println("通过PropertyLoadApplicationContextInitializer注入配置成功");
    }
}
```
- 配置
```Java
test.string=test
test.boolean=true
```
- 注册到spring.factories中去
```Java
org.springframework.context.ApplicationContextInitializer=\
com.qiha.testquartz.initializer.PropertyLoadApplicationContextInitializer


org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener
```
- 在`ApplicationContextInitializedEvent`事件监听器中取注入的配置
```Java
public class ContextInitializedEventApplicationListener implements ApplicationListener<ApplicationContextInitializedEvent> {

    @Override
    public void onApplicationEvent(ApplicationContextInitializedEvent event) {
        ApplicationContext context = event.getApplicationContext();
        Environment environment = context.getEnvironment();
        System.out.println("开始通过ContextInitializedEventApplicationListener获取配置值");
        System.out.println("test.string:" + environment.getProperty("test.string")); // 配置文件中是test,通过addFirst被修改为test1
        System.out.println("test.boolean:" + environment.getProperty("test.boolean")); // 配置文件中是true,通过addLast被修改为false
        System.out.println("test.a:" + environment.getProperty("test.a"));
        System.out.println("test.b:" + environment.getProperty("test.b"));
        System.out.println("test.c:" + environment.getProperty("test.c"));
        System.out.println("配置值输出结束");
    }
}
```
- 输出结果
  - 通过下面的结果可以发现，environment中维护了多套配置（PropertySource），PropertySource之间具有顺序，越靠前优先级越高，越靠后优先级越低
  - 如果在前面的PropertySource中找到了属性名对应的配置，那就直接输出
  - 如果在前面的PropertySource中不存在对应属性名的配置，那就往后找，直到找到
  - 通过ApplicationContextInitializer更新参数比从properies读取优先级更高
```Java
开始通过PropertyLoadApplicationContextInitializer注入配置
通过PropertyLoadApplicationContextInitializer注入配置成功
开始通过ContextInitializedEventApplicationListener获取配置值
test.string:test1 # addFisrt修改了配置，输出了修改后配置
test.boolean:true # addLast修改了配置，输出了修改前配置
test.a:a # 输出注入的配置
test.b:b # 输出注入的配置
test.c:c # 输入命令行参数的值 -Dtest.c=c
配置值输出结束
```

- **Environment配置寻找配置的源码**

```java
@Nullable
protected <T> T getProperty(String key, Class<T> targetValueType, boolean resolveNestedPlaceholders) {
    if (this.propertySources != null) {
       // 逐个遍历propertySource
       // 如果找到了配置，就直接返回;如果没有找到就遍历下一个propertySource
       for (PropertySource<?> propertySource : this.propertySources) {
          if (logger.isTraceEnabled()) {
             logger.trace("Searching for key '" + key + "' in PropertySource '" +
                   propertySource.getName() + "'");
          }
          Object value = propertySource.getProperty(key);
          if (value != null) {
             if (resolveNestedPlaceholders && value instanceof String) {
                value = resolveNestedPlaceholders((String) value);
             }
             logKeyFound(key, propertySource, value);
             return convertValueIfNecessary(value, targetValueType);
          }
       }
    }
    if (logger.isTraceEnabled()) {
       logger.trace("Could not find key '" + key + "' in any property source");
    }
    return null;
}
```

### 测试`ApplicationContextInitializedEvent`-`listeners.contextPrepared(context)`

- ApplicationContextInitailizer接口全部被调用后被监听，对应`SpringApplicationRunListener.contextPrepared`

- 写一个监听事件`ApplicationContextInitializedEvent`的接口

```java
public class ContextInitializedEventApplicationListener implements ApplicationListener<ApplicationContextInitializedEvent> {

    @Override
    public void onApplicationEvent(ApplicationContextInitializedEvent event) {
        ApplicationContext context = event.getApplicationContext();
        Environment environment = context.getEnvironment();
        System.out.println("ApplicationContextInializer已经被调用完成了");
        System.out.println(environment.getProperty("test.string"));
        System.out.println(environment.getProperty("test.boolean"));
    }
}
```

- 在`spring.factories`中注册该监听器

```properties
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener,\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener
```

- 运行结果如下

```java
###############################
#                             #
#                             #
#         开始启动应用          #
#                             #
#                             #
###############################
执行了TestApplicationContextInitializer # applicationContextInializer中输出的
test # applicationContextInializer中输出的
true # applicationContextInializer中输出的
ApplicationContextInializer已经被调用完成了
test
true
```

### 测试`ApplicationPreparedEvent`-`listeners.contextLoaded(context)`

- 在容器准备好，但是容器刷新前被触发，对应`SpringApplicationRunListener.contextLoaded`
- 在这个事件被触发的时候，所有实现了ApplicationContextAware接口的ApplicationContext类被注入了applicationContext

```java
@Override
public void contextLoaded(ConfigurableApplicationContext context) {
    for (ApplicationListener<?> listener : this.application.getListeners()) {
       if (listener instanceof ApplicationContextAware) {
          ((ApplicationContextAware) listener).setApplicationContext(context);
       }
       context.addApplicationListener(listener);
    }
    this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
}
```

- 写一个监听事件`ApplicationPreparedEvent`的接口

```java
public class PreparedEventApplicationListener implements ApplicationListener<ApplicationPreparedEvent>, ApplicationContextAware {

    private ApplicationContext applicationContext;

    @Override
    public void onApplicationEvent(ApplicationPreparedEvent event) {
        System.out.println("容器applicationContext已经准备好了");
        System.out.println(applicationContext == event.getApplicationContext()); // 输出了true，可见完全可以直接通过event获取application
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

- 在`spring.factories`中注册该监听器

```properties
org.springframework.context.ApplicationListener=\
com.qiha.testquartz.listener.StartingEventApplicationListener,\
com.qiha.testquartz.listener.EnvironmentPreparedEventApplicationListener,\
com.qiha.testquartz.listener.FailedEventApplicationListener,\
com.qiha.testquartz.listener.ContextInitializedEventApplicationListener,\
com.qiha.testquartz.listener.PreparedEventApplicationListener
```

- 运行结果如下

```xml
执行了TestApplicationContextInitializer
test
true
run secondInitializer
ApplicationContextInializer已经被调用完成了
test
true
2023-08-26 20:59:38.282  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : Starting Main on MBYP46525326.local with PID 76061 (/Users/ cheng/IdeaProjects/qiha/test-spring-boot-start/build/classes/java/main started by  cheng in /Users/ cheng/IdeaProjects/qiha)
2023-08-26 20:59:38.350  INFO 76061 --- [           main] com.qiha.testquartz.Main                 : No active profile set, falling back to default profiles: default
容器applicationContext已经准备好了
true
```

## 第八步：刷新容器

### 源码

```Java
private void refreshContext(ConfigurableApplicationContext context) {
    // 注册关闭容器的钩子函数
    // 可以通过spring.main.register-shutdown-hook=true开启或者关闭，默认是开启的
    // 详见3.8.2
    if (this.registerShutdownHook) {
       try {
          context.registerShutdownHook();
       }
       catch (AccessControlException ex) {
          // Not allowed in some environments.
       }
    }
    // 刷新容器
    /**
         applicationContext.refresh();
    **/
    refresh((ApplicationContext) context);
}

// AbstractApplicationContext.refresh()
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
       // Prepare this context for refreshing.
       /**
           3.8.3. 刷新容器的准备工作
               1.校验所有的必填配置是否都配置了
       **/
       prepareRefresh();
        
       
       // Tell the subclass to refresh the internal bean factory.
       /**
           3.8.4. 获取beanFactory（DefaultListableBeanFactory）
               1. 设置applicationContext状态为已刷新 refreshed=true
               2. 为beanFactory设置id（id=application）
               3. 在这一步中，还没有加载beanDefinition
           
       **/
       ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

       // Prepare the bean factory for use in this context.
       /**
           3.8.5. 完善beanFactory
               1. 设置classLoader beanClassLoader
               2. 设置el表达式的解析器 StandardBeanExpressionResolver
               3. 添加默认的beanPostProcessor（ApplicationContextAwareProcessor），
                   用来处理各种Aware
                   EnvironmentAware、EmbeddedValueResolverAware、
                   ResourceLoaderAware、ApplicationEventPublisherAware、
                   MessageSourceAware、ApplicationContextAware
               4. 设置默认的beanPostProcessor (ApplicationListenerDetector)
               5. 将environment添加到beanFactory中
               6. 将systemProperties添加在beanFactory中
               7. 将systemEnvironment添加到beanFactory中
       **/
       prepareBeanFactory(beanFactory);

       try {
          // Allows post-processing of the bean factory in context subclasses.
          /**
              见3.8.6
              beanFactory完善后的后置处理，AbstractApplicationContext中该方法是空的
              需要交给不同应用场景的ApplicationContext来自行实现
              比如在ServletWebServerApplicationContext中，因为他是基于Servlet实现的Web应用，所以进行了一些针对Servlet的处理
                  1. 在Servlte中有一个ServletContext类，他是整个Servlet应用唯一的，在ServletWebServerApplicationContext就
                      的postProcessBeanFactory就实现了添加了一个bean后置处理器WebApplicationContextServletContextAwareProcessor
                      该后置处理器用于在实现了ServletContextAware接口的bean中注入ServletContext
                  2. Servlte应用有几种作用域，所以在ServletWebServerApplicationContext中实现了作用域的注册
                      Session域、Request域
          **/
          postProcessBeanFactory(beanFactory);
          
          // Invoke factory processors registered as beans in the context.
          /**
              见3.8.7
              调用beanFactory的后置处理器
              1. BeanFactory的后置处理器一共有两种，
                 一种是普通的BeanFactoryProcessor，另一种是BeanDefinitionRegistryPostProcessor，
                 BeanDefinitionRegistryPostProcessor接口继承了BeanFactoryProcessor接口，具有一个postProcessBeanDefinitionRegistry方法（用来注册bean）
                 后者是用来解析注解，注册Bean进入容器的
              2. 由于BeanDefinitionRegistryPostProcessor解析出来的bean本身也可以是一个BeanDefinitionRegistryPostProcessor，所以有必要在bean注册之后，
                  再次查找容器中BeanDefinitionRegistryPostProcessor类型的bean，直到所有的BeanDefinitionRegistryPostProcessor都被调用了一次
          **/
          invokeBeanFactoryPostProcessors(beanFactory);

          // Register bean processors that intercept bean creation.
          /**
              见3.8.8
              注册bean的后置处理器
          **/
          registerBeanPostProcessors(beanFactory);

          // Initialize message source for this context.
          initMessageSource();

          // Initialize event multicaster for this context.
          initApplicationEventMulticaster();

          // Initialize other special beans in specific context subclasses.
          onRefresh();

          // Check for listener beans and register them.
          registerListeners();

          // Instantiate all remaining (non-lazy-init) singletons.
          finishBeanFactoryInitialization(beanFactory);

          // Last step: publish corresponding event.
          finishRefresh();
       }

       catch (BeansException ex) {
          if (logger.isWarnEnabled()) {
             logger.warn("Exception encountered during context initialization - " +
                   "cancelling refresh attempt: " + ex);
          }

          // Destroy already created singletons to avoid dangling resources.
          destroyBeans();

          // Reset 'active' flag.
          cancelRefresh(ex);

          // Propagate exception to caller.
          throw ex;
       }

       finally {
          // Reset common introspection caches in Spring's core, since we
          // might not ever need metadata for singleton beans anymore...
          resetCommonCaches();
       }
    }
}
```

### 注册容器关闭的钩子函数

#### 测试JAVA自带的程序关闭钩子线程

```Java
public class ShutdownHookTest {

    public static void main(String[] args) {
        Thread thread = new Thread("应用关闭钩子线程") {
            @Override
            public void run() {
                System.out.println("程序正在关闭");
            }
        };
        Runtime.getRuntime().addShutdownHook(thread);


        for (int i = 0; i <= 10; i++) {
            System.out.println(i);
        }
    }
}
```
从结果可以看出，程序关闭钩子线程在程序运行结束之后才会执行，可以在其中进行一些资源的回收，优雅的关闭程序，比如关闭文件、断开连接等

```Java
0
1
2
3
4
5
6
7
8
9
10
程序正在关闭
```

#### SpringBoot应用关闭钩子线程

- 它是基于Java程序关闭钩子线程实现的**，本质上就是Java程序关闭钩子线程**
- 可以通过配置如下，关闭关闭的钩子函数注册，默认是开启的
  - 直接开启就好了，如果不去注册的话，在应用关闭的时候，bean不会被优雅的注销，可能导致内存泄漏、资源不能被回收，比如文件描述符、数据库连接资源等
```Java
spring.main.register-shutdown-hook=false
```

```java
if (this.registerShutdownHook) {
    try {
       context.registerShutdownHook();
    }
    catch (AccessControlException ex) {
       // Not allowed in some environments.
    }
}

@Override
public void registerShutdownHook() {
    if (this.shutdownHook == null) {
       // No shutdown hook registered yet.
       this.shutdownHook = new Thread(SHUTDOWN_HOOK_THREAD_NAME) {
          @Override
          public void run() {
             synchronized (startupShutdownMonitor) {
                doClose();
             }
          }
       };
       Runtime.getRuntime().addShutdownHook(this.shutdownHook);
    }
}
/**
    其内部销毁了所有的bean（调用了bean的destory方法）（实现了接口 DisposableBean）
    其内部关闭了了所有的beanFactory
**/
protected void doClose() {
    // Check whether an actual close attempt is necessary...
    if (this.active.get() && this.closed.compareAndSet(false, true)) {
       if (logger.isDebugEnabled()) {
          logger.debug("Closing " + this);
       }

       LiveBeansView.unregisterApplicationContext(this);

       try {
          // Publish shutdown event.
          publishEvent(new ContextClosedEvent(this));
       }
       catch (Throwable ex) {
          logger.warn("Exception thrown from ApplicationListener handling ContextClosedEvent", ex);
       }

       // Stop all Lifecycle beans, to avoid delays during individual destruction.
       if (this.lifecycleProcessor != null) {
          try {
             this.lifecycleProcessor.onClose();
          }
          catch (Throwable ex) {
             logger.warn("Exception thrown from LifecycleProcessor on context close", ex);
          }
       }

       // Destroy all cached singletons in the context's BeanFactory.
       destroyBeans();

       // Close the state of this context itself.
       closeBeanFactory();

       // Let subclasses do some final clean-up if they wish...
       onClose();

       // Reset local application listeners to pre-refresh state.
       if (this.earlyApplicationListeners != null) {
          this.applicationListeners.clear();
          this.applicationListeners.addAll(this.earlyApplicationListeners);
       }

       // Switch to inactive.
       this.active.set(false);
    }
}
```

#### 测试是否开启程序关闭钩子线程的效果

- 写一个测试Bean
```Java
public class TestDestroyBean implements DisposableBean {

    @Override
    public void destroy() throws Exception {
        System.out.println("容器关闭，注销该bean");
    }
}
```
- 在启动类中定义该bean
```Java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)// 默认就是单例的
public TestDestroyBean testDestroyBean() {
    return new TestDestroyBean();
}
```
- 配置
  - 其实就是默认配置
```Java
spring.main.lazy-initialization=false # 一定不能使用懒加载，因为懒加载只有使用bean时才会创建bean，而在这个测试里面bean并没有被使用就直接关闭程序了，那样容器中就没有该bean，所以不要使用懒加载，就能保证容器中一定有该bean
spring.main.register-shutdown-hook=true
```
- spring.main.register-shutdown-hook=false时结果
  - 使用`kill -15 pid`关闭，直接使用idea关闭不行
```Java
2023-08-27 14:39:31.850  INFO 84860 --- [           main] com.qiha.testquartz.Main                 : Started Main in 3.169 seconds (JVM running for 3.859)

> Task :test-spring-boot-start:Main.main() FAILED
```
- spring.main.register-shutdown-hook=true时结果
  - 使用`kill -15 pid`关闭，直接使用idea关闭不行
```Java
2023-08-27 14:44:22.324  INFO 84990 --- [           main] com.qiha.testquartz.Main                 : Started Main in 3.388 seconds (JVM running for 3.959)
2023-08-27 14:44:32.820  INFO 84990 --- [extShutdownHook] o.s.s.concurrent.ThreadPoolTaskExecutor  : Shutting down ExecutorService 'applicationTaskExecutor'
容器关闭，注销该bean

> Task :test-spring-boot-start:Main.main() FAILED
```

### 容器刷新前的准备工作

#### 源码

```Java
protected void prepareRefresh() {
    // Switch to active.
    this.startupDate = System.currentTimeMillis();
    this.closed.set(false);
    this.active.set(true);

    if (logger.isDebugEnabled()) {
       if (logger.isTraceEnabled()) {
          logger.trace("Refreshing " + this);
       }
       else {
          logger.debug("Refreshing " + getDisplayName());
       }
    }

    // Initialize any placeholder property sources in the context environment.
    /**
        该方法是空的，可以在子类中重写，以对配置做一些修改
        比如添加必填参数
        比如可以加载配置中心的配置
        可以见3.8.3.2
    **/
    initPropertySources();

    // Validate that all properties marked as required are resolvable:
    // see ConfigurablePropertyResolver#setRequiredProperties
    /**
        校验必填参数是否都配置了
        默认的必填参数是空的，可以通过重写initPropertySources方法添加必填参数
        见3.8.3.2
    **/
    getEnvironment().validateRequiredProperties();

    // Store pre-refresh ApplicationListeners...
    if (this.earlyApplicationListeners == null) {
       this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    }
    else {
       // Reset local application listeners to pre-refresh state.
       this.applicationListeners.clear();
       this.applicationListeners.addAll(this.earlyApplicationListeners);
    }

    // Allow for the collection of early ApplicationEvents,
    // to be published once the multicaster is available...
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### 校验是否必填的参数都已经填写了

- 这个必填参数存储的environment对象的`propertyResolver`属性的`requiredProperties`属性中，默认是空的。
- 如果希望设置必填参数就需要自行实现一个ApplicationContext，重写它的initPropertySources方法，可以在该方法中设置必填参数
  - **由此可见，这个initPropertySources方法也是一个可扩展点，可以在这个方法中设置一些参数值，比如可以加载配置中心的配置**
```Java
public class TestApplicationContext extends AnnotationConfigServletWebApplicationContext {
    @Override
    protected void initPropertySources() {
        super.getEnvironment().setRequiredProperties("mysql", "redis", "mongo");
    }
}
```
- 在启动类中设置ApplicationContext类型
```Java
public class TestApplicationContext extends AnnotationConfigServletWebServerApplicationContext {
    @Override
    protected void initPropertySources() {
        // Map<String, Object> map = new HashMap<>();
        // map.put("mysql", "mysql");
        // map.put("redis", "redis");
        // map.put("mongo", "mongo");
        // super.getEnvironment().getPropertySources().addLast(new MapPropertySource("mustq", map));
        super.getEnvironment().setRequiredProperties("mysql", "redis", "mongo");
    }
}
```
- 启动结果（启动失败了）
```Java
org.springframework.core.env.MissingRequiredPropertiesException: The following properties were declared as required but could not be resolved: [mysql, redis, mongo]
```
- 去掉4-8行代码的注释，或者在配置文件中配置上这三个参数（启动成功了）
```Java
mysql=mysql
redis=redis
mongo=mongo
```

### 获取beanFactory（DefaultListableBeanFactory）

```Java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    refreshBeanFactory();
    return getBeanFactory();
}

@Override
protected final void refreshBeanFactory() throws IllegalStateException {
    if (!this.refreshed.compareAndSet(false, true)) {
       throw new IllegalStateException(
             "GenericApplicationContext does not support multiple refresh attempts: just call 'refresh' once");
    }
    this.beanFactory.setSerializationId(getId());
}


@Override
public final ConfigurableListableBeanFactory getBeanFactory() {
    return this.beanFactory;
}
```

### 完善beanFactory

#### 源码

```Java
/**
 * Configure the factory's standard context characteristics,
 * such as the context's ClassLoader and post-processors.
 * @param beanFactory the BeanFactory to configure
 */
protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // Tell the internal bean factory to use the context's class loader etc.
    // 设置类加载器
    beanFactory.setBeanClassLoader(getClassLoader());
    // 设置el表达式解析器
    beanFactory.setBeanExpressionResolver(new StandardBeanExpressionResolver(beanFactory.getBeanClassLoader()));
    beanFactory.addPropertyEditorRegistrar(new ResourceEditorRegistrar(this, getEnvironment()));

    // Configure the bean factory with context callbacks.
    // 添加后置处理器，处理aware接口
    beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(EnvironmentAware.class);
    beanFactory.ignoreDependencyInterface(EmbeddedValueResolverAware.class);
    beanFactory.ignoreDependencyInterface(ResourceLoaderAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationEventPublisherAware.class);
    beanFactory.ignoreDependencyInterface(MessageSourceAware.class);
    beanFactory.ignoreDependencyInterface(ApplicationContextAware.class);

    // BeanFactory interface not registered as resolvable type in a plain factory.
    // MessageSource registered (and found for autowiring) as a bean.
    beanFactory.registerResolvableDependency(BeanFactory.class, beanFactory);
    beanFactory.registerResolvableDependency(ResourceLoader.class, this);
    beanFactory.registerResolvableDependency(ApplicationEventPublisher.class, this);
    beanFactory.registerResolvableDependency(ApplicationContext.class, this);

    // Register early post-processor for detecting inner beans as ApplicationListeners.
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(this));

    // Detect a LoadTimeWeaver and prepare for weaving, if found.
    if (beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
       beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
       // Set a temporary ClassLoader for type matching.
       beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }

    // Register default environment beans.
    // 将各种环境environment、systemProperties、systemEnvironment加载到beanFactory的容器中singletonObjects中
    if (!beanFactory.containsLocalBean(ENVIRONMENT_BEAN_NAME)) {
       beanFactory.registerSingleton(ENVIRONMENT_BEAN_NAME, getEnvironment());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_PROPERTIES_BEAN_NAME)) {
       beanFactory.registerSingleton(SYSTEM_PROPERTIES_BEAN_NAME, getEnvironment().getSystemProperties());
    }
    if (!beanFactory.containsLocalBean(SYSTEM_ENVIRONMENT_BEAN_NAME)) {
       beanFactory.registerSingleton(SYSTEM_ENVIRONMENT_BEAN_NAME, getEnvironment().getSystemEnvironment());
    }
}
```

#### 自定义一个后置处理器BeanPostProcessor

- Aware系列的接口，比如ApplicationContextAware、EnvironmentAware等语义的实现原理都是基于BeanPostProcessor实现的
- **通过bean的后置处理器可以实现动态代理**
  - **比如在后置处理器中处理注解，基于注解实现bean的动态代理，见3.8.5.3**
    - 比如AOP、Mybatis的Mapper实现
- 自定义一个接口`TestStringAware`
  - 后置处理器会处理实现了这个接口的bean，为其注入testString
```Java
public interface TestStringAware extends Aware {
    void setTestString(String testString);
}
```
- 实现接口`TestStringAware`
```Java
public class TestDestroyBean implements TestStringAware {

    private String testString;

    @Override
    public void setTestString(String testString) {
        this.testString = testString;
    }
}
```
- 定义实现该接口语义的后置处理器
  - 该后置处理器会给所有实现了接口`TestStringAware`的bean注入属性`testString`
```Java
@Component
public class TestStringAwareProcessor implements BeanPostProcessor {

    @Value("${test.string}")
    private String testString;

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof TestStringAware) {
            ((TestStringAware) bean).setTestString(testString);
            System.out.println("已经通过TestStringAwareProcessor为" + beanName + "设置了属性testString:" + testString);
        }
        return bean;
    }
}
```
- 测试结果
```Plain
已经通过TestStringAwareProcessor为testDestroyBean设置了属性testString:test1
```

#### 基于BeanPostProcessor实现动态代理-代理对象替换被代理对象

- 定义一个注解，用于标识是否为方法输出日志
```Java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Log {

}
```
- 定义一个后置处理器，处理该注解
```Java
@Component
public class LogProxyProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        Method[] methods = bean.getClass().getDeclaredMethods();
        boolean hasLog = false;
        for (Method method : methods) {
            if (method.isAnnotationPresent(Log.class)) {
                hasLog = true;
            }
            if (hasLog) {
                break;
            }
        }
        // 没有Log注解的类不用生成代理类
        if (!hasLog) {
            return bean;
        }
        // 具有Log代理的类才需要生成代理类
        Enhancer enhancer = new Enhancer();
        enhancer.setCallbackFilter(new CallbackFilter() {
            @Override
            public int accept(Method method) {
                // 1就是过滤，0就是不过滤
                // 被过滤掉的方法不走代理
                // 没有过滤的方法使用代理处理
                // 被Log注解的方法走代理
                return method.isAnnotationPresent(Log.class) ? 0 : 1;
            }
        });//filter要比callbacks先设置
        enhancer.setSuperclass(bean.getClass());
        enhancer.setCallbackTypes(new Class[]{LogMethodInterceptor.class, NoOp.class});
        Class<?> subclass = enhancer.createClass();
        Enhancer.registerCallbacks(subclass, new Callback[]{
                new LogMethodInterceptor(), NoOp.INSTANCE
        });
        System.out.println("=======");
        System.out.println(beanName);
        try {
            // 返回代理类，替换原来的方法
            return subclass.newInstance();
        } catch (InstantiationException | IllegalAccessException e) {
            throw new RuntimeException(e);
        }
    }

    // 代理
    static class LogMethodInterceptor implements MethodInterceptor {
        @Override
        public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
            Object result = null;
            // if (method.isAnnotationPresent(Log.class)) {
                System.out.println("开始调用该方法");
                // 调用被代理类的方法
                // 因为CGLIG是基于继承实现的动态代理
                // 直接调用父类的方法就是调用被代理类的方法
                result = methodProxy.invokeSuper(o, args);
                System.out.println("调用该方法结束");
            // } else {
            //     result = methodProxy.invokeSuper(o, args);
            // }
            return result;
        }
    }
}
```
- 测试类Service
```Java
@Service
public class TestService {
    @Log
    public void logSout() {
        System.out.println("logSout");
    }

    public void sout() {
        System.out.println("sout");
    }
}
```
- 测试结果
  - 可以看到被Log注解的方法被代理了，输出了日志
    - 代理对象在被代理对象的方法两端加了输出，并且在中间调用了被代理对象的方法
  - 没有被Log注解的方法，只是直接在代理对象中调用了被代理对象的方法
```Plain
开始调用该方法
logSout
调用该方法结束
开始调用该方法
logSout
调用该方法结束
开始调用该方法
logSout
调用该方法结束
开始调用该方法
logSout
调用该方法结束
sout
sout
sout
sout
```

### postProcessBeanFactory

```Java
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    super.postProcessBeanFactory(beanFactory);
    if (this.basePackages != null && this.basePackages.length > 0) {
       this.scanner.scan(this.basePackages);
    }
    if (!this.annotatedClasses.isEmpty()) {
       this.reader.register(ClassUtils.toClassArray(this.annotatedClasses));
    }
}

// super.postProcessBeanFactory(beanFactory);
@Override
protected void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) {
    // 注册了一个WebApplicationContextServletContextAwareProcessor bean后置处理器
    // 用来处理ServletContextAware接口，如果bean实现了这个接口，spring容器会给他注入ServletContext
    beanFactory.addBeanPostProcessor(new WebApplicationContextServletContextAwareProcessor(this));
    beanFactory.ignoreDependencyInterface(ServletContextAware.class);
    // 注册scope,两个scope（bean的作用范围），request，session范围 bean的作用范围，
    registerWebApplicationScopes();
}

private void registerWebApplicationScopes() {
    ExistingWebApplicationScopes existingScopes = new ExistingWebApplicationScopes(getBeanFactory());
    WebApplicationContextUtils.registerWebApplicationScopes(getBeanFactory());
    existingScopes.restore();
}

beanFactory.registerScope(WebApplicationContext.SCOPE_REQUEST, new RequestScope());
beanFactory.registerScope(WebApplicationContext.SCOPE_SESSION, new SessionScope());
```

### 执行BeanFactory的后置处理器

#### 源码-invokeBeanFactoryPostProcessors

```Java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 调用BeanFactory的后置处理器
    // 在WebServlet类型的应用中，默认有三个BeanFactory后置处理器
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
    // Detect a LoadTimeWeaver and prepare for weaving, if found in the meantime
    // (e.g. through an @Bean method registered by ConfigurationClassPostProcessor)
    if (beanFactory.getTempClassLoader() == null && beanFactory.containsBean(LOAD_TIME_WEAVER_BEAN_NAME)) {
       beanFactory.addBeanPostProcessor(new LoadTimeWeaverAwareProcessor(beanFactory));
       beanFactory.setTempClassLoader(new ContextTypeMatchClassLoader(beanFactory.getBeanClassLoader()));
    }
}

//     PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, getBeanFactoryPostProcessors());
public static void invokeBeanFactoryPostProcessors(
       ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

    // Invoke BeanDefinitionRegistryPostProcessors first, if any.
    Set<String> processedBeans = new HashSet<>();

    if (beanFactory instanceof BeanDefinitionRegistry) {
       BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
       // 存储普通类型的后置处理器（非BeanDefinitionRegistryPostProcessor类型的）
       List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
       // 存储BeanDefinitionRegistryPostProcessor类型的后置处理器
       List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();
        
       // 遍历后置处理器 
       for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
          // 如果是BeanDefinitionRegistryPostProcessor类型的后置处理器就调用，并且加到registryProcessors列表中
          if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
             BeanDefinitionRegistryPostProcessor registryProcessor =
                   (BeanDefinitionRegistryPostProcessor) postProcessor;
             // 并且调用BeanDefinitionRegistryPostProcessor后置处理器，用来注册bean，进入容器中
             // 这里把容器传入进去了，用来接收被解析出来的bean
             registryProcessor.postProcessBeanDefinitionRegistry(registry);
             registryProcessors.add(registryProcessor);
          }
          // 如果是普通类型的后置处理器，直接加入列表即可
          else {
             regularPostProcessors.add(postProcessor);
          }
       }

       // Do not initialize FactoryBeans here: We need to leave all regular beans
       // uninitialized to let the bean factory post-processors apply to them!
       // Separate between BeanDefinitionRegistryPostProcessors that implement
       // PriorityOrdered, Ordered, and the rest.
       List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

       // First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
       // 因为上面已经调用了所有的BeanDefinitionRegistryPostProcessor类型的后置处理器，所有容器中新增了很多的bean
       而且这些bean里面可能存在BeanDefinitionRegistryPostProcessor类型的bean，对于这种bean，需要再次调用，以注册新的bean
       String[] postProcessorNames =
             beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
       // 这里处理的是实现了PriorityOrdered接口的BeanDefinitionRegistryPostProcessor，因为他的调用顺序高点
       for (String ppName : postProcessorNames) {
          if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
             currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
             processedBeans.add(ppName);
          }
       }
       sortPostProcessors(currentRegistryProcessors, beanFactory);
       registryProcessors.addAll(currentRegistryProcessors);
       // 调用新注册的BeanDefinitionRegistryPostProcessor后置处理器
       invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
       currentRegistryProcessors.clear();

       // Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
       // 这里处理的是实现了Ordered接口的BeanDefinitionRegistryPostProcessor
       postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
       for (String ppName : postProcessorNames) {
          if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
             currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
             processedBeans.add(ppName);
          }
       }
       sortPostProcessors(currentRegistryProcessors, beanFactory);
       registryProcessors.addAll(currentRegistryProcessors);
       invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
       currentRegistryProcessors.clear();

       // Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
       // 调用完一次之后，又加载了新的bean，这个bean里面可能又存在BeanDefinitionRegistryPostProcessor类型的bean
       // 通过循环来处理，
       // 如果一次循环加载的bean中出现了新的BeanDefinitionRegistryPostProcessor.class类型的bean，那就说明需要进入下一轮循环
       // 如果一次循环加载出来的bean中不存在新的BeanDefinitionRegistryPostProcessor.class类型的bean，那就说明循环结束，不需要进入下一轮循环
       boolean reiterate = true;
       while (reiterate) {
          reiterate = false;
          postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
          for (String ppName : postProcessorNames) {
             if (!processedBeans.contains(ppName)) {
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
                reiterate = true;
             }
          }
          sortPostProcessors(currentRegistryProcessors, beanFactory);
          registryProcessors.addAll(currentRegistryProcessors);
          invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
          currentRegistryProcessors.clear();
       }

       // Now, invoke the postProcessBeanFactory callback of all processors handled so far.
       // 最后一轮循环结束的时候，还存在部分BeanDefinitionRegistryPostProcessor.class类型的后置处理器没有调用，最后调用一下
       invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
       // 调用所有没有执行的非BeanDefinitionRegistryPostProcessor.class类型的后置处理器
       invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
    }

    else {
       // Invoke factory processors registered with the context instance.
       invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
    }
   
    // 上面已经调用了所有的BeanDefinitionRegistryPostProcessor类型的后置处理器（包括applicationContext中带有的，以及通过BeanDefinitionRegistryPostProcessor解析出来的）
    // 此外还调用了所有的applicationContext中带有的非BeanDefinitionRegistryPostProcessor类型的后置处理器
    // 但是还没有执行通过BeanDefinitionRegistryPostProcessor后置处理器加载的非BeanDefinitionRegistryPostProcessor类型的后置处理器
    // 下面的逻辑就是调用通过BeanDefinitionRegistryPostProcessor类型后置处理器加载出来的非BeanDefinitionRegistryPostProcessor类型的后置处理器
    
    
    // 为什么要到这后面才调用通过BeanDefinitionRegistryPostProcessor类型后置处理器加载出来的非BeanDefinitionRegistryPostProcessor类型的后置处理器？而不是在上面每一次调用
    // BeanDefinitionRegistryPostProcessor类型后置处理器后，直接调用呢？
    // 这是因为需要对所有的非BeanDefinitionRegistryPostProcessor类型的后置处理器进行一个全局的排序
    // 但是invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);中已经调用了applicationContext中非BeanDefinitionRegistryPostProcessor类型的后置处理器，所以实际上这个排序只是针对后续加载出来的
    // 在调用顺序上，applicationContext的内的后置处理器一定比后续加载出来的高
    
    
    // applicationContext中自带的beanFactory后置处理器是无法通过beanFactory.getBeanNamesForType获取的
    // Do not initialize FactoryBeans here: We need to leave all regular beans
    // uninitialized to let the bean factory post-processors apply to them!
    String[] postProcessorNames =
          beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

    // Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
    // Ordered, and the rest.
    // 将BeanDefinitionRegistryPostProcessor加载出来的全部BeanFactoryPostProcessor类型的bean分为
    // 3类，并且过滤掉已经调用过的后置处理器（即BeanDefinitionRegistryPostProcessor类型的后置处理器）
    // 三类：实现了PriorityOrdered.class接口的、实现了Ordered.class接口的、都没有实现的
    List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    for (String ppName : postProcessorNames) {
       if (processedBeans.contains(ppName)) {
          // skip - already processed in first phase above
       }
       else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
          priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
       }
       else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
          orderedPostProcessorNames.add(ppName);
       }
       else {
          nonOrderedPostProcessorNames.add(ppName);
       }
    }
    // 先调用PriorityOrdered.class类型的
    // First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);
     // 再调用Ordered类型的
    // Next, invoke the BeanFactoryPostProcessors that implement Ordered.
    List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String postProcessorName : orderedPostProcessorNames) {
       orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    sortPostProcessors(orderedPostProcessors, beanFactory);
    invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

    // 最后调用不排序的
    // Finally, invoke all other BeanFactoryPostProcessors.
    List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String postProcessorName : nonOrderedPostProcessorNames) {
       nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
    }
    invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

    // Clear cached merged bean definitions since the post-processors might have
    // modified the original metadata, e.g. replacing placeholders in values...
    beanFactory.clearMetadataCache();
}
```

#### 自定义一个简单的BeanFactoryPostProcessor后置处理器

- 写一个实体类
```Java
@ToString
@Data
public class Student {

    private String name;
    private int age;

}
```

- 自定义一个beanFactory后置处理器，注册Student
```Java
package com.qiha.start.beanfactorypostprocessor;

import com.qiha.start.entity.Student;
import org.springframework.beans.BeanMetadataAttribute;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.beans.factory.config.Scope;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor;
import org.springframework.beans.factory.support.RootBeanDefinition;
import org.springframework.stereotype.Component;

@Component
public class StudentBeanFactoryPostProcessor implements BeanDefinitionRegistryPostProcessor {

    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        /** 虽然能得到bean，但是bean的属性值age=0 name=null
         RootBeanDefinition beanDefinition = new RootBeanDefinition();
         beanDefinition.setBeanClass(Student.class);
         beanDefinition.addMetadataAttribute(new BeanMetadataAttribute("name","zhangsan"));
         beanDefinition.setAttribute("age", 20);
         beanDefinition.setSource("test");
         beanDefinition.setScope(BeanDefinition.SCOPE_SINGLETON);
         registry.registerBeanDefinition("student", beanDefinition);
         **/

        BeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Student.class)
                .addPropertyValue("name", "zhangsan")
                // .addPropertyReference("dao","daoBeanName") 注入另一个bean
                .addPropertyValue("age", "20")
                .setInitMethodName("init")
                .setDestroyMethodName("destroy")
                .setScope(BeanDefinition.SCOPE_SINGLETON)
                .getBeanDefinition();
        registry.registerBeanDefinition("student", beanDefinition);
        System.out.println("完成注册student的BeanDefinition");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        Integer count = beanFactory.getBeanDefinitionCount();
        System.out.println("当前一共存在" + count + "个BeanDefinition");
    }
}
```
- 测试
```Java
package com.qiha.start;

import com.qiha.start.entity.Student;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class) // 在主类的包路径下
@SpringBootTest
public class TestStudent {

    @Autowired(required = true)
    private Student student;

    @Test
    public void testStudent() {
        System.out.println(student);
    }

}
```
- 输出
```Java
完成注册student的BeanDefinition
当前一共存在138个BeanDefinition
初始化Student
Student(name=zhangsan, age=20)
销毁Student
```

### 注册Bean的后置处理器

#### 源码

```java
protected void registerBeanPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.registerBeanPostProcessors(beanFactory, this);
}

public static void registerBeanPostProcessors(
       ConfigurableListableBeanFactory beanFactory, AbstractApplicationContext applicationContext) {
    // 在beanFactory中获取全部的类型为BeanPostProcessor的bean的名称
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanPostProcessor.class, true, false);

    // Register BeanPostProcessorChecker that logs an info message when
    // a bean is created during BeanPostProcessor instantiation, i.e. when
    // a bean is not eligible for getting processed by all BeanPostProcessors.
    // 注册BeanPostProcessorChecker，当在BeanPostProcessor实例化期间创建bean时，即当bean不适合由所有BeanPostProcessors处理时，该Checker会记录信息消息。
    int beanProcessorTargetCount = beanFactory.getBeanPostProcessorCount() + 1 + postProcessorNames.length;
    beanFactory.addBeanPostProcessor(new BeanPostProcessorChecker(beanFactory, beanProcessorTargetCount));

    // Separate between BeanPostProcessors that implement PriorityOrdered,
    // Ordered, and the rest.
    // 对bean进行分组 实现了PriorityOrdered接口、Ordered接口、上述两个接口都没有实现的
    List<BeanPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
    List<BeanPostProcessor> internalPostProcessors = new ArrayList<>();
    List<String> orderedPostProcessorNames = new ArrayList<>();
    List<String> nonOrderedPostProcessorNames = new ArrayList<>();
    // 遍历所有的postProcessorNames
    for (String ppName : postProcessorNames) {
       // 如果bean后置处理器实现了PriorityOrdered接口，就放到priorityOrderedPostProcessors中
       if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
          BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
          priorityOrderedPostProcessors.add(pp);
          if (pp instanceof MergedBeanDefinitionPostProcessor) {
             internalPostProcessors.add(pp);
          }
       }
       // 如果实现了Ordered接口，就放到Ordered，就放到orderedPostProcessorNames中
       else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
          orderedPostProcessorNames.add(ppName);
       }
       // 如果上述两个接口都没有实现的，就放到nonOrderedPostProcessorNames中
       else {
          nonOrderedPostProcessorNames.add(ppName);
       }
    }

    // First, register the BeanPostProcessors that implement PriorityOrdered.
    // 对实现了PriorityOrdered接口的后置处理器进行排序，然后按序注册到beanFactory中
    sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, priorityOrderedPostProcessors);

    // Next, register the BeanPostProcessors that implement Ordered.
    List<BeanPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
    for (String ppName : orderedPostProcessorNames) {
       BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
       orderedPostProcessors.add(pp);
       if (pp instanceof MergedBeanDefinitionPostProcessor) {
          internalPostProcessors.add(pp);
       }
    }
    // 对实现了Ordered接口的后置处理器进行排序，然后按序注册到beanFactory中
    // 顺序在实现了PriorityOrdered的接口的后置处理器后面
    sortPostProcessors(orderedPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, orderedPostProcessors);

    // Now, register all regular BeanPostProcessors.
    List<BeanPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
    for (String ppName : nonOrderedPostProcessorNames) {
       BeanPostProcessor pp = beanFactory.getBean(ppName, BeanPostProcessor.class);
       nonOrderedPostProcessors.add(pp);
       if (pp instanceof MergedBeanDefinitionPostProcessor) {
          internalPostProcessors.add(pp);
       }
    }
    // 注册没有实现PriorityOrdered和Ordered接口的后置处理器
    registerBeanPostProcessors(beanFactory, nonOrderedPostProcessors);

    // Finally, re-register all internal BeanPostProcessors.
    // 将实现了MergedBeanDefinitionPostProcessor的后置处理器重新加入
    // 在加入的时候会先把存在的删除，然后在加入，实际上就是把实现了MergedBeanDefinitionPostProcessor的后置处理器放到队列的最后面，优先级最低
    sortPostProcessors(internalPostProcessors, beanFactory);
    registerBeanPostProcessors(beanFactory, internalPostProcessors);

    // Re-register post-processor for detecting inner beans as ApplicationListeners,
    // moving it to the end of the processor chain (for picking up proxies etc).
    // 在bean后置处理器队列的最后面再次加上一个ApplicationListenerDetector，用于检测ApplicationListener
    // 这里是为了预防前面的bean后置处理器通过代理的方式实现了ApplicationListener,而没有被检测到，所以在最后面再次添加一个ApplicationListenerDetector！
    // 根据ApplicationListenerDetector的equals方法可以发现（只要applicationContext相等，就认为ApplicationListenerDetector是equals的），在前面3.8.5完善beanFactory那一步中增加的ApplicationListenerDetector被移除掉了，这里又添加了一个新的
    // 目的还是为了能够让ApplicationListenerDetector在后置处理器队列的最后面！！！！
    beanFactory.addBeanPostProcessor(new ApplicationListenerDetector(applicationContext));
    
    
    /**
        ApplicationListenerDetector.eqauls()方法
        @Override
        public boolean equals(@Nullable Object other) {
            return (this == other || (other instanceof ApplicationListenerDetector &&
                  this.applicationContext == ((ApplicationListenerDetector) other).applicationContext));
        }
    **/
}
```

#### bean后置处理器总结

可以通过实现`Ordered`和`PriorityOrdered`接口来调整`BeanPostProcessor`的优先级

- `PriorityOrdered`接口的优先级高于`Ordered`接口
- 数值越低，优先级越高
- `MergedBeanDefinitionPostProcessor`类型后置处理器的优先级低于普通的后置处理器优先级
- 实现5个后置处理器
```Java
@Component
public class OneProcessor implements BeanPostProcessor, Ordered {

    @Override
    public int getOrder() {
        return 0;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!"testService".equals(beanName)){
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
        System.out.println("Ordered 优先级0");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}


@Component
public class TwoProcessor implements BeanPostProcessor, Ordered {

    @Override
    public int getOrder() {
        return 1;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!"testService".equals(beanName)) {
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
        System.out.println("Ordered 优先级1");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}

@Component
public class ThreeProcessor implements BeanPostProcessor, PriorityOrdered {

    @Override
    public int getOrder() {
        return 2;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!"testService".equals(beanName)){
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
        System.out.println("PriorityOrdered 优先级2");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}



@Component
public class FourProcessor implements BeanPostProcessor, PriorityOrdered {

    @Override
    public int getOrder() {
        return 3;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!"testService".equals(beanName)){
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
        System.out.println("PriorityOrdered 优先级3");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}


@Component
public class FiveProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (!"testService".equals(beanName)){
            return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
        }
        System.out.println("没有实现Ordered和PriorityOrdered接口");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```
- 输出结果
```Plain
PriorityOrdered 优先级2
PriorityOrdered 优先级3
Ordered 优先级0
Ordered 优先级1
没有实现Ordered和PriorityOrdered接口
```







# 各对象分析

## ApplicationContextInitializer.class对象

### 作用

ApplicationContextInitializer是Spring给我们提供的一个在容器刷新之前对容器进行初始化操作的能力，如果有具体的需要在容器刷新之前处理的业务，可以通过ApplicationContextInitializer接口来实现

##  ApplicationListener.class对象

### 作用

命名我们就可以知道它是一个监听者，[分析springboot启动流程](https://link.juejin.cn?target=https%3A%2F%2Fyidajava.blog.csdn.net%2Farticle%2Fdetails%2F110312421)我们会发现，它其实是用来在整个启动流程中接收不同执行点事件通知的监听者，SpringApplicationRunListener接口规定了SpringBoot的生命周期，在各个生命周期广播相应的事件，调用实际的ApplicationListener类

springboot提供了两个类SpringApplicationRunListeners、SpringApplicationRunListener（EventPublishingRunListener），spring框架还提供了一个ApplicationListener接口

那么这几个类或接口的关系又是如何呢？首先SpringApplicationRunListeners类是SpringApplicationRunListener接口的代理类，可以批量调用SpringApplicationRunListener接口方法，SpringApplicationRunListener接口只有一个实现类EventPublishingRunListener，其有一个属性SimpleApplicationEventMulticaster，SimpleApplicationEventMulticaster即是一个ApplicationListener监听器接口的代理实现类，可以批量的执行监听器的onApplicationEvent方法

调用顺序：SpringApplicationRunListeners-->SpringApplicationRunListener-->EventPublishingRunListener-->SimpleApplicationEventMulticaster

### SpringApplicationRunListeners

```java
class SpringApplicationRunListeners {
    private final Log log;
    private final List<SpringApplicationRunListener> listeners;
    private final ApplicationStartup applicationStartup;

    SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners, ApplicationStartup applicationStartup) {
        this.log = log;
        this.listeners = new ArrayList(listeners);
        this.applicationStartup = applicationStartup;
    }

    void starting(ConfigurableBootstrapContext bootstrapContext, Class<?> mainApplicationClass) {
        this.doWithListeners("spring.boot.application.starting", (listener) -> {
            listener.starting(bootstrapContext);
        }, (step) -> {
            if (mainApplicationClass != null) {
                step.tag("mainApplicationClass", mainApplicationClass.getName());
            }

        });
    }

    void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        this.doWithListeners("spring.boot.application.environment-prepared", (listener) -> {
            listener.environmentPrepared(bootstrapContext, environment);
        });
    }
}
```

### SpringApplicationRunListener

SpringApplicationRunListener是对org.springframework.boot.SpringApplication类的run方法进行监听，SpringApplicationRunListener实现类是通过SpringFactoriesLoader类加载（即springboot SPI）;并且需要声明一个包含SpringApplication实例及String[]的参数构造方法

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

package org.springframework.boot;

import java.time.Duration;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;

public interface SpringApplicationRunListener {
    default void starting(ConfigurableBootstrapContext bootstrapContext) {
    }

    default void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
    }

    default void contextPrepared(ConfigurableApplicationContext context) {
    }

    default void contextLoaded(ConfigurableApplicationContext context) {
    }

    default void started(ConfigurableApplicationContext context, Duration timeTaken) {
        this.started(context);
    }

    /** @deprecated */
    @Deprecated
    default void started(ConfigurableApplicationContext context) {
    }

    default void ready(ConfigurableApplicationContext context, Duration timeTaken) {
        this.running(context);
    }

    /** @deprecated */
    @Deprecated
    default void running(ConfigurableApplicationContext context) {
    }

    default void failed(ConfigurableApplicationContext context, Throwable exception) {
    }
}
```

### EventPublishingRunListener

```java
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {
    private final SpringApplication application;
    private final String[] args;
    private final SimpleApplicationEventMulticaster initialMulticaster;  

public void starting(ConfigurableBootstrapContext bootstrapContext) {
        this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(bootstrapContext, this.application, this.args));
    }

    public void environmentPrepared(ConfigurableBootstrapContext bootstrapContext, ConfigurableEnvironment environment) {
        this.initialMulticaster.multicastEvent(new ApplicationEnvironmentPreparedEvent(bootstrapContext, this.application, this.args, environment));
    }
}
```

### SimpleApplicationEventMulticaster

```java
public class SimpleApplicationEventMulticaster extends AbstractApplicationEventMulticaster {
    @Nullable
    private Executor taskExecutor;
    @Nullable
    private ErrorHandler errorHandler;
    @Nullable
    private volatile Log lazyLogger;
 public void multicastEvent(ApplicationEvent event) {
        this.multicastEvent(event, this.resolveDefaultEventType(event));
    }

    public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
        ResolvableType type = eventType != null ? eventType : this.resolveDefaultEventType(event);
        Executor executor = this.getTaskExecutor();
        Iterator var5 = this.getApplicationListeners(event, type).iterator();

        while(var5.hasNext()) {
            ApplicationListener<?> listener = (ApplicationListener)var5.next();
            if (executor != null) {
                executor.execute(() -> {
                    this.invokeListener(listener, event);
                });
            } else {
                this.invokeListener(listener, event);
            }
        }

    }
}
```

## **BeanFactoryPostProcessor和BeanDefinitionRegistryPostProcessor**

**Spring容器中主要的4个阶段**

- 阶段1：Bean注册阶段，此阶段会完成所有bean的注册
- 阶段2：BeanFactory后置处理阶段
- 阶段3：注册BeanPostProcessor
- 阶段4：bean创建阶段，此阶段完成所有单例bean的注册和装载操作

### Bean注册阶段

spring中所有bean的注册都会在此阶段完成，按照规范，所有bean的注册必须在此阶段进行，其他阶段不要再进行bean的注册

这个阶段spring为我们提供1个接口：BeanDefinitionRegistryPostProcessor，spring容器在这个阶段中会获取容器中所有类型为`BeanDefinitionRegistryPostProcessor`的bean，然后会调用他们的`postProcessBeanDefinitionRegistry`方法，源码如下，方法参数类型是`BeanDefinitionRegistry`，这个类型大家都比较熟悉，即bean定义注册器，内部提供了一些方法可以用来向容器中注册bean

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {
    void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;
}
```

这个接口还继承了`BeanFactoryPostProcessor`接口

当容器中有多个`BeanDefinitionRegistryPostProcessor`的时候，可以通过下面任意一种方式来指定顺序

- 实现`org.springframework.core.PriorityOrdered`接口
- 实现`org.springframework.core.Ordered`接口

**案例-简单实用**

此案例演示`BeanDefinitionRegistryPostProcessor`的简单使用

**自定义一个BeanDefinitionRegistryPostProcessor**

下面我们定义了一个类，需要实现`BeanDefinitionRegistryPostProcessor`接口，然后会让我们实现2个方法，大家重点关注`postProcessBeanDefinitionRegistry`这个方法，另外一个方法来自于`BeanFactoryPostProcessor`，一会我们后面在介绍这个方法，在`postProcessBeanDefinitionRegistry`方法中，我们定义了一个bean，然后通过`registry`将其注册到容器了，代码很简单

```java
package com.javacode2018.lesson003.demo3.test0;
@Component
public class MyBeanDefinitionRegistryPostProcessor implements BeanDefinitionRegistryPostProcessor {
    @Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
        //定义一个字符串类型的bean
        AbstractBeanDefinition userNameBdf = BeanDefinitionBuilder.
                genericBeanDefinition(String.class).
                addConstructorArgValue("路人").
                getBeanDefinition();
        //将userNameBdf注册到spring容器中
        registry.registerBeanDefinition("userName", userNameBdf);
    }
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
    }
}
```

**同包中来个配置类**

```java
package com.javacode2018.lesson003.demo3.test0;
import org.springframework.context.annotation.ComponentScan;
@ComponentScan
public class MainConfig0 {
}
```

**测试用例**

```java
package com.javacode2018.lesson003.demo3;
import com.javacode2018.lesson003.demo3.test0.MainConfig0;
import com.javacode2018.lesson003.demo3.test1.MainConfig1;
import org.junit.Test;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
public class BeanDefinitionRegistryPostProcessorTest {
    @Test
    public void test0() {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(MainConfig0.class);
        context.refresh();
        System.out.println(context.getBean("userName"));
    }
}
```

**运行输出**

```
路人
```

### BeanFactory后置处理阶段

到这个阶段的时候，spring容器已经完成了所有bean的注册，这个阶段中你可以对BeanFactory中的一些信息进行修改，比如修改阶段1中一些bean的定义信息，修改BeanFactory的一些配置等等，此阶段spring也提供了一个接口来进行扩展：`BeanFactoryPostProcessor`，简称`bfpp`，接口中有个方法`postProcessBeanFactory`，spring会获取容器中所有BeanFactoryPostProcessor类型的bean，然后调用他们的`postProcessBeanFactory`，来看一下这个接口的源码

```java
@FunctionalInterface
public interface BeanFactoryPostProcessor {
    void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
}
```

当容器中有多个`BeanFactoryPostProcessor`的时候，可以通过下面任意一种方式来指定顺序

- 实现`org.springframework.core.PriorityOrdered`接口
- 实现`org.springframework.core.Ordered`接口

**案例-这个案例中演示，在BeanFactoryPostProcessor来修改bean中已经注册的bean定义的信息，给一个bean属性设置一个值**

先来定义一个bean

```java
package com.javacode2018.lesson003.demo3.test2;
import org.springframework.stereotype.Component;
@Component
public class LessonModel {
    //课程名称
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "LessonModel{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

上面这个bean有个name字段，并没有设置值，下面我们在BeanFactoryPostProcessor来对其设置值。

**自定义的BeanFactoryPostProcessor**

下面代码中，我们先获取`lessonModel`这个bean的定义信息，然后给其`name`属性设置了一个值

```java
package com.javacode2018.lesson003.demo3.test2;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("准备修改lessonModel bean定义信息!");
        BeanDefinition beanDefinition = beanFactory.getBeanDefinition("lessonModel");
        beanDefinition.getPropertyValues().add("name", "spring高手系列!");
    }
}
```

**测试用例**

```java
@Test
public void test2() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register(MainConfig2.class);
    context.refresh();
    System.out.println(context.getBean(LessonModel.class));
}
```

**运行输出**

```
准备修改lessonModel bean定义信息!
LessonModel{name='spring高手系列!'}
```

结果中可以看出，通过`BeanFactoryPostProcessor`修改了容器中已经注册的bean定义信息。

#### 这个接口的几个重要实现类

**PropertySourcesPlaceholderConfigurer**

这个接口做什么的，大家知道么？来看一段代码

```xml
<bean class="xxxxx">
    <property name="userName" value="${userName}"/>
    <property name="address" value="${address}"/>
</bean>
```

这个大家比较熟悉吧，spring就是在`PropertySourcesPlaceholderConfigurer#postProcessBeanFactory`中来处理xml中属性中的`${xxx}`，会对这种格式的进行解析处理为真正的值

**CustomScopeConfigurer**

向容器中注册自定义的Scope对象，即注册自定义的作用域实现类

**EventListenerMethodProcessor**

处理`@EventListener`注解的，即spring中事件机制

#### 使用注意

`BeanFactoryPostProcessor`接口的使用有一个需要注意的地方，在其`postProcessBeanFactory`方法中，强烈禁止去通过容器获取其他bean，此时会导致bean的提前初始化，会出现一些意想不到的问题，因为这个阶段中`BeanPostProcessor`还未准备好，本文开头4个阶段中有介绍，`BeanPostProcessor`是在第3个阶段中注册到spring容器的，而`BeanPostProcessor`可以对bean的创建过程进行干预，比如spring中的aop就是在`BeanPostProcessor`的一些子类中实现的，`@Autowired`也是在`BeanPostProcessor`的子类中处理的，此时如果去获取bean，此时bean不会被`BeanPostProcessor`处理，所以创建的bean可能是有问题的，还是通过一个案例给大家演示一下把，通透一些

**来一个简单的类**

```java
package com.javacode2018.lesson003.demo3.test3;
import org.springframework.beans.factory.annotation.Autowired;
public class UserModel {
    @Autowired
    private String name; //@1
    @Override
    public String toString() {
        return "UserModel{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

> @1：使用了@Autowired，会指定注入

**来个配置类**

配置类中定义了2个UserModel类型的bean：user1、user2

并且定义了一个String类型的bean：name，这个会注入到UserModel中的name属性中去

```java
package com.javacode2018.lesson003.demo3.test3;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
@Configuration
@ComponentScan
public class MainConfig3 {
    @Bean
    public UserModel user1() {
        return new UserModel();
    }
    @Bean
    public UserModel user2() {
        return new UserModel();
    }
    @Bean
    public String name() {
        return "路人甲Java,带大家成功java高手!";
    }
}
```

**测试用例**

> 输出容器中所有UserModel类型的bean

```java
@Test
public void test3() {
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
    context.register(MainConfig3.class);
    context.refresh();
    context.getBeansOfType(UserModel.class).forEach((beanName, bean) -> {
        System.out.println(String.format("%s->%s", beanName, bean));
    });
}
```

**运行输出**

```
user1->UserModel{name='路人甲Java,带大家成功java高手!'}
user2->UserModel{name='路人甲Java,带大家成功java高手!'}
```

效果不用多解释，大家一看就懂，下面来重点

**添加一个BeanFactoryPostProcessor**

在`postProcessBeanFactory`方法中去获取一下user1这个bean

```java
package com.javacode2018.lesson003.demo3.test3;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanFactoryPostProcessor;
import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
import org.springframework.stereotype.Component;
@Component
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        beanFactory.getBean("user1");
    }
}
```

**再次运行输出**

```java
user1->UserModel{name='null'}
user2->UserModel{name='路人甲Java,带大家成功java高手!'}
```

注意，user1中的name变成null了，什么情况？

**是因为@Autowired注解是在`AutowiredAnnotationBeanPostProcessor`中解析的，spring容器调用`BeanFactoryPostProcessor#postProcessBeanFactory`的使用，此时spring容器中还没有`AutowiredAnnotationBeanPostProcessor`，所以此时去获取user1这个bean的时候，@Autowired并不会被处理，所以name是null**

**4个阶段的源码**

4个阶段的源码为位于下面这个方法中

```java
org.springframework.context.support.AbstractApplicationContext#refresh
    
// 对应阶段1和阶段2：调用上下文中注册为bean的工厂处理器，即调用本文介绍的2个接口中的方法
invokeBeanFactoryPostProcessors(beanFactory);
// 对应阶段3：注册拦截bean创建的bean处理器，即注册BeanPostProcessor
registerBeanPostProcessors(beanFactory);
// 对应阶段3：实例化所有剩余的（非延迟初始化）单例。
finishBeanFactoryInitialization(beanFactory);    
```

## Environment外部化配置管理

Environment的中文意思是环境，它表示整个spring应用运行时的环境信息，它包含两个关键因素

- profiles
- properties

### profiles

profiles这个概念相信大家都已经理解了，最常见的就是不同环境下，决定当前spring容器中的不同配置上下文的解决方案。比如针对开发环境、测试环境、生产环境，构建不同的application.properties配置项，这个时候我们可以通过profiles这个属性来决定当前spring应用上下文中生效的配置项。

实际上，通过profiles可以针对bean的配置进行逻辑分组。 简单来说，我们可以通过profiles来针对不同的bean进行逻辑分组，这个分组和bean本身的定义没有任何关系，无论是xml还是注解方式，都可以配置bean属于哪一个profile分组。

当存在多个profile分组时，我们可以指定哪一个profile生效，当然如果不指定，spring会根据默认的profile去执行。我们来通过一个代码演示一下。

#### ProfileService

创建一个普通的类，代码如下

```java
public class ProfileService {
    private String profile;

    public ProfileService(String profile) {

        this.profile = profile;
    }

    @Override
    public String toString() {
        return "ProfileService{" +
                "profile='" + profile + '\'' +
                '}';
    }
}
```

#### 声明一个配置类

在配置类中，构建两个bean，配置不同的profile

```java
@Configuration
public class ProfileConfiguration {

    @Bean
    @Profile("dev")
    public ProfileService profileServiceDev(){
        return new ProfileService("dev");
    }

    @Bean
    @Profile("prod")
    public ProfileService profileServiceProd(){
        return new ProfileService("prod");
    }
}
```

#### 定义测试方法

```java
public class ProfileMain {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext=new AnnotationConfigApplicationContext();
//        applicationContext.getEnvironment().setActiveProfiles("prod");
        applicationContext.register(ProfileConfiguration.class);
        applicationContext.refresh();
        System.out.println(applicationContext.getBean(ProfileService.class));
    }
}
```

可以通过很多种方式来激活配置，默认情况下不添加`applicationContext.getEnvironment().setActiveProfiles("prod");`时，会发现bean没有被装载。添加了之后，会根据当前激活的profiles来决定装载哪个bean

除此之外，我们还可以在启动参数中增加`-Dspring.profiles.active=prod`来决定当前激活哪个profile。该属性可以配置在系统环境变量、JVM系统属性、等

> 注意配置文件不是单选；可能会同时激活多个配置文件，编程式的使用方法setActiveProfiles()，该方法接收String数组参数,也就是多个配置文件名

```java
applicationContext.getEnvironment().setActiveProfiles("prod","dev");
```

如果没有任何profile配置被激活，默认的profile将会激活。 默认profile配置文件可以更改，通过环境变量的setDefaultProfiles方法，或者是声明的spring.profiles.default属性值

#### profiles总结

通过profiles可以对bean进行逻辑分组，这些逻辑分组的bean会根据Environment上下文中配置的激活的profile来进行加载

- 一个profile就是一组Bean定义的逻辑分组
- 只有当一个profile处于active状态时，它对应的逻辑上组织在一起的这些Bean定义才会被注册到容器中
- Bean添加到profile可以通过XML定义方式或者annotation注解方式
- Environment对于profile所扮演的角色是用来指定哪些profile是当前活跃的缺省

### Properties

properties的作用就是用来存放属性的，它可以帮我们管理各种配置信息。这个配置的来源可以是properties文件、JVM properties、系统环境变量、或者专门的Properties对象等。

我们来看一下Environment这个接口，它继承了PropertyResolver，这个接口和属性的操作有关，也就是我们可以通过Environment来设置和获得相关属性

```java
public interface Environment extends PropertyResolver {
    String[] getActiveProfiles();

    String[] getDefaultProfiles();

    /** @deprecated */
    @Deprecated
    boolean acceptsProfiles(String... var1);

    boolean acceptsProfiles(Profiles var1);
}
```

至此，我们可以可以简单的总结Environment的作用，Environment提供了不同的profile配置，而PropertyResolver提供了配置的操作，由此我们可以知道，Spring 容器可以根据不同的profile来获取不同的配置信息，从而实现Spring容器中运行时环境的处理。

#### environment的应用

- 在spring boot应用中，修改application.properties配置

  ```properties
  env=default
  ```

- 创建一个Controller进行测试

```java
@RestController
public class EnvironementController {

    @Autowired
    Environment environment;

    @GetMapping("/env")
    public String env(){
        return environment.getProperty("env");
    }
```

#### 指定profile属性

在spring boot应用中，默认的外部化配置是application.properties文件，事实上，除了这个默认的配置文件之外，我们还可以使用springboot中的约定命名格式来实现不同环境的配置

当前spring boot应用选择使用哪个properties文件作为上下文环境配置，取决与当前激活的profile。我们可以通过很多种方式来激活，比如在application.properties中增加`spring.profiles.active=dev`这种方式，也可以在JVM参数中增加该配置来指定生效的配置

在不指定的情况下，则使用默认的配置文件，即应用程将加载application-default.properties中的属性

这个功能非常实用，一般的公司里面都会有几套运行环境，比如开发、测试、生产环境，这些环境中会有一些配置信息是不同的，比如服务器地址。那我们需要针对不同的环境使用指定的配置信息，通过这种方式就可以很方便的去解决

## Spring Environment原理设计

结合前面咱们讲过的内容，我们来推测一下Environment的实现原理

![image-20211216204620415](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408172056430.webp)

- 第一部分是属性定义，这个属性定义可以来自于很多地方，比如application.properties、或者系统环境变量等
- 然后根据约定的方式去指定路径或者指定范围去加载这些配置，保存到内存中
- 最后，我们可以根据指定的key从缓存中去查找这个值

下面是表示Environment的类关系图

![image-20211216210040646](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408172056967.webp)

上述类图的核心API说明如下

- Environment接口，继承了PropertyResolver。 PropertyResolver，它主要有两个作用
   - 通过`propertyName`属性名获取与之对应的`propertValue`属性值（getProperty）
   - 把`${propertyName:defaultValue}`格式的属性占位符，替换为实际的值(resolvePlaceholders)
- PropertyResolver的具体实现类是PropertySourcesPropertyResolver，该类是体系中唯一的完整实现类。它以PropertySources属性源集合（内部持有属性源列表List）为属性值的来源，按序遍历每个PropertySource，获取到一个非null的属性值则返回

其中，PropertySourcesPropertyResolver中的List，表示不同属性源的来源，它的类关系图如下，表示针对不同数据源的存储

![image-20211216211933822](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408172056132.webp)














# 参考文档
[Spring Boot启动流程bilibili](https://iilgw85n3f.feishu.cn/docx/DdE1d1wB2oEO8TxKpB7calxdnSc)



