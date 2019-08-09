---
layout: article
title:	Kubernetes从节点安装
date:	2018-07-17 15:15:13
categories:
    - article
tags:
    - docker
    - Kubernetes
---


** 为kubernetes搭建3个从节点。 **

* 192.168.56.101 node1.paas.x
* 192.168.56.102 node2.paas.x
* 192.168.56.103 node3.paas.x


#### 1.安装

```shell
yum install flannel kubernetes-node -y
```

#### 2. 配置/etc/kubernetes/kubelet

```shell
cat << EOF > /etc/kubernetes/kubelet
KUBELET_ADDRESS="--address=0.0.0.0"
KUBELET_PORT="--port=10250"
KUBELET_HOSTNAME="--hostname-override=$HOSTNAME"
KUBELET_API_SERVER="--api-servers=http://master.paas.x:8080"
KUBELET_ARGS="--pod_infra_container_image=docker.x:5000/google_containers/pause-amd64:3.0"
EOF
```

#### 3. 配置/etc/sysconfig/flanneld

```shell
cat << EOF > /etc/sysconfig/flanneld
FLANNEL_ETCD_ENDPOINTS="http://master.paas.x:2379"
FLANNEL_ETCD_PREFIX="/paas.x/network"
FLANNEL_OPTIONS="--iface=enp0s3 -ip-masq=true"
EOF
```

#### 4. 配置/etc/kubernetes/config

```shell
cat << EOF > /etc/kubernetes/config
KUBE_LOGTOSTDERR="--logtostderr=true"
KUBE_LOG_LEVEL="--v=0"
KUBE_ALLOW_PRIV="--allow-privileged=false"
KUBE_MASTER="--master=http://master.paas.x:8080"
KUBE_SERVICE_ADDRESSES="--service-cluster-ip-range=172.17.0.0/16"
EOF
```

#### 5. 启动从节点 **

```shell
for SERVICES in kube-proxy kubelet flanneld docker; do
   systemctl restart $SERVICES
   systemctl enable $SERVICES
   systemctl status $SERVICES
done
```

> 在主节点查看集群是否创建成功
>
> [root@master ~]# kubectl get node
>
> NAME          STATUS    AGE
>
> node1.paas.x   Ready     16m
>
> node2.paas.x   Ready     7m
>
> node3.paas.x   Ready     2m
>
>以上k8s集群就搭建完毕了