---
layout: article
title:	Docker安装
date:	2018-07-29 11:43:17
categories:
    - article
tags:
    - docker
    - linux
---


### 1.删除旧版本

```shell
yum -y remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine
```

### 2.配置yum源以及docker库配置

```shell
yum-config-manager --add-repo   https://download.docker.com/linux/centos/docker-ce.repo
```

> centos没有自带yum-config-manager
>
> 需要自己安装
>
> yum -y install yum-utils


#### 2.1（可选）启用edge和test版，配置存储在docker.repo里

```shell
yum-config-manager --enable docker-ce-edge
yum-config-manager --enable docker-ce-test
```

#### 2.2禁用edge和test版，使用stable版（稳定版）

```shell
yum-config-manager --disable docker-ce-edge
yum-config-manager --disable docker-ce-test
```

### 3.通过yum安装docker-ce

```shell
yum -y install docker-ce
```

### 4.docker开机启动

~~~shell
systemctl daemon-reload
systemctl enable docker
systemctl start docker
~~~

### 4.1添加docker用户组

~~~shell
groupadd docker
usermod -aG docker $USER
~~~

### 5. docker默认配置文件

#### 5.1 /etc/docker/daemon.json 

~~~json
{
    "live-restore": true,
    "bip": "198.168.1.1/24",
    "dns": ["198.168.1.1"],
    "registry-mirrors": ["https://registry.docker-cn.com"],
    "insecure-registries": ["ooips.com:5000"]
}
~~~

> live-restore  关闭docker daemon ，而不关闭容器
>
> bip     docker内部镜像分配的IP地址范围，如果使用kubernetes的话，此配置会被覆盖
>
> registry-mirrors   镜像加速地址
>
> insecure-registries 私有仓库配置
>
> [daemon.json](./daemon.json)

#### 5.2 /var/lib/docker

docker 镜像和容器信息存储位置

#### 5.3 docker 网络代理配置

~~~shell
mkdir -p /etc/systemd/system/docker.service.d

cat << EOF > /etc/systemd/system/docker.service.d/proxy.conf
[Service]
Environment="HTTP_PROXY=http://172.17.18.80:8080"
Environment="HTTPS_PROXY=http://172.17.18.80:8080"
Environment="NO_PROXY=.x,hub.com,h.com,.cae.com"
EOF
~~~

#### 5.4 创建docker用户组

~~~shell
addgroup docker
usermod -aG docker $USER
~~~

#### 5.5 修改配置文件后要重启docker

~~~shell
systemctl daemon-reload
systemctl restart docker
~~~

### 6.搭建docker私有仓库

~~~shell
docker run -d -p 5000:5000 --restart=always --privileged=true --name registry registry:latest
~~~

> --restart=always    容器重启策略
>
> --privileged=true   安全模块selinux把权限禁掉了
>
> 查看docker是否搭建成功 [http://localhost.com:5000/v2/_catalog](http://localhost.com:5000/v2/_catalog)

### 7.安装其它版本

~~~shell
yum -y install docker-ce-17.03.1.ce-1.el7.centos.x86_64 --setopt=obsoletes=0
~~~

### 8.开启docker socket

~~~shell
cat > /etc/systemd/system/docker.service.d/tcp.conf <<EOF
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd -H unix:///var/run/docker.sock -H tcp://0.0.0.0:2375
EOF
~~~

> https://docs.docker.com/engine/api/v1.32/