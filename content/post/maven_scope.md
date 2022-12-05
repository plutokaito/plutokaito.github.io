---
title: "初步了解 Maven Scope"
date: 2022-12-02
tags: ["maven"]
description : "初步了解 Maven 的作用范围和生命周期"
---
该篇文章主要是介绍了 maven 的 scope 和 lifestyle。

# Scope (作用范围)

pom.xml 的使用
```xml
		<dependency>
			<groupId>io.kaitoshy</groupId>
			<artifactId>biz-common</artifactId>
			<!-- 这里的 scope  有以下几种 -->
			<scope>provided</scope>
		</dependency>
```

## compile
默认的作用范围，全依赖，全传递

## provided
和 compile 很像，用于 test 和 compile 时需要的 classpath，常用于像 Servlet-Api 这种**只有接口没有具体实现的 jar 包**中，具体实现需要在应用启动的时候提供。不会传递。

## runtime
运行时需要，用于 test 和 runtime 时需要用的 classpath, 而编译的时候不需要。runtime 的时候一般由运行容器提供，如 Tomcat 这种。

## test
例如
```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

正常使用的 App 不需要这个，运行 `src/test/java` 时的代码时需要，也是非传递，比如 JUnit 和 Mockito。 只有 test 代码编译和执行时需要。

## system

使用本地的 jar 包文件，和 provided 一样，只不过不需要在本地库查找。

```xml
		<dependency>
			<groupId>io.kaioshy</groupId>
			<artifactId>sss-core</artifactId>
			<scope>system</scope>
			<systemPath>C:/ffff</systemPath>
		</dependency>
```

- systemPath 为具体的 jar 位置

## import

使用
```xml
<properties>
    <spring.cloud-version>2021.0.5</spring.cloud-version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>${spring.cloud-version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

定义了在实际 `<dependencies>` 中依赖，但不会影响传递依赖。


# Optional 和 Exclusion

### Exclusion
```XML
 <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-websocket</artifactId>
      <version>9.0.65</version>
      <scope>compile</scope>
      <exclusions>
        <exclusion>
          <artifactId>tomcat-annotations-api</artifactId>
          <groupId>org.apache.tomcat</groupId>
        </exclusion>
      </exclusions>
    </dependency>
```
排除之外

### Optional
相当于默认排除，只有依赖方也依赖了这个包才生效

```XML
		<dependency>
			<groupId>org.apache</groupId>
			<artifactId>apache</artifactId>
			<version>23</version>
			<optional>false</optional>
		</dependency>
```

这个让我想起了公司里在 Common 包中引用 ElasticSearch 相关的包


# 分析命令
`mvn dependency:tree` 和 `mvn dependency:analyze`


# Lifecycle(生命周期)

以下为 idea 中的几个生命周期，这个并不能很好的反映 maven 的生命周期，这个比较常用，但有些省略。

## clean
清空之前构建的包

## validate
验证项目的正确性和所有必要信息的可用性

## compile
将 Java 文件编译成 Class 文件。

## test
用合适的测试框架进行测试，不需要打包和部署，执行时间在 package 之前

## package
将编译后文件打包成合适的格式，如 Jar，war 等

## verify
验证每一条代码准则和 `package` 之后包的有效性，如 checkStyle 这块的检查，详见 [checkStyle 的 Maven 设置](/post/maven_check)

## install
将 `package` 后的包文件部署到本地，供其他项目使用。和 deploy 只能选择一个作为最终执行者。

## site
生成项目的网站文件，有点类似于 Java Doc。

## deploy
将 `package` 后的包部署到远程仓库中



# 参考链接
> https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html
>
> https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html