---
title: Idea集成Docker实现镜像一键打包部署
date: 2020-04-20 13:29:05
tags:
- Docker
- Idea
- Linux
categories:
- Docker
---

## 1.Docker开启远程访问

**修改Docker服务文件**

```bash
vim /lib/systemd/system/docker.service
# 修改ExecStart行
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

<!-- more -->

**刷新 Docker 配置，重启 Docker 服务**

```bash
# 重新加载配置文件
systemctl daemon-reload
# 重启服务
systemctl restart docker.service
# 查看端口是否开启
netstat -nlpt
# 直接curl看是否生效
curl http://127.0.0.1:2375/info
```

## 2.Idea配置Docker

## 3.docker-maven-plugin

```xml
<groupId>com.example</groupId>
<artifactId>demo</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>demo</name>
<description>Demo project for Spring Boot</description>

<properties>
    <java.version>1.8</java.version>
    <docker.image.prefix>7ommymerlin</docker.image.prefix>
</properties>

<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
        <plugin>
            <groupId>com.spotify</groupId>
            <artifactId>docker-maven-plugin</artifactId>
            <version>1.0.0</version>
            <configuration>
                <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                <imageTags>
                    <imageTag>latest</imageTag>
                </imageTags>
                <baseImage>java</baseImage>
                <maintainer>coderhuye coderhuye@coderhuye.com</maintainer>
                <workdir>/ROOT</workdir>
                <cmd>["java","--version"]</cmd>
                <entryPoint>["java","-jar","${project.build.finalName}.jar"]</entryPoint>
                <dockerHost>http://{host}:2375</dockerHost>

                <resources>
                    <resource>
                        <targetPath>/ROOT</targetPath>
                        <directory>${project.build.directory}</directory>
                        <include>${project.build.finalName}.jar</include>
                    </resource>
                </resources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

构建并部署镜像：

```bash
mvn clean package docker:build
```

## 4.扩展配置

绑定 Docker 命令到 maven 各个阶段。

> Docker 分为 build、tag、push，然后分别绑定到 maven 的 package、deploy 阶段。
>
> 只需执行 `mvn deploy` 就可以完成整个 build、tag、push 操作，当执行 `mvn build` 就只完成 build、tag 操作。

```xml
<executions>
    <!--当执行 mvn package 时，执行 mvn packag docker:build-->
    <execution>
        <id>build-image</id>
        <phase>package</phase>
        <goals>
            <goal>build</goal>
        </goals>
    </execution>
</executions>
```

## 5.Idea整合Docker加密认证

[官网示例Demo](https://docs.docker.com/engine/security/https/#create-a-ca-server-and-client-keys-with-openssl)

[B站视频](https://www.bilibili.com/video/BV1xJ411W7Rg?p=33)

```bash
[root@AliHuYe ~]# mkdir -p /usr/local/ca
[root@AliHuYe ~]# cd /usr/local/c


[root@AliHuYe /usr/local/ca]# openssl genrsa -aes256 -out ca-key.pem 4096


[root@AliHuYe /usr/local/ca]# openssl req -new -x509 -days 365 -key ca-key.pem -


[root@AliHuYe /usr/local/ca]# openssl genrsa -out server-key.pem 4096


[root@AliHuYe /usr/local/ca]# openssl req -subj "/CN=www.coderhuye.com" -sha256 -new -key server-key.pem -out server.csr


[root@AliHuYe /usr/local/ca]# echo subjectAltName = IP:127.0.0.1 >> extfile.cnf


[root@AliHuYe /usr/local/ca]# echo extendedKeyUsage = serverAuth >> extfile.cnf


[root@AliHuYe /usr/local/ca]# openssl x509 -req -days 365 -sha256 -in server.csr -CA ca.pem -CAkey ca-key.pem \
-CAcreateserial -out server-cert.pem -extfile extfile.cnf


[root@AliHuYe /usr/local/ca]# openssl genrsa -out key.pem 4096


[root@AliHuYe /usr/local/ca]# openssl req -subj '/CN=client' -new -key key.pem -out client.csr


[root@AliHuYe /usr/local/ca]# echo extendedKeyUsage = clientAuth >> extfile.cnf


[root@AliHuYe /usr/local/ca]# echo extendedKeyUsage = clientAuth > extfile-client.cnf


[root@AliHuYe /usr/local/ca]# openssl x509 -req -days 365 -sha256 -in client.csr -CA ca.pem -CAkey ca-key.pem \
-CAcreateserial -out cert.pem -extfile extfile-client.cnf


[root@AliHuYe /usr/local/ca]# rm -v client.csr server.csr extfile.cnf extfile-client.cnf


[root@AliHuYe /usr/local/ca]# chmod -v 0400 ca-key.pem key.pem server-key.pem


[root@AliHuYe /usr/local/ca]# chmod -v 0444 ca.pem server-cert.pem cert.pem


[root@AliHuYe /usr/local/ca]# cp server-*pem /etc/docker/
[root@AliHuYe /usr/local/ca]# cp ca.pem /etc/docker/


[root@AliHuYe /usr/local/ca]# vim /lib/systemd/system/docker.service

# 将 ExecStart=/usr/bin/dockerd
# 替换为:
ExecStart=/usr/bin/dockerd --tlsverify --tlscacert=/usr/local/ca/ca.pem --tlscert=/usr/local/ca/server-cert.pem --tlskey=/usr/local/ca/server-key.pem -H=tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock


[root@AliHuYe /usr/local/ca]# systemctl daemon-reload
[root@AliHuYe /usr/local/ca]# systemctl restart docker.service
[root@AliHuYe /usr/local/ca]# systemctl restart docker
```

