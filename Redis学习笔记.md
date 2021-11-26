---
title: Redis学习笔记
date: 2020-02-28 09:02:24
tags:
- 数据库
- Java
- Redis
categories:
- Java
- 数据库
---

## 1.概念

redis 是一款高性能的 NoSQL系列的非关系型数据库。

### 1.1 NoSQL

**NoSQL**(NoSQL = Not Only SQL)，意即“不仅仅是 SQL”，是一项全新的数据库理念，泛指非关系型的数据库。

随着互联网 web2.0 网站的兴起，传统的关系数据库在应付 web2.0 网站，特别是超大规模和高并发的 SNS 类型的 web2.0 **纯动态**网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。NoSQL 数据库的产生就是为了解决大规模数据集合多重数据种类带来的挑战，尤其是**大数据应用**难题。

<!-- more -->

#### NoSQL和关系型数据库比较

**优点：**

- 成本：NoSQL 数据库**简单易部署**，基本都是开源软件，相比关系型数据库**价格便宜**。
- 查询速度：NoSQL 数据库将数据存储于缓存之中，关系型数据库将数据存储在硬盘中，**查询速度**远不及 NoSQL 数据库。
- 存储数据的格式：NoSQL 的存储格式是 key,value 形式、文档形式、图片形式等等，所以可以**存储基础类型**以及对象或者是集合等各种格式，而数据库则只支持基础类型。
- 扩展性：关系型数据库有类似 join 这样的多表查询机制的限制导致扩展很艰难。

**缺点：**

- 维护的工具和资料有限，因为 NoSQL 是属于新的技术，不能和关系型数据库十几年的技术同日而语。
- 不提供对 sql 的支持，如果不支持 sql 这样的工业标准，将产生一定用户的学习和使用成本。
- 不提供关系型数据库对事务的处理。

#### 非关系型数据库的优势

- NoSQL 是基于键值对的，可以想象成表中的主键和值的对应关系，而且不需要经过 SQL 层的解析，所以**性能非常高**。
- 可扩展性同样也是因为基于键值对，数据之间没有耦合性，所以非常容易**水平扩展**。

#### 关系型数据库的优势

- 复杂查询可以用 SQL 语句方便的在一个表以及多个表之间做非常**复杂的数据查询**。
- **事务支持**使得对于安全性能很高的数据访问要求得以实现。

#### 小结

- 关系型数据库与 NoSQL 数据库并非对立而是**互补**的关系，即通常情况下使用关系型数据库，在适合使用 NoSQL 的时候使用 NoSQL 数据库，让 NoSQ L数据库对关系型数据库的不足进行弥补。
- 一般会将数据存储在关系型数据库中，在 NoSQL 数据库中**备份**存储关系型数据库的数据

### 1.2 主流的 NoSQL 产品

#### 键值(Key-Value)存储数据库

- 相关产品： Tokyo Cabinet/Tyrant、Redis、Voldemort、Berkeley DB。
- 典型应用： 内容缓存，主要用于处理大量数据的高访问负载。 
- 数据模型： 一系列键值对。
- 优势： 快速查询。
- 劣势： 存储的数据缺少结构化。

#### 列存储数据库

- 相关产品：Cassandra, HBase, Riak。
- 典型应用：分布式的文件系统。
- 数据模型：以列簇式存储，将同一列数据存在一起。
- 优势：查找速度快，可扩展性强，更容易进行分布式扩展。
- 劣势：功能相对局限。

#### 文档型数据库

- 相关产品：CouchDB、MongoDB。
- 典型应用：Web应用（与Key-Value类似，Value是结构化的）。
- 数据模型： 一系列键值对。
- 优势：数据结构要求不严格。
- 劣势： 查询性能不高，而且缺乏统一的查询语法。

#### 图形(Graph)数据库

- 相关数据库：Neo4J、InfoGrid、Infinite Graph。
- 典型应用：社交网络。
- 数据模型：图结构。
- 优势：利用图结构相关算法。
- 劣势：需要对整个图做计算才能得出结果，不容易做分布式的集群方案。

### 1.3 什么是 Redis

Redis 是用 C 语言开发的一个开源的高性能键值对（key-value）数据库，官方提供测试数据，50 个并发执行 100000 个请求,读的速度是 110000 次/s，写的速度是 81000 次/s ，且 Redis 通过提供多种键值数据类型来适应不同场景下的存储需求。

#### Redis 的应用场景

- 缓存（数据查询、短连接、新闻内容、商品内容等等）。
- 聊天室的在线好友列表。
- 任务队列。（秒杀、抢购、12306 等等）。
- 应用排行榜。
- 网站访问统计。
- 数据过期处理（可以精确到毫秒）。
- 分布式集群架构中的 session 分离。

## 2.下载安装

- [官网](https://redis.io/)
- [中文网](http://www.redis.net.cn/)
- 解压直接可以使用：
  - redis.windows.conf：**配置文件**
  - redis-cli.exe：redis的**客户端**
  - redis-server.exe：redis**服务器端**

## 3.命令操作

### 3.1 Redis 的数据结构

Redis 存储的是：key,value 格式的数据，其中 key 都是**字符串**，value 有 5 种不同的数据结构：

- 字符串类型 string
- 哈希类型 hash：map 格式 。
- 列表类型 list：linkedlist 格式，**支持重复元素**。
- 集合类型 set：**不允许重复元素**。
- 有序集合类型 sortedset：**不允许重复元素**，且元素有顺序。

### 3.2 字符串类型 string

- 存储： set key value

```bash
127.0.0.1:6379> set username zhangsan
OK
```

- 获取： get key

```bash
127.0.0.1:6379> get username
"zhangsan"
```

- 删除： del key

```bash
127.0.0.1:6379> del age
(integer) 1
```

### 3.3 哈希类型 hash

- 存储： hset key field value

```bash
127.0.0.1:6379> hset myhash username lisi
(integer) 1
127.0.0.1:6379> hset myhash password 123
(integer) 1
```

- 获取：
```bash
  # hget key field: 获取指定的field对应的值。
  127.0.0.1:6379> hget myhash username
  "lisi"
```

```bash
  # hgetall key：获取所有的field和value。
  127.0.0.1:6379> hgetall myhash
  1) "username"
  2) "lisi"
  3) "password"
  4) "123"
```

- 删除： hdel key field

```bash
127.0.0.1:6379> hdel myhash username
(integer) 1
```

### 3.4 列表类型 list

可以添加一个元素到列表的头部（左边）或者尾部（右边）。

- 添加：
  - lpush key value: 将元素加入列表左表。
  - rpush key value：将元素加入列表右边。

```bash
127.0.0.1:6379> lpush myList a
(integer) 1
127.0.0.1:6379> lpush myList b
(integer) 2
127.0.0.1:6379> rpush myList c
(integer) 3
```

- 获取：lrange key start end ：范围获取。

```bash
127.0.0.1:6379> lrange myList 0 -1
1) "b"
2) "a"
3) "c"
```

- 删除：
  - lpop key： 删除列表最左边的元素，并将元素返回。
  - rpop key： 删除列表最右边的元素，并将元素返回。

### 3.5 集合类型 set

- 存储：sadd key value

```bash
127.0.0.1:6379> sadd myset a
(integer) 1
127.0.0.1:6379> sadd myset a
(integer) 0
```

- 获取：smembers key：获取 set 集合中所有元素。

```bash
127.0.0.1:6379> smembers myset
1) "a"
```

- 删除：srem key value：删除 set 集合中的某个元素。

```bash
127.0.0.1:6379> srem myset a
(integer) 1
```

### 3.6 有序集合类型 sortedset

不允许重复元素，且元素有顺序。每个元素都会关联一个 double 类型的分数。redis 正是通过分数来为集合中的成员进行从小到大的排序。

- 存储：zadd key score value

```bash
127.0.0.1:6379> zadd mysort 60 zhangsan
(integer) 1
127.0.0.1:6379> zadd mysort 50 lisi
(integer) 1
127.0.0.1:6379> zadd mysort 80 wangwu
(integer) 1
```

- 获取：zrange key start end [withscores]

```bash
127.0.0.1:6379> zrange mysort 0 -1
1) "lisi"
2) "zhangsan"
3) "wangwu"

127.0.0.1:6379> zrange mysort 0 -1 withscores
1) "zhangsan"
2) "60"
3) "wangwu"
4) "80"
5) "lisi"
6) "500"
```

- 删除：zrem key value

```bash
127.0.0.1:6379> zrem mysort lisi
(integer) 1
```

### 3.7 通用命令

- keys * : 查询所有的键。
- type key ： 获取键对应的 value 的类型。
- del key：删除指定的key value。

## 4.持久化层

redis 是一个内存数据库，当 redis 服务器重启，或电脑重启，数据会丢失，我们可以将 redis 内存中的数据持久化保存到硬盘的文件中。

redis **持久化机制**：

- RDB：默认方式，不需要进行配置，在一定的间隔时间中，检测 key 的变化情况，然后持久化数据。

  - 1.编辑 redis.windwos.conf  文件。

    ```bash
    #   after 900 sec (15 min) if at least 1 key changed
    save 900 1
    #   after 300 sec (5 min) if at least 10 keys changed
    save 300 10
    #   after 60 sec if at least 10000 keys changed
    save 60 10000
    ```

  - 2.重新启动 redis 服务器，并指定配置文件名称。

  ```bash
  D:\JavaWeb2018\redis\windows-64\redis-2.8.9>redis-server.exe redis.windows.conf
  ```

- AOF：日志记录的方式，可以记录每一条命令的操作。可以每一次命令操作后，持久化数据。

  - 编辑 redis.windwos.conf 文件。

  ```bash
  appendonly no（关闭aof） --> appendonly yes （开启aof）
  				
  # appendfsync always ： 每一次操作都进行持久化
  appendfsync everysec ： 每隔一秒进行一次持久化
  # appendfsync no	 ： 不进行持久化
  ```

## 5.Java客户端 Jedis

Jedis: 一款 java 操作 redis 数据库的工具。

### 使用步骤

- 下载 jedis 的 jar 包。
- 使用

```java
//1. 获取连接
Jedis jedis = new Jedis("localhost",6379);
//2. 操作
jedis.set("username","zhangsan");
//3. 关闭连接
jedis.close();
```

### 操作数据

- 字符串类型 string

```java
//1. 获取连接
Jedis jedis = new Jedis();//如果使用空参构造，默认值 "localhost",6379端口
//2. 操作
//存储
jedis.set("username","zhangsan");
//获取
String username = jedis.get("username");

//可以使用setex()方法存储可以指定过期时间的 key value
jedis.setex("activecode",20,"hehe");//将activecode：hehe键值对存入redis，并且20秒后自动删除该键值对

//3. 关闭连接
jedis.close();
```

- 哈希类型 hash ： map格式 。

```java
// 存储hash
jedis.hset("user","name","lisi");
jedis.hset("user","age","23");
jedis.hset("user","gender","female");

// 获取hash
String name = jedis.hget("user", "name");
// 获取hash的所有map中的数据
Map<String, String> user = jedis.hgetAll("user");

// keyset
Set<String> keySet = user.keySet();
for (String key : keySet) {
    //获取value
    String value = user.get(key);
    System.out.println(key + ":" + value);
}
```

- 列表类型 list ： linkedlist格式，支持重复元素。

```java
// list 存储
jedis.lpush("mylist","a","b","c");//从左边存
jedis.rpush("mylist","a","b","c");//从右边存

// list 范围获取
List<String> mylist = jedis.lrange("mylist", 0, -1);

// list 弹出
String element1 = jedis.lpop("mylist");//c

String element2 = jedis.rpop("mylist");//c

// list 范围获取
List<String> mylist2 = jedis.lrange("mylist", 0, -1);
```

- 集合类型 set  ： 不允许重复元素。

```java
// set 存储
jedis.sadd("myset","java","php","c++");

// set 获取
Set<String> myset = jedis.smembers("myset");
```

- 有序集合类型 sortedset：不允许重复元素，且元素有顺序。

```java
// sortedset 存储
jedis.zadd("mysortedset",3,"亚瑟");
jedis.zadd("mysortedset",30,"后羿");
jedis.zadd("mysortedset",55,"孙悟空");

// sortedset 获取
Set<String> mysortedset = jedis.zrange("mysortedset", 0, -1);
```

### Jedis 连接池：JedisPool

**使用**

```java
//0.创建一个配置对象
JedisPoolConfig config = new JedisPoolConfig();
config.setMaxTotal(50);
config.setMaxIdle(10);

//1.创建Jedis连接池对象
JedisPool jedisPool = new JedisPool(config,"localhost",6379);

//2.获取连接
Jedis jedis = jedisPool.getResource();
//3. 使用
jedis.set("hehe","heihei");
//4. 关闭 归还到连接池中
jedis.close();
```

**连接池工具类**

```java
public class JedisPoolUtils {

    private static JedisPool jedisPool;

    static{
        //读取配置文件
        InputStream is = JedisPoolUtils.class.getClassLoader()
            .getResourceAsStream("jedis.properties");
        //创建Properties对象
        Properties pro = new Properties();
        //关联文件
        try {
            pro.load(is);
        } catch (IOException e) {
            e.printStackTrace();
        }
        //获取数据，设置到JedisPoolConfig中
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(Integer.parseInt(pro.getProperty("maxTotal")));
        config.setMaxIdle(Integer.parseInt(pro.getProperty("maxIdle")));

        //初始化JedisPool
        jedisPool = new JedisPool(config,pro.getProperty("host"),
                                  Integer.parseInt(pro.getProperty("port")));
		}
    
    /**
    * 获取连接方法
    */
    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}
```

## 6.案例

案例需求：

- 提供 index.html 页面，页面中有一个省份 下拉列表。
- 当 页面加载完成后，发送 ajax 请求，加载所有省份。

注意：

- 使用 redis 缓存一些不经常发生变化的数据。
- 数据库的数据一旦发生改变，则需要更新缓存。

[代码](https://gitee.com/TommyMerlin/code-host/tree/master/Java/redis)