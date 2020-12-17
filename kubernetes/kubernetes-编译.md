---
title: "Kubernetes 编译"
date: 2020-09-15T21:41:46+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

### kubernetes  编译

<!--more-->

#### 1. 默认mac 编译会报错

![image-20200917105728178](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-17-025728.png)

/usr/local/go/pkg/tool/linux_amd64/link: signal: killed 通过查看原因，发现是docker 默认在mac 版本上的资源限制问题，解决问题如下 

选择docker 的首选项-设置

 ![image-20200917105907919](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-17-025908.png)

修改完毕，点击应用即可

#### 2.设置docker代理

根据代理软件设置所对应的代理

![image-20201207102056881](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-07-022102.png)



#### 3. 编译

```bash
git clone https://github.com/kubernetes/kubernetes
cd  kubernetes 
git checkout  v1.19.4
./build/run.sh make   开始执行编译(是在容器中编译)
+++ [1207 10:12:19] Placing binaries
Generated bindata file : test/e2e/generated/bindata.go has 8978 test/e2e/generated/bindata.go lines of lovely automated artifacts
No changes in generated bindata file: staging/src/k8s.io/kubectl/pkg/generated/bindata.go
Go version: go version go1.15.2 linux/amd64
+++ [1207 10:12:24] Building go targets for linux/amd64:
    cmd/kube-proxy
    cmd/kube-apiserver
    cmd/kube-controller-manager
    cmd/kubelet
    cmd/kubeadm
    cmd/kube-scheduler
    vendor/k8s.io/kube-aggregator
    vendor/k8s.io/apiextensions-apiserver
    cluster/gce/gci/mounter
    cmd/kubectl
    cmd/gendocs
    cmd/genkubedocs
    cmd/genman
    cmd/genyaml
    cmd/genswaggertypedocs
    cmd/linkcheck
    vendor/github.com/onsi/ginkgo/ginkgo
    test/e2e/e2e.test
    cluster/images/conformance/go-runner
    cmd/kubemark
    vendor/github.com/onsi/ginkgo/ginkgo
    test/e2e_node/e2e_node.test
Env for linux/amd64: GOOS=linux GOARCH=amd64 GOROOT=/usr/local/go CGO_ENABLED= CC=
Coverage is disabled.
Coverage is disabled.
+++ [1207 10:38:11] Placing binaries
+++ [1207 10:38:55] Syncing out of container
#编译完成的二进制在_output 目录中
ls -l  _output/dockerized/bin/linux/amd64                                        
```

![image-20201207104937538](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-07-024941.png)

#### 4. 关于证书期限

**** 修改cert.go 文件

```bash
vim   staging/src/k8s.io/client-go/util/cert/cert.go 
.................
# 默认已经是10年，可以不修改，最高99年，不超过100
NotAfter:              now.Add(duration365d * 10).UTC(),
..........
```

**** 修改constants.go 文件

```bash
vim cmd/kubeadm/app/constants/constants.go
.............
#默认是一年，修改为10年
CertificateValidity = time.Hour * 24 * 365
.............

#修改
CertificateValidity = time.Hour * 24 * 365 * 10 
重新编译kubeadm 
make all WHAT=cmd/kubeadm GOFLAGS=-v
```



#### 5. 制作pause 镜像

```bash
cd build/pause 
make container 
staging-k8s.gcr.io/pause-amd64:3.2  #生成镜像
```

#### 6. 编译完成

```bash
1. 生成镜像
staging-k8s.gcr.io/hyperkube-amd64:v1.18.9   # 包含kube-apiserver 等组件信息的镜像
staging-k8s.gcr.io/pause-amd64:3.1           # pause 镜像
```

