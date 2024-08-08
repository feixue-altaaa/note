# 安装与运行

## 配置Maven国内镜像源

Maven的全局配置文件位于"Maven安装目录/conf/settings.xml"

在文件中找到标签，然后将以下配置代码添加到其中

```xml
<mirror>
   <id>nexus-aliyun</id>
   <mirrorOf>*</mirrorOf>
   <name>Nexus aliyun</name>
   <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

## 单个Maven项目配置镜像源

**右键项目，maven选项，"Open setting.xml"或"Create setting.xml"，在 mirrors 节点添加下面代码**

![在这里插入图片描述](https://raw.githubusercontent.com/feixue-altaaa/picture/master/pic/202408082241005.webp)

```xml
<mirrors>	
    <mirror>
        <id>alimaven</id>
        <name>aliyun maven</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>

    <mirror>
        <id>uk</id>
        <mirrorOf>central</mirrorOf>
        <name>Human Readable Name for this Mirror.</name>
        <url>http://uk.maven.org/maven2/</url>
    </mirror>

    <mirror>
        <id>CN</id>
        <name>OSChina Central</name>
        <url>http://maven.oschina.net/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>

    <mirror>
        <id>nexus</id>
        <name>internal nexus repository</name>
        <url>http://repo.maven.apache.org/maven2</url>
        <mirrorOf>central</mirrorOf>
    </mirror>

</mirrors>
```

