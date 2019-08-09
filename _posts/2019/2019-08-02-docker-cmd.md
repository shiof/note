---
layout: article
title:	Docker - 常用命令
date:	2019-08-02 15:34:00
categories:
    - article
tags:
    - Docker
    - Linux
---

#### 删除为 `none` 镜像

~~~shell
docker images|grep none|awk '{print $3 }'|xargs docker rmi -f
~~~

### 删除所有容器

~~~shell
docker rm `docker ps -a -q`
~~~

### 查看镜像信息

~~~shell
docker inspect tomcat
~~~

### 查看镜像信息某个字段

~~~shell
docker inspect -f {% raw %} {{.NetworkSettings.IPAddress}} {% endraw %} tomcat
~~~

### 安装sonarqube

~~~shell
docker run -itd --name sonarqube \
    -p 9000:9000 \
    -e sonar.jdbc.username=sonar \
    -e sonar.jdbc.password=sonar \
    -e sonar.jdbc.url="jdbc:postgresql://localhost/sonar" \
    --restart=always \
    sonarqube
~~~