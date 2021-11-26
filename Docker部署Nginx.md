---
title: Docker部署Nginx
date: 2020-04-18 12:50:00
tags:
- Nginx
- Docker
- Linux
categories:
- 服务器软件
- Nginx
---

## 1.拉取 Nginx 镜像

```
docker search nginx
docker pull nginx
```

<!-- more -->

## 2.使用 Docker-Composite 部署

### 2.1 建立挂载文件

目录结构：

```
.
├── conf
│   └── nginx.conf
├── docker-compose.yml
├── logs
└── www
```

编写 `nginx.conf`

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;
}
```

### 2.2 编写 `docker-composite.yml`

```yml
version: '3.1'
services:
  nginx:
    image: nginx
    restart: always
    container_name: nginx
    ports:
      - 8000:80
    volumes:
      - ./logs:/var/log/nginx
      - ./www:/usr/share/nginx/html
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
```

### 2.3 启动

```bash
docker-composite up -d
```

## 3.Nginx常用命令

```bash
# 查看 nginx 版本号
nginx -v
# 停止 nginx
nginx -s stop
# 重新加载 nginx
nginx -s reload
```

