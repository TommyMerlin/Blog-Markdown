---
title: Spring Boot学习笔记1:Spring Boot入门
date: 2020-03-06 09:20:08
tags:
- Java
- Spring Boot
categories:
- Java
- Spring
- Spring Boot
---

## 1.Spring Boot 简介

> 简化Spring应用开发的一个框架；
>
> 整个Spring技术栈的一个大整合；
>
> J2EE开发的一站式解决方案；

<!-- more -->

## 2.微服务

2014，martin fowler

微服务：架构风格（服务微化）

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

单体应用：ALL IN ONE

微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

[微服务文档](https://martinfowler.com/articles/microservices.html#MicroservicesAndSoa) 

[中文版](http://blog.cuicc.com/blog/2015/07/22/microservices/)

## 3.Spring Boot HelloWorld

实现功能：浏览器发送 hello 请求，服务器接受请求并处理，响应 Hello World 字符串；

### 3.1 创建一个maven工程（jar）

### 3.2 导入spring boot相关的依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

### 3.3 编写一个主程序，启动Spring Boot应用

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

### 3.4 编写相关的Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World!";
    }
}
```

### 3.5 运行主程序测试

### 3.6 简化部署

```xml
<!-- 这个插件，可以将应用打包成一个可执行的jar包；-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```

将这个应用打成 jar 包，直接使用 `java -jar` 的命令进行执行。

## 4.Hello World探究

### 4.1 POM文件

#### 父项目

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.9.RELEASE</version>
</parent>

他的父项目是
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-dependencies</artifactId>
  <version>1.5.9.RELEASE</version>
  <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
该父项目来真正管理Spring Boot应用里面的所有依赖版本；
```

Spring Boot 的**版本仲裁中心**；
以后我们**导入依赖默认不需要写版本**；（没有在dependencies里面管理的依赖自然需要声明版本号）

#### 启动器

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

**spring-boot-starter-*web***：
- spring-boot-starter：spring-boot **场景启动器**；帮我们导入了 web 模块正常运行所依赖的组件。
Spring Boot将所有的功能场景都抽取出来，做成一个个的 starters（启动器），只需要在项目里面引入这些 starter 相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器。

### 4.2 主程序类，主入口类

```java
/**
 *  @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 */
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

        // Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

@**SpringBootApplication**:    Spring Boot 应用标注在某个类上说明这个类是 SpringBoot 的主配置类， SpringBoot 就应该运行这个类的 main 方法来启动 SpringBoot 应用；

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

@**SpringBootConfiguration**：Spring Boot的配置类，标注在某个类上，表示这是一个Spring Boot的配置类。
- @**Configuration**：配置类上来标注这个注解，配置类代替配置文件；配置类也是容器中的一个组件。
- @**EnableAutoConfiguration**：开启自动配置功能。

  ```java
  @AutoConfigurationPackage
  @Import(EnableAutoConfigurationImportSelector.class)
  public @interface EnableAutoConfiguration {
  ```

  - @**AutoConfigurationPackage**：自动配置包。
    - @**Import**(AutoConfigurationPackages.Registrar.class)：Spring的底层注解 **@Import**，给容器中导入一个组件；导入的组件由AutoConfigurationPackages.Registrar.class 将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器。

  - @**Import**(EnableAutoConfigurationImportSelector.class)：给容器中导入组件。
    **EnableAutoConfigurationImportSelector**：导入哪些组件的选择器，将所有需要导入的组件以全类名的方式返回，这些组件就会被添加到容器中。最终会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件。有了自动配置类，免去了我们手动编写配置注入功能组件等的工作。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE%E7%B1%BB-d4fd7b.png" alt="自动配置类" width="60%" />

## 5.使用 Spring Initializer 快速创建 Spring Boot 项目

### IDEA：使用 Spring Initializer 快速创建项目

选择我们需要的模块，向导会联网创建 Spring Boot 项目。
默认生成的 Spring Boot 项目：
- 主程序已经自动生成。
- resources 文件夹中目录结构
  - static：保存所有的静态资源； js css  images。
  - templates：保存所有的模板页面；（Spring Boot 默认 jar 包使用嵌入式的 Tomcat，默认不支持 JSP 页面）；可以使用模板引擎（freemarker、thymeleaf）。
  - application.properties：Spring Boot 应用的配置文件；可以修改一些默认设置。
