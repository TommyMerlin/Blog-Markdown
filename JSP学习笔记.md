---
title: JSP学习笔记
date: 2020-02-27 14:18:34
tags:
- JavaWeb
- Servlet
- JSP
categories:
- Java
- JavaWeb
- JSP
---

## 概念

Java Server Pages： Java 服务器端页面。

- 可理解为：一个特殊的页面，其中既可定义html标签，又可定义 Java 代码。
- 用于**简化书写**。

<!-- more -->

## 原理

JSP 本质上就是一个 **Servlet**。

## JSP 脚本

JSP 定义 Java 代码的方式：

- 1.<%  代码 %>：定义的java代码，在service方法中。service方法中可以定义什么，该脚本中就可以定义什么。
- 2.<%! 代码 %>：定义的java代码，在jsp转换后的java类的成员位置。
- 3.<%= 代码 %>：定义的java代码，会输出到页面上。输出语句中可以定义什么，该脚本中就可以定义什么。