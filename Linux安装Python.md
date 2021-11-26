---
title: Linux安装Python
date: 2020-05-11 20:15:11
tags:
- Linux
- Python
- Jupyter Notebook
categories:
- Python
---

# 1.安装 Python

### 1.1 下载 python3 源码包

```bash
wget https://www.python.org/ftp/python/3.7.6/Python-3.7.6.tgz
```

<!-- more -->

### 1.2 下载 python3 编译的依赖包 

```bash
yum install -y gcc patch libffi-devel python-devel  zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```

### 1.3 解压缩源码包

```bash
tar -zxvf Python-3.7.6.tgz
```

### 1.4 编译安装

```bash
cd Python-3.7.6
./configure --prefix=/opt/python37   # 指定安装目录为/opt/python37
make && make install
```

### 1.5 更改 linux 的 path 变量

```bash
vim  /etc/profile
```

`/etc/profile`

```
PATH=/opt/python37/bin:$PATH
export PATH
```

```bash
source /etc/profile
```

***

# 2.安装 Jupyter Notebook

### 2.1 pip 安装

```bash
pip install jupyter
```

### 2.2 更改默认工作路径

- 生成配置文件

```bash
jupyter-notebook --generate-config
```

- 打开配置文件

```bash
vim ~/.jupyter/jupyter_notebook_config.py
```

- 设置默认路径

```py
c.NotebookApp.notebook_dir = '/root/JupyterNotebook'
```

