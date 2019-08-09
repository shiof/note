---
layout: article
title:	Hadoop常用命令
date:	2018-06-10 17:20:31
categories:
    - article
tags:
    - Hadoop
    - BigData
---

在上一章节已经配置过hadoop的环境变量，现在可以直接使用**hadoop**和**hdfs**命令

##### 初始化namenode

~~~shell
hadoop namenode -formate
~~~

##### 在hadoop集群运行jar

~~~shell
hadoop jar helloworld.jar
~~~

##### 对文件的操作命令  `hadoop fs`和 `hdfs dfs`

`hadoop fs` 它可以指向任何文件系统，如本地，HDFS等

`hdfs dfs` 可用于与HDFS相关的所有操作

跟linux上对文件操作基本一致，只不过前面加上`hadoop fs`或`hdfs dfs`表示对HDFS里的文件执行操作

##### 在HDFS下，新建/test目录

~~~shell
hdfs dfs -mkdir /test
~~~

##### 查看是否新建了/test

~~~shell
[root@hadoop_master logs]# hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2018-08-04 04:06 /test
~~~

##### 重命名(移动)/test为/test1

~~~shell
[root@hadoop_master logs]# hdfs dfs -mv /test /test1
[root@hadoop_master logs]# hdfs dfs -ls /
Found 1 items
drwxr-xr-x   - root supergroup          0 2018-08-04 04:14 /test1
~~~

##### 删除/test1目录

~~~shell
[root@hadoop_master logs]#  hdfs dfs -rm -r /test1
18/08/04 04:11:31 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /test1
~~~
或者
~~~shell
[root@hadoop_master logs]#  hdfs dfs -rmdir  /test1
18/08/04 04:11:31 INFO fs.TrashPolicyDefault: Namenode trash configuration: Deletion interval = 0 minutes, Emptier interval = 0 minutes.
Deleted /test1
~~~

##### 从主机复制文件夹到HDFS
将主机的`localdir`目录复制到HDFS根目录下

~~~shell
[root@hadoop_master ~]# hadoop fs -copyFromLocal ~/localdir /
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root supergroup          0 2018-08-04 04:22 /localdir
drwxr-xr-x   - root supergroup          0 2018-08-04 04:14 /test1
~~~
或者
~~~shell
hadoop fs -put ~/localdir /
hdfs dfs -put ~/localdir /
~~~

> hdfs dfs 没有`-copyFromLocal`参数

##### 从HDFS复制文件夹到主机
在HDFS上，新建`hdfsdir`目录并复制到主机`~`目录下

~~~shell
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root supergroup          0 2018-08-04 04:31 /hdfsdir
drwxr-xr-x   - root supergroup          0 2018-08-04 04:22 /localdir
[root@hadoop_master ~]# hdfs dfs -copyToLocal /hdfsdir ~
[root@hadoop_master ~]# ll
total 3
drwxr-xr-x  2 root root    6 Aug  4 04:32 hdfsdir
drwxr-xr-x  2 root root    6 Aug  4 04:20 localdir
~~~

> hdfs dfs 没有`-copyToLocal`参数

##### 在HDFS查看文件内容
在主机新建`helloworld.txt`文件，写入"hello world"，通过上面学习的命令写入到HDFS的`localdir`目录下，并在HDFS文件内容

~~~shell
[root@hadoop_master ~]# hdfs dfs -ls /localdir
Found 1 items
-rw-r--r--   2 root supergroup         12 2018-08-04 04:52 /localdir/helloworld.txt
[root@hadoop_master ~]# hdfs dfs -cat /localdir/helloworld.txt
hello world
~~~

##### 将`helloworld.txt`文件复制到/hdfsdir目录下

~~~shell
[root@hadoop_master ~]#  hdfs dfs -cp /localdir/helloworld.txt /hdfsdir/helloworld.txt 
[root@hadoop_master ~]#  hdfs dfs -ls /hdfsdir
Found 1 items
-rw-r--r--   2 root spwang         12 2018-08-04 05:07 /hdfsdir/helloworld.txt
~~~

##### 修改文件价所属的组

~~~shell
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root supergroup          0 2018-08-04 04:31 /hdfsdir
drwxr-xr-x   - root supergroup          0 2018-08-04 04:52 /localdir
[root@hadoop_master ~]# hdfs dfs -chgrp -R spwang /hdfsdir
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - root spwang              0 2018-08-04 04:31 /hdfsdir
drwxr-xr-x   - root supergroup          0 2018-08-04 04:52 /localdir
~~~

##### 修改文件夹拥有者

~~~shell
[root@hadoop_master ~]# hdfs dfs -chown -R spwang /hdfsdir
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxr-xr-x   - spwang spwang              0 2018-08-04 04:31 /hdfsdir
drwxr-xr-x   - root   supergroup          0 2018-08-04 04:52 /localdir
~~~

##### 修改文件夹权限

~~~shell
[root@hadoop_master ~]# hdfs dfs -chmod -R 777 /hdfsdir
[root@hadoop_master ~]# hdfs dfs -ls /
Found 2 items
drwxrwxrwx   - spwang spwang              0 2018-08-04 04:31 /hdfsdir
drwxr-xr-x   - root   supergroup          0 2018-08-04 04:52 /localdir
~~~

##### `hadoop fs`帮助命令

~~~shell
[root@hadoop_master ~]#  hadoop fs
Usage: hadoop fs [generic options]
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] [-h] <path> ...]
        [-cp [-f] [-p | -p[topax]] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] <path> ...]
        [-expunge]
        [-find <path> ... <expression> ...]
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getfattr [-R] {-n name | -d} [-e en] <path>]
        [-getmerge [-nl] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-d] [-h] [-R] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] [-l] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setfattr {-n name [-v value] | -x name} <path>]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-truncate [-w] <length> <path> ...]
        [-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
~~~

##### `hdfs dfs`帮助命令

~~~shell
[root@hadoop_master ~]# hdfs dfs
Usage: hadoop fs [generic options]
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] [-h] <path> ...]
        [-cp [-f] [-p | -p[topax]] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] <path> ...]
        [-expunge]
        [-find <path> ... <expression> ...]
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getfattr [-R] {-n name | -d} [-e en] <path>]
        [-getmerge [-nl] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-d] [-h] [-R] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] [-l] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setfattr {-n name [-v value] | -x name} <path>]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-truncate [-w] <length> <path> ...]
        [-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
~~~













