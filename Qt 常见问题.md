---
title: Qt 常见问题
date: 2021-07-09 11:03:53
tags:
- C++
- Qt
- Visual Studio
- 常见问题
categories:
- C++
- Qt
---

## 1.exe文件运行依赖问题

**问题**：运行 exe 文件提示如下问题：

![](https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-07-09-image-20210709111235972-6a092d.png)

<!-- more -->

**解决**：使用 `windeployqt.exe` 工具自动检测运行程序所需动态库。

工具路径：`D:\Qt\Qt5.14.2\5.14.2\msvc2017_64\bin`

```bash
windeployqt.exe 目标.exe
```

## 2.中文乱码

声明文件使用的编码格式：

```c++
#pragma execution_character_set("utf-8")
```

<img src="https://aliyun-oss-coderhuye.oss-cn-hangzhou.aliyuncs.com/blog/2021-07-09-image-20210709114815625-0083f5.png" alt="image-20210709114815625" style="width:50%;" />

## 3.VTK8.2.0在Windows10+VS2017+Qt 5.12环境下编译安装

[VTK8.2.0在Windows10+VS2017+Qt 5.12环境下编译安装](https://blog.csdn.net/annjeff/article/details/88597051)

[详细的VTKVSQT配置vtk8.2.0+vs2017+QT5.9.2](https://blog.csdn.net/wilsonass/article/details/89192007)

