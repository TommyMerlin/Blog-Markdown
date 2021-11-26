---
title: 2.Spring学习笔记:Bean的装配与管理
date: 2020-03-03 21:36:19
tags:
- Java
- Spring
categories:
- Java
- Spring
- Spring
description: Spring学习笔记:Bean的装配与管理
---

## 1.Spring 中 Bean 的细节

### 三种创建 bean 对象的方式

1. 使用**默认构造函数**创建

   在 spring 的配置文件中，使用 id 和 class 属性之后，且没有其他属性和标签时，采用的就是默认构造函数创建 bean 对象，此时如果类中没有默认构造函数，则对象无法创建。

   ```xml
   <bean id = "accountService" class = "com.itheima.service.impl.AccountServiceImpl"></bean>
   ```

2. 使用**普通工厂中的方法**创建对象（使用某个类中的方法创建对象，并存入  Spring 容器），如下

   ```java
   /**
    *模拟一个工厂类，该类可能存在于jar包中，无法通过修改源码的方式来提供默认构造函数
    * 
    */
   public class InstanceFactory {
       public IAccountService getAccountService() {
           return new AccountServiceImpl();
       }
   }
   ```

   配置方式如下：

   ```xml
   <bean id = "instanceFactory" class = "com.itheima.factory.InstanceFactory"></bean>
   <bean id = "accountService" factory-bean="instanceFactory" factory-method
             ="getAccountService"></bean>
   ```

3. 使用**工厂中的静态方法**创建对象（使用某个类中的静态方法创建对象，并存入 Spring 容器），如下：

   ```java
   public class StaticFactory {
       public  static IAccountService getAccountService() {
           return new AccountServiceImpl();
       }
   }
   ```

   配置方式如下：

   ```xml
   <bean id = "accountService" class = "com.itheima.factory.StaticFactory" factory-method="getAccountService"></bean>
   ```

## 2.bean 的作用范围调整

1. bean 标签的 **scope** 属性
   作用：用于指定 bean 的**作用范围。**
   取值：常用的就是**单例**和**多例。**

   - **singleton** : 单例的（**default**）。
   - **prototype** : 多例的。
   - request : 作用于 web 应用的请求范围。
   - session : 作用于 web  应用的会话范围。
   - global-session : 作用于集群的会话范围（全局会话范围），当不是集群范围时，它就是 session。

2. bean 对象的**声明周期**
   **单例**对象：
   - 出生：当容器创建时发生。
   - 活着：只要容器还在对象就一直活着。
   - 死亡：容器销毁，对象消亡。

   总结：单例对象的**声明周期和容器相同**。

   **多例**对象：
   - 出生：当我们使用对象时 Spring 框架为我们创建。
   - 活着：对象只要是在使用过程中就活着。
   - 死亡：当对象长时间不用，且没有别的对象引用时，由 Java 的 GC 回收。
