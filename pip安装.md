---
title: pip安装
date: 2020-09-16 19:41:11
tags:
- Python
- pip
categories:
- Python
---

## 1.安装 pip

### 1.1 方法1

下载 [get-pip.py](https://bootstrap.pypa.io/get-pip.py) 文件

```shell
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
```

<!-- more -->

安装 pip

```shell
python get-pip.py
```

### 1.2 方法2

下载 [pip](https://pypi.org/project/pip/#files) 安装文件

下载完成之后，解压到一个文件夹，用 CMD 进入解压文件的目录，然后，在文件目录下，输入：

```shell
python setup.py install
```

## 2.pypi 镜像（清华源）使用帮助

pypi 镜像每 5 分钟同步一次。

### 2.1 临时使用

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，`simple` 不能少, 是 `https` 而不是 `http`

### 2.2 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```shell
pip install pip -U
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```shell
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```