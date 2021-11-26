---
title: Docker安装IK中文分词器
date: 2020-05-16 14:22:56
tags:
- Docker
- ElasticSearch
categories:
- Java
- ElasticSearch
---

## 1.下载插件

[medcl / elasticsearch-analysis-ik](https://github.com/medcl/elasticsearch-analysis-ik/releases)

<!-- more -->

## 2.拷贝插件到容器

```bash
docker cp /root/elasticsearch-analysis-ik-7.7.0.zip <container_id>:/usr/share/elasticsearch/plugins/
```

## 3.安装插件

```bash
# 进入容器
docker exec -it <container_id> bash
mkdir /usr/share/elasticsearch/plugins/ik
# 解压插件到 ik 文件夹
mv plugins/elasticsearch-analysis-ik-7.7.0.zip plugins/ik
unzip elasticsearch-analysis-ik-7.7.0.zip 
rm -rf elasticsearch-analysis-ik-7.7.0.zip 
# 退出容器
ctrl P+Q
# 重启容器
docker restart <container_id>
```

