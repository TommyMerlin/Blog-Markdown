---
title: 数据库基础6:用户管理DCL
date: 2020-03-04 16:11:35
tags:
- 数据库
- MySql
categories:
- 数据库
---

## SQL分类

- DDL：操作数据库和表
- DML：增删改表中数据
- DQL：查询表中数据
- DCL：管理用户，授权

<!-- more -->

## DCL：管理用户，授权

### 1.管理用户

- 添加用户
  - CREATE USER '用户名'@'主机名' IDENTIFIED BY '密码';
- 删除用户
  - DROP USER '用户名'@'主机名';
- 修改用户密码

```sql
UPDATE USER SET PASSWORD = PASSWORD('新密码') WHERE USER = '用户名';
UPDATE USER SET PASSWORD = PASSWORD('abc') WHERE USER = 'lisi';

SET PASSWORD FOR '用户名'@'主机名' = PASSWORD('新密码');
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123');
```

- 查询用户
  - 通配符： % 表示可以在任意主机使用用户登录数据库

```sql
-- 1. 切换到mysql数据库
USE myql;
-- 2. 查询user表
SELECT * FROM USER;
```

### 2.权限管理

- 查询权限

```sql
-- 查询权限
SHOW GRANTS FOR '用户名'@'主机名';
SHOW GRANTS FOR 'lisi'@'%';
```

- 授予权限

```sql
-- 授予权限
grant 权限列表 on 数据库名.表名 to '用户名'@'主机名';
-- 给张三用户授予所有权限，在任意数据库任意表上

GRANT ALL ON *.* TO 'zhangsan'@'localhost';
```

- 撤销权限

```sql
-- 撤销权限：
revoke 权限列表 on 数据库名.表名 from '用户名'@'主机名';
REVOKE UPDATE ON db3.`account` FROM 'lisi'@'%';
```

