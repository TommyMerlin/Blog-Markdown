---
title: 5.Spring学习笔记:Spring整合Junit
date: 2020-03-04 09:35:14
tags:
- Java
- Spring
categories:
- Java
- Spring
- Spring
description: Spring学习笔记:Spring整合Junit
---

1.导入 Spring 整合 Junit 的 jar ( 坐标 )
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.0.2.RELEASE</version>
</dependency>
```

2.使用 Junit 提供的一个注解把原有的 main 方法替换了，替换成 Spring 提供的。
- **@Runwith**

3.告知 Spring 的运行器， Spring 和 ioc 创建是基于 xml 还是注解的，并且说明位置，用到的注解如下
- **@ContextConfiguration**
  - Locations : 指定 xml 文件的位置，加上 classpath 关键字，表示在类路径下。
  - classes : 指定注解类所在地位置。

4.使用 @Autowired 给测试类中的变量注入数据。
5.当使用 spring 5.x 版本的时候，要求 junit 的 jar 必须是 4.12 及以上。

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = SpringConfiguration.class)
public class AccountServiceTest {

    @Autowired
    private IAccountService as = null;


    @Test
    public void testFindAll() {
        //3.执行方法
        List<Account> accounts = as.findAllAccount();
        for(Account account : accounts){
            System.out.println(account);
        }
    }
}
```

