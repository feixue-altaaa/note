# 连接数据库

## MySQL

**先从菜单View→Tool Windows→Database打开数据库工具窗口，如下图所示**

![image-20240806154336571](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408061543989.png)

**单击“+”按钮添加新数据源，并从列表中选择“MySQL”**

填写所需信息

- 主机：localhost（或您的MySQL服务器地址）
- 端口：3306（MySQL的默认端口）
- 用户：您的MySQL用户名
- 密码：您的MySQL密码
- 数据库：您的MySQL数据库名称

单击“Test Connection”按钮测试连接。如果成功，请单击“OK”保存配置

![image-20240806154447551](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408061544584.png)

**连接时报错Server returns invalid timezone. Need to set 'serverTimezone' property**

**解决操作**

- 设置`mysql`的时区
- `mysql`驱动的版本

进入命令窗口（`Win + R`），连接数据库 `mysql -hlocalhost -uroot -p`，回车，输入密码，回车
继续输入 `show variables like'%time_zone';` （注意不要漏掉后面的分号），回车，如下图

![img](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408061546397.png)

显示 `SYSTEM` 就是没有设置时区

输入`set global time_zone = '+8:00';` 注意不要漏掉后面的分号），回车

重新连接下数据库，连接成功

# 在Idea中隐藏指定文件/文件

​		既然是在Idea下做隐藏功能，肯定隶属于Idea的设置，设置方式如下。

**步骤①**：打开设置，【Files】→【Settings】

<img src="https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021106090.png" alt="image-20211122173835517" style="zoom:80%;" />

**步骤②**：打开文件类型设置界面后，【Editor】→【File Types】→【Ignored Files and Folders】，忽略文件或文件夹显示

<img src="https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021106120.png" alt="image-20211122174020028" style="zoom: 67%;" />

**步骤③**：添加你要隐藏的文件名称或文件夹名称，可以使用*号通配符，表示任意，设置完毕即可。

​	到这里就做完了，其实就是Idea的一个小功能

**总结**

1. Idea中隐藏指定文件或指定类型文件
   1. 【Files】→【Settings】
   2. 【Editor】→【File Types】→【Ignored Files and Folders】
   3. 输入要隐藏的名称，支持*号通配符
   4. 回车确认添加

# **自动提示功能消失解决方案**

​		在做程序的过程中，可能有些小伙伴会基于各种各样的原因导致配置文件中没有提示，这个确实很让人头疼，所以下面给大家说一下如果自动提示功能消失了怎么解决

​		先要明确一个核心，就是自动提示功能不是SpringBoot技术给我们提供的，是我们在Idea工具下编程，这个编程工具给我们提供的。明白了这一点后，再来说为什么会出现这种现象。其实这个自动提示功能消失的原因还是蛮多的，如果想解决这个问题，就要知道为什么会消失，大体原因有如下2种：

- Idea认为你现在写配置的文件不是个配置文件，所以拒绝给你提供提示功能

- Idea认定你是合理的配置文件，但是Idea加载不到对应的提示信息

  这里我们主要解决第一个现象，第二种现象到原理篇再讲解。第一种现象的解决方式如下：

**步骤①**：打开设置，【Files】→【Project Structure...】

![image-20211126160548690](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021438895.png)

**步骤②**：在弹出窗口中左侧选择【Facets】，右侧选中Spring路径下对应的模块名称，也就是你自动提示功能消失的那个模块

<img src="https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021438558.png" alt="image-20211126160726589" style="zoom:67%;" />![image-20211126160844372](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021438640.png)

**步骤③**：点击Customize Spring Boot按钮，此时可以看到当前模块对应的配置文件是哪些了。如果没有你想要称为配置文件的文件格式，就有可能无法弹出提示

![image-20211126160946448](D:\课件\笔记\note\CS\Java\框架\springboot\img\image-20211126160946448.png)<img src="https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021438688.png" alt="image-20211126160954338" style="zoom:80%;" />

**步骤④**：选择添加配置文件，然后选中要作为配置文件的具体文件就OK了

<img src="D:\课件\笔记\note\CS\Java\框架\springboot\img\image-20211126161145082.png" alt="image-20211126161145082" style="zoom:80%;" /><img src="D:/课件/笔记/note/CS/Java/框架/springboot/img/image-20211126161156324.png" alt="image-20211126161156324" style="zoom: 67%;" />

​		到这里就做完了，其实就是Idea的一个小功能

![image-20211126161301699](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303021438994.png)



**总结**

1. 指定SpringBoot配置文件

   - Setting → Project Structure → Facets
   - 选中对应项目/工程
   - Customize Spring Boot
   - 选择配置文件

# 安装新版本时无法打开

![image-20230310151445102](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303101514495.png)

![image-20230310151513668](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202303101515716.png)

+ 删除 C:\Users\qls\AppData 路径下以上两个文件夹下的JetBrains文件

# IDEA破解

https://www.exception.site/essay/idea-reset-eval
