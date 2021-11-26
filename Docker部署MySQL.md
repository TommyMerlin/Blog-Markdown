---
title: Docker部署MySQL
date: 2020-04-23 21:21:35
tags:
- 数据库
- MySql
- Docker
categories:
- Docker
---

## 1.部署 MySql

拉取镜像

```bash
docker pull mysql
```

<!-- more -->

准备 `docker-compose.yml`

```yml
version: '3.1'
services:
  mysql:
    image: mysql
    restart: always
    container_name: mysql
    environment:
      MYSQL_ROOT_PASSWORD: ***
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql
      - ./conf:/etc/mysql/conf.d
      - ./logs:/var/log/mysql
```

开启容器

```bash
docker-compose up -d
```

## 2.错误解决

SQL客户端连接出现 `2059-authentication plugin' caching sha2 password' cannot be loaded` 错误

```bash
docker exec -it <container_id> bash
mysql --user=root --password
alter user 'root' identified with mysql_native_password by '123456';
```



