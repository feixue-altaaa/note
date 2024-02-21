# 优缺点

## 优点

+ 支持多种数据类型
  + redis支持set,zset,list,hash,string这五种数据类型，操作非常方便

+ 持久化存储
  + 作为一个内存数据库，最担心的，就是万一机器死机宕机，数据就会消失掉。redis使用RDB和AOF做数据的持久化存储

+ 速度快

## 缺点

- Redis的数据库容量受到物理内存的限制，不能用作海量数据的高性能读写，因此Redis适合的场景主要局限在较小数据量的高性能操作和运算上；
- 在主节点宕机前有部分数据不能及时同步到从节点上；
- 单线程操作，无法利用多核CPU的优势

# Redis应用场景

> ## 缓冲（buffer）和缓存（cache）区别
>
> - 缓存（cache）是在读取硬盘中的数据时，把最常用的数据保存在内存的缓存区中，再次读取该数据时，就不去硬盘中读取了，而在缓存中读取
> - 缓冲（buffer）是在向硬盘写入数据时，先把数据放入缓冲区,然后再一起向硬盘写入，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而提高系统性能
> - 简单来说，缓存（cache）是用来加速数据从硬盘中"读取"的，而缓冲（buffer）是用来加速数据"写入"硬盘的

## 缓存

+ 毫无疑问这是Redis当今最为人熟知的使用场景。在提升服务器性能方面非常有效

### 为什么查询更快

**redis可以承担Redis读的速度是110000次/s,写的速度是81000次/s ，也就是说redis的qps大概在10W量级上，这个量是很大的**

+ 内存存储：Redis 将数据存储在内存中，而不是硬盘上，因此可以快速读写数据。内存的读写速度比硬盘快得多

+ 单线程模型：Redis 是单线程模型的（单线程指的是网络请求模块使用了一个线程（所以不需考虑并发安全性），即一个线程处理所有网络请求，其他模块仍用了多个线程），这意味着 Redis 在任何时候都只有一个线程在执行，避免了**线程切换**和**上下文切换**的开销，从而提高了性能

  > 为何使用单线程？
  >
  > - **不需要各种锁的性能消耗**
  > - 避免了**线程切换**和**上下文切换**的开销

  > **并发(concurrency)**：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。
  > **并行(parallel)**：指在同一时刻，有多条指令在多个处理器上同时执行。所以无论从微观还是从宏观来看，二者都是一起执行的

  > **上下文切换**
  >
  > 多个线程可以执行在单核或多核CPU上，单核CPU也支持多线程执行代码，CPU通过给每个线程分配CPU时间片(机会)来实现这个机制。CPU为了执行多个线程，就需要不停的切换执行的线程，这样才能保证所有的线程在一段时间内都有被执行的机会。
  >
  > 此时，CPU分配给每个线程的执行时间段，称作它的时间片。CPU时间片一般为几十毫秒。CPU通过时间片分配算法来循环执行任务，当前任务执行一个时间片后切换到下一个任务。
  >
  > 但是，在切换前会保存上一个任务的状态，以便下次切换回这个任务时，可以再加载这个任务的状态。所以任务从保存到再加载的过程就是一次上下文切换。
  >
  > 根据多线程的运行状态来说明：多线程环境中，当一个线程的状态由Runnable转换为非Runnable(Blocked、Waiting、Timed_Waiting)时，相应线程的上下文信息(包括CPU的寄存器和程序计数器在某一时间点的内容等)需要被保存，以便相应线程稍后再次进入Runnable状态时能够在之前的执行进度的基础上继续前进。而一个线程从非Runnable状态进入Runnable状态可能涉及恢复之前保存的上下文信息。这个对线程的上下文进行保存和恢复的过程就被称为

+ IO 多路复用：Redis 将所有产生事件的套接字都放到一个队列里面，以有序、同步、每次一个套接字的方式向文件事件分派器传送套接字，文件事件分派器根据套接字对应的事件选择响应的处理器进行处理，从而实现了高效的网络请求

  + I/O多路复用
    为什么Redis中要使用 I/O 多路复用这种技术呢？因为Redis 是跑在单线程中的，所有的操作都是按照顺序线性执行的，但是由于读写操作等待用户输入 或 输出都是阻塞的，所以 I/O 操作在一般情况下往往不能直接返回，这会导致某一文件的 I/O 阻塞导，致整个进程无法对其它客户提供服务。而 I/O 多路复用就是为了解决这个问题而出现的。为了让单线程(进程)的服务端应用同时处理多个客户端的事件，Redis采用了IO多路复用机制

  + 这里“多路”指的是多个网络连接客户端，“复用”指的是复用同一个线程(单进程)。I/O 多路复用其实是使用一个线程来检查多个Socket的就绪状态，在单个线程中通过记录跟踪每一个socket（I/O流）的状态来管理处理多个I/O流。如下图是Redis的I/O多路复用模型

  + ![img](https://img-blog.csdnimg.cn/20210818104310346.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1Nla3lfZmVp,size_16,color_FFFFFF,t_70)


  > (1)一个socket客户端与服务端连接时，会生成对应一个套接字描述符(套接字描述符是文件描述符的一种)，每一个socket网络连接其实都对应一个文件描述符
  >
  > (2)多个客户端与服务端连接时，Redis使用 I/O多路复用程序 将客户端socket对应的FD注册到监听列表(一个队列)中，并同时监控多个文件描述符（fd）的读写情况。当客服端执行accept、read、write、close等操作命令时，I/O多路复用程序会将命令封装成一个事件，并绑定到对应的FD上
  >
  > (3)当socket有文件事件产生时，I/O 多路复用模块就会将那些产生了事件的套接字fd传送给文件事件分派器
  >
  > (4)文件事件分派器接收到I/O多路复用程序传来的套接字fd后，并根据套接字产生的事件类型，将套接字派发给相应的事件处理器来进行处理相关命令操作
  >
  > (5)整个文件事件处理器是在单线程上运行的，但是通过 I/O 多路复用模块的引入，实现了同时对多个 FD 读写的监控，当其中一个client端达到写或读的状态，文件事件处理器就马上执行，从而就不会出现I/O堵塞的问题，提高了网络通信的性能
  >
  > (6)如上图，Redis的I/O多路复用模式使用的是 Reactor设计模式的方式来实现

+ 数据结构简单：Redis 支持多种数据结构，如字符串、哈希表、列表、集合等，这些数据结构都是基于简单的键值对存储的，因此可以快速读写数据

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303131653366.png)

## 排行榜

+ 如果使用传统的关系型数据库来做这个事儿，非常的麻烦，而利用Redis的SortSet数据结构能够非常方便搞定

## 计算器

+ 利用Redis中原子性的自增操作，我们可以统计类似用户点赞数、用户访问数等，这类操作如果用MySQL，频繁的读写会带来相当大的压力

## 限流

### 限流场景

+ 秒杀活动，有人使用软件恶意刷单抢货，需要限流防止机器参与活动

+ 某api被各式各样系统广泛调用，严重消耗网络、内存等资源，需要合理限流

+ 淘宝获取ip所在城市接口、微信公众号识别微信用户等开发接口，免费提供给用户时需要限流，更具有实时性和准确性的接口需要付费

### API限流自定义注解

+ 首先我们编写注解类 AccessLimit ，使用注解方式在方法上限流更优雅更方便！
+ 三个参数分别代表有效时间、最大访问次数、是否需要登录，可以理解为 seconds 内最多访问 maxCount 次

```java
package com.maxuan.service;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface AccessLimit {
    int seconds();
    int maxCount();
    boolean needLogin() default true;
}
```

### 限流的思路

+ 通过 **ip:api** 路径作为key，访问次数为value的方式对某一用户的某一请求进行唯一标识
+ 每次访问的时候判断 key 是否存在，是否 count 超过了限制的访问次数
+ 若访问超出限制，则应 response 返回 msg:请求过于频繁 给前端予以展示

```java
package com.maxuan.component;
import com.maxuan.service.AccessLimit;
import org.springframework.stereotype.Component;
import org.springframework.web.method.HandlerMethod;
import org.springframework.web.servlet.HandlerInterceptor;
import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
@Component
public class AccessLimtInterceptor implements HandlerInterceptor {
    @Resource
    private RedisUtil redisUtil;
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse
                        response, Object handler) throws Exception {
        if (handler instanceof HandlerMethod) {
            HandlerMethod hm = (HandlerMethod) handler;
            AccessLimit accessLimit = hm.getMethodAnnotation(AccessLimit.class);
            if (null == accessLimit) {
                return true;
            }
            int seconds = accessLimit.seconds();
            int maxCount = accessLimit.maxCount();
            boolean needLogin = accessLimit.needLogin();
            if (needLogin) {
            //判断是否登录
            }
            //客户端ip地址
            String ip = request.getRemoteAddr();
            String key = ip + ":" + request.getServletPath();
            Integer count = (Integer) redisUtil.get(key);
            //第一次访问
            if (null == count || -1 == count) {
                redisUtil.set(key, 1);
                //设置过期时间
                redisUtil.expire(key, seconds);
                return true;
            }
            //如果访问次数<最大次数，则加1操作
            if (count < maxCount) {
                redisUtil.incr(key, 1);
                return true;
            }
            //超过最大值返回操作频繁
            if (count >= maxCount) {
                System.out.println("count=="+count);
                response.setContentType("text/html;charset=utf-8");
                response.getWriter().write("请求过于频繁，请稍后再试");
                return false;
            }
        }
        return true;
    }
}
```

**注册拦截器并配置拦截路径和不拦截路径**

```java
package com.maxuan.component;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
@Configuration
public class IntercepterConfig implements WebMvcConfigurer {
    @Autowired
    private AccessLimtInterceptor accessLimtInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(accessLimtInterceptor)
            .addPathPatterns("/access/accessLimit")
            .excludePathPatterns("/access/login");
    }
}

```

**在 Controller 层的方法上直接可以使用注解 @AccessLimit**

```java
package com.maxuan.controller;
import com.maxuan.service.AccessLimit;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.RestController;
@RestController
@RequestMapping("access")
public class AccessControler {
    @ResponseBody
    @GetMapping("accessLimit")
    //3秒内最多访问10次
    @AccessLimit(seconds = 3, maxCount = 10)
    public String accessLimit() {
        return "it is ok";
    }
}
```

## 好友关系

* 通过set求交集、并集、差集等。可以方便搞定一些共同好友、共同爱好之类的功能

[微服务Spring Boot 整合 Redis 实现 好友关注][https://blog.csdn.net/weixin_45526437/article/details/128276229]

## Session共享

[session + redis 实现session 共享原理和原因][https://blog.csdn.net/qq_37306041/article/details/107948763]

## 分布式锁

## 简单消息队列(不常用)

+ 除了Redis自身的**发布/订阅模式**，我们也可以利用**List**来实现一个队列机制，比如：到货通知、邮件发送之类的需求，不需要高可靠，但是会带来非常大的DB压力，完全可以用List来完成异步解耦；
+ [Redis 实现消息队列][https://juejin.cn/post/6917576292808261645]

