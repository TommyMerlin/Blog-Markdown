---
title: 3.Spring学习笔记:依赖注入
date: 2020-03-03 21:42:54
tags:
- Java
- Spring
categories:
- Java
- Spring
- Spring
description: Spring学习笔记:依赖注入
---

## 1.概述

### **能注入的数据**
  - 基本类型和 String
  - 其他 bean 类型（在配置文件中或者注解中配置过的 bean）
  - 复杂类型/集合类型

### **IOC** 的作用

**减低程序间的耦合**（即依赖关系）

在当前类需要用到其他类的对象，由 Spring 为我们提供，而我们在配置文件中说明依赖关系的维护，这种方式就称为依赖注入。

## 2.注入方式

注入的方式：
- 第一种：使用**构造函数**提供
- 第二种：使用 **set方法**提供
- 第三种：使用**注解**提供

### 2.1 操作实例：

- 接口如下：

     ```java
     public interface IAccountDao {
     
         void saveAccount();
     }
     
     public interface IAccountService {
         /**
          * 模拟保存账户
          */
         void saveAccount();
     }
  ```
  
- 实现类：
  
     ```java
     @Service("accountService")
     public class AccountServiceImpl implements IAccountService {
     
         @Autowired
         @Qualifier("accountDao2")
         private IAccountDao accountDao = null;
         
         public void  saveAccount() {
     		accountDao.saveAccount();
     	}
     }        
     ```

    ```java
    @Repository("accountDao1")
    public class AccountDaoImpl implements IAccountDao {
    
        public void  saveAccount() {
            System.out.println("对象创建了111");
        }
    }          
    ```

    ```java
    @Repository("accountDao2")
    public class IAccountDaoImpl2 implements IAccountDao{
    
        public void  saveAccount() {
            System.out.println("对象创建了222");
        }
    }
    ```

### 2.2 使用构造函数注入

使用的**标签**：**constructor-arg**
标签出现的**位置**：bean 标签的内部
标签中的**属性**：
- type：用于指定要注入的数据的数据类型，该数据类型也是构造函数中某个或某些参数的类型
- index：用于指定要注入的数据给构造函数中指定索引位置的参数赋值。索引的位置是从0开始
  **name**：用于指定给构造函数中指定名称的参数赋值（**常用**）
- =============以上三个用于指定给构造函数中哪个参数赋值===============================
- **value**：用于提供基本类型和String类型的数据
- **ref**：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象

**优势**：
- 在获取bean对象时，注入数据是必须的操作，否则对象无法创建成功。

**弊端**：
- 改变了bean对象的实例化方式，使我们在创建对象时，如果用不到这些数据，也必须提供。

```xml
<bean id="accountService" class="com.itheima.service.impl.AccountServiceImpl">
    <constructor-arg name="name" value="泰斯特"></constructor-arg>
    <constructor-arg name="age" value="18"></constructor-arg>
    <constructor-arg name="birthday" ref="now"></constructor-arg>
</bean>

<!-- 配置一个日期对象 -->
<bean id="now" class="java.util.Date"></bean>
```

### 2.3 使用 set 方法注入（常用）

使用的**标签**：**property**
出现的**位置**：bean 标签的内部
标签的**属性**：
- **name**：用于指定注入时所调用的set方法名称
- **value**：用于提供基本类型和String类型的数据
- **ref**：用于指定其他的bean类型数据。它指的就是在spring的Ioc核心容器中出现过的bean对象

**优势**：
- 创建对象时没有明确的限制，可以直接使用默认构造函数

**弊端**：
- 如果有某个成员必须有值，则获取对象是有可能set方法没有执行。

```xml
<bean id="accountService2" class="com.itheima.service.impl.AccountServiceImpl2">
    <property name="name" value="TEST" ></property>
    <property name="age" value="21"></property>
    <property name="birthday" ref="now"></property>
</bean>
```

### 2.4 复杂类型的注入/集合类型的注入

用于给 List 结构集合注入的标签：
- list
- array
- set

用于个 Map 结构集合注入的标签:
- map 
- props

结构相同，标签可以**互换**

```xml
<bean id="accountService3" class="com.itheima.service.impl.AccountServiceImpl3">
	<property name="myStrs">
		<set>
			<value>AAA</value>
			<value>BBB</value>
			<value>CCC</value>
		</set>
	</property>

	<property name="myList">
		<array>
			<value>AAA</value>
			<value>BBB</value>
			<value>CCC</value>
		</array>
	</property>

	<property name="mySet">
		<list>
			<value>AAA</value>
			<value>BBB</value>
			<value>CCC</value>
		</list>
	</property>

	<property name="myMap">
		<props>
			<prop key="testC">ccc</prop>
			<prop key="testD">ddd</prop>
		</props>
	</property>

	<property name="myProps">
		<map>
			<entry key="testA" value="aaa"></entry>
			<entry key="testB">
				<value>BBB</value>
			</entry>
		</map>
	</property>
</bean>
```

### 2.5 使用注解注入

#### 如何使用？

第一步：在类或方法的前面加上**注解关键字**
第二步：**引入约束**，注意此处约束多了 xmlns:context...
第三步：**添加配置**，告知 Spring 在创建容器时要扫描的包，配置所需的标签不是在 bean 约束中，而是一个名称为 context 的名称空间和约束中，完整配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="com.itheima"></context:component-scan>
</beans>
```

#### 有哪些注解？
##### 用于**创建对象**的
作用：等同于 xml 配置文件中编写一个 <bean> 标签
- **@Component**
  - 形式：@Component(value=" ")   /  @Component(" ")
  - 作用：用于把当前类对象存入 Spring 容器中
  - 属性：
    value : 用于指定 bean 的 id，当我们不写的时候，它的默认值是当前类名，且首字母改小写;当值只有一个的时候可以省略

以下三个注解的作用与 @Component 完全一样，它们是 Spring 提供的更明确的划分，使三层对象更加清晰
- **@Controller**   用于**表现层**
- **@Service**        用于**业务层**
- **@Repository**  用于**持久层**

##### 用于**注入数据**的
作用：等同于在 <bean> 标签中写一个 <property> 标签。
- **@Autowired**
  - 作用：**自动按照类型注入**，只要容器中有**唯一**的一个 bean 对象类型和要注入的变量类型匹配，就可以注入成功，如果 IOC 容器中没有任何  bean 的类型和要注入的变量类型匹配，则报错。
  - 出现位置：可以是变量上，也可以是方法上。
  - 细节：在使用注解注入时，set 方法就不是必须的了。 
  
- **@Qualifier**     
  - 作用：在按照类型注入的基础上再**按照名称注入**，它**在给类成员注入时不能单独使用(需结合@Autowired)，但是在给方法参数注入时可以**。     
  - 属性：value : 用于指定注入的 bean 的  id
  
- **@Resource**     
     - 作用：直接按照 bean 的 id 注入，可以直接使用。 
     - 属性：name : 用于指定 bean 的 id     
     - 等同于@Autowired+@Qualifier   

以上三个注入都**只能注入其他 bean 类型的数据**，而基本类型和 String 类型的数据无法使用上述注解实现。另外，**集合类型的注入只能通过 xml 配置文件实现**。

   - **@Value**
     - 作用：用于注入基本类型和 String 类型的数据。
     - 属性：
       value : 用于指定数据的值，它可以使用 Spring 中 Spel (即spring的el表达式) 
       Spel 的写法：${表达式}
##### 用于改变范围的
作用：等同于在 <bean> 标签中使用 scope 属性。
- **@Scope**
  - 作用：用于指定 bean 的作用范围。
  - 属性：value : 指定范围的取值，常用为 singleton ,  prototype。
##### 和生命周期相关（了解）
作用：等同于在 <bean> 标签中使用 init-method 和 destroy-method。
- **@PreDestory**
  作用：用于指定**销毁方法**  
- **@Postcontrust** 
    作用：用于指定**初始化方法**

