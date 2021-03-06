---
title: 1.Spring学习笔记:入门案例
date: 2020-03-03 21:28:00
tags:
- Java
- Spring
categories:
- Java
- Spring
- Spring
description: Spring学习笔记:入门案例
---

### 1.实例

1. 通过 Idea 创建 maven 项目
2. 配置 Spring 配置文件 ApplicationContext.xml
3. 编写接口及实现类

- IaccountDao

  ```java
  /**
   * 账户的持久层接口
   */
  public interface IAccountDao {
  
      /**
       * 模拟保存账户
       */
      void saveAccount();
  }
  ```

- IaccountService

  ```java
  /**
   * 账户业务层的接口
   */
  public interface IAccountService {
  
      /**
       * 模拟保存账户
       */
      void saveAccount();
  }
  ```

- AccountDaoImpl

  ```java
  /**
   * 账户的持久层实现类
   */
  public class AccountDaoImpl implements IAccountDao {
  
      public  void saveAccount(){
  
          System.out.println("保存了账户");
      }
  }
  ```

- AccountServiceImpl

  ```java
  /**
   * 账户的业务层实现类
   */
  public class AccountServiceImpl implements IAccountService {
  
      private IAccountDao accountDao = new AccountDaoImpl();
  
      public void  saveAccount(){
          accountDao.saveAccount();
      }
  }
  ```

4.编写测试类 Client

```java
/**
 * 模拟一个表现层，用于调用业务层
 */
public class Client {
    /**
     *
     *获取IOC的核心容器，并根据id获取对象
     * @param args
     */
     public static void main(String[] args) {
          ApplicationContext ac = new ClassPathXmlApplicationContext("beans.xml");
          // 两种不同的方式获取Bean对象
          IAccountService as = (IAccountService) ac.getBean("accountService");
          IAccountDao adao = ac.getBean("accountDao",IAccountDao.class);
          System.out.println(as);
          System.out.println(adao);
          // as.saveAccount();
     }
}
```



### 2.知识点

1. **ApplicationContext** 的三个常用实现类：
   - **ClassPathXmlApplicationContext**： 它可以加载**路径下**的配置文件，要求配置文件必须在路径下，否则加载不了。

     ```java
     ApplicationContext ac = new ClassPathXmlApplicationContext("bea
     ns.xml");
     ```

   - **FileSyetemXmlApplicationContext**：它可以加载**磁盘任意路径下**的配置文件（**必须有访问权限**）。
     加载方式如下：

     ```java
     ApplicationContext ac = new FileSystemXmlApplicationContext("C:\\user\\greyson\\...")
     ```

   - **AnnotationConfigApplicationContext**：它是用于**读取注解创建容器**的

2. 核心容器的两个接口引发出来的问题
   - ApplicationContext：它在创建核心容器时，创建对象采取的策略是采用立即加载的方式，也就是说，只要一读取完配置文件就马上创建配置文件中配置的对象。
     - **单例对象**适用。
     - 开发中常采用此接口。
   - BeanFactory: 它在构建核心容器时，创建对象的策略是采用延迟加载的方式，什么时候获取 id 对象了，什么时候就创建对象。
     - **多例对象**适用。
