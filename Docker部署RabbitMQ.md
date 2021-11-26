---
title: Docker部署RabbitMQ
date: 2020-05-01 12:50:00
tags:
- RabbitMQ
- Docker
- Linux
categories:
- Docker
---

## 1.拉取 RabbitMQ 镜像

```
docker pull qabbitmq:management
```

<!-- more -->

## 2.编写 `docker-compose.yml`

```yml
version: '3.1'
services:
  rabbitmq:
    restart: always
    image: rabbitmq:management
    container_name: rabbitmq
    ports:
      - 5672:5672
      - 15672:15672
    environment:
      TZ: Asia/Shanghai
      RABBITMQ_DEFAULT_USER: rabbit
      RABBITMQ_DEFAULT_PASS: 123456
    volumes:
      - ./data:/var/lib/rabbitmq
```

## 3.启动

```bash
docker-compose up -d
```

