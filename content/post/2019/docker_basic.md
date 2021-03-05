---
title: "【十八般兵器】Docker 学习笔记整理"
date: 2019-03-05T13:05:14+08:00
draft: true
categories: ['十八般兵器']
---

　　话说自己从17年开始接触docker为代表的容器化相关的概念，也在工作过程中也用到了一些docker相关的技术，这其中也零散的记录了一些笔记，但都比较凌乱，今天写这篇文章就想把这些散乱的笔记整理一下，形成一篇系统的学习笔记。

# 1.Docker环境的安装

　　我的测试环境为ubuntu操作系统，所以这里以ubuntu为例。访问官网的下载列表：`https://download.docker.com/linux/ubuntu/dists/${version}/pool/stable/amd64/`，其中`${version}`是根据自己的操作系统版本找到对应的版本代号。

　　下载其中的如下三个安装包： docker-ce、docker-ce-cli、containerd.io，并使用如下命令进行安装：

```
sudo dpkg -i  docker-ce_ver_os.deb
sudo dpkg -i  docker-ce-cli_ver_os.deb
sudo dpkg -i  containerd.io_ver_os.deb
```

​		安装成功之后，可以通过如下命令进行验证，是否安装成功。

```
sudo docker run hello-world
```

​		如何出现“Hello from Docker!”则证明docker环境就安装成功，并且docker引擎也已经正常启动。

# 2.Docker镜像

　　docker的所有容器都是基于镜像来创建的，这点有点类似与面向对象变成中类和对象之间的关系，镜像就是事先设计好的“类"， 真正工作的是在运行时，容器就是通过”类“所创建的”对象“。

​		一般我们常用的镜像在docker的官网上都已经存在了，我们只需要直接下载下来使用即可。

## 2.1 列出当前已有的镜像：

```
docker images

REPOSITORY         TAG          IMAGE ID       CREATED         SIZE
compose_test_web   latest       1e49cd6698a8   2 hours ago     196MB
hadoop_stack       0.3          05ac13d265cf   4 days ago      6.31GB
ubuntu             latest       f63181f19b2f   6 weeks ago     72.9MB
mysql              5.7          a70d36bc331a   6 weeks ago     449MB
redis              alpine       933c79ea2511   7 weeks ago     31.6MB
python             3.7-alpine   72e4ef8abf8e   2 months ago    41.1MB
hello-world        latest       bf756fb1ae65   14 months ago   13.3kB
```

可以看到列出了已有的镜像，并可以查看到如下信息：

- REPOSITORY： 镜像来自于哪个仓库
- TAG：镜像的标签，一般标识版本号，latest 标识最新的版本
- IMAGE ID：镜像的ID
- CREATED：镜像创建时间
- SIZE：镜像大小

## 2.2 下载镜像：

使用下面的命令可以下载指定版本的镜像：

```
docker pull image_name:tag
```

如果下载最新的镜像可以直接使用：

```
docker pull image_name
```

## 2.3 更新国内镜像地址

在下载镜像的时候，如果速度比较慢的话，建议修改一下配置，改用国内的镜像，可以大幅提升镜像的下载速度，这里是知名的几个国内镜像：

- 科大镜像：**https://docker.mirrors.ustc.edu.cn/**
- 网易：**https://hub-mirror.c.163.com/**
- 七牛云加速器：**https://reg-mirror.qiniu.com**
- 阿里云：**https://xxx.mirror.aliyuncs.com**  (需要[注册](https://cr.console.aliyun.com/)阿里云账号，xxx为阿里云系统分配的id)



修改daemon配置文件/etc/docker/daemon.json来使用加速器

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xxxx.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 2.4 搜索镜像

可以使用下面的命令来搜索镜像名称，以mysql为例：

```
docker search mysql
```

```
NAME                              DESCRIPTION                                     STARS     OFFICIAL AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10575     [OK]
mariadb                           MariaDB Server is a high performing open sou…   3956      [OK]
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   775                  [OK]
percona                           Percona Server is a fork of the MySQL relati…   527       [OK]
centos/mysql-57-centos7           MySQL 5.7 SQL database server                   87
mysql/mysql-cluster               Experimental MySQL Cluster Docker images. Cr…   79
centurylink/mysql                 Image containing mysql. Optimized to be link…   59                   [OK]
bitnami/mysql                     Bitnami MySQL Docker Image                      48                   [OK]
deitch/mysql-backup               REPLACED! Please use http://hub.docker.com/r…   41                   [OK]
databack/mysql-backup             Back up mysql databases to... anywhere!         39
prom/mysqld-exporter                                                              37                   [OK]
tutum/mysql                       Base docker image to run a MySQL database se…   35
schickling/mysql-backup-s3        Backup MySQL to S3 (supports periodic backup…   29                   [OK]
linuxserver/mysql                 A Mysql container, brought to you by LinuxSe…   27
centos/mysql-56-centos7           MySQL 5.6 SQL database server                   20
circleci/mysql                    MySQL is a widely used, open-source relation…   20
mysql/mysql-router                MySQL Router provides transparent routing be…   18
arey/mysql-client                 Run a MySQL client from a docker container      17                   [OK]
fradelg/mysql-cron-backup         MySQL/MariaDB database backup using cron tas…   12                   [OK]
yloeffler/mysql-backup            This image runs mysqldump to backup data usi…   7                    [OK]
openshift/mysql-55-centos7        DEPRECATED: A Centos7 based MySQL v5.5 image…   6
devilbox/mysql                    Retagged MySQL, MariaDB and PerconaDB offici…   3
ansibleplaybookbundle/mysql-apb   An APB which deploys RHSCL MySQL                2                    [OK]
widdpim/mysql-client              Dockerized MySQL Client (5.7) including Curl…   1                    [OK]
jelastic/mysql                    An image of the MySQL database server mainta…   1
```

## 2.5 删除镜像

使用如下命令删除镜像，其中-f 参数标识强制删除。

```
docker rmi [-f] image_name 
```

# 3.Docker容器

## 3.1 启动容器

启动容器有几种模式：

1. 例如： 以交互模式并在新的终端中启动容器后，启动bash环境，并给容器命名为my_container

   ```
   docker run -i -t --name my_container ubuntu /bin/bash
   ```

2. 以后台形式运行：

   ```
   docker run -i -t -d --name my_container ubuntu /bin/bash
   ```

## 3.2 停止容器

```
docker stop container_id/name
```

## 3.3 重启容器

```
docker start container_id/name
```

## 3.4 删除容器

```
docker rm -f container_id/name
```

## 3.5 查看容器

- 显示正在运行的容器：

  ```
  docker ps 
  ```

- 显示所有容器：

  ```
  docker ps -a
  ```

- 显示结果：

  ```
  root@ali-zjk:~# docker ps -a
  CONTAINER ID   IMAGE              COMMAND                  CREATED             STATUS                           PORTS     NAMES
  e837dbc40ac7   ubuntu             "/bin/bash"              5 seconds ago       Exited (0) 4 seconds ago                   c1
  ecb19464015d   ubuntu             "bash"                   About an hour ago   Exited (130) About an hour ago             zen_sammet
  60a09906be82   ubuntu             "bash"                   About an hour ago   Exited (0) About an hour ago               romantic_cori
  6107a5142640   ubuntu             "bash"                   About an hour ago   Exited (130) About an hour ago             strange_panini
  ```

  

# 4.Docker网络相关

# 5.其他知识点