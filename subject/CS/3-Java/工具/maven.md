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

# pom文件结构

## what

### 什么是POM文件？

POM（Project Object Model）文件是Maven项目的核心文件之一。它是一个XML文件，描述了项目的基本信息、依赖项、构建和发布等信息。POM文件是Maven的重要组成部分，可以帮助开发者管理和构建项目。在使用Maven进行项目构建时，需要根据项目的需要配置POM文件

### POM文件的基本结构

POM文件是一个XML文件，包含多个元素，每个元素代表一个特定的配置项。下面是一个POM文件的基本结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
         http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>example-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- 依赖项配置 -->
    </dependencies>

    <build>
        <!-- 构建配置 -->
    </build>

    <repositories>
        <!-- 仓库配置 -->
    </repositories>
</project>
```

## POM文件的常用配置项

### 坐标信息

坐标信息指的是<groupId>、<artifactId>和<version>元素。<groupId>定义了项目的组织ID，<artifactId>定义了项目的唯一标识符，<version>定义了项目的版本号。这些信息对于Maven的依赖管理和构建过程非常重要

```xml
<groupId>com.example</groupId>
<artifactId>example-project</artifactId>
<version>1.0-SNAPSHOT</version>
```

### 依赖项配置

依赖项配置用于定义项目所依赖的外部库。Maven会自动下载并管理这些依赖项。依赖项配置包含在<dependencies>元素中，每个依赖项使用一个<dependency>元素进行描述

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-core</artifactId>
        <version>5.3.13</version>
    </dependency>
</dependencies>
```

>  dependencies与dependencyManagement的区别
>
>  **dependencyment**
>
>  在我们项目中，我们会发现在父模块的pom文件中常常会出现dependencyMent元素，这是因为我们可以通过其来管理子模块的版本号，也就是说我们在父模块中声明号依赖的版本，但是并不实现引入；
>
>  **dependencies**
>
>  上面说到dependencyment只是声明一个依赖，而不实现引入，故我们在子模块中也需要对依赖进行声明，倘若不声明子模块自己的依赖，是不会从父模块中继承的；只有子模块中也声明了依赖。并且没有写对应的版本号它才会从父类中继承；并且version和scope都是取自父类；此外要是子模块中自己定义了自己的版本号，是不会继承自父类的
>
>  **总结**
>
>  dependencyment只是用来管理依赖，规定未添加版本号的子模块依赖继承自它，dependencies是用来声明子模块自己的依赖，可以在其中来写自己需要的版本号；

### 构建配置

构建配置用于定义项目的构建过程。它包含在<build>元素中，包括了多个子元素。其中比较常用的子元素有

```xml
<build>
    <plugins>
        <!-- 插件配置 -->
    </plugins>
    <resources>
        <!-- 资源文件配置 -->
    </resources>
    <testResources>
        <!-- 测试资源文件配置 -->
    </testResources>
</build>
```

### 插件配置

插件是Maven的一个重要特性，它可以用于扩展Maven的功能。Maven自带了一些插件，比如maven-compiler-plugin、maven-jar-plugin等。插件配置包含在<plugins>元素中，每个插件使用一个<plugin>元素进行描述

```xml
<plugins>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-compiler-plugin</artifactId>
        <version>3.8.1</version>
        <configuration>
            <source>1.8</source>
            <target>1.8</target>
        </configuration>
    </plugin>
</plugins>
```

### 仓库配置

仓库配置用于定义Maven的依赖项下载地址。Maven默认使用中央仓库，但是也可以配置私有仓库或者本地仓库。仓库配置包含在<repositories>元素中，每个仓库使用一个<repository>元素进行描述

```xml
<repositories>
    <repository>
        <id>central</id>
        <url>https://repo.maven.apache.org/maven2</url>
    </repository>
</repositories>
```

## 父子 POM 示例

**Maven 父 POM** （或超级 POM）用于构造项目，以**避免重复或重复使用 pom 文件之间的\*继承配置\***。 它有助于长期轻松维护。

如果在父 POM 和子 POM 中都使用不同的值配置了任何依赖项或属性，则子 POM 值将具有优先级

### 父 POM 内容

可以使用包`pom`声明父 POM。 它不打算分发，因为仅从其他项目中引用了它。

Maven 父 pom 可以包含几乎所有内容，并且可以继承到子 pom 文件中，例如：

*   通用数据 – 开发人员的姓名，SCM 地址，分发管理等
*   常数 – 例如版本号
*   共同的依赖项 – 所有子项共同的。 与在单个 pom 文件中多次写入它们具有相同的效果。
*   属性 – 例如插件，声明，执行和 ID。
*   配置
*   资源

### 父 POM 和子 POM 示例

为了匹配父 POM，Maven 使用两个规则：

1.  在项目的根目录或给定的相对路径中有一个 pom 文件。
2.  子 POM 文件中的引用包含与父 POM 文件中所述相同的坐标。

#### 父 POM

此处，父 POM 为 JUnit 和 spring 框架配置了基本项目信息和两个[依赖项](//howtodoinjava.com/maven/maven-dependency-management/)。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd;
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.howtodoinjava.demo</groupId>
	<artifactId>MavenExamples</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>pom</packaging>

	<name>MavenExamples Parent</name>
	<url>http://maven.apache.org</url>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<junit.version>3.8.1</junit.version>
		<spring.version>4.3.5.RELEASE</spring.version>
	</properties>

	<dependencies>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>

	</dependencies>
</project>

```

#### 子 POM

现在，子 POM 需要使用`parent`标签并指定`groupId/artifactId/version`属性来引用父 POM。 这个 pom 文件将从父 POM 继承所有属性和依赖项，并且还可以包括子项目特定的依赖项。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<!--The identifier of the parent POM-->
	<parent>
		<groupId>com.howtodoinjava.demo</groupId>
		<artifactId>MavenExamples</artifactId>
		<version>0.0.1-SNAPSHOT</version>
	</parent>

	<modelVersion>4.0.0</modelVersion>
	<artifactId>MavenExamples</artifactId>
	<name>MavenExamples Child POM</name>
	<packaging>jar</packaging>

	<dependencies>		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-security</artifactId>
			<version>${spring.version}</version>
		</dependency>
	</dependencies>

</project>

```

### 父POM相对路径

默认情况下，Maven 首先在项目的根目录下查找父 POM，然后在本地仓库中查找，最后在远程仓库中查找。 如果父 POM 文件不在任何其他位置，则可以使用代码标签。 该**相对路径应相对于项目根**

如果未明确给出相对路径，则默认为`..`，即当前项目的父目录中的 pom

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

	<!--The identifier of the parent POM-->
	<parent>
		<groupId>com.howtodoinjava.demo</groupId>
		<artifactId>MavenExamples</artifactId>
		<version>0.0.1-SNAPSHOT</version>
		<relativePath>../baseapp/pom.xml</relativePath>
	</parent>

	<modelVersion>4.0.0</modelVersion>
	<artifactId>MavenExamples</artifactId>
	<name>MavenExamples Child POM</name>
	<packaging>jar</packaging>

</project>
```











