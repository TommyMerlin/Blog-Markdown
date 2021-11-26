---
title: Maven中Scope的区别
date: 2020-02-29 18:24:17
tags:
- Maven
- Java
categories:
- Java
- Maven
---

## compile

编译范围，不设置**默认**为 compile，在工程环境的classpath（编译环境）和打包（如果是WAR包，会包含在WAR包中）时候都有效，属于**强依赖**。

## test

该依赖仅仅参与**测试**相关的内容，包括测试用例的编译和执行，一般是单元测试场景使用，在编译环境加入classpath，但打包时不会加入，如 junit 等。

<!-- more -->

## provided

容器或 JDK 已提供范围，表示该依赖包已经由目标容器（如tomcat）和 JDK 提供，只在编译的 classpath 中加载和使用，打包的时候不会包含在目标包中。最常见的是 j2ee 规范相关的 servlet-api 和 jsp-api 等 jar 包，一般由 servlet 容器提供，无需再打包到 war 包中，如果不配置为 provided ，把这些包打包到工程 war 包中，在 tomcat6 以上版本会出现冲突无法正常运行程序（版本不符的情况）。

## runtime

一般是运行和测试环境使用，编译时候不用加入 classpath，打包时候会打包到目标包中。一般是通过动态加载或接口反射加载的情况比较多。一般这种类库都是接口与实现相分离的类库，比如 JDBC 类库，在编译之时仅依赖相关的接口，在具体的运行之时，才需要具体的 mysql、oracle 等等数据的驱动程序。

## system

使用上与provided相同，不同之处在于该依赖不从maven仓库中提取，而是从本地文件系统中提取，其会参照systemPath的属性进行提取依赖。

```xml
<dependency>
    <groupId>sun.jdk</groupId>
    <artifactId>tools</artifactId>
    <version>1.5.0</version>
    <scope>system</scope>
    <systemPath>${java.home}/../lib/tools.jar</systemPath>
</dependency>
```



| 依赖范围 | 对于编译有效 | 对于测试有效 | 对于运行时有效 | 例子                  |
| -------- | :----------: | :----------: | :------------: | :-------------------- |
| compile  |      Y       |      Y       |       Y        | spring-core           |
| test     |      —       |      Y       |       —        | Junit                 |
| provided |      Y       |      Y       |       —        | servlet-api           |
| runtime  |      —       |      Y       |       Y        | JDBC驱动              |
| system   |      Y       |      Y       |       —        | Maven仓库之外的本地库 |

