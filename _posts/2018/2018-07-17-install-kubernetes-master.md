---
layout: article
title:	Kubernetes主节点安装
date:	2018-07-17 15:15:13
categories:
    - article
tags:
    - docker
    - Kubernetes
---


* ** 192.168.56.100 master.paas.x 为主节点 **

#### 1. 安装

```shell
yum install etcd kubernetes-master flannel  -y
```

#### 2. 配置/etc/kubernetes/config

```shell
cat << EOF > /etc/kubernetes/config
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
KUBE_ALLOW_PRIV="--allow-privileged=false"
KUBE_MASTER="--master=http://master.paas.x:8080"
EOF
```

#### 3. 配置/etc/sysconfig/flanneld **

```shell
cat << EOF > /etc/sysconfig/flanneld
FLANNEL_ETCD_ENDPOINTS="http://master.paas.x:2379"
FLANNEL_ETCD_PREFIX="/paas.x/network"
FLANNEL_OPTIONS="--iface=enp0s3 -ip-masq=true"
EOF
```

#### 4. 配置/etc/etcd/etcd.conf

```shell
cat << EOF > /etc/etcd/etcd.conf
ETCD_NAME=default
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_CLIENT_URLS="http://0.0.0.0:2379"
ETCD_ADVERTISE_CLIENT_URLS="http://master.paas.x:2379"
EOF
```

#### 5. 配置/etc/kubernetes/apiserver

```shell
cat << EOF > /etc/kubernetes/apiserver
KUBE_API_ADDRESS="--address=0.0.0.0"
KUBE_API_PORT="--port=8080"
KUBELET_PORT="--kubelet-port=10250"
KUBE_ETCD_SERVERS="--etcd-servers=http://master.paas.x:2379"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=172.17.0.0/16"
KUBE_API_ARGS="--service-node-port-range=30000-62767 --enable-swagger-ui=true --apiserver-count=3 --audit-log-maxage=30 --audit-log-maxbackup=3 --audit-log-maxsize=100 --audit-log-path=/var/log/k8s/audit.log --event-ttl=1h"
EOF
```

#### 6. 配置ETCD网络

```shell
systemctl start etcd
etcdctl mkdir /paas.x/network
etcdctl set /paas.x/network/config '{"Network":"172.17.0.0/16"}'
```

#### 7. 启动主节点

```shell
for SERVICES in etcd docker kube-apiserver kube-controller-manager kube-scheduler; do
   systemctl restart $SERVICES
   systemctl enable $SERVICES
   systemctl status $SERVICES
done
```

> 最后能够访问 http://192.168.56.100:8080/ 说明主节点搭建成功！