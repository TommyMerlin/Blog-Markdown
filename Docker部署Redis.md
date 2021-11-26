---
title: Docker部署Redis
date: 2020-05-01 18:57:00
tags:
- Redis
- Docker
- Linux
categories:
- Docker
---

## 1.拉取 Redis 镜像

```bash
docker pull redis
```

## 2.启动 Redis

```bash
docker run -d --name redis6379 -p 6379:6379 redis --requirepass "zjuhuye"
```

## 3.进入 Redis Cli

```bash
docker exec -it redis6379 redis-cli -a zjuhuye
```

