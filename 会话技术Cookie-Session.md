---
title: 会话技术Cookie&Session
date: 2020-02-27 12:23:35
tags:
- Java
- HTTP
- Cookie
- Session
categories:
- Java
- JavaWeb
- HTTP
---

## 1.会话技术

**会话**：一次会话中包含多次请求和响应。

- 一次会话：浏览器第一次给服务器资源发送请求，会话建立，直到有一方断开为止。

**功能**：在一次会话的范围内的多次请求间，共享数据。

**方式**：

- 客户端会话技术：**Cookie**
- 服务器端会话技术：**Session**

<!-- more -->

## 2.Cookie

### 2.1 概念

客户端会话技术，将数据保存到客户端。

### 2.2 使用步骤

- 创建Cookie对象，绑定数据。

  ```java
  new Cookie(String name, String value) 
  ```

- 发送Cookie对象。

  ```java
  response.addCookie(Cookie cookie) 
  ```

- 获取Cookie，拿到数据。

  ```java
  Cookie[]  request.getCookies()
  ```

###  2.3 实现原理

基于响应头 set-cookie 和请求头 cookie 实现

### 2.4 cookie 的细节

#### 1.一次发送多个 cookie

- 可以创建多个 Cookie 对象，使用response调用多次 addCookie 方法发送 cookie 即可。

#### 2.cookie 在浏览器中保存多长时间

- 默认情况下，当浏览器关闭后，Cookie 数据被销毁。
- **持久化存储**：setMaxAge(int seconds)
  - 正数：将 Cookie 数据写到硬盘的文件中。持久化存储。并指定 cookie 存活时间，时间到后，cookie 文件自动失效。
  - 负数：默认值。
  - 零：删除cookie信息。

#### 3.cookie 能不能存中文

- 在 tomcat 8 之前，cookie中不能直接存储中文数据。
  - 需要将中文数据转码---一般采用URL编码(%E3)。
- 在 tomcat 8 之后，cookie支持中文数据。特殊字符还是不支持，建议使用URL编码存储，URL解码解析。

#### 4.cookie 共享问题

- 假设在一个 tomcat 服务器中，部署了多个 web 项目，那么在这些 web 项目中 cookie 能不能共享？
  - 默认情况下 cookie 不能共享。
  - setPath(String path)：设置 cookie 的获取范围。默认情况下，设置当前的虚拟目录。如果要共享，则可以将 path 设置为"/"。
- 不同的 tomcat 服务器间 cookie 共享问题？
  - setDomain(String path)：如果设置一级域名相同，那么多个服务器之间cookie可以共享。
    setDomain(".baidu.com")，那么 tieba.baidu.com 和 news.baidu.com 中 cookie 可以共享。

#### 5.Cookie的特点和作用

- 特点：
  - cookie 存储数据在客户端浏览器。
  - 浏览器对于单个 cookie 的大小有限制(4kb) 以及对同一个域名下的总 cookie 数量也有限制(20个)。
- 作用：
  - cookie 一般用于存出少量的不太敏感的数据。
  - 在不登录的情况下，完成服务器对客户端的身份识别。

#### 6.案例：记住上一次访问时间

**需求**：

- 访问一个 Servlet，如果是第一次访问，则提示：您好，欢迎您首次访问。
- 如果不是第一次访问，则提示：欢迎回来，您上次访问时间为:显示时间字符串。

**分析**：

- 可以采用 Cookie 来完成。
- 在服务器中的 Servlet 判断是否有一个名为 lastTime 的cookie：
  - 有：不是第一次访问
    - 响应数据：欢迎回来，您上次访问时间为:2018年6月10日11:50:20
    - 写回 Cookie：lastTime=2018年6月10日11:50:01
  - 没有：是第一次访问
    - 响应数据：您好，欢迎您首次访问
    - 写回 Cookie：lastTime=2018年6月10日11:50:01

## 3.Session

### 3.1 概念

服务器端会话技术，在一次会话的多次请求间共享数据，将数据保存在服务器端的对象中。HttpSession

### 3.2 快速入门

- 1.获取 HttpSession 对象：

```java
HttpSession session = request.getSession();
```

- 2.使用 HttpSession 对象：

```java
Object getAttribute(String name)  
void setAttribute(String name, Object value)
void removeAttribute(String name) 
```

### 3.3 原理

Session 的实现是依赖于 **Cookie** 的。

### 3.4 细节

- 当客户端关闭后，服务器不关闭，两次获取 session 是否为同一个？
  - 默认情况下不是。
  - 如果需要相同，则可以创建 Cookie，键为 JSESSIONID，设置最大存活时间，让 cookie 持久化保存。

```java
Cookie c = new Cookie("JSESSIONID",session.getId());
c.setMaxAge(60*60);
response.addCookie(c);
```

- 客户端不关闭，服务器关闭后，两次获取的 session 是同一个吗？
  - 不是同一个，但是要确保数据不丢失。Tomcat 自动完成以下工作：
    - session 的钝化：在服务器正常关闭之前，将 session 对象系列化到硬盘上。
    - session 的活化：在服务器启动后，将 session 文件转化为内存中的 session 对象即可。
- session 什么时候被销毁？
  - 服务器关闭.
  - session 对象调用 invalidate() 。
  - session 默认失效时间 30分钟。
    - 选择性配置修改

```xml
<session-config>
    <session-timeout>30</session-timeout>
</session-config>
```

### 3.5 Session 的特点

- session 用于存储一次会话的多次请求的数据，存在服务器端。
- session 可以存储任意类型，任意大小的数据。

## 4.Session 与 Cookie 的区别

- session 存储数据在服务器端，Cookie 在客户端。
- session没有数据大小限制，Cookie有。
- session数据安全，Cookie相对于不安全。

