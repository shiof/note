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

### docker网络命令

|命令|描述|示例|
|-------------------------|-------------------|---------------------------------------------|
|docker network create    |创建网络            |docker network create my-network             |
|docker network connect	  |容器加入网络         |docker network connect my-network tomcat     |
|docker network disconnect|断开容器的网络       |docker network disconnect my-network tomcat  |
|docker network inspect	  |查看网络信息         |docker network inspect my-network            |
|docker network ls	      |查看网络列表         |docker network ls                            |
|docker network prune	  |删除所有未被使用的网络|docker network prune                          |
|docker network rm	      |删除网络            |docker network rm my-network                  |


> [1] [https://docs.docker.com/engine/reference/commandline/network_create/#specify-advanced-options](https://docs.docker.com/engine/reference/commandline/network_create/#specify-advanced-options)