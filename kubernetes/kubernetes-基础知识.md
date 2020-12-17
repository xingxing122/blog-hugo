---

title: "Kubernetes 基础知识"
date: 2019-06-13T23:24:16+08:00
draft: false  
categories: [Kubernetes]
tags: [Kubernetes]

---

Kubernetes 是什么？ 

<!--more-->

Kubernetes 是谷歌严格保密十几年的秘密武器-Borg的一个开源版本，Brog 是谷歌一个久负盛名的内部使用的大规模集群管理系统，他基于容器技术，目的是实现资源管理的自动化，以及跨多个数据中心的资源利用率的最大化，直到2015年才被谷歌首次公开。

kubernetes 是一个可移植，可扩展的开源平台，用于管理容器化工作负载和服务，有助于声明性配置和自动化。



Kubernetes 能解决什么？

使用Kubernetes 解决方案可以不必头疼于服务监控、部署实施、故障处理，使开发人员更加专注于业务本身，而且降低了运维难度和运维成本。 

Kubernetes 是一个分布式系统支撑平台。Kubernetes 具有完备的集群管理能力，包括多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建的智能负载均衡器、强大的故障发现和自我修复能力、服务滚动升级和在线扩容能力、可扩展的资源自动调度机制，以及多粒度的资源配额管理能力。



Kubernetes 基本概念：

master 指的是集群控制节点，master 是负责整个集群的管理和控制

master 上运行的进程有:

```bash
Kubernetes  API   server(kube-apiserver) : 提供了HTTP  Rest 接口的关键服务进程，是kubernetes 里所有资源的增、删、改、查等操作的唯一入口，也是集群控制的入口进程。
kubernetes Controller Manager(kube-controller-manager):  Kubernetes 里所有资源对象的控制中心，处理集群中常规任务的后台线程，主要有以下功能：
    ​ 节点控制器：当节点移除时，负责注意和响应
    ​ 副本控制器:  负责维护系统中每个副本控制器对象正确数量的pod 
    ​ 端点控制器:  填充端点(Endpoints)对象(即连接Service&Pods)
    ​ 服务账户和令牌控制器: 为新的命名空间创建默认账户和API访问令牌
kubernetes Scheduler(kube-scheduler): 负责资源调度(Pod调度)的进程 
Etcd: 数据存储
```

Node 上运行的进程有:

```bash
kubelet: 负责pod 对应容器的创建，启停等任务，同时与master 密切协作，实现集群管理的基本功能
kube-proxy: 实现kubernetes Service 的通信与负载机制的重要组件
Docker: 负责本机的容器创建和管理工作
```

Pod 

```bash
pod 是kubernetes 最重要的基本概念，每个pod 都有一个特殊的被称为"根容器"的pause容器。Pause 容器对应的镜像属于kubernetes 平台的一部分，出了pause容器，每个pod 还包含一个或多个紧密相关的用户业务容器
```

![image-20190702231943987](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-02-151948.png)



pause 容器作用

```bash
在pod 中担任linux 命名空间共享的基础；
启用pid 命名空间，开启init 进程；
```

Pod 有两种类型：

```bash
普通的pod 和静态pod，
静态pod 并不存储在kubernetes 的etcd 存储，而是存放在某个具体Node上的一个具体文件中，并且只在此Node上启动，运行，用kubelet--pod-mainfest-path=<the directory>

普通的pod 会被存放在etcd 存储中，随后会被kubernetes master 调度到某个node 节点上启动运行
```

label 标签

```bash
label标签是kubernetes 系统中另外一个核心概念，一个label 是一个kye=value 的键值对，其中key 与value 由用户自己指定，label 可以被附加到各种资源对象上，例如Node pod Service RC 等，一个资源对象可以定义任意数量的label，同一个label 也可以被添加到任意数量的资源对象上,label 通常在资源对象定义时确定。
```

例子

label 被定义在metadata中:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myweb
  labels:
    app: myweb
```

管理RC和Service 则通过Selector 字段设置需要关联Pod的Label

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 1
  selector:
    app: myweb
  template:
  .......

apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  selector:
    app: myweb
  ports:
  - port: 8080
```

Replication Controller (RC)

---
```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: rc-demo
  labels:
    app: rc
spec:
  replicas: 3
  selector:
      app: rc
  template:
    metadata:
       labels:
         app: rc
    spec:
      containers:

- name: nginx-demo
  image: nginx
  ports:
  - containerPort: 80
```

Replication Controller 已经升级为Replica setReplication Set与RC 当前的唯一区别是，Replica Sets支持基于集合的Label  selector(Set-based selector)

RC 的特性：

```bash
RC 实现Pod的创建及副本数量
RC 通过Label Selector 机制实现对Pod副本的自动控制
```

Deployment 

Deployment 相对于RC的最大升级是我们可以随时知道当前pod部署进度

---
```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    app:  nginx-demo
spec:
  replicas: 3
  minReadySeconds: 5 
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
      app: rc
  template:
    metadata:
       labels:
         app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```



```bash
以下参数为更新应用时使用

  minReadySeconds: 5 
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1

参数为滚动更新应用时使用
```





