# 单点登录

## 前端同域 + 后端同 Redis

#### 设计思路

首先我们分析一下多个系统之间，为什么无法同步登录状态？

- 前端的 `Token` 无法在多个系统下共享
- 后端的 `Session` 无法在多个系统间共享

所以我们就可以使用

- 共享Cookie来解决Token共享问题
- 使用Redis来解决Session共享问题

共享Cookie就是主域名Cookie在二级域名下的共享

共享Redis就是并不需要我们把所有项目的数据都放在同一个Redis中，Sa-Token提供了 **[权限缓存与业务缓存分离]** 的解决方案

Sa-Token默认的Redis集成方式会把权限数据和业务缓存放在一起，但在部分场景下我们需要将他们彻底分离开来，比如：搭建两个Redis服务器，一个专门用来做业务缓存，另一台专门存放Sa-Token权限数据

![img](https://oss.dev33.cn/sa-token/doc/g/g3--alone-redis.gif)

![模式1.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0ce4ddac8f2d486ab8d114ea84776029~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp?)

## 前端不同域 + 后端同 Redis

**设计思路**

- 使用 `url重定向传播` 来解决 Token 共享问题。
- 使用 `Redis` 来解决 Session 共享问题

![img](https://img-blog.csdnimg.cn/92a11dd4972d4db2908184a8a7c732f8.png)

- 用户进入 A 系统，没有登录凭证（ticket），A 系统给他跳到 SSO
- SSO 没登录过，也就没有 sso 系统下没有凭证（注意这个和前面 A ticket 是两回事），输入账号密码登录
- SSO 账号密码验证成功，通过接口返回做两件事：一是种下 sso 系统下凭证（记录用户在 SSO 登录状态）；二是下发一个 ticket
- 客户端拿到 ticket，保存起来，带着请求系统 A 接口
- 系统 A 校验 ticket，成功后正常处理业务请求
- 此时用户第一次进入系统 B，没有登录凭证（ticket），B 系统给他跳到 SSO
- SSO 登录过，系统下有凭证，不用再次登录，只需要下发 ticket
- 客户端拿到 ticket，保存起来，带着请求系统 B 接口

对浏览器来说，SSO 域下返回的数据要怎么存，才能在访问 A 的时候带上？浏览器对跨域有严格限制，cookie、localStorage 等方式都是有域限制的

- 在 SSO 域下，SSO 不是通过接口把 ticket 直接返回，而是通过一个带 code 的 URL 重定向到系统 A 的接口上，这个接口通常在 A 向 SSO 注册时约定
- 浏览器被重定向到 A 域下，带着 code 访问了 A 的 callback 接口，callback 接口通过 code 换取 ticket
- 这个 code 不同于 ticket，code 是一次性的，暴露在 URL 中，只为了传一下换 ticket，换完就失效
- callback 接口拿到 ticket 后，在自己的域下 set cookie 成功
- 在后续请求中，只需要把 cookie 中的 ticket 解析出来，去 SSO 验证就好
- 访问 B 系统也是一样

[单点登录原理、解决方案、基于satoken的实现](https://blog.csdn.net/love_eat_peach/article/details/131307064?spm=1001.2101.3001.6650.16&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-131307064-blog-127717892.235%5Ev38%5Epc_relevant_sort_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-16-131307064-blog-127717892.235%5Ev38%5Epc_relevant_sort_base3&utm_relevant_index=22)