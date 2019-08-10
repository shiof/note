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