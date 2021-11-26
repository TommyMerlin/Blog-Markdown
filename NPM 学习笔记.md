---
title: NPM 学习笔记
date: 2021-05-18 10:30:53
tags:
- 前端
- npm
categories:
- 前端
- 前端工具
---

## 1.概念

[NPM](https://www.npmjs.com/) 的全称是 Node Package Manager，是一个 [NodeJS](https://nodejs.org/) **包管理和分发**工具。

<!-- more -->

## 2.NPM 镜像设置与查看

- 搭建环境时通过以下代码将默认镜像改为淘宝源。

```bash
npm config set registry https://registry.taobao.org --global
npm config set disturl https://npm.taobao.org/dist --global
```

- 查看镜像的配置结果。

```bash
npm config get registry
npm config get disturl
```

- 使用 nrm 工具切换淘宝源

```bash
npm nrm use taobao
```

- 如果之后需要切换回官方源可使用

```bash
npm nrm use npm
```

## 3.NPM 的使用

```bash
npm -v
npm install <ModuleName>			# 在当前目录下安装模块
npm install <ModuleName> -g			# 全局安装
npm -g install npm@5.9.1			# 安装指定版本的模块
npm uninstall <ModuleName>			# 卸载模块
npm update <ModuleName>				# 更新模块
npm list -g							# 查看所有全局安装的模块
npm list vue						# 查看某个模块的版本号
npm install --save <ModuleName>     # -save 在 package 文件中的 dependencies 节点写入依赖
npm install --save-dev <ModuleName> # -save-dev|-D 在 package 文件中的 devDependencies 节点写入依赖

npm init (--yes/-y)					# 初始化 npm 项目 (--yes 参数设定为默认配置)
npm install 						# 根据 package.json 文件下载依赖
```

## 4. package.json 文件

```json
{
  "name": "npm-demo",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "HuYe",
  "license": "ISC",
// dependencies：运行时的依赖，发布后，即生产环境下还需要用的模块
  "dependencies": {				
    "jquery": "^3.6.0"			
// ^3.6.0 代表安装 3.X.X 中的最新版本
// ~3.6.0 代表安装 3.6.X 中的最新版本
// 3.6.0  固定版本
  },
// devDependencies：开发时的依赖，发布时不需要，如项目中使用的压缩 css、js 的模块，这些模块在项目部署后是不需要的
  "devDependencies": {
    "bootstrap": "^5.0.1"
  }
}
```

