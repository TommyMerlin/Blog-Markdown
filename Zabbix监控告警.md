---
title: Zabbix监控告警
date: 2020-05-22 10:19:04
tags:
- Zabbix
- 微服务
- 监控告警
categories:
- 微服务
- 监控告警
---

[官网链接](https://www.zabbix.com/cn)

## Zabbix安装

[下载安装链接](https://www.zabbix.com/download?zabbix=5.0&os_distribution=centos&os_version=7&db=mysql&ws=apache)

**centos 7 添加阿里云镜像**

```bash
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```

[Docker安装](https://github.com/zabbix/zabbix-docker)

<!-- more -->

## Zabbix Agent安装

- 下载安装

```bash
rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum clean all
yum install zabbix-agent
```

- 修改配置

```bash
vim /etc/zabbix/zabbix_agentd.conf
```

修改以下配置

```
# zabbix server端IP
Server=192.168.10.24

# zabbix server端IP
ServerActive=192.168.10.24

# zabbix客户端主机名
Hostname=agent_1
```

- 启动Zabbix Agent服务

```bash
systemctl start zabbix-agent.service
```

