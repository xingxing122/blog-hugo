---
title: "Kubernetes V1.14-编译k8s(一)"
date: 2019-03-26T08:59:33+08:00
draft: false  
categories: ["kubernetes"]
tags: ["kubernetes"]
---

 编译kubernetes 所需要的二进制文件和镜像

<!--more-->
1. 安装docker

   ```bash
    yum install -y yum-utils device-mapper-persistent-data lvm2
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum list docker-ce --showduplicates | sort -r
    yum install docker-ce-<VERSION_STRING>   # 安装docker 最新版本
   ```
   
2. 编译k8s的二进制 

   ```bash
    git clone https://github.com/kubernetes/kubernetes 
    git checkout  v1.14.0  
   ```

   ​     
    编译二进制  
   ​     
   ​    

```bash
cd  kubernetes  
./build/run.sh  make   
# 编译的时候，会拉取一个镜像k8s.gcr.io/kube-cross:v1.12.1-2 需要翻墙拉取  
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
Coverage is disabled.
+++ [0326 13:51:21] Placing binaries
+++ [0326 13:52:32] Syncing out of container
+++ [0326 13:52:32] Stopping any currently running rsyncd container
+++ [0326 13:52:32] Starting rsyncd container
+++ [0326 13:52:34] Running rsync
+++ [0326 13:53:24] Stopping any currently running rsyncd container
# 编译完成会生成一个_output目录 
# 生成的二进制包
```

​       ![](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-03-26-055618.png) 
​       

   制作镜像

```bash
cd cluster/images/hyperkube
make build VERSION=v1.11.0 ARCH=amd64  
# 编译所需要拉取镜像 k8s.gcr.io/debian-hyperkube-base-amd64:0.12.1  
    
拉取到镜像之后需要修改
docker build  --pull  -t ${REGISTRY}/hyperkube-${ARCH}:${VERSION} ${TEMP_DIR}
修改为
docker build  -t ${REGISTRY}/hyperkube-${ARCH}:${VERSION} ${TEMP_DIR}
生成的镜像 staging-k8s.gcr.io/hyperkube-amd64:v1.11.0
```

   制作pause 镜像 

```bash
cd  build/pause 
make container 
```

   修改   
   vim Makefile 
   只留下amd64 的，其他平台的不需要
   ALL_ARCH = amd64 
        

```bash
最后生成镜像
staging-k8s.gcr.io/pause-amd64:3.1
```

  总结: 
```bash
生成镜像
kubelet kube-apiserver 等启动配置的镜像
staging-k8s.gcr.io/hyperkube-amd64:v1.11.0
   
pause 镜像
staging-k8s.gcr.io/pause-amd64:3.1      
```


​        


