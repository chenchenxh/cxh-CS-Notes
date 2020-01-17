# Maven

Maven是专门为Java项目打造的管理和构建工具，它的主要功能有：

- 提供了一套标准化的项目结构；
- 提供了一套标准化的构建流程（编译，测试，打包，发布……）；
- 提供了一套依赖管理机制。

默认的项目目录结构如下：

```ascii
a-maven-project
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       ├── java
│       └── resources
└── target
```

其中最关键的就是pom.xml文件，这个文件就是该项目的描述文件。用来声明项目的名称和版本等信息，最主要是该项目的依赖。

## 依赖关键

Java项目中非常重要的就是包的使用，也就会牵涉到包的管理的问题。因此Maven的一个重要功能就是包的依赖管理，可以自动分析并管理依赖。

依赖关系包括以下几种：

| scope    | 说明                                          | 示例            |
| :------- | :-------------------------------------------- | :-------------- |
| compile  | 编译时需要用到该jar包（默认）                 | commons-logging |
| test     | 编译Test时需要用到该jar包                     | junit           |
| runtime  | 编译时不需要，但运行时需要用到                | mysql           |
| provided | 编译时需要用到，但运行时由JDK或某个服务器提供 | servlet-api     |

对于某个依赖，Maven只需要3个变量即可唯一确定某个jar包：

- groupId：属于组织的名称，类似Java的包名；
- artifactId：该jar包自身的名称，类似Java的类名；
- version：该jar包的版本。

示例如下：

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.3.2</version>
    <scope>test</scope>
</dependency>
```

## 构建流程

Maven的构建是由很多个阶段（phase）构成的，可以操作Maven运行到某个phase为止。

对此，常用的命令有：

```java
mvn clean	//清理所有生成的class和jar；

mvn clean compile	//先清理，再执行到compile；

mvn clean test	//先清理，再执行到test，因为执行test前必须执行compile，所以这里不必指定compile；

mvn clean package	//先清理，再执行到package。
```

**Goal**：上述的phase中的真正的执行单元，一般mvn+phase就会执行到该phase中的默认goal，特殊情况下可以直接mvn+phase:goal。

## 模块管理

一个project中如果只有一个模块就会很繁杂，降低复杂度的方式就是分成几个模块。

Maven就可以对这些模块进行管理。

## mvnw

mvnw是Maven Wrapper的缩写。通常情况下，系统使用的Maven是全局安装的Maven版本，但是可以对某个或者某些特殊的项目使用特定的Maven，就是使用mvnw。

在项目目录安装后，查看项目结构：

```ascii
my-project
├── .mvn
│   └── wrapper
│       ├── MavenWrapperDownloader.java
│       ├── maven-wrapper.jar
│       └── maven-wrapper.properties
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        ├── java
        └── resources
```

发现多了`mvnw`、`mvnw.cmd`和`.mvn`目录，我们只需要把`mvn`命令改成`mvnw`就可以使用跟项目关联的Maven。