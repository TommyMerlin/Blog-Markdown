---
title: Tomcat部署项目的三种方式
date: 2020-02-24 19:55:04
tags:
- Tomcat
categories:
- 服务器软件
- Tomcat
---

## 1.直接将项目放到webapps目录下

- 项目的访问路径-->虚拟路径
- 简化部署：将项目打包成一个war包，再将war包放置到webapps目录下
  - war包会自动解压缩（热部署）

<!-- more -->

## 2.配置conf/server.xml文件

在 `Host` 标签体中配置

> <Context daoBase="D:\hello" path="/hello"/>

- docBase：项目存放的路径
- path：虚拟路径

## 3.在conf\Catalina\localhost创建任意名称的xml文件

在 xml 文件中编写`<Context daoBase="D:\hello" />`

- 虚拟目录：xml 文件的名称

## 4.Tomcat 在 Windows 中后台运行

一、修改 tomcat 里面的配置

1. 找到 tomcat下 `bin/setclasspath.bat` 文件打开。
2. 在文件中找到 `set_RUNJAVA="%JRE_HOME\bin\java"`, 并修改为 `set_RUNJAVA="%JRE_HOME\bin\javaw"` 。
3. 然后重启 tomcat，命令行窗口即会消失，不会出现在任务栏上，而只是在后台运行。

二、把 tomcat 注册成服务，这种方式配置可以用程序来控制tomcat的启停，无需手动来控制启停。

1. 进入Tomcat `\bin` 目录 找到 `service.bat` 的文件。
2. cmd 进入 `\bin` 目录。 
3. 启动服务 `service.bat install [service_name]`   删除服务 `service.bat remove  [service_name]`。
4. 在打开计算机->管理->服务这一栏可以看到在服务中添加了一个 tomcat 的服务，只需要将此服务开启即可，若要开机启动就将服务设成是自动的。