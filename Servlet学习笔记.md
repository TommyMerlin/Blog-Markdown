---
title: Servlet学习笔记
date: 2020-02-25 08:51:04
tags:
- Java
- Servlet
categories:
- Java
- JavaWeb
- Servlet
---

## 1.Servlet 概述

- **概念**：运行在服务器端的小程序。

  - Servlet 就是一个**接口**，定义了 Java 类被浏览器访问（Tomcat识别）的规则。


<!-- more -->

- **快速入门**：

  - 1.创建 JavaEE 项目。
  - 2.定义一个类，实现 Servlet 接口。

  ```java
  public class ServerletDemo1 implements Servlet
  ```

  - 3.实现接口中的抽象方法。
  - 4.配置 Servlet：`web\WEB-INF\web.xml`。

  ```xml
  <servlet>
      <servlet-name>demo1</servlet-name>
      <servlet-class>com.coderhuye.web.servlet.ServerletDemo1</servlet-class>
  </servlet>
  
  <servlet-mapping>
      <servlet-name>demo1</servlet-name>
      <url-pattern>/demo1</url-pattern>
  </servlet-mapping>
  ```

- **执行原理**：

  - 1.当服务器接受到客户端浏览器的请求后，会解析请求URL路径，获取访问的Servlet的资源路径。
  - 2.查找web.xml文件，是否有对应的<url-pattern>标签体内容。
  - 3.如果有，则在找到对应的<servlet-class>全类名。
  - 4.tomcat会将字节码文件加载进内存，并且创建其对象。
  - 5.调用其方法。

---

## 2.Servlet 生命周期

- 1.**创建**：执行 init 方法，仅执行一次。
  - Servlet 什么时候被创建？
    - 默认情况下，第一次访问时被创建。
    - 可以在<servelet>标签下配置指定 Servlet 被创建的时机。
    <img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E9%85%8D%E7%BD%AEServlet%E5%88%9B%E5%BB%BA%E6%97%B6%E6%9C%BA-126753.png" alt="配置Servlet创建时机" width="60%" />
- Servlet 在内存中只有一个对象，是单例的。
    - 多个用户同时访问时，存在线程安全问题。
    - 解决：尽量不在 Servlet 中定义成员变量。即使定义了成员变量，也不要对其修改值。
- 2.**提供服务**：执行 sevice 方法，每访问一次 Servlet ，就会执行一次。
- 3.**销毁**：服务器正常关闭时执行一次。

---

## 3.Servlet3.0
- 支持注解配置
- 步骤：
  - 创建 JavaEE 项目，选择 Servlet 版本 >3.0，可不创建 web.xml。
  - 定义一个 Servlet 实现类。
  - 复写方法
    - 在类上使用 @WebServlet 注解。
      - ```java
        @WebServlet(urlPatterns = "/demo1")
        or
        @WebServlet("/demo1")
        ```

## 4.Servlet 体系结构

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-Servlet%E7%BB%A7%E6%89%BF%E5%85%B3%E7%B3%BB-24f00b.png" alt="Servlet继承关系" width="40%" />

- GenericServlet：将Servlet接口中其他的方法做了默认空实现，只将service()方法作为抽象方法。
- HttpServlet：对http协议的一种封装，简化操作。
  - 定义类继承HttpServlet。
  - 复写doGet/doPost方法。

---

## 5.Servlet 相关配置

- urlPatterns：一个Servlet可以定义多个访问路径 ：
  - ```java
    @WebServlet({"/d4","/demo4","/dd4"})
    ```
- 路径定义规则：
  - /xxx：路径匹配
  - /xxx/xxx：多层路径，目录结构
  - *.do：扩展名匹配