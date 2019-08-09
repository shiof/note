---
layout: article
title:	Hadoop集群环境准备
date:	2018-06-10 15:11:56
categories:
    - article
tags:
    - Hadoop
    - BigData
---


准备三台虚拟机，安装centos7系统，配置ip和hostname


~~~
192.168.56.110 hadoop-master
192.168.56.111 hadoop-node-1
192.168.56.112 hadoop-node-2
~~~

关闭防火墙

~~~shell
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
~~~

> *注意centos默认是开启防火墙的，如果不关闭的话，导出子节点无法连接上主节点

配置host文件

~~~shell
cat << EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.110 hadoop-master
192.168.56.111 hadoop-node-1
192.168.56.112 hadoop-node-2
EOF
~~~

配置集群免密码登录

~~~shell
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh
chmod 700 ~/.ssh/authorized_keys
#hadoop-master节点的公钥发到hadoop-node-1、hadoop-node-2上
ssh-copy-id -i hadoop-node-1
ssh-copy-id -i hadoop-node-2
#hadoop-node-1节点的公钥发到hadoop-master、hadoop-node-2上
ssh-copy-id -i hadoop-master
ssh-copy-id -i hadoop-node-2
#hadoop-node-2节点的公钥发到hadoop-master、hadoop-node-1上
ssh-copy-id -i hadoop-master
ssh-copy-id -i hadoop-node-1
#测试hadoop-master能否登录上hadoop-node-1
ssh root@192.168.56.111
~~~

## 安装java环境（省略）




