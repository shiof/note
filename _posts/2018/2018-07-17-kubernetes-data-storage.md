---
layout: article
title:	Kubernetes集群数据持久化存储
date:	2018-07-17 20:31:29
categories:
    - article
tags:
    - docker
    - Kubernetes
---


#### 1. 各节点安装glusterd和heketi

```shell
yum install centos-release-gluster -y
yum install -y glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
systemctl start glusterd.service
systemctl enable glusterd.service

yum install heketi -y
systemctl enable heketi
systemctl restart heketi
systemctl status heketi
```

#### 2. 集群免密码登录

```shell
ssh-keygen -t rsa
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.56.101
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.56.102
ssh-copy-id -i /root/.ssh/id_rsa.pub root@192.168.56.103
```

#### 3. 在从节点创建磁盘分区

```shell
[root@node1 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0xa80b2b92.

Command (m for help): n
Partition type:
  p   primary (0 primary, 0 extended, 4 free)
  e   extended
Select (default p):
Using default response p
Partition number (1-4, default 1):
First sector (2048-16777215, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-16777215, default 16777215):
Using default value 16777215
Partition 1 of type Linux and of size 8 GiB is set

Command (m for help): p

Disk /dev/sdb: 8589 MB, 8589934592 bytes, 16777216 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0xa80b2b92

  Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    16777215     8387584   83  Linux

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
[root@node1 ~]# pvcreate /dev/sdb1
 Physical volume "/dev/sdb1" successfully created.
```

#### 4. 在主节点安装heketi-cli

```shell
yum install heketi-client -y
```

##### 4.1 将heketi用户改为root

```shell
sed -i "s/User=heketi/User=root/g" /usr/lib/systemd/system/heketi.service
systemctl daemon-reload
```

##### 4.2 配置 /etc/heketi/heketi.json

```json
{
 "_port_comment": "Heketi Server Port Number",
 "port": "9090",

 "_use_auth": "Enable JWT authorization. Please enable for deployment",
 "use_auth": false,

 "_jwt": "Private keys for access",
 "jwt": {
   "_admin": "Admin has access to all APIs",
   "admin": {
     "key": "My Secret"
   },
   "_user": "User only has access to /volumes endpoint",
   "user": {
     "key": "My Secret"
   }
 },

 "_glusterfs_comment": "GlusterFS Configuration",
 "glusterfs": {
   "_executor_comment": [
     "Execute plugin. Possible choices: mock, ssh",
     "mock: This setting is used for testing and development.",
     "      It will not send commands to any node.",
     "ssh:  This setting will notify Heketi to ssh to the nodes.",
     "      It will need the values in sshexec to be configured.",
     "kubernetes: Communicate with GlusterFS containers over",
     "            Kubernetes exec api."
   ],
   "executor": "ssh",

   "_sshexec_comment": "SSH username and private key file information",
   "sshexec": {
     "keyfile": "/root/.ssh/id_rsa",
     "user": "root",
     "port": "22",
     "fstab": "/etc/fstab"
   },

   "_kubeexec_comment": "Kubernetes configuration",
   "kubeexec": {
     "host" :"https://kubernetes.host:8443",
     "cert" : "/path/to/crt.file",
     "insecure": false,
     "user": "kubernetes username",
     "paasword": "paasword for kubernetes user",
     "namespace": "OpenShift project or Kubernetes namespace",
     "fstab": "Optional: Specify fstab file on node.  Default is /etc/fstab"
   },

   "_db_comment": "Database file name",
   "db": "/var/lib/heketi/heketi.db",

   "_loglevel_comment": [
     "Set log level. Choices are:",
     "  none, critical, error, warning, info, debug",
     "Default is warning"
   ],
   "loglevel" : "debug"
 }
}
```

> [root@master ~]# curl http://192.168.56.100:9090/hello
> Hello from Heketi
>
> PS:可能 /usr/lib/systemd/system/heketi.service 配置有错
>
> 修改之前：ExecStart=/usr/bin/heketi -config=/etc/heketi/heketi.json
>
> 修改之后：ExecStart=/usr/bin/heketi --config=/etc/heketi/heketi.json

##### 4.3 配置topology.json

```json
{
   "clusters":[
       {
           "nodes":[
               {
                   "node":{
                       "hostnames":{
                           "manage":[
                               "192.168.56.101"
                           ],
                           "storage":[
                               "192.168.56.101"
                           ]
                       },
                       "zone":1
                   },
                   "devices":[
                       "/dev/sdb1"
                   ]
               },
               {
                   "node":{
                       "hostnames":{
                           "manage":[
                               "192.168.56.102"
                           ],
                           "storage":[
                               "192.168.56.102"
                           ]
                       },
                       "zone":1
                   },
                   "devices":[
                       "/dev/sdb1"
                   ]
               },
               {
                   "node":{
                       "hostnames":{
                           "manage":[
                               "192.168.56.103"
                           ],
                           "storage":[
                               "192.168.56.103"
                           ]
                       },
                       "zone":1
                   },
                   "devices":[
                       "/dev/sdb1"
                   ]
               }
           ]
       }
   ]
}
```

```shell
echo "export HEKETI_CLI_SERVER=http://192.168.56.100:9090" >> ~/.bashrc
source ~/.bashrc

#heketi-cli --server http://192.168.33.21:9090 topology load --json=/root/k8smonitor/topology.json
heketi-cli topology load --json=topology.json
heketi-cli topology info
```

#### 5. 创建分布式卷

```shell
heketi-cli  volume create --size=1 --replica=3 --name=hadoop
heketi-cli  volume list
heketi-cli  volume info [id]
```

>[root@master ~]# heketi-cli  volume create --size=1 --replica=3 --name=hadoop
>heketi-cli  volume listName: hadoop
>Size: 1
>Volume Id: fb559a0070217ff58b05f1e6d9eca147
>Cluster Id: d81812cbe25f2011b3cff508b6b751c5
>Mount: 192.168.56.101:hadoop
>Mount Options: backup-volfile-servers=192.168.56.102,192.168.56.103
>Durability Type: replicate
>Distributed+Replica: 3
>
>[root@master ~]# heketi-cli  volume list
>Id:fb559a0070217ff58b05f1e6d9eca147    Cluster:d81812cbe25f2011b3cff508b6b751c5    Name:hadoop
>[root@master ~]# heketi-cli  volume info fb559a0070217ff58b05f1e6d9eca147
>Name: hadoop
>Size: 1
>Volume Id: fb559a0070217ff58b05f1e6d9eca147
>Cluster Id: d81812cbe25f2011b3cff508b6b751c5
>Mount: 192.168.56.101:hadoop
>Mount Options: backup-volfile-servers=192.168.56.102,192.168.56.103
>Durability Type: replicate
>Distributed+Replica: 3

#### 6. 创建k8s资源

##### 6.1 glusterfs-endpoints..yaml

```yaml
---
kind: Endpoints
apiVersion: v1
metadata:
 name: glusterend
subsets:
- addresses:
 - ip: 192.168.56.101
 ports:
 - port: 1
- addresses:
 - ip: 192.168.56.102
 ports:
 - port: 1
- addresses:
 - ip: 192.168.56.103
 ports:
 - port: 1
```

##### 6.2 gluster-service.yaml

```yaml
---
kind: Service
apiVersion: v1
metadata:
 name: glusterend
spec:
 ports:
 - port: 1
```

##### 6.3 storage-class.yaml

```yaml
apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
 name: gluster-heketi
provisioner: kubernetes.io/glusterfs
parameters:
 resturl: "http://192.168.56.100:9090"
 restauthenabled: "false"
 restuser: "admin"
 secretNamespace: "default"
 secretName: "heketi-secret"
```

##### 6.4 pv.yaml

```yaml
---
kind: PersistentVolume
apiVersion: v1
metadata:
 name: hadoop-pv
 labels:
   use: hadoop
 annotations:
   volume.beta.kubernetes.io/storage-class: gluster-heketi
spec:
 capacity:
   storage: 1Gi
 glusterfs:
   endpoints: glusterend
   path: hadoop
 accessModes:
 - ReadWriteMany
 persistentVolumeReclaimPolicy: Retain
status: {}
```

> path 要与heketi-cli  volume create --size=1 --replica=3 --name=hadoop的name对应。