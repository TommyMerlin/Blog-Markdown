---
title: ElasticStack学习笔记
date: 2020-05-18 16:07:56
tags:
- ElasticSearch
- Beats
- Kibana
- Logstash
- ELK
categories:
- Java
- ElasticSearch
---

## 1.ElasticSearch

[官网链接](https://www.elastic.co/cn/downloads/elasticsearch)

### 1.1 RESTful API

[RESTful API](chrome-extension://aejoelaoggembcahagimdiliamlcdmfm/index.html#requests)

<!-- more -->

### 1.2 JAVA API

- 引入 maven 依赖

```xml
<!-- Spring Boot 2.3 -->
<!--<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>-->

<!-- https://mvnrepository.com/artifact/org.elasticsearch/elasticsearch -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.7.0</version>
</dependency>
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>7.7.0</version>
</dependency>
```

- configuration

`ElasticSearchConfig.java`

```java
import org.apache.commons.lang.StringUtils;
import org.apache.http.HttpHost;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import org.elasticsearch.client.RestHighLevelClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.Objects;

/**
 * @author: HuYe
 * @date: 2020/5/17
 * @description: com.coderhuye.elasticsearch.config
 */
@Configuration
public class ElasticSearchConfig {
    private static final int TIME_OUT = 5 * 60 * 1000;
    private static final int ADDRESS_LENGTH = 2;
    private static final String HTTP_SCHEME = "http";

    @Value("${elasticsearch.ip}")
    private String[] ipAddr;

    @Bean
    public RestClientBuilder restClientBuilder() {
        HttpHost[] hosts = Arrays.stream(ipAddr)
                .map(this::makeHttpHost)
                .filter(Objects::nonNull)
                .toArray(HttpHost[]::new);
        return RestClient.builder(hosts);
    }

    @Bean(name = "highLevelClient")
    public RestHighLevelClient highLevelClient(@Autowired RestClientBuilder restClientBuilder) {
        restClientBuilder.setRequestConfigCallback(
                requestConfigBuilder -> requestConfigBuilder.setSocketTimeout(TIME_OUT)
        );
        //TODO 此处可以进行其它操作
        return new RestHighLevelClient(restClientBuilder);
    }


    private HttpHost makeHttpHost(String s) {
        assert StringUtils.isNotEmpty(s);
        String[] address = s.split(":");
        if (address.length == ADDRESS_LENGTH) {
            String ip = address[0];
            int port = Integer.parseInt(address[1]);
            return new HttpHost(ip, port, HTTP_SCHEME);
        } else {
            return null;
        }
    }
}
```

- 配置 `application.yml`

```yml
server:
  port: 8100
elasticsearch:
  ip: 47.94.232.51:9200
```

- 使用

`ElasticSearchController.java`

```java
package com.coderhuye.elasticsearch.controller;

import com.coderhuye.elasticsearch.entity.User;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.action.index.IndexResponse;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.*;

import java.io.IOException;

/**
 * @author: HuYe
 * @date: 2020/5/17
 * @description: com.coderhuye.elasticsearch.controller
 */
@RestController
@RequestMapping("/")
public class ElasticSearchController {
    @Autowired
    private RestHighLevelClient highLevelClient;
    @Value("${server.port}")
    private int port;
    private static final ObjectMapper MAPPER = new ObjectMapper();

    @GetMapping("/index")
    public String index() {
        return "port:" + port;
    }

    @GetMapping("/test")
    public String test() throws IOException {
        /*Map<String, Object> data = new HashMap<>();
        data.put("name", "name8");
        data.put("age", 90);
        data.put("email", "1655676@163.com");
        data.put("hobby", "游泳，绘画");*/

        User user = new User("name9", 90, "123456789@163.com", "游泳，电影");
        String json = MAPPER.writeValueAsString(user);

        sendRequest("coderhuye",json);
        return "Success";
    }

    @PostMapping("/createData")
    public String test(@RequestBody User user) throws IOException {
        String json = MAPPER.writeValueAsString(user);
        sendRequest("coderhuye",json);
        return "Success";
    }

    public void sendRequest(String index, String json) throws IOException {
        IndexRequest indexRequest = new IndexRequest(index).source(json, XContentType.JSON);
        IndexResponse indexResponse = this.highLevelClient.index(indexRequest, RequestOptions.DEFAULT);
        System.out.println("index-->" + indexResponse.getIndex());
        System.out.println("id-->" + indexResponse.getId());
        System.out.println("version-->" + indexResponse.getVersion());
        System.out.println("result-->" + indexResponse.getResult());
        System.out.println("shardInfo-->" + indexResponse.getShardInfo());
    }
}
```

## 2.Beats

[官网链接](https://www.elastic.co/cn/beats)

### 2.1 Filebeat

- 配置文件

`coderhuye-nginx.yml`

```yml
filebeat.inputs:
#- type: log
#  enabled: true
#  paths:
#    - /usr/local/nginx/logs/*.log
#  tags: ["nginx"]
#  fields:
#    from: LinuxHuYe-nginx
#  fields_under_root: true
setup.template.settings:
  index.number_of_shards: 1
output.elasticsearch:
  hosts: ["127.0.0.1:9200"]
filebeat.config.modules:
  path: ${path.config}/modules.d/*.yml
  reload.enabled: false
setup.kibana:
  host: "127.0.0.1:5601"
```

- 启动 nginx 模块

```bash
./filebeat modules enable nginx
```

- 配置 nginx 模块

`modules.d/nginx.yml`

```yml
# Module: nginx
# Docs: https://www.elastic.co/guide/en/beats/filebeat/7.7/filebeat-module-nginx.html

- module: nginx
  # Access logs
  access:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/usr/local/nginx/logs/access.log*"]

  # Error logs
  error:
    enabled: true

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    #var.paths:

  # Ingress-nginx controller logs. This is disabled by default. It could be used in Kubernetes environments to parse ingress-nginx logs
  ingress_controller:
    enabled: false

    # Set custom paths for the log files. If left empty,
    # Filebeat will choose the paths depending on your OS.
    var.paths: ["/usr/local/nginx/logs/error.log*"]
```

- 启动 filebeat

```bash
./filebeat -e -c coderhuye-nginx.yml
```

### 2.2 Metricbeat

- 配置文件

`metricbeat.yml`

```yml
# versions, this URL points to the dashboard archive on the artifacts.elastic.co
# website.
#setup.dashboards.url:

#============================== Kibana =====================================

# Starting with Beats version 6.0.0, the dashboards are loaded via the Kibana API.
# This requires a Kibana endpoint configuration.
setup.kibana:

  # Kibana Host
  # Scheme and port can be left out and will be set to the default (http and 5601)
  # In case you specify and additional path, the scheme is required: http://localhost:5601/path
  # IPv6 addresses should always be defined as: https://[2001:db8::1]:5601
  host: "127.0.0.1:5601"

  # Kibana Space ID
  # ID of the Kibana Space into which the dashboards should be loaded. By default,
  # the Default Space will be used.
  #space.id:

#============================= Elastic Cloud ==================================

# These settings simplify using Metricbeat with the Elastic Cloud (https://cloud.elastic.co/).

# The cloud.id setting overwrites the `output.elasticsearch.hosts` and
# `setup.kibana.host` options.
# You can find the `cloud.id` in the Elastic Cloud web UI.
#cloud.id:
```

- 启动 nginx 模块

```bash
./metricbeat modules enable redis
```

- 配置 nginx 模块

`modules.d/redis.yml`

```yml
# Module: redis
# Docs: https://www.elastic.co/guide/en/beats/metricbeat/7.7/metricbeat-module-redis.html

- module: redis
  metricsets:
    - info
    - keyspace
  period: 10s

  # Redis hosts
  hosts: ["127.0.0.1:6379"]

  # Network type to be used for redis connection. Default: tcp
  #network: tcp

  # Max number of concurrent connections. Default: 10
  #maxconn: 10

  # Redis AUTH password. Empty by default.
  password: ******
```

- 启动 metricbeat

```bash
./metricbeat -e
```

## 3.Kibana

[官网链接](https://www.elastic.co/cn/downloads/kibana)

[中文文档](https://www.elastic.co/guide/cn/kibana/current/index.html)

- 配置文件

`kibana.yml`

```yml
# Kibana is served by a back end server. This setting specifies the port to use.
server.port: 5601

# Specifies the address to which the Kibana server will bind. IP addresses and host names are both valid values.
# 云服务器未内网地址或 0.0.0.0
server.host: "172.17.189.12"

# The URLs of the Elasticsearch instances to use for all your queries.
elasticsearch.hosts: ["http://172.17.189.12:9200"]
```

- 启动 kibana

```bash
./bin/kibana
# ./bin/kibana --allow-root
```

## 4.Logstash

[官网链接](https://www.elastic.co/cn/downloads/logstash)

- 入门

```bash
./bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

- 配置文件

`coderhuye-pipeline.conf`

```
input {
  file {
    path => "/usr/local/logstash/log/app.log"
    start_position => "beginning"
  }
}

filter {
  mutate {
    split => { "message" => "|" }
  }
}

#output {
#  stdout { codec => rubydebug }
#}

output {
  elasticsearch {
    hosts => ["47.94.232.51:9200"]
  }
}
```

- 启动

```bash
./bin/logstash -f codehuye-pipeline.conf
```

## 5.综合练习

### 5.1 生成日志（APP）

```java
# 日志格式
DAU|6917|评论商品|2020-05-18 19:19:03
 ↓    ↓     ↓       ↓ 
前缀|用户id|操作类型|时间戳
```

### 5.2 收集日志（Filebeat）

`coderhuye-demo.yml`

```yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /root/Spring.log
setup.template.settings:
  index.number_of_shards: 1
output.logstash:
  hosts: ["172.17.189.12:5044"]
```

```bash
./filebeat -e -c coderhuye-demo.yml
```

### 5.3 处理日志（Logstash）

`coderhuye-pipeline.conf`

```
input {
  beats {
    port => 5044
  }
}

filter {
  mutate {
    split => { "message" => "|" }
  }
  
  mutate {
    add_field => {
      "userId" => "%{[message][1]}"
      "visit" => "%{[message][2]}"
      "date" => "%{[message][3]}"
    }
  }
  
  mutate {
    convert => {
      "userId" => "integer"
      "visit" => "string"
      "date" => "string"
    }
  }
}

#output {
#  stdout { codec => rubydebug }
#}

output {
  elasticsearch {
    hosts => ["172.17.189.12:9200"]
  }
}
```

```bash
./bin/logstash -f codehuye-pipeline.conf
```

### 5.4 可视化日志（Kibana）

[教程视频](https://www.bilibili.com/video/BV1iJ411c7Az?p=64)

