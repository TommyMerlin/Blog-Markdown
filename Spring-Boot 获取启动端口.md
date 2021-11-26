---
title: Spring Boot 获取启动端口
date: 2021-05-10 11:23:08
tags:
- Java
- Spring Boot
categories:
- Java
- Spring
- Spring Boot
---


## 1. 通过 environment 获取

```java
@Autowired
Environment environment;

public String getPort(){
    return environment.getProperty("local.server.port");
}
```

 <!-- more -->

## 2.通过注解获取

```java
@Value("${server.port}")
private String port ;

@LocalServerPort
private String port ;
```

注：当 `application.yml` 配置文件中没有指定服务启动端口时，不能使用 `@LocalServerPort` 和 `@Value("${server.port}")` 的方法获取端口号，只能使用 `environment` 的方式。