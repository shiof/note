---
layout: article
title:	在Centos下配置Kubernetes集群环境
date:	2018-07-17 15:11:56
categories:
    - article
tags:
    - Docker
    - Kubernetes
---


#### 1. 配置局域网以及主机名

```shell
hostnamectl set-hostname master.paas.x
sed -i "s/BOOTPROTO=dhcp/BOOTPROTO=static" /etc/sysconfig/network-scripts/ifcfg-enp0s3
sed -i "s/ONBOOT=no/ONBOOT=yes" /etc/sysconfig/network-scripts/ifcfg-enp0s3
echo "IPADDR=192.168.56.100" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
echo "NETMASK=255.255.255.0" >> /etc/sysconfig/network-scripts/ifcfg-enp0s3
```


#### 2. 关闭防火墙

```shell
chkconfig iptables off
systemctl stop firewalld
systemctl disable firewalld
echo "SELINUX=disabled" > /etc/selinux/config
#centos7默认没有iptables.service
#需要安装yum install iptables-services -y
iptables -F
iptables -I INPUT  -j ACCEPT
iptables -I OUTPUT -j ACCEPT
iptables -P FORWARD ACCEPT
service iptables save
```

#### 3. 配置hosts文件

```shell
cat << EOF > /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.56.100 master.paas.x
192.168.56.101 node1.paas.x
192.168.56.102 node2.paas.x
192.168.56.103 node3.paas.x
192.168.56.110 docker.x
EOF
```

#### 4. 安装常用的命令工具

```shell
   yum install wget vim docker lrzsz iptables-services traceroute ntp lsof -y
   #ntp为时区同步
   timedatectl set-timezone Asia/Shanghai
   systemctl enable ntpd
```