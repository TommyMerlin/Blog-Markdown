---
title: SpringMVC学习笔记4:响应数据和结果视图
date: 2020-03-05 15:01:20
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
---

## 1.返回值分类

### 返回字符串

Controller 方法返回字符串可以指定逻辑视图的名称，根据视图解析器为物理视图的地址。

```java
@RequestMapping(value="/hello")
public String sayHello() {
    System.out.println("Hello SpringMVC!!");
    // 跳转到XX页面
    return "success";
}
```

<!-- more -->

### 返回值是 void

如果控制器的方法返回值编写成 void，执行程序报 404 的异常，默认查找 JSP 页面没有找到。

- 默认会跳转到@RequestMapping(value="/initUpdate") initUpdate的页面。

可以使用请求转发或者重定向跳转到指定的页面。

```java
@RequestMapping(value="/initAdd")
public void initAdd(HttpServletRequest request,HttpServletResponse response) throws
Exception {
    System.out.println("请求转发或者重定向");
    // 请求转发
    // request.getRequestDispatcher("/WEB-INF/pages/add.jsp").forward(request,
    response);
    // 重定向
    // response.sendRedirect(request.getContextPath()+"/add2.jsp");
    response.setCharacterEncoding("UTF-8");
    response.setContentType("text/html;charset=UTF-8");
    // 直接响应数据
    response.getWriter().print("你好");
    return;
}
```

### 返回值是 ModelAndView 对象

ModelAndView 对象是 Spring 提供的一个对象，可以用来调整具体的 JSP 视图。

```java
/**
* 返回ModelAndView对象
* 可以传入视图的名称（即跳转的页面），还可以传入对象。
* @return
* @throws Exception
*/
@RequestMapping(value="/findAll")
public ModelAndView findAll() throws Exception {
    ModelAndView mv = new ModelAndView();
    // 跳转到list.jsp的页面
    mv.setViewName("list");
    // 模拟从数据库中查询所有的用户信息
    List<User> users = new ArrayList<>();
    User user1 = new User();
    user1.setUsername("张三");
    user1.setPassword("123");
    User user2 = new User();
    user2.setUsername("赵四");
    user2.setPassword("456");
    users.add(user1);
    users.add(user2);
    // 添加对象
    mv.addObject("users", users);
    return mv;
}
```

```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <h3>查询所有的数据</h3>
    <c:forEach items="${ users }" var="user">
    ${ user.username }
    </c:forEach>
</body>
</html>
```

## 2.SpringMVC 框架提供的转发和重定向

### forward 请求转发

controller 方法返回 String 类型，想进行请求转发也可以编写成

```java
/**
* 使用forward关键字进行请求转发
* "forward:转发的JSP路径"，不走视图解析器了，所以需要编写完整的路径
* @return
* @throws Exception
*/
@RequestMapping("/delete")
public String delete() throws Exception {
    System.out.println("delete方法执行了...");
    // return "forward:/WEB-INF/pages/success.jsp";
    return "forward:/user/findAll";
}
```

### redirect 重定向

controller 方法返回 String 类型，想进行重定向也可以编写成

```java
/**
* 重定向
* @return
* @throws Exception
*/
@RequestMapping("/count")
public String count() throws Exception {
	System.out.println("count方法执行了...");
    return "redirect:/add.jsp";
    // return "redirect:/user/findAll";
}
```

## 3.ResponseBody 响应 json 数据

1.DispatcherServlet 会拦截到所有的资源，导致一个问题就是静态资源（img、css、js）也会被拦截到，从而不能被使用。解决问题就是需要**配置静态资源不进行拦截**，在 `springmvc.xml` 配置文件添加如下配置：

- `<mvc:resources>` 标签配置不过滤
  - location 元素表示 webapp 目录下的包下的所有文件
  - mapping 元素表示以 /static 开头的所有请求路径，如 /static/a 或者 /static/a/b

```xml
<!-- 设置静态资源不过滤 -->
<mvc:resources location="/css/" mapping="/css/**"/> <!-- 样式 -->
<mvc:resources location="/images/" mapping="/images/**"/> <!-- 图片 -->
<mvc:resources location="/js/" mapping="/js/**"/> <!-- javascript -->
```

2.使用 **@RequestBody** 获取请求体数据

```javascript
// 页面加载
$(function(){
    // 绑定点击事件
    $("#btn").click(function(){
        $.ajax({
            url:"user/testJson",
            contentType:"application/json;charset=UTF-8",
            data:'{"addressName":"aa","addressNum":100}',
            dataType:"json",
            type:"post",
            success:function(data){
            alert(data);
            alert(data.setAddressName);
            }
        });
    });
});
```

```java
/**
* 获取请求体的数据
* @param body
*/
@RequestMapping("/testJson")
public void testJson(@RequestBody String body) {
	System.out.println(body);
}
```

3.使用 **@RequestBody** 注解把 json 的字符串转换成 JavaBean 的对象

```java
/**
* 获取请求体的数据
* @param body
*/
@RequestMapping("/testJson")
public void testJson(@RequestBody Address address) {
	System.out.println(address);
}
```

4.使用 **@ResponseBody** 注解把 JavaBean 对象转换成 json 字符串，直接响应

- 要求方法需要返回 JavaBean 的对象

```java
@RequestMapping("/testJson")
public @ResponseBody Address testJson(@RequestBody Address address) {
System.out.println(address);
	address.setAddressName("上海");
	return address;
}
```

5.json 字符串和 JavaBean 对象互相转换的过程中，需要使用 jackson 的 jar 包

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.0</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.0</version>
</dependency>
```

