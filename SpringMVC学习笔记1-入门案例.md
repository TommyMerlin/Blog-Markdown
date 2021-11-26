---
title: SpringMVC学习笔记1:入门案例
date: 2020-03-05 10:12:56
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
---

## 1.MVC 模型

MVC 全名是 Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，是一种用于设计创建 Web 应用程序**表现层**的模式。

- **Model**（模型）
  - 通常指的就是我们的数据模型。作用一般情况下用于封装数据。
- **View**（视图）
  - 通常指的就是我们的jsp或者html。作用一般就是展示数据。
  - 通常视图是依据模型数据创建的。
- **Controller**（控制器）
  - 是应用程序中处理用户交互的部分。作用一般就是处理程序逻辑的。

<!-- more -->

## 2.SpringMVC 概述

SpringMVC 是一种基于 Java 的实现 MVC 设计模型的请求驱动类型的轻量级 Web 框架，属于 Spring FrameWork 的后续产品，已经融合在 Spring Web Flow 里面。Spring 框架提供了构建 Web 应用程序的全功能 MVC 模块。

## 3.SpringMVC 入门案例

#### 3.1 创建 WEB 工程，引入开发的 jar 包

- `pom.xml`

```xml
<!-- 版本锁定 -->
<properties>
	<spring.version>5.0.2.RELEASE</spring.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-web</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>${spring.version}</version>
    </dependency>
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>javax.servlet.jsp</groupId>
        <artifactId>jsp-api</artifactId>
        <version>2.0</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

#### 3.2 配置核心的控制器（配置 DispatcherServlet）

- `web.xml`

```xml
<!-- SpringMVC的核心控制器 -->
<servlet>
    <servlet-name>dispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servletclass>
    <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:springmvc.xml</param-value>
    </init-param>
    <!-- 配置servlet启动时加载对象 -->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

#### 3.3 编写 springmvc.xml 的配置文件

- `springmvc.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/mvc
    http://www.springframework.org/schema/mvc/spring-mvc.xsd
    http://www.springframework.org/schema/context
    http://www.springframework.org/schema/context/spring-context.xsd">
    
    <!-- 配置spring创建容器时要扫描的包 -->
    <context:component-scan base-package="com.itheima"></context:component-scan>
    
    <!-- 配置视图解析器 -->
    <bean id="viewResolver"
    class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"></property>
        <property name="suffix" value=".jsp"></property>
    </bean>
    
    <!-- 配置spring开启注解mvc的支持
    <mvc:annotation-driven></mvc:annotation-driven>-->
</beans>
```

#### 3.4 编写 index.jsp 和 HelloController 控制器类

- `index.jsp`

```html
<body>
    <h3>入门案例</h3>
    <a href="${ pageContext.request.contextPath }/hello">入门案例</a>
</body>
```

- `HelloController`

```java
// 控制器
@Controller
public class HelloController {
    
    // 接收请求
    @RequestMapping(path="/hello")
    public String sayHello() {
        System.out.println("Hello SpringMVC!!");
        return "success";
    }
}
```

#### 3.5 在 WEB-INF 目录下创建 pages 文件夹，编写 success.jsp 的成功页面

```html
<body>
	<h3>入门成功！！</h3>
</body>
```

#### 3.6 启动Tomcat服务器，进行测试

## 4.入门案例的执行过程分析

### 4.1 入门案例的执行流程

- 1.当启动 Tomcat 服务器的时候，因为配置了 load-on-startup 标签，所以会创建 DispatcherServlet 对象，就会加载 springmvc.xml 配置文件。
- 2.开启了注解扫描，那么 HelloController 对象就会被创建
- 3.从 index.jsp 发送请求，请求会先到达 DispatcherServlet 核心控制器，根据配置 @RequestMapping 注解找到执行的具体方法。
- 4.根据执行方法的返回值，再根据配置的视图解析器，去指定的目录下查找指定名称的 JSP 文件。
- 5.Tomcat 服务器渲染页面，做出响应。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-03-a700c7.bmp" alt="03" width="70%" />

### 4.2 入门案例设计组件

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-04-f2ae19.bmp" alt="04" width="80%" />

- **DispatcherServlet**：前端控制器
  - 用户请求到达前端控制器，它就相当于 mvc 模式中的 c ，dispatcherServlet 是整个流程控制的中心，由它调用其它组件处理用户的请求，dispatcherServlet 的存在降低了组件之间的耦合性。
- **HandlerMapping**：处理器映射器
  - HandlerMapping 负责根据用户请求找到 Handler 即处理器，SpringMVC 提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。
- **Handler**：处理器
  - 它就是我们开发中要编写的具体业务控制器。由 DispatcherServlet 把用户请求转发到 Handler。由 Handler 对具体的用户请求进行处理。
- **HandlAdapter**：处理器适配器
  - 通过 HandlerAdapter 对处理器进行执行，这是适配器模式的应用，通过扩展适配器可以对更多类型的处理器进行执行。
- **View Resolver**：视图解析器
  - View Resolver 负责将处理结果生成 View 视图，View Resolver 首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成 View 视图对象，最后对 View 进行渲染将处理结果通过页面展示给用户。
- **View**：视图
  - SpringMVC 框架提供了很多的 View 视图类型的支持，包括：jstlView、freemarkerView、pdfView 等。我们最常用的视图就是 jsp。
- `<mvc:annotation-driven>` 标签说明
  - 在SpringMVC 的各个组件中，处理器映射器、处理器适配器、视图解析器称为 SpringMVC 的三大组件。
  - 使用 `<mvc:annotation-driven>` 自动加载 RequestMappingHandlerMapping（处理映射器）和 RequestMappingHandlerAdapter（处理适配器），可用在 SpringMVC.xml 配置文件中使用 `<mvc:annotation-driven>` 替代注解处理器和适配器的配置。
  - 它就相当于在xml中配置了：

```xml
<!-- 上面的标签相当于 如下配置--> 
<!-- HandlerMapping --> 
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping"></bean> 
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"></bean> 

<!-- HandlerAdapter --> 
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter"></bean> 
<bean class="org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter"></bean> 
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"></bean> 

<!-- HadnlerExceptionResolvers --> 
<bean class="org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver"></bean> 
<bean class="org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver"></bean> 
<beanclass="org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver"></bean>
```

## 5.RequestMapping注解

RequestMapping 注解的作用是建立请求 URL 和处理方法之间的对应关系。

RequestMapping注解可以作用在方法和类上：

- 作用在**类**上：**第一级**的访问目录。
- 作用在**方法**上：**第二级**的访问目录。

**注意**：

> 路径可以不编写 / 表示应用的根目录开始
>
> ${ pageContext.request.contextPath } 也可以省略不写，但是路径上不能写 /

RequestMapping 的**属性**：

- path：指定请求路径的url
- value：value属性和path属性是一样的
- mthod：指定该方法的请求方式
- params：指定限制请求参数的条件
- headers：发送的请求中必须包含的请求头