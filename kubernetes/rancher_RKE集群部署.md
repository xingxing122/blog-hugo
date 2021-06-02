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

   [下载RKE]:https://github.com/rancher/rke/releases

   ```bash
   mv rke_linux-amd64 rke
   chmod +x rke
   rke --version
   ```

   使用RKE 安装需要生成一个cluster.yml 文件可以使用rke 的命令来生成

   rke config  来生成这个文件

   ```yaml
   rke config
   
   [+] Cluster Level SSH Private Key Path [~/.ssh/id_rsa]: 
   [+] Number of Hosts [1]: 3
   [+] SSH Address of host (1) [none]: 10.39.15.23
   [+] SSH Port of host (1) [22]: 
   [+] SSH Private Key Path of host (10.39.15.23) [none]: 
   [-] You have entered empty SSH key path, trying fetch from SSH key parameter
   [+] SSH Private Key of host (10.39.15.23) [none]: 
   [-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
   [+] SSH User of host (10.39.15.23) [ubuntu]: root
   [+] Is host (10.39.15.23) a Control Plane host (y/n)? [y]: y
   [+] Is host (10.39.15.23) a Worker host (y/n)? [n]: y
   [+] Is host (10.39.15.23) an etcd host (y/n)? [n]: y
   [+] Override Hostname of host (10.39.15.23) [none]: 
   [+] Internal IP of host (10.39.15.23) [none]: 
   [+] Docker socket path on host (10.39.15.23) [/var/run/docker.sock]: 
   [+] SSH Address of host (2) [none]: 10.39.15.24
   [+] SSH Port of host (2) [22]: 
   [+] SSH Private Key Path of host (10.39.15.24) [none]: 
   [-] You have entered empty SSH key path, trying fetch from SSH key parameter
   [+] SSH Private Key of host (10.39.15.24) [none]: 
   [-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
   [+] SSH User of host (10.39.15.24) [ubuntu]: root
   [+] Is host (10.39.15.24) a Control Plane host (y/n)? [y]: y
   [+] Is host (10.39.15.24) a Worker host (y/n)? [n]: y
   [+] Is host (10.39.15.24) an etcd host (y/n)? [n]: y
   [+] Override Hostname of host (10.39.15.24) [none]: 
   [+] Internal IP of host (10.39.15.24) [none]: 
   [+] Docker socket path on host (10.39.15.24) [/var/run/docker.sock]: 
   [+] SSH Address of host (3) [none]: 10.39.15.25
   [+] SSH Port of host (3) [22]: 
   [+] SSH Private Key Path of host (10.39.15.25) [none]: 
   [-] You have entered empty SSH key path, trying fetch from SSH key parameter
   [+] SSH Private Key of host (10.39.15.25) [none]: 
   [-] You have entered empty SSH key, defaulting to cluster level SSH key: ~/.ssh/id_rsa
   [+] SSH User of host (10.39.15.25) [ubuntu]: root
   [+] Is host (10.39.15.25) a Control Plane host (y/n)? [y]: y
   [+] Is host (10.39.15.25) a Worker host (y/n)? [n]: y
   [+] Is host (10.39.15.25) an etcd host (y/n)? [n]: y
   [+] Override Hostname of host (10.39.15.25) [none]: 
   [+] Internal IP of host (10.39.15.25) [none]: 
   [+] Docker socket path on host (10.39.15.25) [/var/run/docker.sock]: 
   [+] Network Plugin Type (flannel, calico, weave, canal, aci) [canal]: calico
   [+] Authentication Strategy [x509]: 
   [+] Authorization Mode (rbac, none) [rbac]: none  
   [+] Kubernetes Docker image [rancher/hyperkube:v1.20.5-rancher1]: 
   [+] Cluster domain [cluster.local]: 
   [+] Service Cluster IP Range [10.43.0.0/16]: 192.168.0.0/16
   [+] Enable PodSecurityPolicy [n]: 
   [+] Cluster Network CIDR [10.42.0.0/16]: 172.16.0.0/16
   [+] Cluster DNS Service IP [10.43.0.10]: 
   [+] Add addon manifest URLs or YAML files [no]: 
   ```

   cluster.yml 生成之后就可以执行

   ```bash
   rke up  #来部署kubernetes 集群
   export KUBECONFIG=$(pwd)/kube_config_cluster.yml  #rke up 执行完毕使用这个命令
   #将kube_config_cluster.yml 文件拷贝到$HOME/.kube/config 
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