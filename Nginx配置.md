---
title: Nginx配置
date: 2020-04-18 18:24:00
tags:
- Nginx
- Linux
categories:
- 服务器软件
- Nginx
---

## 反向代理

```
server {
    listen         80;
    server_name    47.94.232.51;
    location / {
    	proxy_pass  http://47.94.232.51:8080;
    }
    location ~ /baidu/ {
    	proxy_pass  http://47.94.232.51:8081;
    }
}
```

<!-- more -->

## 负载均衡

**nginx 分类服务器策略**

- 轮询（默认）：每个请求按时间顺序注意分配到不同后端服务器，若后端服务器 down 掉，能自动剔除。
- weight：权重，默认为1，权重越高被分配的客户端越多。
- ip_hash：每个请求按访问的 ip 的 hash 结果分配，每个访客固定访问一个后端服务器，可解决 session 问题。
- fair：按后端服务器的响应时间分配请求，响应时间短的优先分配。

```
upstream myserver{
    server 47.94.232.51:8080 weight=1;
    server 47.94.232.51:8081 weight=1;
}

server {
    listen         80;
    server_name    47.94.232.51;
    location / {
    	proxy_pass  http://myserver;
    }
}
```

## 动静分离

创建 `www` 和 `image` 文件夹，分别存放 html 页面和图片。

```
server {
    listen         80;
    server_name    47.94.232.51;
    location /www/ {
    	root /data/;
    	index index.html index.htm;
    }
    location /image/ {
    	root /data/;
    	autoindex on;
    }
}
```

## 配置高可用

nginx 宕机，请求无法到达指定服务器。