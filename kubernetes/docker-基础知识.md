---
title: "Docker 基础知识"
date: 2020-12-16T10:42:55+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

# Docker 基础知识

<!--more-->

### 1. Docker 是什么？

```bash
Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够将应用程序与基础架构分开，从而可以快速交付软件，减少编写代码和在生产环境中运行代码之间的延迟
```

### 2. Docker 安装

```bash
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io
#yum list docker-ce --showduplicates | sort -r
#yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
systemctl start docker
```

### 3. Docker 基本命令

```bash
docker    restart/stop/start  重启，关闭，启动
docker    run  
             -d   后台运行
             -p   端口 
             -h   指定主机名
             --dns 指定dns 服务器
             --name 容器的名字
             --net=“bridge” 容器的网络选择
             --privileged=false   容器特权
             -v     挂载存储   
日志
docker  log 
        rmi  删除镜像
        rm   删除容器

```

### 4. Docker 网络

```bash
当docker 启动时，会自动在主机上创建一个docker0 虚拟网桥，实际上是Linux 的一个bridge，可以理解为软件交换机，它会在挂载到它的网口之间进行转发。

创建网络
docker network create -d bridge my-net  
网络模式
bridge    为每一个容器分配  设置ip等，并将容器连接到docker0  虚拟网桥，默认为该模式
host      容器不会虚拟出自己的网卡，配置自己的ip等，而是使用宿主机的ip和端口
node      容器有独立的network  namespace，但并没有对其进行任何网络设置，如分配veth  和网桥连接   ip 等 
container 新创建的容器不会创建自己的网卡和配置自己的ip，而是和一个指定的容器共享ip、端口范围等
```

### 5. Dockerfile







