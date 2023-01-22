# Redis应用场景

## 缓存

毫无疑问这是Redis当今最为人熟知的使用场景。在提升服务器性能方面非常有效

### 为什么查询更快

我们都知道内存读写是比磁盘读写快很多的。Redis是基于内存存储实现的数据库，相对于数据存在磁盘的数据库，就省去磁盘磁盘I/O的消耗。MySQL等磁盘数据库，需要建立索引来加快查询效率，而Redis数据存放在内存，直接操作内存，所以就很快

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d7be13173814a43a60960fe59a48c61~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.image)

## 排行榜

如果使用传统的关系型数据库来做这个事儿，非常的麻烦，而利用Redis的SortSet数据结构能够非常方便搞定

## 计算器

利用Redis中原子性的自增操作，我们可以统计类似用户点赞数、用户访问数等，这类操作如果用MySQL，频繁的读写会带来相当大的压力

## 限流

### 限流场景

+ 秒杀活动，有人使用软件恶意刷单抢货，需要限流防止机器参与活动

+ 某api被各式各样系统广泛调用，严重消耗网络、内存等资源，需要合理限流

+ 淘宝获取ip所在城市接口、微信公众号识别微信用户等开发接口，免费提供给用户时需要限流，更
  具有实时性和准确性的接口需要付费

### API限流自定义注解

+ 首先我们编写注解类 AccessLimit ，使用注解方式在方法上限流更优雅更方便！
  三个参数分别代表有效时间、最大访问次数、是否需要登录，可以理解为 seconds 内最多访问
  maxCount 次。

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

+ 通过 **ip:api** 路径的作为key，访问次数为value的方式对某一用户的某一请求进行唯一标识
  每次访问的时候判断 key 是否存在，是否 count 超过了限制的访问次数
  若访问超出限制，则应 response 返回 msg:请求过于频繁 给前端予以展示

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

## 简单消息队列(不常用)

+ 除了Redis自身的**发布/订阅模式**，我们也可以利用**List**来实现一个队列机制，比如：到货通知、邮件发送之类的需求，不需要高可靠，但是会带来非常大的DB压力，完全可以用List来完成异步解耦；
+ [Redis 实现消息队列][https://juejin.cn/post/6917576292808261645]

