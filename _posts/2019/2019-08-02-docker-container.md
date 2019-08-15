---
layout: article
title:	Docker - 常用容器安装命令
date:	2019-08-10 20:09:00
categories:
    - article
tags:
    - Docker
    - Linux
---

### 安装docker私有仓库

~~~shell
docker run -d -p 5000:5000 --restart=always --privileged=true --name registry registry:latest
~~~

### 安装MySql

~~~shell
docker run --itd --name mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
~~~

### 安装SonarQube

~~~shell
docker run -itd --name sonarqube \
    -p 9000:9000 \
    -e sonar.jdbc.username=sonar \
    -e sonar.jdbc.password=sonar \
    -e sonar.jdbc.url="jdbc:postgresql://localhost/sonar" \
    --restart=always \
    sonarqube
~~~

### 安装zookeeper

~~~yml
version: '3.1'

services:
  zoo1:
    image: zookeeper
    restart: always
    hostname: zoo1
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=0.0.0.0:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo2:
    image: zookeeper
    restart: always
    hostname: zoo2
    ports:
      - 2182:2181
    environment:
      ZOO_MY_ID: 2
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=0.0.0.0:2888:3888;2181 server.3=zoo3:2888:3888;2181

  zoo3:
    image: zookeeper
    restart: always
    hostname: zoo3
    ports:
      - 2183:2181
    environment:
      ZOO_MY_ID: 3
      ZOO_SERVERS: server.1=zoo1:2888:3888;2181 server.2=zoo2:2888:3888;2181 server.3=0.0.0.0:2888:3888;2181
~~~

> docker-compose -f stack.yml up -d


### 参考

> [1] [docker官方例子](https://docs.docker.com/samples/)