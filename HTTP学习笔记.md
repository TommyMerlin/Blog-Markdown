---
title: HTTP学习笔记
date: 2020-02-25 11:12:48
tags:
- JavaWeb
- HTTP
- Servlet
categories:
- Java
- JavaWeb
- HTTP
---

## 1.概述

**概念**：Hyper Text Transfer Protocol 超文本传输协议。

**传输协议**：定义了客户端和服务器端通信时，发送数据的格式。

**特点**：

- 基于 TCP/IP 的高级协议。
- 默认端口号：80。
- 基于请求/响应模型：一次请求对应一次响应。
- 无状态：每次请求之间相互独立，不能交互数据。

<!-- more -->

**历史版本**：

- 1.0：每一次请求响应都会建立新的连接。
- 1.1：复用连接。

## 2.请求消息数据格式

### 2.1 **请求行**

- 格式：请求方式 请求url 请求协议/版本
- 请求方式：7中，常用2种：
  - GET：
    - 请求参数在请求行中，在url后。
    - 请求的url长度有限制。
    - 不太安全。
  - POST：
    - 请求参数在请求体中。
    - 请求的url长度没有限制。
    - 相对安全。

### 2.2 **请求头**

- 格式：请求头名称：请求头值
- 常见请求头：
  - User-Agent：告诉服务器访问所使用的浏览器版本信息，解决浏览器兼容性问题。
  - Referer：告诉服务器请求来源。
    - 防盗链
    - 统计工作

### 2.3 **请求空行**

- 空行，用于分割POST请求的请求头和请求体。

### 2.4 **请求体（正文）**

- 封装POST请求消息的请求参数。

字符串格式：

> GET /login.html HTTP/1.1 
> Host: localhost:8080 
> Connection: keep-alive 
> Upgrade-Insecure-Requests: 1 
> User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.116 Safari/537.36 Sec-Fetch-Dest: document 
> Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
> Accept-Encoding: gzip, deflate, br 
> Accept-Language: zh-CN,zh;q=0.9,en;q=0.8 
> Referer: http://localhost/login.html

## 3.Request对象

### 3.1 request 继承体系结构

- ServletRequest(接口) <— HttpServletRequest(接口) —org.apache.catalina.connector.RequestFacade 类(tomcat)

### 3.2 request 功能

#### 3.2.1.获取请求消息数据

##### 获取**请求行**数据

- GET /demo/show？name=zhangsan HTTP/1.1
- 方法：
  - 获取请求方式：GET
    - String getMethod()
  - **获取虚拟目录**：/demo
    - String getContextPath()
  - 获取 Servlet 路径 /show
    - String getServletPath()
  - 获取 get 方式请求参数：name=zhangsan
    - String getQueryString()
  - **获取请求URI**：/demo/show
    - String getRequestURI()：/demo/show
    - StringBuffer getRequestURL()：http://localhost/demo/show
    - URI：统一资源标识符。
    - URL:统一资源定位符。
  - 获取版本及协议：HTTP/1.1。
    - String getProtocol()
  - 获取客户机的IP地址。
    - String getRemoteAddr()

##### 获取**请求头**数据

- **String getHeader(String name)**：通过请求头的名称获取请求头的值。
- Enumeration<String> getHeaderNames()：获取所有的请求头名称。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-09-%E8%8E%B7%E5%8F%96%E8%AF%B7%E6%B1%82%E5%A4%B4-0957e7.png" alt="获取请求头" width="50%" />

##### 获取**请求体**数据

- 1.获取流对象。
    - BufferedReader getReader()：获取字符输入流，只能操作字符数据。
    - ServletInputStream getInputStream()：获取字节输入流，可操作所有类型数据。
- 2.从流对象中获取数据。

#### 3.2.2 其他功能

##### 获取请求参数
- 通用方式：get 和 post 请求都可以用。
- String getParameter(String name)：根据参数名称获取参数值    username=zs&password=123
- String[] getParameterValues(String name)：根据参数名称获取参数值的数组  hobby=xx&hobby=game
- Enumeration<String> getParameterNames()：获取所有请求的参数名称
- Map<String,String[]> getParameterMap():获取所有参数的map集合

##### 请求转发
- 一种在服务器内部的资源跳转方式。
- 步骤：
    - 1.通过request对象获取请求转发器对象：
        RequestDispatcher getRequestDispatcher(String path)
    - 2.使用RequestDispatcher对象来进行转发：forward(ServletRequest request, ServletResponse response) 
- 特点：
    - 浏览器地址栏路径不发生变化。
    - 只能转发到当前服务器内部资源中。
    - 请求转发只是一次请求。

##### 共享数据
- 域对象：一个有作用范围的对象，可以在范围内共享数据。
- request域：代表一次请求的范围，一般用于请求转发的多个资源中共享数据。
- 方法：
  - void setAttribute(String name,Object obj)：存储数据。
  - Object getAttitude(String name)：通过键获取值。
  - void removeAttribute(String name)：通过键移除键值对。

##### 获取 ServletContext
- ServletContext getServletContext()

### 3.3 案例：用户登录

#### **需求**

- 1.编写 `login.html` 登录页面。
  username & password 两个输入框
- 2.使用 Druid 数据库连接池技术,操作 mysql ，test01数据库中 stuinfo 表。
- 3.使用 JdbcTemplate 技术封装 JDBC。
- 4.登录成功跳转到 SuccessServlet 展示：登录成功！用户名,欢迎您。
- 5.登录失败跳转到 FailServlet 展示：登录失败，用户名或密码错误。

#### **案例分析**

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-09-%E7%99%BB%E5%BD%95%E6%A1%88%E4%BE%8B%E5%88%86%E6%9E%90-804520.bmp" alt="登录案例分析" width="60%" />

#### **步骤**

- 1.编写 **[login.html](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/web/login.html)** 登陆页面。
- 2.创建数据库环境。
```sql
CREATE DATABASE test01;
USE test01;
CREATE TABLE stuinfo(
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(32) UNIQUE NOT NULL,
    PASSWORD VARCHAR(32) NOT NULL
);
```
- 3.创建实体类 **[User](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/domain/User.java)**。
- 4.编写工具类 **[JDBCUtils](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/util/JDBCUtils.java)**。
- 5.创建包 `com.coderhuye.dao` ，创建类 **[UserDao](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/dao/UserDao.java)**,提供 login 方法。
- 6.编写 **[LoginServlet](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/web/servlet/LoginServlet.java)** 类。
- 7.编写 **[FailServlet](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/web/servlet/FailServlet.java)** 和 **[SuccessServlet](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginTest/src/com/coderhuye/web/servlet/SuccessServlet.java)** 类。
- 8.login.html 中 form 表单的 action 路径的写法。
  - 虚拟目录+Servlet的资源路径
- 9.BeanUtils工具类，简化数据封装。
  - commons-beanutils-1.8.0.jar
  - [使用代码](https://gitee.com/TommyMerlin/code-host/blob/master/Java/LoginServlet.java)
  - 1.JavaBean：标准的Java类
    - 要求：
      - 1.类必须被public修饰。
      - 2.必须提供空参的构造器。
      - 3.成员变量必须使用private修饰。
      - 4.提供公共setter和getter方法。
    - 功能：封装数据。
  - 2.方法：
    - setProperty()
    - getProperty()
    - populate(Object obj , Map map):将map集合的键值对信息，封装到对应的JavaBean对象中。

## 4.响应消息数据格式

### 4.1 响应行

- 组成：协议/版本 响应状态码 状态码描述
- 响应状态码：服务器告诉客户端浏览器本次请求和响应的一个状态。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-09-%E7%8A%B6%E6%80%81%E7%A0%81-df4fa7.png" alt="状态码" width="60%" />

### 4.2 响应头

- 格式：    头名称： 值
- 常见的响应头：
  - Content-Type：服务器告诉客户端本次响应体数据格式以及编码格式。
  - Content-disposition：服务器告诉客户端以什么格式打开响应体数据。
    - in-line：默认值，在当前页面内打开。
    - attachment;filename=xxx：以附件形式打开响应体，文件下载

### 4.3 响应空行

### 4.4 响应体

- 传输的数据

### 响应字符串格式

>HTTP/1.1 200 OK
>Content-Type: text/html;charset=UTF-8
>Content-Length: 101
>Date: Wed, 06 Jun 2018 07:08:42 GMT

## 5.Response对象

### 5.1 功能

#### 设置响应消息

- 设置响应行
  - 格式：HTTP/1.1 200 ok
  - 设置状态码：setStatus(int sc) 
- 设置响应头
  - setHeader(String name, String value) 
- 设置响应体
  - 1.获取输出流
    字符输出流：PrintWriter getWriter()
    字节输出流：ServletOutputStream getOutputStream()
  - 2.使用输出流，将数据输出到客户端浏览器

### 5.2 案例

#### 完成重定向

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-09-%E9%87%8D%E5%AE%9A%E5%90%91-f9a3a2.bmp" alt="重定向" width="80%" />

```java
//访问/responseDemo1，自动跳转到/responseDemo2资源
//1. 设置状态码为302
response.setStatus(302);
//2.设置响应头location
response.setHeader("location","/Response/responseDemo2");

//动态获取虚拟目录
String contextPath = request.getContextPath();
//简单的重定向方法
response.sendRedirect(contextPath+"/responseDemo2");
//response.sendRedirect("https://tommymerlin.gitee.io/");
```

##### **重定向**的特点：redirect

- 地址栏发生变化。
- 重定向可以访问其他站点(服务器)的资源。
- 重定向是**两次请求**，不能使用request对象来共享数据。

##### **转发**的特点：forward

- 转发地址栏路径不变。
- 转发只能访问当前服务器下的资源。
- 转发是**一次请求**，可以使用request对象来共享数据。

##### 路径写法：

- 路径分类：
  - **相对路径**：通过相对路径不可以确定唯一资源。
    如：./index.html  不以/开头，以 . 开头
    - 规则：找到当前资源和目标资源之间的相对位置关系。
      - ./：当前目录
      - ../：后退一级目录
  - **绝对路径**：通过绝对路径可以确定唯一资源。
    如：http://localhost/day15/responseDemo2
    ​        /day15/responseDemo2
    - 规则：判断定义的路径是给谁用的？判断请求将来从哪发出。
      - 给客户端浏览器使用：需要加虚拟目录(项目的访问路径)。
        - 建议虚拟目录动态获取：request.getContextPath()
      - 给服务器使用：不需要加虚拟目录。
        - 转发路径

#### 服务器输出字符数据到浏览器
步骤
- 获取字符输出流
- 输出数据

乱码问题
- PrintWriter pw = response.getWriter();获取的流的默认编码是ISO-8859-1。
- 设置编码（在获取流之前设置）。
```java
  response.setContentType("text/html;charset=utf-8");
  PrintWriter writer = response.getWriter();
  writer.write("<h1>Hello Java</h1><br><a href='http://www.baidu.com'>百度</a>");
```

  

#### 服务器输出字节数据到浏览器

步骤
- 获取字节输出流
- 输出数据

```java
response.setContentType("text/html;charset=utf-8");
//1.获取字节输出流
ServletOutputStream sos = response.getOutputStream();
//2.输出数据
sos.write("你好".getBytes("utf-8"));
```

#### 验证码
[代码](https://gitee.com/TommyMerlin/code-host/blob/master/Java/CheckCodeServlet.java)

## 6.ServletContext对象
### 6.1 概念

代表整个 web 应用，可以和程序的容器(服务器)来通信

### 6.2 获取

- 通过 **request** 对象获取。
  - request.getServletContext();
- 通过 **HttpServlet** 获取。
  - this.getServletContext();

### 6.3 功能

- 获取 **MIME** 类型：
  - MIME 类型:在互联网通信过程中定义的一种文件数据类型。
  - 格式： 大类型/小类型   text/html		image/jpeg
  - 获取：String getMimeType(String file)

- 域对象：**共享数据**
  - setAttribute(String name,Object value)
  - getAttribute(String name)
  - removeAttribute(String name)
  - ServletContext **对象范围**：所有用户所有请求的数据

- 获取文件的**真实(服务器)路径**

  ```java
  String getRealPath(String path)  
  String b = context.getRealPath("/b.txt");                //web目录下资源访问
  System.out.println(b);
  	
  String c = context.getRealPath("/WEB-INF/c.txt");        //WEB-INF目录下的资源访问
  System.out.println(c);
  	
  String a = context.getRealPath("/WEB-INF/classes/a.txt");//src目录下的资源访问
  System.out.println(a);
  ```

### 6.4 案例

#### 文件下载需求

- 页面显示超链接。
- 点击超链接后弹出下载提示框。
- 完成图片文件下载。

#### 分析

- 超链接指向的资源如果能够被浏览器解析，则在浏览器中展示，如果不能解析，则弹出下载提示框-->**不满足需求**
- 任何资源都必须弹出下载提示框。
- 使用响应头设置资源的打开方式：
  		* content-disposition:attachment;filename=xxx

#### 步骤

- 1.定义页面，编辑超链接 href 属性，指向 Servlet，传递资源名称 filename。
- 2.定义Servlet。
  - a.获取文件名称
  - b.使用字节输入流加载文件进内存
  - c.指定response的响应头
    content-disposition:attachment;filename=xxx
  - d.将数据写出到response输出流

#### 中文文件名问题

- 获取客户端使用的浏览器版本信息。
- 根据不同的版本信息，设置filename的编码方式不同。

#### 代码

> html文件

```html
<a href="/Response/downloadServlet?filename=1.jpg">图片1</a>
```

> DownloadServlet.java

```java
//1.获取请求参数，文件名称
String filename = request.getParameter("filename");
//2.使用字节输入流加载文件进内存
//2.1找到文件服务器路径
ServletContext servletContext = this.getServletContext();
String realPath = servletContext.getRealPath("/img/" + filename);
//2.2用字节流关联
FileInputStream fis = new FileInputStream(realPath);

//3.设置response的响应头
//3.1设置响应头类型：content-type
String mimeType = servletContext.getMimeType(filename);  //获取文件的mime类型
response.setHeader("content-type",mimeType);  //3.2设置响应头打开方式:content-disposition

//解决中文文件名问题
//1.获取user-agent请求头、
String agent = request.getHeader("user-agent");
//2.使用工具类方法编码文件名即可
filename = DownLoadUtils.getFileName(agent, filename);

response.setHeader("content-disposition","attachment;filename="+filename);
//4.将输入流的数据写出到输出流中
ServletOutputStream sos = response.getOutputStream();
byte[] buff = new byte[1024 * 8];
int len = 0;
while((len = fis.read(buff)) != -1){
    sos.write(buff,0,len);
}

fis.close();
```

> 工具类

```java
public class DownLoadUtils {

    public static String getFileName(String agent, String filename) throws 
        UnsupportedEncodingException {
        if (agent.contains("MSIE")) {
            // IE浏览器
            filename = URLEncoder.encode(filename, "utf-8");
            filename = filename.replace("+", " ");
        } else if (agent.contains("Firefox")) {
            // 火狐浏览器
            BASE64Encoder base64Encoder = new BASE64Encoder();
            filename = "=?utf-8?B?" + base64Encoder.encode(
                filename.getBytes("utf-8")) + "?=";
        } else {
            // 其它浏览器
            filename = URLEncoder.encode(filename, "utf-8");
        }
        return filename;
    }
}
```

