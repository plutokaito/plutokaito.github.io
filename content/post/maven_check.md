---
title: "如何在 MAVEN 项目中 checkstyle"
date: 2022-03-12
tags: ["代码规范", "maven-plugin"]
description : "如何在 MAVEN 项目中 checkstyle"
---
本文章是记录了怎么简单的通过 MAVEN 的 checkstyle 插件进行代码规范的管理。

## 前言
众所周知，代码规范是软件开发中一项比较重要的事情，那么怎么落地呢？有人说 code-review，也有人说写代码规范文档按照文档上执行即可。以上都是不错的办法，但这些都是靠人工的方式去保持的，这里介绍一种比较简单又容易执行的方法，就是利用 MAVEN 的 checkstyle 插件去执行。

## 需求
在使用 `maven clean package` 时，可以自动检测代码是否相应的规范。

## 执行
我们通过查看 [Apache Maven Checkstyle Plugin - Usage](https://maven.apache.org/plugins/maven-checkstyle-plugin/usage.html) 下有一条：`Checking for Violations as Part of the Build` (检查规范作为构建的一部分) ，那么就使用该下面的示例，在 pom.xml 文件中增加

```xml
<build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-checkstyle-plugin</artifactId>
                <version>3.1.2</version>
                <dependencies>
                    <dependency>
                        <!-- 指定 checkstyle 的版本 -->
                        <groupId>com.puppycrawl.tools</groupId>
                        <artifactId>checkstyle</artifactId>
                        <version>8.41</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <!-- configLocation 表示的是 checkstyle.xml 所在项目中的位置 -->
                    <configLocation>checkstyle.xml</configLocation>
                    <encoding>UTF-8</encoding>
                    <!-- 是否在控制台处输出 -->
                    <consoleOutput>true</consoleOutput>
                    <!-- 是否在错误处失败 -->
                    <failsOnError>true</failsOnError>
                </configuration>
                <executions>
                    <!-- 执行 -->
                    <execution>
                        <!-- 执行的标识 -->
                        <id>validate</id>
                        <!-- 在构建生命周期内，执行目标的绑定对象 -->
                        <phase>validate</phase>
                        <!-- 执行目标 -->
                        <goals>
                            <goal>check</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

在项目根目录新建 `checkstyle.xml`, 在该文件下输入：

```xml
<!DOCTYPE module PUBLIC
        "-//Checkstyle//DTD Checkstyle Configuration 1.3//EN"
        "https://checkstyle.org/dtds/configuration_1_3.dtd">
<!-- 每个 checkstyle 文件都有个 Checker 的根节点 -->
<module name="Checker">
    <module name="TreeWalker">
        <!-- 未使用的导入 -->
        <module name="UnusedImports" />
    </module>
</module>
```
这里查看 [checks](https://checkstyle.org/checks.html) 的介绍结合公司的业务需要去配置。几个可以参照的对象: [dubbo](https://github.com/apache/dubbo/blob/3.0/codestyle/checkstyle.xml), [shenyu_checkstyle.xml](https://github.com/apache/incubator-shenyu/blob/master/script/shenyu_checkstyle.xml)。配置后使用命令 `mvn clean install -D maven.test.skip` 时观察日志会有以下输出

```log
[INFO] --- maven-checkstyle-plugin:3.1.2:check (validate) @ case-demo ---
[INFO] 开始检查……
检查完成。
[INFO] You have 0 Checkstyle violations.
```
这说明 checkstyle 插件已经起效果了。



