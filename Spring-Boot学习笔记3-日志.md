---
title: Spring Boot学习笔记3:日志
date: 2020-03-06 15:03:11
tags:
- Java
- Spring Boot
- 日志
categories:
- Java
- Spring
- Spring Boot
---

## 1.日志框架

**市面上的日志框架；**
JUL、JCL、Jboss-logging、logback、log4j、log4j2、slf4j....

| 日志门面  （日志的抽象层）                                   | 日志实现                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| ~~JCL（Jakarta  Commons Logging）~~  <br/>SLF4j（Simple  Logging Facade for Java）<br/> ~~jboss-logging~~ | Log4j <br/> JUL（java.util.logging）<br/> Log4j2 <br/> **Logback** |

<!-- more -->

左边选一个门面（抽象层）、右边来选一个实现；

- 日志门面：  SLF4J；
- 日志实现：Logback；

SpringBoot：底层是 Spring 框架，Spring 框架默认是用 JCL。
**SpringBoot 选用 SLF4j 和 logback**

## 2.SLF4j使用

### 2.1 如何在系统中使用SLF4j   https://www.slf4j.org

以后开发的时候，日志记录方法的调用，不应该来直接调用日志的实现类，而是调用日志抽象层里面的方法；
给系统里面导入 slf4j 的 jar 和 logback 的实现 jar。
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class HelloWorld {
  public static void main(String[] args) {
    Logger logger = LoggerFactory.getLogger(HelloWorld.class);
    logger.info("Hello World");
  }
}
```

图示
![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-slf4j-f56eba.png)

每一个日志的实现框架都有自己的配置文件。使用slf4j以后，**配置文件还是做成日志实现框架自己本身的配置文件。**

### 2.2 遗留问题

（slf4j+logback）: Spring（commons-logging）、Hibernate（jboss-logging）、MyBatis、xxxx
统一日志记录，即使是别的框架如何让一起统一使用 slf4j 进行输出？
![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-legacy-912d7c.png)

**如何让系统中所有的日志都统一到 slf4j**

- 1.将系统中其他日志框架先排除出去；
- 2.用中间包来替换原有的日志框架；
- 3.导入 slf4j 其他的实现。

## 3.SpringBoot日志关系

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

SpringBoot 使用它来做日志功能；

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-logging</artifactId>
</dependency>
```

**底层依赖关系**

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0-6f9465.png" alt="底层实现" width="50%" />

**总结**

- SpringBoot 底层也是使用 slf4j+logback 的方式进行日志记录。
- SpringBoot 把其他的日志都替换成了 slf4j。
- 如果要引入其他框架，一定要把这个框架的默认日志依赖移除掉。
  - Spring 框架用的是 commons-logging；

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-core</artifactId>
    <exclusions>
        <exclusion>
            <groupId>commons-logging</groupId>
            <artifactId>commons-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

**SpringBoot能自动适配所有的日志，而且底层使用 slf4j+logback 的方式记录日志，引入其他框架的时候，只需要把这个框架依赖的日志框架排除掉即可。**

## 4.日志使用

### 4.1 默认配置

SpringBoot 默认帮我们配置好了日志；

```java
//记录器
Logger logger = LoggerFactory.getLogger(getClass());
@Test
public void contextLoads() {
    //日志的级别；
    //由低到高   trace<debug<info<warn<error
    //可以调整输出的日志级别；日志就只会在这个级别以以后的高级别生效
    logger.trace("这是trace日志...");
    logger.debug("这是debug日志...");
    //SpringBoot默认给我们使用的是info级别的，没有指定级别的就用SpringBoot默认规定的级别
    logger.info("这是info日志...");
    logger.warn("这是warn日志...");
    logger.error("这是error日志...");
}
```

    日志输出格式：
    %d表示日期时间，
    %thread表示线程名，
    %-5level：级别从左显示5个字符宽度
    %logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
    %msg：日志消息，
    %n是换行符
    -->
    %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n

SpringBoot 修改日志的默认配置

```properties
logging.level.com.atguigu=trace

# 不指定路径在当前项目下生成springboot.log日志
# 可以指定完整的路径；
#logging.file.name=G:/springboot.log

# 在当前磁盘的根路径下创建spring文件夹和里面的log文件夹；使用 spring.log 作为默认文件
logging.path=/spring/log

# 在控制台输出的日志的格式
logging.pattern.console=%d{yyyy-MM-dd} [%thread] %-5level %logger{50} - %msg%n
# 指定文件中日志输出的格式
logging.pattern.file=%d{yyyy-MM-dd} === [%thread] === %-5level === %logger{50} ==== %msg%n
```

| logging.file.name | logging.file.path | Example  | Description                        |
| ----------------- | ----------------- | -------- | ---------------------------------- |
| (none)            | (none)            |          | 只在控制台输出                     |
| 指定文件名        | (none)            | my.log   | 输出日志到my.log文件               |
| (none)            | 指定目录          | /var/log | 输出到指定目录的 spring.log 文件中 |

### 4.2 指定配置

给类路径下放上每个日志框架自己的配置文件即可；SpringBoot 就不使用默认配置了。

| Logging System          | Customization                                                |
| ----------------------- | ------------------------------------------------------------ |
| Logback                 | `logback-spring.xml`, `logback-spring.groovy`, `logback.xml` or `logback.groovy` |
| Log4j2                  | `log4j2-spring.xml` or `log4j2.xml`                          |
| JDK (Java Util Logging) | `logging.properties`                                         |

logback.xml：直接就被日志框架识别了；

[**logback-spring.xml**](https://gitee.com/TommyMerlin/code-host/blob/master/Java/springboot/logback-spring.xml)：日志框架就不直接加载日志的配置项，由 SpringBoot 解析日志配置，可以使用SpringBoot 的高级 Profile 功能

```xml
<springProfile name="staging">
    <!-- configuration to be enabled when the "staging" profile is active -->
  	可以指定某段配置只在某个环境下生效
</springProfile>
```

如：

```xml
<appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <!--
        日志输出格式：
			%d表示日期时间，
			%thread表示线程名，
			%-5level：级别从左显示5个字符宽度
			%logger{50} 表示logger名字最长50个字符，否则按照句点分割。 
			%msg：日志消息，
			%n是换行符
        -->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <springProfile name="dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ----> [%thread] ---> %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
            <springProfile name="!dev">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} ==== [%thread] ==== %-5level %logger{50} - %msg%n</pattern>
            </springProfile>
        </layout>
    </appender>
```

如果使用 logback.xml 作为日志配置文件，还要使用profile功能，会有以下错误

 `no applicable action for [springProfile]`

## 5.切换日志框架

可以按照 slf4j 的日志适配图，进行相关的切换。

**slf4j+log4j 的方式**

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
  <exclusions>
    <exclusion>
      <artifactId>logback-classic</artifactId>
      <groupId>ch.qos.logback</groupId>
    </exclusion>
    <exclusion>
      <artifactId>log4j-over-slf4j</artifactId>
      <groupId>org.slf4j</groupId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>slf4j-log4j12</artifactId>
</dependency>

```

**切换为 log4j2**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <artifactId>spring-boot-starter-logging</artifactId>
            <groupId>org.springframework.boot</groupId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```
