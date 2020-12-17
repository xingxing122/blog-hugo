---
title: "Kubernetes V1.14组件安装(五)"
date: 2019-04-09T14:20:09+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

k8s 网络组件calico、dns 等安装

<!--more-->

1. 安装calico  

   [calico 官网参考资料](https://docs.projectcalico.org/v3.5/getting-started/kubernetes/)

   下载calico 的yaml 配置文件

   ```bash
   wget   https://docs.projectcalico.org/v3.5/getting-started/kubernetes/installation/hosted/calico.yaml 
   ```

   

2. 配置calico

   vim   calico.yaml 

   ~~~yaml
            etcd_endpoints:"https://10.39.30.116:2379,https://10.39.30.117:2379,https://10.39.30.118:2379" 
      etcd_ca:  "/calico-secrets/etcd-ca" 
      etcd_cert:  "/calico-secrets/etcd-cert"  
      etcd_key:  "/calico-secrets/etcd-key"
      ```
      # 这里面要写入 base64 的信息
      
      data:
       etcd-key: (cat /etc/kubernetes/ssl/etcd-key.pem | base64 | tr -d '\n')
       etcd-cert: (cat /etc/kubernetes/ssl/etcd.pem | base64 | tr -d '\n')
       etcd-ca: (cat /etc/kubernetes/ssl/ca.pem | base64 | tr -d '\n')
   ~~~

3. 修改kubelet

   vim /etc/systemd/system/kubelet.service

   ```yaml
   ..........
   --network-plugin=cni \
   ........
   ```

4. 重启kubelet  

   ```bash
   systemctl daemon-reload
   systemctl   restart kubelet 
   ```

   

5. 创建calico 

   ```bash
   [root@k8s-116 ~]# kubectl apply -f calico.yaml
   configmap/calico-config created
   secret/calico-etcd-secrets created
   daemonset.extensions/calico-node created
   serviceaccount/calico-node created
   deployment.extensions/calico-kube-controllers created
   serviceaccount/calico-kube-controllers created
   clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
   clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
   clusterrole.rbac.authorization.k8s.io/calico-node created
   clusterrolebinding.rbac.authorization.k8s.io/calico-node created
   ```

6. 部署k8s-dns 

   ```bash
   wget  <https://raw.githubusercontent.com/kubernetes/kubernetes/v1.14.0/cluster/addons/dns/kube-dns/kube-dns.yaml.base> 
   [root@k8s-116 ~]# mv kube-dns.yaml.base kube-dns.yam
   ```

7. 

8. 配置kube-dns 

   修改之前

   [root@k8s-116 ~]# vim kube-dns.yaml

   ```yaml
   clusterIP: __PILLAR__DNS__SERVER__
   - --domain=__PILLAR__DNS__DOMAIN__.
   - --server=/__PILLAR__DNS__DOMAIN__/127.0.0.1#10053    
   - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.__PILLAR__DNS__DOMAIN__,5,SRV
   - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.__PILLAR__DNS__DOMAIN__,5,SRV
   ```

   

   修改之后

   [root@k8s-116 ~]# vim kube-dns.yaml    

   ```bash
    clusterIP: 10.254.0.2
   - --domain=cluster.local
   - --server=/cluster.local./127.0.0.1#10053
   - --probe=kubedns,127.0.0.1:10053,kubernetes.default.svc.cluster.local,5,SRV
   - --probe=dnsmasq,127.0.0.1:53,kubernetes.default.svc.cluster.local,5,SRV
   ```

   ```bash
   kubectl apply -f kube-dns.yaml 
   ```

   查看

   ```bash
   [root@k8s-116 ~]# kubectl get pod -n kube-system
   NAME                                       READY   STATUS    RESTARTS   AGE
   calico-kube-controllers-5bc85548d5-sqtk2   1/1     Running   1          126m
   calico-node-24wqt                          1/1     Running   2          134m
   calico-node-bzr4j                          1/1     Running   0          134m
   calico-node-pl5j7                          1/1     Running   1          134m
   calico-node-t2mmp                          1/1     Running   0          134m
   kube-apiserver-k8s-116                     1/1     Running   13         94m
   kube-apiserver-k8s-117                     1/1     Running   3          94m
   kube-apiserver-k8s-118                     1/1     Running   24         94m
   kube-controller-manager-k8s-116            1/1     Running   1          94m
   kube-controller-manager-k8s-117            1/1     Running   3          94m
   kube-controller-manager-k8s-118            1/1     Running   1          94m
   kube-dns-bc77b69d6-46bss                   3/3     Running   0          45s
   kube-proxy-qc86s                           1/1     Running   1          87m
   kube-scheduler-k8s-116                     1/1     Running   3          94m
   kube-scheduler-k8s-117                     1/1     Running   3          94m
   kube-scheduler-k8s-118                     1/1     Running   1          94m
   ```

   

   备注: kube-dns 需要拉取以下三个镜像，需要翻墙拉取

   ```bash
   image: k8s.gcr.io/k8s-dns-kube-dns:1.14.13
   image: k8s.gcr.io/k8s-dns-dnsmasq-nanny:1.14.13
   image: k8s.gcr.io/k8s-dns-sidecar:1.14.13
   ```

9. 使用coreDNS 

   [coreDNS配置文件下载地址](https://raw.githubusercontent.com/kubernetes/kubernetes/v1.14.0/cluster/addons/dns/coredns/coredns.yaml.base)

   

   

   

    