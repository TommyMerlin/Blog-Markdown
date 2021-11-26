---
title: Bootstrap学习笔记
date: 2020-02-27 15:54:53
tags:
- 前端
- Bootstrap
categories:
- 前端
- 前端框架
---

[官网](https://www.bootcss.com/)

## 1.概念

一个前端开发的**框架**，来自 Twitter，是目前很受欢迎的前端框架。Bootstrap 基于 HTML、CSS、JavaScript ，简洁灵活，使得 Web 开发更加快捷。

**响应式布局**：同一套页面可以兼容不同分辨率的设备。

<!-- more -->

## 2.快速入门

- 1.下载 Bootstrap。
- 2.在项目中将三个文件夹复制。

```
bootstrap/
├── css/
│   ├── bootstrap.css
│   ├── bootstrap.css.map
│   ├── bootstrap.min.css
│   ├── bootstrap.min.css.map
│   ├── bootstrap-theme.css
│   ├── bootstrap-theme.css.map
│   ├── bootstrap-theme.min.css
│   └── bootstrap-theme.min.css.map
├── js/
│   ├── bootstrap.js
│   └── bootstrap.min.js
└── fonts/
    ├── glyphicons-halflings-regular.eot
    ├── glyphicons-halflings-regular.svg
    ├── glyphicons-halflings-regular.ttf
    ├── glyphicons-halflings-regular.woff
    └── glyphicons-halflings-regular.woff2
```

- 3.创建html页面，引入必要的资源文件

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <!-- 上述3个meta标签*必须*放在最前面，任何其他内容都*必须*跟随其后！ -->
    <title>Bootstrap Hello World</title>

    <!-- Bootstrap -->
    <link href="css/bootstrap.min.css" rel="stylesheet">

    <!-- jQuery (Bootstrap 的所有 JavaScript 插件都依赖 jQuery，所以必须放在前边) -->
    <script src="js/jquery-3.2.1.min.js"></script>
    <!-- 加载 Bootstrap 的所有 JavaScript 插件。你也可以根据需要只加载单个插件。 -->
    <script src="js/bootstrap.min.js"></script>

</head>
<body>
</body>
</html>
```

## 3.响应式布局

同一套页面可以兼容**不同分辨率**的设备。

实现：依赖于**栅格系统**：将一行平均分成12个格子，可以指定元素占几个格子。

步骤：

- 定义容器，相当于 table。
  - 容器分类：
    - container：两边留白。
    - container-fluid：每一种设备都是100%宽度。
- 定义行，相当于 tr 。  样式：row
- 定义元素。指定该元素在不同的设备上，所占的格子数目。样式：col-设备代号-格子数目。
  - xs：超小屏幕 手机 (<768px)：col-xs-12
  - sm：小屏幕 平板 (≥768px)
  - md：中等屏幕 桌面显示器 (≥992px)
  - lg：大屏幕 大桌面显示器 (≥1200px)

**注意：**

  - 一行中如果格子数目超过12，则超出部分自动换行。
  - 栅格类属性可以**向上兼容**。栅格类适用于与屏幕宽度大于或等于分界点大小的设备。
  - 如果真实设备宽度小于了设置栅格类属性的设备代码的最小值，会一个元素沾满一整行。

## 4. CSS 样式和 JS 插件

### 4.1 全局 CSS 样式

**按钮**：class="btn btn-default"

**图片**：
- class="img-responsive"：图片在任意尺寸都占100%。
- 图片形状：

  ```html
  <img src="..." alt="..." class="img-rounded">：方形
  <img src="..." alt="..." class="img-circle"> ： 圆形
  <img src="..." alt="..." class="img-thumbnail"> ：相框
  ```

**表格**
- table
- table-bordered
- table-hover

**表单**
- 给表单项添加：class="form-control" 

### **4.2 组件**

- 导航条
- 分页条

### **4.3 插件**

- 轮播图 Carousel 