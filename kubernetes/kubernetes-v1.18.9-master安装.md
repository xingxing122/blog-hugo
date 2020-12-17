---
title: "Kubernetes-v1.18.9 master安装"
date: 2020-09-24T14:44:33+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

Kubernetes  v.1.18.9 master 的安装   

<!--more-->

kubectl 与kube-apiserver 的安全端口通信，需要为安全通信提供TLS证书和秘钥  

1. 制作证书

   ```json
   # admin-csr.json 文件
   vim  admin-csr.json
   {
       "CN": "admin",
       "hosts": [],
       "key": {
         "algo": "rsa",
         "size": 2048
       },
       "names": [
         {
           "C": "CN",
           "ST": "Beijing",
           "L": "Beijing",
           "O": "system:masters",
           "OU": "System"
         }
       ]
       }
   
   # 生成证书和私钥 
   ../cfssl gencert -ca ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes admin-csr.json  | ../cfssljson  -bare admin
   # 生成admin-csr.json  admin-key.pem   admin.csr       admin.pem 证书文件
   
   # kubernetes 证书 
   vim  kubernetes-csr.json
   {
    "CN": "kubernetes",
    "hosts": [
      "127.0.0.1",
      "10.39.68.22",
      "10.39.68.23",
      "10.39.68.24",
      "10.39.68.249",
      "10.254.0.1",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "Beijing",
        "L": "Beijing",
        "O": "k8s",
        "OU": "System"
      }
    ]
   }
   
   # 生成证书，这里的ip 是三个master 的ip，有一个是lb的ip 是apiserver 高可用时连接，也可以写域名
   ../cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes kubernetes-csr.json  | ../cfssljson  -bare kubernetes
   # 生成证书文件  kubernetes-csr.json   kubernetes-key.pem kubernetes.csr    kubernetes.pem
   
   # 将证书分发到master 服务器 
   ansible -i hosts etcd -m copy  -a "src=../ssl/   dest=/etc/kubernetes/ssl"
   ```

2. 拷贝二进制文件(kubectl  kubelet  kubeadm)

3. 给二进制文件添加执行权限

   ```bash
   ansible -i hosts  etcd   -m shell -a " chmod +x /usr/local/bin/*"
   ```

4. 创建核心组件的静态pod 存储目录

   ```bash
   ansible -i hosts etcd  -m shell  -a "mkdir /etc/kubernetes/manifests"
   ```

5. 创建kube-apiserver.json 文件 

   vim kube-apiserver.json 

   ```json
   {
             "kind": "Pod",
             "apiVersion": "v1",
             "metadata": {
               "name": "kube-apiserver",
               "namespace": "kube-system",
               "creationTimestamp": null,
               "labels": {
                 "component": "kube-apiserver",
                 "tier": "control-plane"
               }
             },
             "spec": {
               "volumes": [
                 {
                   "name": "certs",
                   "hostPath": {
                     "path": "/etc/ssl/certs"
                   }
                 },
                 {
                   "name": "pki",
                   "hostPath": {
                     "path": "/etc/kubernetes"
                   }
                 }
               ],
               "containers": [
                 {
                   "name": "kube-apiserver",
                   "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0",
                   "imagePullPolicy": "Always",
                   "command": [
                     "/apiserver",
                     "--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,PersistentVolumeLabel,DefaultStorageClass,ResourceQuota",
                     "--anonymous-auth=false",
                     "--experimental-encryption-provider-config=/etc/kubernetes/encryption-config.yaml",
                     "--advertise-address=10.39.30.116",
                     "--allow-privileged=true",
                     "--apiserver-count=3",
                     "--audit-policy-file=/etc/kubernetes/audit-policy.yaml",
                     "--audit-log-maxage=30",
                     "--audit-log-maxbackup=3",
                     "--audit-log-maxsize=100",
                     "--audit-log-path=/var/log/kubernetes/audit.log",
                     "--authorization-mode=Node,RBAC",
                     "--bind-address=10.39.30.116",
                     "--secure-port=6443",
                     "--client-ca-file=/etc/kubernetes/ssl/ca.pem",
                     "--kubelet-client-certificate=/etc/kubernetes/ssl/kubernetes.pem",
                     "--kubelet-client-key=/etc/kubernetes/ssl/kubernetes-key.pem",
                     "--enable-swagger-ui=true",
                     "--etcd-cafile=/etc/kubernetes/ssl/ca.pem",
                     "--etcd-certfile=/etc/kubernetes/ssl/etcd.pem",
                     "--etcd-keyfile=/etc/kubernetes/ssl/etcd-key.pem",
                     "--etcd-servers=https://10.39.30.116:2379,https://10.39.30.117:2379,https://10.39.30.118:2379",
                     "--event-ttl=1h",
                     "--kubelet-https=true",
                     "--insecure-bind-address=127.0.0.1",
                     "--insecure-port=8080",
                     "--service-account-key-file=/etc/kubernetes/ssl/ca-key.pem",
                     "--service-cluster-ip-range=10.254.0.0/18",
                     "--service-node-port-range=30000-32000",
                     "--tls-cert-file=/etc/kubernetes/ssl/kubernetes.pem",
                     "--tls-private-key-file=/etc/kubernetes/ssl/kubernetes-key.pem",
                     "--enable-bootstrap-token-auth",
                     "--v=2"
                   ],
                   "resources": {
                     "requests": {
                       "cpu": "250m"
                     }
                   },
                   "volumeMounts": [
                     {
                       "name": "certs",
                       "mountPath": "/etc/ssl/certs"
                     },
                     {
                       "name": "pki",
                       "readOnly": true,
                       "mountPath": "/etc/kubernetes/"
                     }
                   ],
                   "livenessProbe": {
                     "httpGet": {
                       "path": "/healthz",
                       "port": 8080,
                       "host": "127.0.0.1"
                     },
                     "initialDelaySeconds": 15,
                     "timeoutSeconds": 15,
                     "failureThreshold": 8
                   }
                 }
               ],
               "hostNetwork": true
             }
           }
   
   
   ```

   

6. 

7. 

8. 

9. 

10. 

11. 

12. 

13. 

14. 

15. 

16. 

17. 

18. 

19. 

20. 

21. 

22. 

23. 

24. 

25. 

26. 

27. 

28. 

29. 