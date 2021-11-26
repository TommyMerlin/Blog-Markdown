---
title: SpringMVC学习笔记7:拦截器
date: 2020-03-05 20:03:56
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
---

## 1.拦截器的概述

- SpringMVC 框架中的拦截器用于对处理器进行**预处理**和**后处理**的技术。
- 可以定义**拦截器链**，连接器链就是将拦截器按着一定的顺序结成一条链，在访问被拦截的方法时，拦截器链中的拦截器会按着定义的顺序执行。
- 拦截器和过滤器的功能比较类似，有区别
  - 过滤器是 Servlet 规范的一部分，任何框架都可以使用过滤器技术。
  - 拦截器是 SpringMVC 框架独有的。
  - 过滤器配置了 /*，可以拦截任何资源。
  - 拦截器只会对控制器中的方法进行拦截。
- 拦截器也是 AOP 思想的一种实现方式。
- 想要自定义拦截器，需要实现 **HandlerInterceptor** 接口。

<!-- more -->

## 2.自定义拦截器步骤

- 创建类，实现 HandlerInterceptor 接口，重写需要的方法。

```java
/**
* 自定义拦截器
*/
public class MyInterceptor1 implements HandlerInterceptor{
    /**
    * controller方法执行前，进行拦截的方法
    * return true放行
    * return false拦截
    * 可以使用转发或者重定向直接跳转到指定的页面。
    */
    public boolean preHandle(HttpServletRequest request,
							HttpServletResponse response,
    						Object handler)
							throws Exception {
        System.out.println("拦截器执行了...");
        return true;
    }
}
```

- 在 springmvc.xml 中配置拦截器类。

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 哪些方法进行拦截 -->
        <mvc:mapping path="/user/*"/>

        <!-- 哪些方法不进行拦截
        <mvc:exclude-mapping path=""/>
        -->

        <!-- 注册拦截器对象 -->
        <bean class="cn.itcast.demo1.MyInterceptor1"/>
    </mvc:interceptor>
</mvc:interceptors>
```

## 3.HandlerInterceptor 接口中的方法

- preHandle 方法是 controller 方法执行前拦截的方法。
  - 可以使用 request 或者 response 跳转到指定的页面。
  - return true 放行，执行下一个拦截器，如果没有拦截器，执行 controller 中的方法。
  - return false 不放行，不会执行 controller 中的方法。
- postHandle 是 controller 方法执行后执行的方法，在 JSP 视图执行前。
  - 可以使用 request 或者 response 跳转到指定的页面。
  - 如果指定了跳转的页面，那么 controller 方法跳转的页面将不会显示。
- postHandle 方法是在 JSP 执行后执行。
  - request 或者 response 不能再跳转页面了。

## 4.配置多个拦截器

- 1.再编写一个拦截器的类
- 2.配置2个拦截器

```xml
<!-- 配置拦截器 -->
<mvc:interceptors>
    <mvc:interceptor>
        <!-- 哪些方法进行拦截 -->
        <mvc:mapping path="/user/*"/>
        <!-- 注册拦截器对象 -->
        <bean class="cn.itcast.demo1.MyInterceptor1"/>
    </mvc:interceptor>
    
    <mvc:interceptor>
        <mvc:mapping path="/**"/>
        <bean class="cn.itcast.demo1.MyInterceptor2"/>
    </mvc:interceptor>
</mvc:interceptors>
```

