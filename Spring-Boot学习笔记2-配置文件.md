---
title: Spring Boot学习笔记2:配置文件
date: 2020-03-06 09:56:15
tags:
- Java
- Spring Boot
categories:
- Java
- Spring
- Spring Boot
---

## 1.配置文件

SpringBoot 使用一个**全局**的配置文件，配置**文件名是固定**的。

- application.properties
- application.yml

<!-- more -->

**配置文件的作用**：修改 SpringBoot 自动配置的默认值；SpringBoot在底层都给我们自动配置好。

**YAML**（YAML Ain't Markup Language）

- YAML  A Markup Language：是一个标记语言
- YAML   isn't Markup Language：不是一个标记语言；

**标记语言**：

- 以前的配置文件；大多都使用的是  **xxxx.xml**文件；
- YAML：**以数据为中心**，比 json、xml 等更适合做配置文件；

**配置实例**

- YAML：
```yaml
server:
  port: 8081
```

- XML：
```xml
<server>
	<port>8081</port>
</server>
```

## 2.YAML语法

### 2.1 基本语法

k:(空格)v：表示一对键值对（空格必须有）；

以**空格**的缩进来控制层级关系；只要是左对齐的一列数据，都是同一个层级的

```yaml
server:
    port: 8081
    path: /hello
```

属性和值也是大小写敏感；

### 2.2 值的写法

#### 字面量：普通的值（数字，字符串，布尔）

k: v：

- 字面直接来写；
- 字符串默认不用加上单引号或者双引号；
  - ""：双引号；会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
  name:   "zhangsan \n lisi"：输出；zhangsan 换行  lisi
  - ''：单引号；不会转义特殊字符，特殊字符最终只是一个普通的字符串数据
  name:   ‘zhangsan \n lisi’：输出；zhangsan \n  lisi

#### 对象、Map（属性和值）（键值对）

k: v：在下一行来写对象的属性和值的关系；注意缩进。
- 对象还是k: v的方式
```yaml
friends:
    lastName: zhangsan
    age: 20
```

- 行内写法：
```yaml
friends: {lastName: zhangsan,age: 18}
```

#### 数组（List、Set）

用 - 值表示数组中的一个元素

```yaml
pets:
 - cat
 - dog
 - pig
```

行内写法

```yaml
pets: [cat,dog,pig]
```

## 3.配置文件值注入

配置文件：

```yaml
person:
    lastName: hello
    age: 18
    boss: false
    birth: 2017/12/12
    maps: {k1: v1,k2: 12}
    lists:
      - lisi
      - zhaoliu
    dog:
      name: 小狗
      age: 12
```

javaBean：

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *
 */
@Component
@ConfigurationProperties(prefix = "person")
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

我们可以导入配置文件处理器，以后编写配置就有提示。

```xml
<!--导入配置文件处理器，配置文件进行绑定就会有提示-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

### 1.@Value 获取值和 @ConfigurationProperties 获取值比较

|                      | @ConfigurationProperties | @Value     |
| -------------------- | ------------------------ | ---------- |
| 功能                 | 批量注入配置文件中的属性 | 一个个指定 |
| 松散绑定（松散语法） | 支持                     | 不支持     |
| SpEL                 | 不支持                   | 支持       |
| JSR303数据校验       | 支持                     | 不支持     |
| 复杂类型封装         | 支持                     | 不支持     |

- 配置文件 yml 还是 properties 都能获取到值。
- 如果我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用 @Value。
- 如果我们专门编写了一个 javaBean 来和配置文件进行映射，我们就直接使用 @ConfigurationProperties。

### 2.配置文件注入值数据校验

```java
@Component
@ConfigurationProperties(prefix = "person")
@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

   //lastName必须是邮箱格式
    @Email
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")
    private Integer age;
    //@Value("true")
    private Boolean boss;

    private Date birth;
    private Map<String,Object> maps;
    private List<Object> lists;
    private Dog dog;
```

### 3.@PropertySource & @ImportResource & @Bean

@**PropertySource**：加载指定的配置文件；

```java
/**
 * 将配置文件中配置的每一个属性的值，映射到这个组件中
 * @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
 *      prefix = "person"：配置文件中哪个下面的所有属性进行一一映射
 *
 * 只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 *  @ConfigurationProperties(prefix = "person")默认从全局配置文件中获取值；
 *
 */
@PropertySource(value = {"classpath:person.properties"})
@Component
@ConfigurationProperties(prefix = "person")
//@Validated
public class Person {

    /**
     * <bean class="Person">
     *      <property name="lastName" value="字面量/${key}从环境变量、配置文件中获取值/#{SpEL}"></property>
     * <bean/>
     */

   //lastName必须是邮箱格式
   // @Email
    //@Value("${person.last-name}")
    private String lastName;
    //@Value("#{11*2}")
    private Integer age;
    //@Value("true")
    private Boolean boss;

```

- @**ImportResource**：导入 Spring 的配置文件，让配置文件里面的内容生效。
- Spring Boot里面没有 Spring 的配置文件，我们自己编写的配置文件，也不能自动识别。
- 想让 Spring 的配置文件生效，需要手动加载进来；将 @**ImportResource **标注在一个配置类上。

```java
@ImportResource(locations = {"classpath:beans.xml"})
导入Spring的配置文件让其生效
```

编写 Spring 的配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">


    <bean id="helloService" class="com.atguigu.springboot.service.HelloService"></bean>
</beans>
```

SpringBoot 推荐给容器中添加组件的方式；推荐使用**全注解**的方式
- 1.配置类**@Configuration**------>Spring配置文件
- 2.使用**@Bean**给容器中添加组件
```java
/**
 * @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 *
 * 在配置文件中用<bean><bean/>标签添加组件
 *
 */
@Configuration
public class MyAppConfig {

    //将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println("配置类@Bean给容器中添加组件了...");
        return new HelloService();
    }
}
```

## 4.配置文件占位符

### 4.1 随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}
```

### 4.2 占位符获取之前配置的值

**如果没有可以使用 : 指定默认值**

## 5.Profile

### 5.1 多Profile文件

我们在主配置文件编写的时候，文件名可以是   `application-{profile}.properties/yml`

默认使用 application.properties 的配置；

### 5.2 yml支持多文档块方式

```yml
server:
  port: 8081
spring:
  profiles:
    active: prod

---
server:
  port: 8083
spring:
  profiles: dev


---

server:
  port: 8084
spring:
  profiles: prod  #指定属于哪个环境
```

### 5.3 激活指定profile

- 在配置文件中指定  spring.profiles.active=dev
- 命令行：
  - java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；
  - 可以直接在测试的时候，配置传入命令行参数
- 虚拟机参数；
  - -Dspring.profiles.active=dev

## 6.配置文件加载位置

springboot 启动会扫描以下位置的 application.properties 或者 application.yml 文件作为 Spring boot 的默认配置文件：
- –file:./config/
- –file:./
- –classpath:/config/
- –classpath:/
优先级由高到底，高优先级的配置会**覆盖**低优先级的配置。

SpringBoot会从这四个位置全部加载主配置文件；**互补配置。**

我们还可以通过 **spring.config.location** 来改变默认的配置文件位置。

**项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置。**

```bash
java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=G:/application.properties
```

## 7.外部配置加载顺序

**SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置。**

- **1.命令行参数**
  - 所有的配置都可以在命令行上进行指定

    ```bash
    java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=/abc
    ```

  - 多个配置用空格分开； --配置项=值
- 2.来自 java:comp/env 的 JNDI 属性
- 3.Java 系统属性（System.getProperties()）
- 4.操作系统环境变量
- 5.RandomValuePropertySource 配置的 random.* 属性值
**由 jar 包外向 jar 包内进行寻找；**
**优先加载带 profile**
- **6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件**
- **7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件**
**再来加载不带profile**
- **8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件**
- **9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件**
- 10.@Configuration 注解类上的 @PropertySource
- 11.通过 SpringApplication.setDefaultProperties 指定的默认属性

## 8.自动配置原理

[配置文件能配置的属性参照](https://docs.spring.io/spring-boot/docs/2.2.4.RELEASE/reference/html/appendix-application-properties.html)

### 8.1 自动配置原理

1.SpringBoot 启动的时候加载主配置类，开启了自动配置功能 **@EnableAutoConfiguration** 
2.@EnableAutoConfiguration 作用：
 - 利用 EnableAutoConfigurationImportSelector 给容器中导入一些组件
**将 类路径下  META-INF/spring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中；**
```java
// Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\

//......
//......

org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration
```
每一个这样的  xxxAutoConfiguration 类都是容器中的一个组件，都加入到容器中，用他们来做自动配置。
3.每一个自动配置类进行自动配置功能；
4.以 **HttpEncodingAutoConfiguration（Http编码自动配置）** 为例解释自动配置原理；
```java
@Configuration   //表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class)  //启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中

@ConditionalOnWebApplication //Spring底层@Conditional注解（Spring注解版），根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；    判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)  //判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = "spring.http.encoding", value = "enabled", matchIfMissing = true)  //判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
//即使我们配置文件中不配置pring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
  
  	//他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;
  
   //只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   //给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) //判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```

根据当前不同的条件判断，决定这个配置类是否生效。
一但这个配置类生效，这个配置类就会给容器中添加各种组件，这些组件的属性是从对应的 **properties 类**中获取的，这些类里面的**每一个属性又是和配置文件绑定**的。
5.所有在配置文件中能配置的属性都是在 xxxxProperties 类中封装，配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = "spring.http.encoding")  //从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName("UTF-8");
```

### 8.2 精髓

- SpringBoot 启动会加载大量的自动配置类。
- 我们看需要的功能有没有 SpringBoot 默认写好的自动配置类。
- 再看这个自动配置类中到底配置了哪些组件；（只要需要的组件有，我们就不需要再来配置）给容器中自动配置类添加组件的时候，会从 properties 类中获取某些属性。我们就可以在配置文件中指定这些属性的值。
**xxxAutoConfigurartion**：自动配置类，给容器中添加组件。
**xxxProperties**：封装配置文件中相关属性。

### 8.3 细节

**@Conditional** 派生注解（Spring 注解版原生的 @Conditional 作用）

作用：必须是 @Conditional 指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效。

| @Conditional扩展注解            | 作用（判断是否满足当前指定条件）                 |
| ------------------------------- | ------------------------------------------------ |
| @ConditionalOnJava              | 系统的java版本是否符合要求                       |
| @ConditionalOnBean              | 容器中存在指定Bean；                             |
| @ConditionalOnMissingBean       | 容器中不存在指定Bean；                           |
| @ConditionalOnExpression        | 满足SpEL表达式指定                               |
| @ConditionalOnClass             | 系统中有指定的类                                 |
| @ConditionalOnMissingClass      | 系统中没有指定的类                               |
| @ConditionalOnSingleCandidate   | 容器中只有一个指定的Bean，或者这个Bean是首选Bean |
| @ConditionalOnProperty          | 系统中指定的属性是否有指定的值                   |
| @ConditionalOnResource          | 类路径下是否存在指定资源文件                     |
| @ConditionalOnWebApplication    | 当前是web环境                                    |
| @ConditionalOnNotWebApplication | 当前不是web环境                                  |
| @ConditionalOnJndi              | JNDI存在指定项                                   |

**我们可以通过启用  debug=true 属性；来让控制台打印自动配置报告**，这样我们就可以很方便的知道哪些自动配置类生效；

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches:（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched:
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches:（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration:
      Did not match:
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
```
