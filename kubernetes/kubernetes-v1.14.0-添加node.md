---
title: "Kubernetes V1.14添加node(四)"
date: 2019-04-07T18:06:07+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

kubernetes 添加node 节点  

<!--more-->

1. ##### 制作kube-proxy 证书

​      vim  kube-proxy-csr.json

```json
{
      "CN": "system:kube-proxy",
      "hosts": [],
      "key": {
        "algo": "rsa",
        "size": 2048
      },
      "names": [
        {
          "C": "CN",
          "ST": "ShenZhen",
          "L": "ShenZhen",
          "O": "k8s",
          "OU": "System"
        }
      ]
 }
```

2. 生成kube-proxy 证书

```bash
[root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem -config=config.json  -profile=kubernetes kube-proxy-csr.json  | cfssljson -bare kube-proxy
```



3.创建kube-proxy kubeconfig  文件

​       以下操作在k8s-116 mstart 上操作

创建node 的token 

```bash
[root@k8s-116 ~]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:k8s-119 --kubeconfig ~/.kube/config
znfaju.l0vvizj7zfxnwfin
```

配置集群

```bash
[root@k8s-116 ~]# kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem  --embed-certs=true  --server=https://10.39.30.116:6443 --kubeconfig=kube-proxy.kubeconfig
```

配置客户端认证

```bash
[root@k8s-116 ~]# kubectl config set-credentials kube-proxy   --client-certificate=/etc/kubernetes/ssl/kube-proxy.pem   --client-key=/etc/kubernetes/ssl/kube-proxy-key.pem   --embed-certs=true   --kubeconfig=kube-proxy.kubeconfig
```

配置关联

```bash
[root@k8s-116 ~]# kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
```

配置默认关联

```bash
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig 
scp kube-proxy.kubeconfig  root@10.39.30.119:/etc/kubernetes/
```

##### 拷贝证书到/etc/kubernetes/pki/ 目录

```bash
[root@devops ssl]# scp kube-proxy* root@10.39.30.119:/etc/kubernetes/pki/
```



4. ##### 生成k8s-119 的bootstrap.kubeconfig

 配置集群参数

```bash
kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem  --embed-certs=true  --server=https://10.39.10.116:6443 --kubeconfig=kubernetes-182-bootstrap.kubeconfi
```

配置客户端认证

```bash
kubectl config set-credentials  kubelet-bootstrap  --token=znfaju.l0vvizj7zfxnwfin   --kubeconfig=k8s-119-bootstrap.kubeconfig
```

配置关联

```bash
kubectl config set-context default  --cluster=kubernetes  --user=kubelet-bootstrap  --kubeconfig=k8s-119-bootstrap.kubeconfig
```

配置默认关联

```bash
kubectl config use-context default  --kubeconfig=k8s-119-bootstrap.kubeconfig
```



拷贝文件到node 节点

 ```bash
   scp  k8s-119-bootstrap.kubeconfig   root@10.39.30.119:/etc/kubernetes/bootstrap.kubeconfig
 ```

##### 5.配置kubelet 

kubelet.config.json文件

 ```json
   {
     "kind": "KubeletConfiguration",
     "apiVersion": "kubelet.config.k8s.io/v1beta1",
     "authentication": {
       "x509": {
         "clientCAFile": "/etc/kubernetes/ssl/ca.pem"
       },
       "webhook": {
         "enabled": true,
         "cacheTTL": "2m0s"
       },
       "anonymous": {
         "enabled": false
       }
     },
     "authorization": {
       "mode": "Webhook",
       "webhook": {
         "cacheAuthorizedTTL": "5m0s",
         "cacheUnauthorizedTTL": "30s"
       }
     },
     "address": "10.39.30.119",
     "port": 10250,
     "readOnlyPort": 10255,
     "cgroupDriver": "systemd",
     "hairpinMode": "promiscuous-bridge",
     "serializeImagePulls": false,
     "RotateCertificates": true,
     "featureGates": {
       "RotateKubeletClientCertificate": true,
       "RotateKubeletServerCertificate": true
     },
     "MaxPods": "512",
     "failSwapOn": false,
     "containerLogMaxSize": "10Mi",
     "containerLogMaxFiles": 5,
     "clusterDomain": "cluster.local",
     "clusterDNS": ["10.254.0.2"]
   }
 ```



创建kubelet.service

vim /etc/systemd/system/kubelet.service

```bash
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/local/bin/kubelet \
      --hostname-override=k8s-119 \
      --pod-infra-container-image=harbor.enncloud.cn/enncloud/pause-amd64:3.1 \
      --bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
      --kubeconfig=/etc/kubernetes/kubelet.config \
      --config=/etc/kubernetes/kubelet.config.json \
      --cert-dir=/etc/kubernetes/pki \
      --allow-privileged=true \
      --kube-reserved cpu=500m,memory=512m \
      --image-gc-high-threshold=85  --image-gc-low-threshold=70 \
      --logtostderr=true \
      --enable-controller-attach-detach=true  \
      --v=2
[Install]
 WantedBy=multi-user.target
```



启动kubelet 

```bash
systemctl   restart kubelet  
```

查看集群

```bash
kubectl get node
NAME      STATUS   ROLES    AGE     VERSION
k8s-116   Ready    master   4d5h    v1.14.0
k8s-117   Ready    master   3d23h   v1.14.0
k8s-118   Ready    master   3d21h   v1.14.0
k8s-119   Ready    node     14h     v1.14.0
```



##### 6.部署kube-proxy

在master 上创建kube-proxy 的fs 

[<https://github.com/kubernetes/kubernetes/tree/v1.14.0/cluster/addons/kube-proxy>]()

vim  kube-proxy.yaml 

```yaml
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  labels:
    component: kube-proxy
    k8s-app: kube-proxy
    kubernetes.io/cluster-service: "true"
    name: kube-proxy
    tier: node
  name: kube-proxy
  namespace: kube-system
spec:
  selector:
    matchLabels:
      component: kube-proxy
      k8s-app: kube-proxy
      kubernetes.io/cluster-service: "true"
      name: kube-proxy
      tier: node
  template:
    metadata:
      annotations:
        scheduler.alpha.kubernetes.io/affinity: '{"nodeAffinity":{"requiredDuringSchedulingIgnoredDuringExecution":{"nodeSelectorTerms":[{"matchExpressions":[{"key":"beta.kubernetes.io/arch","operator":"In","values":["amd64"]}]}]}}}'
        scheduler.alpha.kubernetes.io/tolerations: '[{"key":"dedicated","value":"master","effect":"NoSchedule"}]'
      labels:
        component: kube-proxy
        k8s-app: kube-proxy
        kubernetes.io/cluster-service: "true"
        name: kube-proxy
        tier: node
    spec:
      containers:
      - command:
        - /proxy
        - --cluster-cidr=10.254.64.0/18
        - --kubeconfig=/run/kubeconfig
        - --logtostderr=true
        - --proxy-mode=iptables
        - --v=2
        image: harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0
        imagePullPolicy: IfNotPresent
        name: kube-proxy
        securityContext:
          privileged: true
        volumeMounts:
        - mountPath: /var/run/dbus
          name: dbus
        - mountPath: /run/kubeconfig
          name: kubeconfig
        - mountPath: /etc/kubernetes/pki
          name: pki
      dnsPolicy: ClusterFirst
      hostNetwork: true
      nodeSelector:
        kube-proxy: proxy
      restartPolicy: Always
      volumes:
      - hostPath:
          path: /etc/kubernetes/kube-proxy.kubeconfig
        name: kubeconfig
      - hostPath:
          path: /var/run/dbus
        name: dbus
      - hostPath:
          path: /etc/kubernetes/pki
        name: pki
  updateStrategy:
    type: OnDelete
```

```bash
kubectl apply -f  kube-proxy.yaml 
```

打标签，需要在节点上跑kube-proxy 的需要打上标签

```bash
kubectl label node k8s-119   kube-proxy=proxy 
```

```bash
kubectl get pod -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
kube-apiserver-k8s-116            1/1     Running   12         4d5h
kube-apiserver-k8s-117            1/1     Running   0          3d23h
kube-apiserver-k8s-118            1/1     Running   23         3d22h
kube-controller-manager-k8s-116   1/1     Running   0          4d5h
kube-controller-manager-k8s-117   1/1     Running   0          3d23h
kube-controller-manager-k8s-118   1/1     Running   0          3d22h
kube-proxy-xg9rt                  1/1     Running   0          72m
kube-scheduler-k8s-116            1/1     Running   2          4d5h
kube-scheduler-k8s-117            1/1     Running   0          3d23h
```



