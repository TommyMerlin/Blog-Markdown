---
title: SpringMVC学习笔记3:常用注解
date: 2020-03-05 14:12:23
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
description: 常用注解。
---

## 1.RequestParam 注解

### 作用
把请求中的指定名称的参数传递给控制器中的形参赋值。

### 属性
- value：请求参数中的名称。
- required：请求参数中是否必须提供此参数，默认值是true，必须提供。

### 代码

```java
/**
* 接收请求
*/
@RequestMapping(path="/hello")
public String sayHello(@RequestParam(value="username",required=false)String name) {
    System.out.println("aaaa");
    System.out.println(name);
    return "success";
}
```

## 2.RequestBody 注解

### 作用

用于获取**请求体**的内容（注意：**get方法不可以**）

### 属性

- required：是否必须有请求体，默认值是true。

### 代码

```java
/**
* 接收请求
*/
@RequestMapping(path="/hello")
public String sayHello(@RequestBody String body) {
    System.out.println(body);
    return "success";
}
```

## 3.PathVariable 注解

### 作用

用于绑定 url 中的占位符的。例如：url 中有 /delete/{id}，{id} 就是占位符

### 属性

- value：指定 url 中的占位符名称.

### Restful 风格的 URL

- 请求路径一样，可以根据不同的请求方式去执行后台的不同方法。
- restful 风格的 URL 优点
  - 结构清晰
  - 符合标准
  - 易于理解
  - 扩展方便

### 代码

```java
<a href="user/hello/1">入门案例</a>

/**
* 接收请求
* @return
*/
@RequestMapping(path="/hello/{id}")
public String sayHello(@PathVariable(value="id") String id) {
    System.out.println(id);
    return "success";
}
```

## 4.RequestHeader 注解

### 作用

获取指定**请求头**的值。

### 属性

- value：请求头的名称。

### 代码

```java
@RequestMapping(path="/hello")
public String sayHello(@RequestHeader(value="Accept") String header) {
    System.out.println(header);
    return "success";
}
```

## 5.CookieValue 注解

### 作用

用于获取指定 **cookie** 的名称的值。

### 属性

- value：cookie的名称。

### 代码

```java
@RequestMapping(path="/hello")
public String sayHello(@CookieValue(value="JSESSIONID") String cookieValue) {
    System.out.println(cookieValue);
    return "success";
}
```

## 6.ModelAttribute 注解

### 作用

- 出现在**方法**上：表示当前方法会在控制器方法执行前先执行。
- 出现在**参数**上：获取指定的数据给参数赋值。

### 应用场景

- 当提交表单数据不是完整的实体数据时，保证没有提交的字段使用数据库原来的数据。

### 代码

- 修饰的方法有返回值

```java
/**
* 作用在方法，先执行
* @param name
* @return
*/
@ModelAttribute
public User showUser(String name) {
    System.out.println("showUser执行了...");
    // 模拟从数据库中查询对象
    User user = new User();
    user.setName("哈哈");
    user.setPassword("123");
    user.setMoney(100d);
    return user;
}

/**
* 修改用户的方法
* @param cookieValue
* @return
*/
@RequestMapping(path="/updateUser")
public String updateUser(User user) {
    System.out.println(user);
    return "success";
}
```

- 修饰的方法没有返回值

```java
/**
* 作用在方法，先执行
* @param name
* @return
*/
@ModelAttribute
public void showUser(String name,Map<String, User> map) {
    System.out.println("showUser执行了...");
    // 模拟从数据库中查询对象
    User user = new User();
    user.setName("哈哈");
    user.setPassword("123");
    user.setMoney(100d);
    map.put("abc", user);
}

/**
* 修改用户的方法
* @param cookieValue
* @return
*/
@RequestMapping(path="/updateUser")
public String updateUser(@ModelAttribute(value="abc") User user) {
    System.out.println(user);
    return "success";
}
```

## 7.SessionAttributes 注解

### 作用

用于多次执行控制器方法间的**参数共享**。

### 属性

- value：指定存入属性的名称。

### 代码

```java
@Controller
@RequestMapping(path="/user")
@SessionAttributes(value= {"username","password","age"},types=
{String.class,Integer.class}) // 把数据存入到session域对象中
public class HelloController {
    /**
    * 向session中存入值
    * @return
    */
    @RequestMapping(path="/save")
    public String save(Model model) {
        System.out.println("向session域中保存数据");
        model.addAttribute("username", "root");
        model.addAttribute("password", "123");
        model.addAttribute("age", 20);
        return "success";
    }
    
    /**
    * 从session中获取值
    * @return
    */
    @RequestMapping(path="/find")
    public String find(ModelMap modelMap) {
        String username = (String) modelMap.get("username");
        String password = (String) modelMap.get("password");
        Integer age = (Integer) modelMap.get("age");
        System.out.println(username + " : "+password +" : "+age);
        return "success";
    }
    
    /**
    * 清除值
    * @return
    */
    @RequestMapping(path="/delete")
    public String delete(SessionStatus status) {
        status.setComplete();
        return "success";
    }
}
```

