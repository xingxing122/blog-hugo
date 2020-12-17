---
title: "Rancher_RKE集群部署"
date: 2020-07-22T09:52:34+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

Rancher  RKE集群的安装

<!--more-->

1. 安装docker 

   ```bash
   curl https://releases.rancher.com/install-docker/18.09.sh | sh
   ```

2. 安装kubernetes 命令行工具kubectl  

   [参考官网]: https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl

   ```bash
   curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/linux/amd64/kubectl
   chmod +x ./kubectl
   mv ./kubectl /usr/local/bin/kubectl
   kubectl version --client
   ```

3. 安装RKE 

4. 

   [下载RKE]:https://github.com/rancher/rke/releases

   ```bash
   mv rke_linux-amd64 rke
   chmod +x rke
   rke --version
   ```

   使用RKE 安装需要生成一个cluster.yml 文件可以使用rke 的命令来生成

   rke config  来生成这个文件

   ```bash
   rke config
   ```

   cluster.yml 生成之后就可以执行

   ```bash
   rke up  #来部署kubernetes 集群
   export KUBECONFIG=$(pwd)/kube_config_cluster.yml  #rke up 执行完毕使用这个命令
   [root@i-3lhxok9k ~]# kubectl get nodes                               
   NAME         STATUS   ROLES                      AGE   VERSION
   10.39.12.2   Ready    controlplane,etcd,worker   17h   v1.18.6
   10.39.12.3   Ready    controlplane,etcd,worker   17h   v1.18.6
   10.39.12.7   Ready    controlplane,etcd,worker   17h   v1.18.6
   ```

5. 安装rancher 

   rancher 的安装需要helm 

   ```bash
   #添加仓库
   helm repo add rancher-stable https://releases.rancher.com/server-charts/stable
   ```

   为rancher 创建Namespace 

   ```go
   kubectl create namespace cattle-system
   ```

   选择已有证书安装rancher 

   >  ```go
   >  1. >  helm install rancher rancher-stable/rancher \
   >     >  >  --namespace cattle-system \
   >     >  >  --set hostname=rancher-test.enncloud.cn \
   >     >  >  --set ingress.tls.source=secret
   >  ```
   >
   >  
   >

   添加已有证书

   ```go
   kubectl -n cattle-system create secret tls tls-rancher-ingress   --cert=/opt/enneloud.cer  --key=/opt/privateKey.key
   ```

6. 第一次登录的时候需要设置一下密码 

   ![image-20200731100134135](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-07-31-020134.png)

7. 部署过程会遇到一些错误，可以清楚之后，重新安装

   ```bash
   sudo df -h|grep kubelet |awk -F % '{print $2}' |xargs umount
   sudo rm /var/lib/kubelet/* -rf
   sudo rm /etc/kubernetes/* -rf
   sudo rm /etc/cni/* -rf
   sudo rm /var/lib/rancher/* -rf
   sudo rm /var/lib/etcd/* -rf
   sudo rm /var/lib/cni/* -rf
   sudo rm /opt/cni/* -rf
   sudo ip link del flannel.1
   sudo ip link del cni0
   sudo iptables -F && iptables -t nat -F
   sudo docker ps -a|awk '{print $1}' |xargs docker rm -f
   sudo docker volume ls|awk '{print $2}' |xargs docker volume rm
   sudo systemctl restart docker
   ```