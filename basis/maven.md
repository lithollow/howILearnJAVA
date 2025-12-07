# 1 介绍

Maven是专门为Java项目打造的管理和构建工具，主要功能：

- 提供一套标准化的项目结构；
- 提供一套标准化的构建流程（编译，测试，打包，发布……）；
- 提供一套依赖管理机制。



Maven 项目结构

```
a-maven-project				// 项目名
├── pom.xml					// 项目描述文件
├── src
│   ├── main
│   │   ├── java			// Java源码目录
│   │   └── resources		// 资源文件目录
│   └── test
│       ├── java			// 测试源码
│       └── resources		// 测试资源
└── target					// 所有编译、打包生成的文件
```



pom.xml : 

```xml
<project ...>
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.itranswarp.learnjava</groupId>
	<artifactId>hello</artifactId>
	<version>1.0</version>
	<packaging>jar</packaging>
	<properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<maven.compiler.release>17</maven.compiler.release>
	</properties>
	<dependencies>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.16</version>
        </dependency>
	</dependencies>
</project>
```

`groupId`类似于Java的包名，通常是公司或组织名称

`artifactId`类似于Java的类名，通常是项目名称

再加上`version`，一个Maven工程就是由`groupId`，`artifactId`和`version`作为唯一标识



引用其他第三方库的时候，也是通过这3个变量确定。例如，上面的代码块中引用依赖`org.slfj4:slf4j-simple:2.0.16`:

使用`<dependency>`声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中

Maven维护了一个中央仓库（[repo1.maven.org](https://repo1.maven.org/)），所有第三方库将自身的jar以及相关信息上传至中央仓库，Maven从中央仓库把所需依赖下载到本地

一个jar包一旦被下载过，就会被Maven自动缓存在本地目录（用户主目录的`.m2`目录）



`<properties>`定义了一些属性，常用的属性有：

- `project.build.sourceEncoding`：表示项目源码的字符编码，通常应设定为`UTF-8`；
- `maven.compiler.release`：表示使用的JDK版本，例如`21`；
- `maven.compiler.source`：表示Java编译器读取的源码版本；
- `maven.compiler.target`：表示Java编译器编译的Class版本。

从Java 9开始，推荐使用`maven.compiler.release`属性，保证编译时输入的源码和编译输出版本一致。

如果源码和输出版本不同，则应该分别设置`maven.compiler.source`和`maven.compiler.target`



---



# 2 依赖管理

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>1.4.2.RELEASE</version>
</dependency>
```

声明一个`spring-boot-starter-web`依赖时，Maven会自动解析并判断最终需要的（大概二三十个）其他依赖，避免了手动管理依赖



依赖关系

| scope    | 说明                                          | 示例            |
| -------- | --------------------------------------------- | --------------- |
| compile  | 编译时需要用到该jar包（默认）                 | commons-logging |
| test     | 编译Test时需要用到该jar包                     | junit           |
| runtime  | 编译时不需要，但运行时需要用到                | mysql           |
| provided | 编译时需要用到，但运行时由JDK或某个服务器提供 | servlet-api     |

```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.3.2</version>
    <scope>test</scope>
</dependency>
```



---



# 3 构建流程

## lifecycle 生命周期

Maven的生命周期由一系列阶段（phase）构成，以内置的生命周期`default`为例，它包含以下phase：

- validate
- initialize
- generate-sources
- process-sources
- generate-resources
- process-resources
- compile
- process-classes
- generate-test-sources
- process-test-sources
- generate-test-resources
- process-test-resources
- test-compile
- process-test-classes
- test
- prepare-package
- package
- pre-integration-test
- integration-test
- post-integration-test
- verify
- install
- deploy



运行`mvn package`，Maven就会执行`default`生命周期，它会从开始一直运行到`package`这个phase为止

运行`mvn compile`，Maven也会执行`default`生命周期，只会运行到`compile`

运行`mvn clean`，执行3个phase：`pre-clean`, `clean `, `post-clean`



使用`mvn`命令时，后面的参数是phase，Maven自动根据生命周期运行到指定的phase；可以指定多个phase

常用phase：

- clean：清理
- compile：编译
- test：运行测试
- package：打包



执行一个phase又会触发一个或多个goal：

| 执行的Phase | 对应执行的Goal                     |
| ----------- | ---------------------------------- |
| compile     | compiler:compile                   |
| test        | compiler:testCompile surefire:test |

goal的命名总是`abc:xyz`这种形式

goal是最小任务单元



运行`mvn compile`，Maven执行`compile`这个phase，这个phase会调用`compiler`插件执行关联的`compiler:compile`这个goal

执行每个phase，都是通过某个插件（plugin）来执行的，Maven只是负责找到对应的`compiler`插件，然后执行默认的`compiler:compile`这个goal来完成编译

可以自定义插件


---



# 4 模块管理

在软件开发中，把一个大项目分拆为多个模块是降低软件复杂度的有效方法：

```
                        ┌ ─ ─ ─ ─ ─ ─ ┐
                          ┌─────────┐
                        │ │Module A │ │
                          └─────────┘
┌──────────────┐ split  │ ┌─────────┐ │
│Single Project│───────▶  │Module B │
└──────────────┘        │ └─────────┘ │
                          ┌─────────┐
                        │ │Module C │ │
                          └─────────┘
                        └ ─ ─ ─ ─ ─ ─ ┘
```

```
multiple-project
├── pom.xml
├── parent
│   └── pom.xml
├── module-a
│   ├── pom.xml
│   └── src
├── module-b
│   ├── pom.xml
│   └── src
└── module-c
    ├── pom.xml
    └── src
```

Maven可以有效地管理多个模块，只需要把每个模块当作一个独立的Maven项目，它们有各自独立的`pom.xml`



提取 module-a 和 module-b 的`pom.xml`高度相似的共同部分作为`parent`：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>parent</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>

    <name>parent</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.5.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

注意到parent的`<packaging>`是`pom`而不是`jar`，因为`parent`本身不含任何Java代码

module-a中使用：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.itranswarp.learnjava</groupId>
        <artifactId>parent</artifactId>
        <version>1.0</version>
        <relativePath>../parent/pom.xml</relativePath>
    </parent>

    <artifactId>module-a</artifactId>
    <packaging>jar</packaging>
    <name>module-a</name>
</project>
```

模块A依赖模块B

```xml
    ...
    <dependencies>
        <dependency>
            <groupId>com.itranswarp.learnjava</groupId>
            <artifactId>module-b</artifactId>
            <version>1.0</version>
        </dependency>
    </dependencies>
```

根`pom.xml`统一编译：

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.itranswarp.learnjava</groupId>
    <artifactId>build</artifactId>
    <version>1.0</version>
    <packaging>pom</packaging>
    <name>build</name>

    <modules>
        <module>parent</module>
        <module>module-a</module>
        <module>module-b</module>
        <module>module-c</module>
    </modules>
</project>
```



---



# 5 mvnw

Maven Wrapper，给某个项目提供一个独立的，指定版本的Maven给它使用，而非全局安装的Maven版本

安装Maven Wrapper：在项目的根目录（即`pom.xml`所在的目录）下运行安装命令`mvn wrapper:wrapper -Dmaven=3.9.0`



---



# 6 发布Artifact

把自己的开源库放到Maven的repo中，供他人引用

略，参考[发布Artifact - Java教程 - 廖雪峰的官方网站](https://liaoxuefeng.com/books/java/maven/deploy/index.html)