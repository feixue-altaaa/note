# RocketMQ单机安装

## 下载

- 下载地址     [rocketmq下载](https://rocketmq.apache.org/download/)
- 选择Binary进行下载
- ![在这里插入图片描述](https://img-blog.csdnimg.cn/3f17cd5bf716427fbfd743881445ce5e.png)

+ 将下载的安装包上传到Linux

## 修改配置文件

- 修改runserver.sh
- 使用vim命令打开bin/runserver.sh文件。现将这些值修改为如下

![12](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211802838.jpg)

- 修改runbroker.sh
- 使用vim命令打开bin/runbroker.sh文件。现将这些值修改为如下

![13](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211802874.jpg)

## 启动

### 启动NAMESERVER 

```bash
nohup sh bin/mqnamesrv &
tail -f ~/logs/rocketmqlogs/namesrv.log
```

![14](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211802631.jpg)

### 启动broker

```bash
nohup sh bin/mqbroker -n localhost:9876 &
tail -f ~/logs/rocketmqlogs/broker.log
```

![15](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202402211802136.jpg)

### 关闭Server

+ 无论是关闭name server还是broker，都是使用bin/mqshutdown命令

```bash
[root@mqOS rocketmq]# sh bin/mqshutdown broker
The mqbroker(1740) is running...
Send shutdown request to mqbroker(1740) OK
[root@mqOS rocketmq]# sh bin/mqshutdown namesrv
The mqnamesrv(1692) is running...
Send shutdown request to mqnamesrv(1692) OK
[2]+ 退出 143 nohup sh bin/mqbroker -n localhost:9876
```

***注意：需在Java8环境下启动***

# RocketMQ插件部署

- 下载
- 编译 
  用CMD进入‘\rocketmq-console’文件夹，执行maven命令编译生成jar包

```java
mvn clean package -Dmaven.test.skip=true
```

- 启动 
  编译成功之后，Cmd进入‘target’文件夹，执行命令，启动‘rocketmq-console-ng-2.0.0.jar’。

```java
java -jar rocketmq-console-ng-2.0.0.jar
```

- 测试 
  浏览器中输入‘127.0.0.1:配置端口’，成功后即可查看。 
  eg：http://127.0.0.1:8088
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/495f35154631403c9cbe4819d1304f56.png)

# Springboot整合RocketMQ

## 引入依赖

```java
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.2.2</version>
</dependency>
```

## 创建生产者

@RocketMQMessageListener
消息监听器，消费者通过参数topic、selectorExpression来监听生产者发送消息的topic和tags

```java
@Slf4j
@RocketMQMessageListener(consumerGroup = "rocketmq-consumer",
        topic = "rocketmqTest",
        selectorExpression = "tags"
)
public class ConsumerManagerImpl implements ConsumerManager {
    
    @Override
    public void onMessage(String msg) {
        log.info("msg: {}", msg);
    }
}
```

## yml配置

```java
#rocketmq配置
rocketmq:
  producer:
    group: rocketmq
  name-server: 127.0.0.1:9876
```

## 创建测试接口

```java
@RestController
@RequestMapping("/MQTest") 
public class MQController {

    @Resource
    ProducerManagerImpl producerManager;

    @GetMapping("/sendMessage")
    public String sendMessage(@RequestParam("message") String message){
        producerManager.sendMessage("rocketmqTest", message);
        return "消息发送完成";
    }
}
```



