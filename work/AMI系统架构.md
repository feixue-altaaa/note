# 框架总体设计

## 架构整体图

设计架构分成接口层(interfaces)、应用层(Applications)、领域层(Domain)以及基础设施层(Infrastructure)

**架构分层图**

![image-20240724142757696](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407241427724.png)

**六边形架构**

![image-20240724142908296](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407241429320.png)

- 面向用户侧，提供http端口，并使用SpringMVC框架的RequestMapping、Controller等组件实现对http 请求的解析，转化为Application层可识别的业务dto对象，这里的Controller+RequestMapping便起着适配器的作用以此提供Restful风格架构，接口符合Restful规范。通过RESTful风格的接口契约对外提供主机开放服务。借助SpringMVC实现
- 面向电表集中器等硬件设备，通过netty框架使用tcp通信，采集数据
- 面向存储层，通过mongoclient的适配，访问mongodb；通过mybatis的适配或者Spring jpa，访问mysql；通过redisclient的适配，访问redis；
- 面向消息中间件，通过mqclient的适配，访问 rabbitMQ

## 分层架构风格

- User Interface —— 用户接口层。对外提供各种协议形式的服务，并提供Validation参数校验，authenticate权限认证等
- Application —— 应用服务层。组合多个业务实体、基础设施层的各种组件完成业务服务
- Domain —— 业务领域层。DDD概念中的核心业务层，封装所有业务逻辑，包含entity、value object、domain service、domain event等
- Infrastructure —— 基础设施层。提供公共组件，如：Logging、rabbitMq、Netty等

![image-20240724143212371](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202407241432411.png)