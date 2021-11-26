---
title: SpringMVC学习笔记2:请求参数绑定
date: 2020-03-05 11:56:28
tags:
- Java
- SpringMVC
categories:
- Java
- Spring
- SpringMVC
---

## 1.请求参数的绑定说明

### 1.1 绑定机制

- 表单提交的数据都是 k=v 格式的 username=haha&password=123
- SpringMVC 的参数绑定过程是把表单提交的请求参数，作为控制器中方法的参数进行绑定的。
- 要求：提交表单的 name 和参数的名称是相同的。

<!-- more -->

### 1.2 支持的数据类型

- 基本数据类型和字符串类型。
- 实体类型（JavaBean）。
- 集合数据类型（List、map集合等）。

## 2.基本数据类型和字符串类型

- 提交表单的 name 和参数的名称是相同的。
- 区分大小写。

## 3.实体类型（JavaBean）

- 提交表单的 name 和 JavaBean 中的属性名称需要一致。
- 如果一个 JavaBean 类中包含其他的引用类型，那么表单的 name 属性需要编写成：对象.属性 例如：
   address.name。

## 4.给集合属性数据封装

- JSP页面编写方式：list[0].属性

## 5.请求参数中文乱码的解决

在 `web.xml` 中配置Spring提供的过滤器类。

```xml
<!-- 配置过滤器，解决中文乱码的问题 -->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filterclass>
    <!-- 指定字符集 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>

<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 6.自定义类型转换器

表单提交的任何数据类型全部都是字符串类型，但是后台定义 Integer 类型，数据也可以封装上，说明 Spring 框架内部会**默认进行数据类型转换**。

如果想自定义数据类型转换，可以实现 Converter 的接口:

- 自定义类型转换器

```java
// 把字符串转换成日期的转换器
public class StringToDateConverter implements Converter<String, Date>{

	// 进行类型转换的方法
    public Date convert(String source) {
    // 判断
    if(source == null) {
    	throw new RuntimeException("参数不能为空");
    }
    try {
        DateFormat df = new SimpleDateFormat("yyyy-MM-dd");
        // 解析字符串
        Date date = df.parse(source);
        return date;
    } catch (Exception e) {
        throw new RuntimeException("类型转换错误");
        }
    }
}
```

- 注册自定义类型转换器，在 springmvc.xml 配置文件中编写配置

```xml
<!-- 注册自定义类型转换器 -->
<bean id="conversionService"
class="org.springframework.context.support.ConversionServiceFactoryBean">
    <property name="converters">
        <set>
        	<bean class="cn.itcast.utils.StringToDateConverter"/>
        </set>
    </property>
</bean>

<!-- 开启Spring对MVC注解的支持 -->
<mvc:annotation-driven conversion-service="conversionService"/>
```

## 7.在控制器中使用原生的 ServletAPI 对象

只需要在控制器的方法参数定义 HttpServletRequest 和 HttpServletResponse 对象

```java
@RequestMapping("/testServlet")
public String testServlet(HttpServletRequest request, HttpServletResponse response){
    HttpSession session = request.getSession();
    ServletContext servletContext = session.getServletContext();
    return "success";
}
```

