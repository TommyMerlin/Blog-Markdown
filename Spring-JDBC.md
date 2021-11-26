---
title: Spring JDBC
date: 2020-02-26 12:04:40
tags:
- 数据库
- Java
categories:
- Java
- 数据库
---

## 概念

**Spring** 框架对 JDBC 的简单封装，提供了一个 JDBCTemplate 对象简化 JDBC 的开发。

## 步骤

#### 1.导入 jar 包。

#### 2.创建 JdbcTemplate 对象。依赖于数据源DataSource

```java
JdbcTemplate template = new JdbcTemplate(ds);
```

<!-- more -->

#### 3.调用JdbcTemplate的方法来完成CRUD的操作。

- update()：执行DML语句。增、删、改语句。
- queryForMap()：查询结果将结果集封装为map集合，将列名作为key，将值作为value 将这条记录封装为一个map集合。
  - 注意：这个方法查询的结果集长度只能是1。
- queryForList()：查询结果将结果集封装为list集合。
  - 注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中。
- query()：查询结果，将结果封装为JavaBean对象。
  - query的参数：RowMapper
    - 一般使用BeanPropertyRowMapper实现类，可以完成数据到JavaBean的自动封装。
    - new BeanPropertyRowMapper<类型>(类型.class)
- queryForObject：查询结果，将结果封装为对象。
  - 一般用于聚合函数的查询。

#### 4.示例

需求：

- 修改1号数据的 salary 为 10000。
- 添加一条记录。
- 删除刚才添加的记录。
- 查询id为1的记录，将其封装为Map集合。
- 查询所有记录，将其封装为List。
- 查询所有记录，将其封装为[Emp对象](https://gitee.com/TommyMerlin/code-host/blob/master/Java/Emp.java)的List集。
- 查询总记录数。

数据表：

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E8%A1%A8-10e0a4.png" alt="表" width="50%" />

代码：

```java
//获取JDBCTemplate对象
private JdbcTemplate template = new JdbcTemplate(JDBCUtils.getDataSource());
```
```java
//1. 修改1号数据的 salary 为 10000
public void test1(){
    //定义sql
    String sql = "update emp set salary = 10000 where id = 1001";
    //执行sql
    int count = template.update(sql);
    System.out.println(count);
}
```

```java
//2. 添加一条记录
public void test2(){
    String sql = "insert into emp(id,ename,dept_id) values(?,?,?)";
    int count = template.update(sql, 1015, "郭靖", 10);
    System.out.println(count);
}
```

```java
//3.删除刚才添加的记录
public void test3(){
    String sql = "delete from emp where id = ?";
    int count = template.update(sql, 1015);
    System.out.println(count);
}
```

```java
//4.查询id为1001的记录，将其封装为Map集合
//注意：这个方法查询的结果集长度只能是1
public void test4(){
    String sql = "select * from emp where id = ?";
    Map<String, Object> map = template.queryForMap(sql, 1001);
    System.out.println(map);
    //查询结果：{id=1001, ename=孙悟空, job_id=4, mgr=1004, joindate=2000-12-17, salary=10000.00, bonus=null, dept_id=20}
}
```

```java
//5. 查询所有记录，将其封装为List
public void test5(){
    String sql = "select * from emp";
    List<Map<String, Object>> list = template.queryForList(sql);
    for (Map<String, Object> stringObjectMap : list) 			           			         	System.out.println(stringObjectMap);
    }
}
```

```java
//6. 查询所有记录，将其封装为Emp对象的List集合-->手动实现RowMapper接口
public void test6(){
	String sql = "select * from emp";
	List<Emp> list = template.query(sql, new RowMapper<Emp>() {
			
		@Override
		public Emp mapRow(ResultSet rs, int i) throws SQLException {
            Emp emp = new Emp();
            int id = rs.getInt("id");
            String ename = rs.getString("ename");
            int job_id = rs.getInt("job_id");
            int mgr = rs.getInt("mgr");
            Date joindate = rs.getDate("joindate");
            double salary = rs.getDouble("salary");
            double bonus = rs.getDouble("bonus");
            int dept_id = rs.getInt("dept_id");
			
            emp.setId(id);
            emp.setEname(ename);
            emp.setJob_id(job_id);
            emp.setMgr(mgr);
            emp.setJoindate(joindate);
            emp.setSalary(salary);
            emp.setBonus(bonus);
            emp.setDept_id(dept_id);
			
			return emp;
		}
	});
	for (Emp emp : list) {
		System.out.println(emp);
	}
}
```

**重点掌握**

```java
//6. 查询所有记录，将其封装为Emp对象的List集合-->自动封装emp对象
public void test6(){
	String sql = "select * from emp";
	List<Emp> list = template.query(sql, new BeanPropertyRowMapper<Emp>(Emp.class));
	for (Emp emp : list) {
		System.out.println(emp);
	}
}
```

```java
//7. 查询总记录数
public void test7(){
    String sql = "select count(id) from emp";
    Long total = template.queryForObject(sql, Long.class);
    System.out.println(total);
}
```



