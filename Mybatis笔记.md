---
title: Mybatis笔记
date: 2020-03-27 08:54:45
tags:
- 数据库
- Java
- Mybatis
categories:
- Java
- 数据库
---

# 1、简介

## 1.1、什么是MyBatis

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-mybatis-b37104.png)

<!-- more -->

## 简介

### 什么是 MyBatis？
- MyBatis 是一款优秀的**持久层**框架。
- 它支持定制化 SQL、存储过程以及高级映射。
- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
- MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录。

如何获得MyBatis？
- Maven仓库
- ```xml
  <!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
  <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis</artifactId>
      <version>3.5.3</version>
  </dependency>
  ```

- github：https://github.com/mybatis/mybatis-3/releases
- 中文文档：https://mybatis.org/mybatis-3/zh/index.html

## 1.2、持久化

数据持久化
- 持久化就是将程序的数据在持久状态和瞬时状态转化的过程
- 内存：**断电即失**
- 数据库(JDBC),io文件持久化。

## 1.3、持久层

Dao 层、Service 层、Controller 层.....
- 完成持久化工作的代码块。
- 层界限十分明显。

## 1.4、为什么需要Mybatis？
- 帮助程序员将数据存入到数据库中。
- 传统的 JDBC 代码太复杂了。简化、框架、自动化。
- 不用 Mybatis 也可以。更容易上手。
- 优点：
  - 简单易学。
  - 灵活。
  - sql 和代码的分离，提高了可维护性。
  - 提供映射标签，支持对象与数据库的 orm 字段关系映射。
  - 提供对象关系映射标签，支持对象关系组建维护。
  - 提供 xml 标签，支持编写动态 sql。

# 2、第一个Mybatis程序

思路：搭建环境-->导入Mybatis-->编写代码-->测试！

## 2.1、搭建环境

搭建数据库
```sql
CREATE DATABASE `mybatis`;
use `mybatis`;
CREATE TABLE `user`(
`id` INT(20) not null PRIMARY KEY,
`name` VARCHAR(30) DEFAULT NULL,
`pwd` VARCHAR(30) DEFAULT NULL
)ENGINE=INNODB DEFAULT CHARSET=utf8;

INSERT INTO `user` (`id`,`name`,`pwd`) VALUES
(1,'狂神','123456'),
(2,'张三','123456'),
(3,'李四','123890')
```

新建项目
1. 新建一个普通的 maven 项目
2. 删除 src 目录
3. 导入maven依赖

```java
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--父工程-->
    <groupId>com.rui</groupId>
    <artifactId>MyBatis-Study</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!--导入依赖-->
    <dependencies>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.17</version>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.2</version>
        </dependency>
        <!--junit-->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
    </dependencies>

</project>
```

```xml
<!--在build中配置resources，来防止我们资源导出失败的问题-->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
            </includes>
            <filtering>true</filtering>
        </resource>
    </resources>
</build>
```

## 2.2、创建一个模块
- 编写 mybatis 的**核心配置文件**
  ```xml
  <?xml version="1.0" encoding="UTF-8" ?>
  <!DOCTYPE configuration
          PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
          "http://mybatis.org/dtd/mybatis-3-config.dtd">
  <!--configuration核心配置文件-->
  <configuration>
      <!--environments配置环境组-->
      <!--default默认环境-->
      <environments default="development">
          <!--environment单个环境-->
          <environment id="development">
              <!--transactionManager配置事务管理器-->
              <transactionManager type="JDBC"/>
              <!--配置连接池-->
              <dataSource type="POOLED">
                  <property name="driver" value="com.mysql.jdbc.Driver"/>
                  <property name="url" value="jdbc:mysql://localhost:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UFT-8"/>
                  <property name="username" value="root"/>
                  <property name="password" value="Cc105481"/>
              </dataSource>
          </environment>
      </environments>
  
  </configuration>
  ```

- 编写 mybatis **工具类**
```java
package com.rui.utils;

import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;

//sqlSessionFactory--->SessionFactory
public class MyBatisUtils {
    private static SqlSessionFactory sqlSessionFactory;
    static {
        try{
            //使用mybatis第一步、获取sqlSessionFactory对象
            String resource = "mybatis-config.xml";
            InputStream inputStream = Resources.getResourceAsStream(resource);
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }catch(IOException e) {
            e.printStackTrace();
        }
    }

    //既然有了 SqlSessionFactory，顾名思义，我们就可以从中获得 SqlSession 的实例了。
    // SqlSession 完全包含了面向数据库执行 SQL 命令所需的所有方法。
    // 你可以通过 SqlSession 实例来直接执行已映射的 SQL 语句。
    public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession();
    }
}
```

## 2.3、编写代码
- 实体类
  ```java
  package com.rui.pojo;
  
  public class user {
      private int id;
      private String name;
      private String pwd;
  
      public user() {
      }
  
      public user(int id, String name, String pwd) {
          this.id = id;
          this.name = name;
          this.pwd = pwd;
      }
  
      public int getId() {
          return id;
      }
  
      public void setId(int id) {
          this.id = id;
      }
  
      public String getName() {
          return name;
      }
  
      public void setName(String name) {
          this.name = name;
      }
  
      public String getPwd() {
          return pwd;
      }
  
      public void setPwd(String pwd) {
          this.pwd = pwd;
      }
  
      @Override
      public String toString() {
          return "user{" +
                  "id=" + id +
                  ", name='" + name + '\'' +
                  ", pwd='" + pwd + '\'' +
                  '}';
      }
  }
  ```

- Dao接口
  ```java
  package com.rui.dao;
  
  import com.rui.pojo.User;
  
  import java.util.List;
  
  public interface UserDao {
      List<User> getUserList();
  }
  
  ```

- 接口实现类由原来的UserImpl转变为一个Mapper配置文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!--namespace=绑定一个对应的mapper接口-->
<mapper namespace="com.rui.dao.UserDao">
    <!--select查询语句-->
   <select id="getUserList" resultType="com.rui.pojo.User">
       /*定义sql*/
       select * from mybatis.user
   </select>
</mapper>
```

## 2.4、测试
 注意点：

org.apache.ibatis.binding.BindingException: Type interface com.rui.dao.UserDao is not known to the MapperRegistry.

**MapperRegistry** 是什么?
核心配置文件中注册 mappers

- junit测试
  ```java
  package com.rui;
  
  import com.rui.dao.UserDao;
  import com.rui.pojo.User;
  import com.rui.utils.MyBatisUtils;
  import org.apache.ibatis.session.SqlSession;
  import org.junit.Test;
  
  import java.util.List;
  
  public class UserDaoTest {
      @Test
      public void test(){
          //第一步：获得SqlSession对象
          SqlSession sqlSession = MyBatisUtils.getSqlSession();
          //执行SQL
          UserDao mapper = sqlSession.getMapper(UserDao.class);
          List<User> userList = mapper.getUserList();
          for (User user : userList) {
              System.out.println(user);
          }
          //关闭SqlSession
          sqlSession.close();
      }
  }
  
  ```

  **可能会遇到的问题**：
  1. 配置文件没有注册
  2. 绑定接口错误
  3. 方法名不对
  4. 返回类型不对
  5. Maven导出资源问题

# 3、CRUD

## 1、namespace
namespace 中的包名要和 Dao/mapper 接口的包名保持一致

## 2、select

选择查询语句；
- id：就是对应的namespace中的方法名；
- resultType：Sql语句执行的返回值！
- parameterType：参数类型！
1. 编写接口

   ```java
   package com.rui.dao;
   
   import com.rui.pojo.User;
   
   import java.util.List;
   
   public interface UserMapper {
       //根据id查询用户
       User getUserById(int id);
   }
   
   ```

2. 编写对应的mapper中的sql语句
   ```java
   <select id="getUserById" resultType="com.rui.pojo.User" parameterType="int">
          /*定义sql*/
          select * from mybatis.user where id = #{id};
   </select>
   ```

3. 测试 <!--增删改需要提交事务-->
   ```java
   @Test
   public void getUserById(){
       SqlSession sqlSession = MyBatisUtils.getSqlSession();
       UserMapper mapper = sqlSession.getMapper(UserMapper.class);
       User user = mapper.getUserById(1);
       System.out.println(user);
       sqlSession.close();
   }
   ```

## 3、Insert
## 4、Update
## 5、Delete
**注意点：增删改需要提交事务**
## 6、分析错误
- 标签不要匹配错误
- resource 绑定 mapper，需要使用路径！
- 程序配置文件必须符合规范
- NullPointerException，没有注册到资源
- 输出的 xml 文件中存在中文乱码问题
- maven 资源没有导出问题

## 7、万能Map
假设，我们的实体类，或者数据库中的表，字段或者参数过多，我们应当考虑使用Map！

```java
//万能Map
int addUser2(Map<String,Object> map);
```

```xml
<!--对象中的属性，可以直接取出来 parameterType=传递map中的key-->
<insert id="addUser2" parameterType="map">
    insert into mybatis.user (id, name, pwd) values (#{userId},#{userName},#{password});
</insert>
```

```java
//万能map
@Test
public void addUser2(){
    SqlSession sqlSession = MyBatisUtils.getSqlSession();
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    HashMap<String, Object> map = new HashMap<>();
    map.put("userId",4);
    map.put("userName","王五");
    map.put("password","23333");
    mapper.addUser2(map);
    //提交事务
    sqlSession.commit();
    sqlSession.close();
}
```

Map传递参数，直接在sql中取出key即可！【parameterType="map"】
对象传递参数，直接在sql中取对象的属性即可！【parameterType="Object"】
只有一个基本类型参数的情况下，可以直接在sql中取到！
多个参数用Map，**或者注解！**

## 8、思考
模糊查询怎么写？
1. Java代码执行的时候传递通配符 %%
   ```java
   List<User> userList=mapper.getUserLike("%李%");
   ```

2. 在 sql 拼接中使用通配符！
   ```sql
   select * from mybatis.user where name like "%"#{value}"%"
   ```


# 4、配置解析
## 1、核心配置文件
- mybatis-config.xml
- MyBatis的配置文件包含了会深深影响MyBatis行为的设置和属性信息
```xml
configuration（配置）
properties（属性）
settings（设置）
typeAliases（类型别名）
typeHandlers（类型处理器）
objectFactory（对象工厂）
plugins（插件）
environments（环境配置）
environment（环境变量）
transactionManager（事务管理器）
dataSource（数据源）
databaseIdProvider（数据库厂商标识）
mappers（映射器）
```

## 2、环境配置（environments）
MyBatis 可以配置成适应多种环境
**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
学会使用配置多套运行环境！
MyBatis默认的事务管理器就是JDBC，连接池：POOLED
## 3、属性（properties）
我们可以通过properties属性来实现引用配置文件
这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。【db.properties】
编写一个配置文件
db.properties
```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/mybatis?
useSSL=true&useUnicode=true&characterEncoding=utf8
username=root
password=Cc105481
```

在核心配置文件中引入
```properties
 <!--引入外部配置文件-->
 <properties resource="db.properties">
     <property name="username" value="root"/>
     <property name="password" value="Cc105481"/>
 </properties>
```

- 可以直接引入外部文件
- 可以在其中增加一些属性配置
- 如果两个文件有同一字段，优先使用外部配置文件的！
## 4、类型别名（typeAliases）
- 类型别名是为 Java 类型设置一个短的名字。
- 存在的意义仅在于用来减少类完全限定名的冗余。
  ```xml
  <!--可以给实体类起别名-->
  <typeAliases>
      <typeAlias type="com.rui.pojo.User" alias="User"/>
  </typeAliases>
  ```

  也可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean,比如：
  扫描实体类的包，他的默认别名就为这个类的类名，首字母小写！
  ```xml
   <!--可以给实体类起别名-->
  <typeAliases>
      <package name="com.rui.pojo"/>
  </typeAliases>
  ```

在实体类比较少的时候，使用第一种方式。
如果实体类十分多，建议使用第二种方式。
第一种可以DIY别名，第二种则不行，如果非要改，需要在实体类（pojo）上增加@Alias注解
```java
@Alias("author")
public class Author {
    ...
}
```

## 5、设置
这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。
![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-Settings-b2f373.png)

## 6、生命周期和作用域
生命周期，和作用域是至关重要的，因为错误的使用会导致非常严重的**并发问题**。

**SqlSessionFactoryBuilder**

- 一旦创建了SqlSessionFactory，就不再需要它了
- 局部变量

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-mybatis%E8%BF%90%E8%A1%8C%E6%B5%81%E7%A8%8B-7debb1.png" alt="mybatis运行流程" width="50%" />

**SqlSessionFactory：**

- 可以想象为：数据库连接池
- SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，**没有任何理由丢弃它或重新创建另一个实例**。
- 因此SqlSessionFactory的最佳作用域是应用作用域。
- 最简单的就是使用 **单例模式** 或者静态单例模式

**SqlSession：**
- 连接到连接池的一个请求！
- SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域。
- 用完之后需要赶紧关闭，否则会占用资源

  <img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-SqlSessionFactory-3d0b08.png" alt="SqlSessionFactory" width="60%" />

这里的每一个Mapper，就代表一个具体的业务！

# 5、解决属性名和字段名不一致的问题

## 1、问题
数据库中的字段

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%AD%E7%9A%84%E5%AD%97%E6%AE%B5-81723f.png)

新建一个项目，拷贝之前的，测试实体类字段不一致的情况。

```java
public class User {
    private int id;
    private String name;
    private String pwd;
}
```

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%B5%8B%E8%AF%95%E9%94%99%E8%AF%AF-4d056c.png)

```java
//select * from mybatis.user where id = #{id}
//类型处理器
//select id,name,pwd from mybatis.user where id = #{id}
```

解决方法：
- 起别名
```sql
select id,name,pwd as password from mybatis.user where id = #{id}
```

## 2、resultMap
结果集映射
```
id name pwd
id name password
```

```xml
<!--结果集映射-->
    <resultMap id="UserMap" type="User">
        <!--column数据库中的字段，property实体类中的属性-->
        <result column="	id" property="id"/>
        <result column="name" property="name"/>
        <result column="pwd" property="password"/>
    </resultMap>
    <select id="getUserById" resultMap="UserMap" parameterType="int">
       /*定义sql*/
       select * from mybatis.user where id = #{id};
   </select>
```

- `resultMap` 元素是 MyBatis 中最重要最强大的元素
- ResultMap 的设计思想是，对于简单的语句根本不需要配置显式的结果映射，而对于复杂一点的语句只需要描述它们的关系就行了。
- `ResultMap` 最优秀的地方在于，虽然你已经对它相当了解了，但是根本就不需要显式地用到他们。

# 6、日志
## 6.1、日志工厂
如果一个数据库操作，出现了异常，我们需要排错。日志就是最好的助手！
曾经：sout、debug
现在：日志工厂
![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%97%A5%E5%BF%97-9652b5.png)

- SLF4J 
- LOG4J【掌握】
- LOG4J2
- JDK_LOGGING
- COMMONS_LOGGING 
- STDOUT_LOGGING【掌握】
- NO_LOGGING

在Mybatis中具体使用那个日志实现，在设置中设定！

**STDOUT_LOGGING标准日志输出**

在mybatis核心配置文件中，配置我们的日志！

```xml
<settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
</settings>
```

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%97%A5%E5%BF%97%E9%85%8D%E7%BD%AE-fbf6c3.png)

## 6.2、Log4j

什么是log4j
- Log4j 是 [Apache](https://baike.baidu.com/item/Apache/8512995) 的一个开源项目，通过使用 Log4j，我们可以控制日志信息输送的目的地是[控制台](https://baike.baidu.com/item/控制台/2438626)、文件、[GUI](https://baike.baidu.com/item/GUI)组件
- 我们也可以控制每一条日志的输出格式
- 通过定义每一条日志信息的级别，我们能够更加细致地控制日志的生成过程。
- 通过一个[配置文件](https://baike.baidu.com/item/配置文件/286550)来灵活地进行配置，而不需要修改应用的代码。

1.先导入log4j的包

```xml
<dependencies>
    <!-- https://mvnrepository.com/artifact/log4j/log4j -->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
</dependencies>
```

2.log4j.properties
```properties
log4j.rootLogger=debug, stdout, R
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
# Pattern to output the caller's file name and line number.
log4j.appender.stdout.layout.ConversionPattern=%5p [%t] (%F:%L) - %m%n
log4j.appender.R=org.apache.log4j.RollingFileAppender
log4j.appender.R.File=example.log
log4j.appender.R.MaxFileSize=100KB
# Keep one backup file
log4j.appender.R.MaxBackupIndex=5
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=%p %t %c - %m%n

```

3.配置log4j为日志实现
```xml
<settings>
    <setting name="logImpl" value="LOG4J"/>
</settings>
```

4.log4j的使用！直接测试运行刚才的查询
```
DEBUG [main] (LogFactory.java:105) - Logging initialized using 'class org.apache.ibatis.logging.log4j.Log4jImpl' adapter.
DEBUG [main] (LogFactory.java:105) - Logging initialized using 'class org.apache.ibatis.logging.log4j.Log4jImpl' adapter.
DEBUG [main] (PooledDataSource.java:353) - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] (PooledDataSource.java:353) - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] (PooledDataSource.java:353) - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] (PooledDataSource.java:353) - PooledDataSource forcefully closed/removed all connections.
DEBUG [main] (JdbcTransaction.java:136) - Opening JDBC Connection
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
DEBUG [main] (PooledDataSource.java:424) - Created connection 2049051802.
DEBUG [main] (JdbcTransaction.java:100) - Setting autocommit to false on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7a220c9a]
DEBUG [main] (BaseJdbcLogger.java:143) - ==>  Preparing: /*定义sql*/ select * from mybatis.user where id = ?; 
DEBUG [main] (BaseJdbcLogger.java:143) - ==> Parameters: 1(Integer)
DEBUG [main] (BaseJdbcLogger.java:143) - <==      Total: 1
User{id=1, name='狂神', password='123456'}
DEBUG [main] (JdbcTransaction.java:122) - Resetting autocommit to true on JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7a220c9a]
DEBUG [main] (JdbcTransaction.java:90) - Closing JDBC Connection [com.mysql.cj.jdbc.ConnectionImpl@7a220c9a]
DEBUG [main] (PooledDataSource.java:381) - Returned connection 2049051802 to pool.


Disconnected from the target VM, address: '127.0.0.1:58296', transport: 'socket'

Process finished with exit code 0
```

**简单使用**
1. 在要使用Log4j 的类中，导入org.apache.log4j.Logger;
2. 日志对象，加载参数为当前类的class
   ```java
    static Logger logger = Logger.getLogger(UserDaoTest.class);
   ```

3. 日志级别
   ```java
   logger.info("info:进入了testLog4j方法");
   logger.debug("debug:进入了testLog4j");
   logger.error("error:进入了testLog4j");
   ```

# 7、分页
**思考：为什么要分页？**
- 减少数据的处理量
## 7.1、使用Limit分页
```sql
select * from user limit startIndex,pageSize
```
使用Mybatis实现分页，核心SQL
1. 接口
   ```java
   //分页
   List<User> getUserByLimit(Map<String,Integer> map);
   ```
2. Mapper.xml
   ```xml
   <!--分页-->
   <select id="getUserByLimit" parameterType="map" resultMap="UserMap">
       select * from mybatis.user limit #{startIndex},#{pageSize}
   </select>
   ```
3. 测试
   ```java
   @Test
   public void getUserByLimit(){
       SqlSession sqlSession = MyBatisUtils.getSqlSession();
       UserMapper mapper = sqlSession.getMapper(UserMapper.class);
       HashMap<String, Integer> map = new HashMap<>();
       map.put("startIndex",0);
       map.put("pageSize",2);
   
   
       List<User> userList = mapper.getUserByLimit(map);
       for (User user : userList) {
           System.out.println(user);
       }
       sqlSession.close();
   }
   ```
## 7.2、RowBounds分页
不再使用SQL实现分页
1. 接口
   ```java
   List<User> getUserByRowBounds();
   ```
   
2. mapper.xml
   ```xml
   <!--分页2-->
   <select id="getUserByRowBounds" resultMap="UserMap">
       select * from mybatis.user
   </select>
   ```

3. 测试
   ```java
   @Test
   public void getUserByRowBounds(){
       SqlSession sqlSession = MyBatisUtils.getSqlSession();
       //RowBounds实现
       RowBounds rowBounds = new RowBounds(1, 2);
   
       //通过java代码层面实现分页
       List<User> userList = sqlSession.selectList("com.rui.dao.UserMapper.getUserByRowBounds",null,rowBounds);
   
       for (User user : userList) {
           System.out.println(user);
       }
       sqlSession.close();
   }
   ```

## 7.3、分页插件
<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-image-20191203175719275-106b17.png" alt="image-20191203175719275" width="50%" />

# 8、使用注解开发

## 8.1、面向接口编程
**面向接口编程的根本原因：<u>解耦</u>，可拓展，提高复用，分层开发中、上层不用管具体的实现，大家都遵守共同的标准，使得开发变得容易，规范性好**

## 8.2、使用注解开发
1. 注解在接口上实现

   ```JAVA
   @Select(value = "select * from user")
   List<User> getUsers();
   ```

2. 需要在核心配置文件中绑定接口！

   ```XML
   <!--绑定接口-->
   <mappers>
       <mapper class="rui.dao.UserMapper"/>
   </mappers>
   ```

3. 测试

```JAVA
public class UserMapperTest {
    @Test
    public void test(){
        SqlSession sqlSession = MyBatisUtils.getSqlSession();
        //底层主要应用反射
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        List<User> users = mapper.getUsers();
        for (User user : users) {
            System.out.println(user);
        }
        sqlSession.close();
    }
}
```

本质：反射机制实现
底层：动态代理！
## 8.3、CRUD
我们可以在工具类创建的时候实现自动提交事务！
```java
 public static SqlSession getSqlSession(){
        return sqlSessionFactory.openSession(true);
    }
```

编写接口，增加注解
```JAVA
public interface UserMapper {
    @Select(value = "select * from user")
    List<User> getUsers();

    //方法存在多个参数，所有的参数前面必须加上@Param注解
    @Select("select * from user where id = #{id} or name = #{name}")
    User getUserByID(@Param("id")int id,@Param("name")String name);

    @Insert("insert into user(id,name,pwd) values (#{id},#{name},#{password})")
    int addUser(User user);

    @Update("update user set name = #{name},pwd = #{password} where id = #{id}")
    int updateUser(User user);

    @Delete("delete from user where id = #{uid}")
    int deleteUser(@Param("uid") int id);
}
```

测试类
【注意：我们必须要将接口注册绑定到我们的核心配置文件中！】

**关于@Param()注解**
- 基本类型的参数或者String类型，需要加上
- 引用类型不需要加
- 如果只有一个基本类型的话，可以忽略，但是建议大家都加上
- 我们在SQL中引用的就是我们这里的@Param()中设定的属性名

**#{}   ${}区别**

# 9、Lombok

```java
Project Lombok is a java library that automatically plugs into your editor and build tools, spicing up your java.
Never write another getter or equals method again, with one annotation your class has a fully featured builder, Automate your logging variables, and much more.
```

使用步骤：
1. 在IDEA中安装Lombok插件
2. 在项目中导入lombok的jar包

   ```xml
    <!-- https://mvnrepository.com/artifact/org.projectlombok/lombok -->
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <version>1.18.10</version>
           </dependency>
   ```

3. 在实体类上加注解即可


   ```java
   @Getter and @Setter
   @FieldNameConstants
   @ToString
   @EqualsAndHashCode
   @AllArgsConstructor, @RequiredArgsConstructor and @NoArgsConstructor
   @Log, @Log4j, @Log4j2, @Slf4j, @XSlf4j, @CommonsLog, @JBossLog, @Flogger, @CustomLog
   @Data
   @Builder
   @SuperBuilder
   @Singular
   @Delegate
   @Value
   @Accessors
   @Wither
   @With
   @SneakyThrows
   @val
   @var
   experimental @var
   @UtilityClass
   Lombok config system
   ```

**说明：**
```java
@Data:无参构造，get、set、toSring、hashcode、equals
@AllArgsConstructor
@NoArgsConstructor
@ToString
@EqualsAndHashCode
```

# 10、多对一处理

多对一：
- 多个学生，对应一个老师
- 对于学生这边而言，**关联**...多个学生，关联一个老师【多对一】
- 对于老师而言，**集合**，一个老师又很多学生【一对多】

SQL:

```sql
CREATE TABLE `teacher`(
`id` int(10) Not null,
`name` VARCHAR(30) DEFAULT NULL,
PRIMARY KEY (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO teacher(`id`,`name`) VALUES (1,'秦老师');

CREATE TABLE `student`(
`id` int(10) Not null,
`name` VARCHAR(30) DEFAULT NULL,
`tid` INT(10) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `fktid`(`tid`),
CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
)ENGINE=INNODB DEFAULT CHARSET=utf8

INSERT INTO student(`id`,`name`,`tid`) VALUES (1,'小明',1);
INSERT INTO student(`id`,`name`,`tid`) VALUES (2,'小红',1);
INSERT INTO student(`id`,`name`,`tid`) VALUES (3,'小张',1);
INSERT INTO student(`id`,`name`,`tid`) VALUES (4,'小李',1);
INSERT INTO student(`id`,`name`,`tid`) VALUES (5,'小王',1);
```

## 测试环境
1. 导入lombok
2. 新建实体类Teacher，Student
3. 新建Mapper接口
4. 建立Mapper.XML文件
5. 在核心配置文件中绑定注册我们的MApper接口或者文件！【方式很多，随意选】
6. 测试查询是否成功！

## 按照查询嵌套处理
```XML
<!--
思路：
    1、查询所有的学生信息
    2、根据查询出来的学生的id的tid，寻找对应的老师！ -子查询

-->

<select id="getStudent" resultMap="StudentTeacher">
select * from student
</select>

<resultMap id="StudentTeacher" type="com.rui.pojo.Student">
    <!--复杂的属性，我们需要单独处理  对象：association  集合：collection-->
    <association property="teacher" column="tid" javaType="com.rui.pojo.Teacher" select="getTeacher"/>
</resultMap>

<select id="getTeacher" resultType="com.rui.pojo.Teacher">
    select * from teacher where id = #{id}
</select>
```

## 按照结果嵌套处理
```XML
<!--按照结果嵌套处理-->
<select id="getStudent2" resultMap="StudentTeacher2">
    select s.id sid,s.name sname,t.name tname,t.id tid
    from student s,teacher t
    where s.tid=t.id;
</select>

<resultMap id="StudentTeacher2" type="com.rui.pojo.Student">
    <result property="id" column="sid"/>
    <result property="name" column="sname"/>
    <association property="teacher" javaType="com.rui.pojo.Teacher">
        <result property="id" column="tid"></result>
        <result property="name" column="tname"></result>
    </association>

</resultMap>
```

回顾Mysql多对一查询方式：
- 子查询
- 联表查询

# 11、一对多处理
比如：一个老师拥有多个学生！
对于老师而言，就是一对多的关系！
## 环境搭建
1. 环境搭建，和刚才一样

实体类
```JAVA
@Data
public class Teacher {
    private int id;
    private String name;

    //一个老师拥有多个学生
    private List<Student> students;
}
```

```JAVA
@Data
public class Student {
    private int id;
    private String name;
    private int tid;

}
```

## 按照结果嵌套处理
```XML
<!--按结果嵌套查询-->
<select id="getTeacher" resultMap="TeacherStudent">
    select s.id sid,s.name sname,t.name tname,t.id tid
    from student s,teacher t
    where s.tid=t.id and t.id = #{tid}
</select>
<resultMap id="TeacherStudent" type="com.rui.pojo.Teacher">
    <result property="id" column="tid"/>
    <result property="name" column="tname"/>
    <!--复杂的属性，我们需要单独处理  对象：association  集合：collection
        javaType="" 指定属性的类型
        集合中的泛型信息，我们使用ofType获取
    -->
    <collection property="students" ofType="com.rui.pojo.Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <result property="tid" column="tid"/>
    </collection>
</resultMap>
```

## 按照查询嵌套处理
```XMl
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from mybatis.teacher where id = #{tid}
</select>
<resultMap id="TeacherStudent2" type="com.rui.pojo.Teacher">
    <collection property="students" javaType="ArrayList" ofType="com.rui.pojo.Student" select="getStudentByTeacherId" column="id"/>
</resultMap>

<select id="getStudentByTeacherId" resultType="com.rui.pojo.Student">
    select * from mybatis.student where tid = #{tid}
</select>
```

## 小节
1. 关联-association【多对一】
2. 集合-collection   【一对多】
3. javaType & ofType
   1. JavaType用来指定实体类中属性的类型
   2. ofType用来指定映射到List或者集合中的pojo类型，泛型中的约束类型！

注意点：
- 保证SQL的可读性，尽量保证通俗易懂
- 注意一对多和多对一中，属性名和字段的问题！
- 如果问题不好排查错误，可以使用日志，建议使用Log4j

**慢SQL		1S		1000S**

面试高频
- Mysql引擎
- InnoDB底层原理
- 索引
- 索引优化！

# 12、动态SQL
**什么事动态SQL：动态SQL就是指根据不同的条件生成不同的SQL语句**
利用动态SQL这一特性可以彻底摆脱这种痛苦
动态 SQL 元素和 JSTL 或基于类似 XML 的文本处理器相似。在 MyBatis 之前的版本中，有很多元素需要花时间了解。MyBatis 3 大大精简了元素种类，现在只需学习原来一半的元素便可。MyBatis 采用功能强大的基于 OGNL 的表达式来淘汰其它大部分元素。
- if
- choose (when, otherwise)
- trim (where, set)
- foreach

## 搭建环境
```sql
CREATE TABLE `bolg`(
	`id` VARCHAR(50) NOT NULL COMMENT '博客id',
	`title` VARCHAR(100) not null comment '博客标题',
	`author` VARCHAR(30) not null comment '博客作者',
	`creat_time` datetime not null comment '创建时间',
	`views` int(30) not null comment '浏览量'
)ENGINE=InnoDB DEFAULT CHARSET=utf8
```

创建一个基础工程
1. 导包
2. 编写配置文件
3. 编写实体类
   ```java
   @Data
   public class Blog {
       private int id;
       private String title;
       private String author;
       private Date creatTime;
       private int views;
   }
   ```

4. 编写实体类对应的Mapper接口和Mapper.xml
## IF
```XML
<select id="queryBlogIF" parameterType="map" resultType="com.rui.pojo.Blog">
    select * from mybatis.bolg where 1=1
    <if test="title != null">
        and title = #{title}
    </if>
    <if test="author != null">
        and author = #{author}
    </if>
</select>
```

```JAVA
@Test
public void queryBlogIF(){
    SqlSession sqlSession = MyBatisUtils.getSqlSession();
    BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
    HashMap map = new HashMap();
    map.put("author","尹锐");
    List<Blog> blogs = mapper.queryBlogIF(map);
    for (Blog blog : blogs) {
        System.out.println(blog);
    }
    sqlSession.close();
}
```

## choose (when, otherwise)
```XML
<select id="queryBlogChoose" parameterType="map" resultType="com.rui.pojo.Blog">
    select * from mybatis.bolg
    <where>
        <choose>
            <when test="title != null">
                title=#{title}
            </when>
            <when test="author!=null">
                and author = #{author}
            </when>
            <otherwise>
                and views = #{views}
            </otherwise>
        </choose>
    </where>
</select>
```

## trim, (where, set)
```xml
select * from mybatis.bolg
<where>
<if test="title != null">
    title = #{title}
</if>
<if test="author != null">
    and author = #{author}
</if>
</where>
```

```XML
<update id="updateBlog" parameterType="map">
    update mybatis.bolg
    <set>
        <if test="title != null">
            title = #{title},
        </if>
        <if test="author != null">
            author = #{author},
        </if>
    </set>
    where id = #{id}
</update>
```

**所谓的动态SQL，本质还是SQL语句，只是我们可以在SQL层面，去执行一些逻辑代码**
if
Where,set,choose,when
## SQL片段
有的时候，我们可能会将一些公共的部分抽取出来，方便复用！
1. 使用SQL标签抽取公共的部分

   ```XML
   <sql id="if-title-author">
       <if test="title != null">
           title = #{title}
       </if>
       <if test="author != null">
           and author = #{author}
       </if>
   </sql>
   ```

2. 在需要使用的地方使用Include标签引用即可
   ```XML
   <select id="queryBlogIF" parameterType="map" resultType="com.rui.pojo.Blog">
       select * from mybatis.bolg
       <where>
           <include refid="if-title-author"></include>
       </where>
   </select>
   ```

   注意事项：
   - 最好基于单表来定义SQL片段！
   - 不要存在where或者set**标签**，片段里尽量只有if就好了

   ## Foreach
   ```sql
   select * from user where 1=1 and 
     <foreach item="id" index="index" collection="ids"
         open="(" separator="or" close=")">
           #{id}
     </foreach>
   
   (id=1 or id=2 or id=3)
   ```
   
   <img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-image-20191205141204114-bb38d5.png" alt="image-20191205141204114" width="90%" />

   ```xml
   <!--
   select * from mybatis.bolg where 1=1 and (id=1 or id=2 or id=3)
   
   我们现在传递一个万能的map，这个map中可以存在一个map
   -->
   <select id="queryBlogForeach" parameterType="map" resultType="com.rui.pojo.Blog">
       select * from mybatis.bolg
   
       <where>
       <foreach collection="ids" item="id" open="(" close=")" separator="or">
           id = #{id}
       </foreach>
       </where>
   </select>
   ```
   
   **动态SQL就是在拼接SQL语句，我们只要保证SQL的正确性，按照SQL的格式，去排列组合就可以了**
   
   建议：
   
   - 先在Mysql中写出完整的SQL，在对应的去修改称为我们的动态SQL

# 13、缓存
## 13.1、简介
```
查询	：		连接数据库，耗资源！
		一次查询的结果，给他暂存在一个可以直接取到的地方！--->内存	：	缓存
		
我们再次查询相同数据的时候，直接走缓存，就不用走数据库了
```

1. 什么是缓存[Cache]?
   - 存在内存中的临时数据。
   - 将用户经常查询的数据放在缓存（内存）中，用户去查询数据就不用从磁盘上（关系型数据库数据文件）查询，
     从缓存中查询，从而提高查询效率，解决了高并发系统的性能问题。
2. 为什么使用缓存？
   - 减少和数据库的交互次数，减少系统开销，提高系统效率。
3. 什么样的数据能使用缓存？
   - 经常查询并且不经常改变的数据。

## 13.2、Mybatis缓存
- MyBatis包含一个非常强大的查询缓存特性，它可以非常方便地定制和配置缓存。缓存可以极大的提升查询效率。
- MyBatis系统中默认定义了两级缓存：**一级缓存**和**二级缓存**
  - 默认情况下，只有一级缓存开启。（SqlSession级别的缓存，也称为本地缓存）
  - 二级缓存需要手动开启和配置，他是基于namespace级别的缓存。
  - 为了提扩展性，MyBatis定义了缓存接口Cache。我们可以通过实现Cache接口来自定义二级缓存

## 13.3、一级缓存
- 一级缓存也叫本地缓存：SqlSession
  - 与数据库同义词会话期间查询到的数据会放在本地缓存中。
  - 以后如果需要获取相同的数据，直接从缓存中拿，没有必要再去查询数据；

**测试步骤：**
1. 开启日志!
2. 测试在一个Session中查询两次相同的记录
3. 查看日志输出

**缓存失效的情况：**
1. 查询不同的东西
2. 增删改操作，可能会改变原来的数据，所以必定会刷新缓存！
3. 查询不同的Mapper.xml
4. 手动清理缓存！
   ```java
   sqlsession.clearCache(); //手动清理缓存
   ```

小节：一级缓存默认是开启的，只在一次SqlSession中有效，也就是拿到连接到关闭连接这个区间段！

一级缓存就是一个Map。
## 13.4、二级缓存
- 二级缓存也叫全局缓存，一级缓存作用域太低了，所以诞生了二级缓存
- 基于namespace级别的缓存，一个名称空间，对应一个二级缓存；
- 工作机制
  - 一个会话查询一条数据，这个数据就会被放在当前会话的一级缓存中；
  - 如果当前会话关闭了，这个会话对应的一级缓存就没了；但是我们想要的是，会话关闭了，一级缓存中的数据会被保存到二级缓存中；
  - 新的会话查询信息，就可以从二级缓存中获取内容；
  - 不同的mapper查出的数据会放在自己对应的缓存（map）中；

步骤：
1. 开启全局缓存
   ```xml
   <!--显式的开启全局缓存-->
   <setting name="cacheEnabled" value="true"/>
   ```

2. 在要使用二级缓存的Mapper中开启
   ```xml
   <!--在当前Mapper.xml中使用二级缓存-->
   <cache/>
   也可以自定义参数
   <cache eviction="FIFO"
          flushInterval="60000"
          size="512"
          readOnly="true"/>
   ```

3. 测试
   1. 问题：我们需要将实体类序列化！否则就会报错

      ```java
       java.io.NotSerializableException: com.rui.pojo.User
      ```


小结：
- 只要开启了二级缓存，在同一个Mapper下就有效
- 所有的数据都会先放在一级缓存中；
- 只有当会话提交，或者关闭的时候，才会提交到二级缓存中！

## 13.5、缓存原理

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-image-20191205170549715-74e4a5.png)

## 13.6、自定义缓存-encache
```xml
EhCache 是一个纯Java的进程内缓存框架，具有快速、精干等特点，是Hibernate中默认的CacheProvider。
```

要在程序中使用ehcache，先要导包！
```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.caches/mybatis-ehcache -->
<dependency>
    <groupId>org.mybatis.caches</groupId>
    <artifactId>mybatis-ehcache</artifactId>
    <version>1.1.0</version>
</dependency>

```

然后在mapper中指定使用ehcache缓存实现
```xml
<!--在当前Mapper.xml中使用二级缓存-->
<cache type="org.mybatis.caches.ehcache.EhcacheCache"/>
```

导入配置文件 ehcache.xml
```xml

<?xml version="1.0" encoding="UTF-8"?>
<ehcache xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="http://ehcache.org/ehcache.xsd"
         updateCheck="false">
    <!--
       diskStore：为缓存路径，ehcache分为内存和磁盘两级，此属性定义磁盘的缓存位置。参数解释如下：
       user.home – 用户主目录
       user.dir  – 用户当前工作目录
       java.io.tmpdir – 默认临时文件路径
     -->
    <diskStore path="java.io.tmpdir/Tmp_EhCache"/>
    <!--
       defaultCache：默认缓存策略，当ehcache找不到定义的缓存时，则使用这个缓存策略。只能定义一个。
     -->
    <!--
      name:缓存名称。
      maxElementsInMemory:缓存最大数目
      maxElementsOnDisk：硬盘最大缓存个数。
      eternal:对象是否永久有效，一但设置了，timeout将不起作用。
      overflowToDisk:是否保存到磁盘，当系统当机时
      timeToIdleSeconds:设置对象在失效前的允许闲置时间（单位：秒）。仅当eternal=false对象不是永久有效时使用，可选属性，默认值是0，也就是可闲置时间无穷大。
      timeToLiveSeconds:设置对象在失效前允许存活时间（单位：秒）。最大时间介于创建时间和失效时间之间。仅当eternal=false对象不是永久有效时使用，默认是0.，也就是对象存活时间无穷大。
      diskPersistent：是否缓存虚拟机重启期数据 Whether the disk store persists between restarts of the Virtual Machine. The default value is false.
      diskSpoolBufferSizeMB：这个参数设置DiskStore（磁盘缓存）的缓存区大小。默认是30MB。每个Cache都应该有自己的一个缓冲区。
      diskExpiryThreadIntervalSeconds：磁盘失效线程运行时间间隔，默认是120秒。
      memoryStoreEvictionPolicy：当达到maxElementsInMemory限制时，Ehcache将会根据指定的策略去清理内存。默认策略是LRU（最近最少使用）。你可以设置为FIFO（先进先出）或是LFU（较少使用）。
      clearOnFlush：内存数量最大时是否清除。
      memoryStoreEvictionPolicy:可选策略有：LRU（最近最少使用，默认策略）、FIFO（先进先出）、LFU（最少访问次数）。
      FIFO，first in first out，这个是大家最熟的，先进先出。
      LFU， Less Frequently Used，就是上面例子中使用的策略，直白一点就是讲一直以来最少被使用的。如上面所讲，缓存的元素有一个hit属性，hit值最小的将会被清出缓存。
      LRU，Least Recently Used，最近最少使用的，缓存的元素有一个时间戳，当缓存容量满了，而又需要腾出地方来缓存新的元素的时候，那么现有缓存元素中时间戳离当前时间最远的元素将被清出缓存。
   -->
    <defaultCache
            eternal="false"
            maxElementsInMemory="10000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="259200"
            memoryStoreEvictionPolicy="LRU"/>
 
    <cache
            name="cloud_user"
            eternal="false"
            maxElementsInMemory="5000"
            overflowToDisk="false"
            diskPersistent="false"
            timeToIdleSeconds="1800"
            timeToLiveSeconds="1800"
            memoryStoreEvictionPolicy="LRU"/>
 
</ehcache>
```

