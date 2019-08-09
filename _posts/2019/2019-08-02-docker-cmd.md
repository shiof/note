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