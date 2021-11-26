---
title: Docker安装
date: 2020-04-21 9:55:05
tags:
- Docker
- Linux
categories:
- Docker
---

## 新装(CenOS7)

```bash
# 确定 CentOS 版本
cat /etc/redhat-release
# 安装 gcc 相关
yum -y install gcc
yum -y install gcc-c++
# 卸载旧版本
yum -y remove docker docker-common docker-selinux docker-engine
# 安装需要的软件包
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置 stable 镜像仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 更新 yum 软件包索引
yum makecache fast
# 安装 docker-ce
yum install -y docker-ce
# 启动 Docker
systemctl enable docker
systemctl start docker
# 测试
docker version
```

<!-- more -->

- **配置镜像加速**

[阿里云镜像加速](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)

## 卸载重装

- 卸载

```bash
systemctl stop docker
rpm -qa | grep docker
yum remove -y docker-ce
rm -rf /var/lib/docker
```

- 重装

```bash
yum list docker-ce --showduplicates | sort -r
yum install -y docker-ce
systemctl enable docker
systemctl start docker
```

## Docker-Compose

### 安装

```bash
sudo curl -L https://github.com/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 验证安装

```bash
docker-compose --version
```

### 卸载

```bash
sudo rm /usr/local/bin/docker-compose
```

