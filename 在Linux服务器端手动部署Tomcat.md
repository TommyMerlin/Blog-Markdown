---
title: 在Linux服务器端手动部署Tomcat
date: 2020-02-24 21:05:18
tags:
- Tomcat
- Linux
categories:
- Linux
---

## 1.安装 jdk 并配置路径

将 jdk 解压放到 `/usr/local/` 目录下。

修改 `/etc/profile` 文档，在末尾添加如下语句：

```
export JAVA_HOME=/usr/local/jdk1.8.0_191
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```
刷新配置文件：

```bash
source /etc/profile
```

F:\Blog\source\_posts

## 2.安装 Tomcat

解压Tomcat压缩包。

修改 `/conf/server.xml`文件，将端口8080改为80。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E4%BF%AE%E6%94%B9Server%E9%85%8D%E7%BD%AE-c3a7a7.png" alt="修改Server配置" />

## 3.在阿里云服务器上添加安全组规则

添加80端口的监听。

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-%E6%B7%BB%E5%8A%A0%E5%AE%89%E5%85%A8%E7%BB%84%E8%A7%84%E5%88%99-f44069.png" alt="添加安全组规则" width="50%" />

## 4.启动 Tomcat 服务器

执行 `/bin/startup.sh`脚本，启动服务器，访问服务器地址就能看到部署成功！

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-06-19-Tomcat-99c683.png" alt="Tomcat" width="60%" />