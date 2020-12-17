---

title: "Kubernetes V1.14 集群安装master(三)"
date: 2019-03-28T15:59:31+08:00
draft: false  
categories: ["kubernetes"]
tags: ["kubernetes"]

---

 master 节点的安装，使用hy

<!--more-->

1. admin证书制作  

    kubectl 与 kube-apiserver 的安全端口通信，需要为安全通信提供TLS证书和秘钥 

 vim admin-csr.json

```bash
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
```

   生成admin 证书和私钥  

```bash
    [root@devops ssl]# cfssl gencert -ca ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes admin-csr.json  | cfssljson -bare admin
    [root@devops ssl]# ll admin*
     -rw-r--r-- 1 root root 1009 Mar 28 16:56 admin.csr
     -rw-r--r-- 1 root root  297 Mar 28 16:47 admin-csr.json
     -rw------- 1 root root 1679 Mar 28 16:56 admin-key.pem
     -rw-r--r-- 1 root root 1399 Mar 28 16:56 admin.pem
```

2. 创建kubernetes 证书  

   创建kubernetes-csr.json 文件


   vim  kubernetes-csr.json  

```bash
      {
     "CN": "kubernetes",
     "hosts": [
       "127.0.0.1",
       "10.39.30.116",
       "10.39.30.117",
       "10.39.30.118",
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
```

   ​制作kubernetes证书

```bash
 [root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes kubernetes-csr.json  | cfssljson  -bare kubernetes
```


   查看证书 

```bash
[root@devops ssl]# ll kubernetes*
     -rw-r--r-- 1 root root 1261 Mar 28 17:22 kubernetes.csr
     -rw-r--r-- 1 root root  588 Mar 28 17:21 kubernetes-csr.json
     -rw------- 1 root root 1679 Mar 28 17:22 kubernetes-key.pem
     -rw-r--r-- 1 root root 1627 Mar 28 17:22 kubernetes.pem
```

3. 证书分发  


```bash
[root@devops k8s]# ansible -i hosts k8s -m copy -a "src=./ssl/   dest=/etc/kubernetes/ssl" 	        
```

4. copy 二进制文件

​    拷贝kubectl   kubelet  kuebeadm  

```bash
[root@devops k8s]# ansible -i hosts master -m copy -a "src=/data/kubernetes/_output/dockerized/bin/linux/amd64/kubectl dest=/usr/local/bin"
[root@devops k8s]# ansible -i hosts master -m copy -a "src=/data/kubernetes/_output/dockerized/bin/linux/amd64/kubeadm dest=/usr/local/bin"
[root@devops k8s]# ansible -i hosts master -m copy -a "src=/data/kubernetes/_output/dockerized/bin/linux/amd64/kubelet dest=/usr/local/bin"
```

​     添加执行权限

```bash
[root@devops k8s]# ansible -i hosts master  -m shell -a " chmod +x /usr/local/bin/*" 添加执行权限
```

3. 创建核心组件的启动文件

创建目录/etc/kubernetes/manifests

```bash
[root@devops k8s]# ansible -i hosts k8s -m shell  -a "mkdir /etc/kubernetes/manifests"
```

登录某一台master主机
登录10.39.30.11

创建kube-apiserver.json 文件

 vim  kube-apiserver.json

```bash
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
 #备注 images为之前编译好的镜像 staging-k8s.gcr.io/hyperkube-amd64:v1.11.0 
```

 vim /etc/kubernetes/audit-policy.yaml

```bash
apiVersion: audit.k8s.io/v1beta1
kind: Policy
  rules:
  - level: Metadata
```

创建kube-controller-manager.json 文件

 vim  kube-controller-manager.json

```bash
 {
          "kind": "Pod",
          "apiVersion": "v1",
          "metadata": {
            "name": "kube-controller-manager",
            "namespace": "kube-system",
            "labels": {
              "component": "kube-controller-manager",
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
              },
              {
                "name": "plugin",
                "hostPath": {
                  "path": "/usr/libexec/kubernetes/kubelet-plugins"
                }
              }
            ],
            "containers": [
              {
                "name": "kube-controller-manager",
                "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0",
                "imagePullPolicy": "Always",
                "command": [
                  "/controller-manager",
                  "--address=127.0.0.1",
                  "--master=http://127.0.0.1:8080",
                  "--allocate-node-cidrs=false",
                  "--service-cluster-ip-range=10.254.0.0/18",
                  "--cluster-cidr=10.254.64.0/18",
                  "--cluster-name=kubernetes",
                  "--cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem",
                  "--cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem",
                  "--service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem",
                  "--root-ca-file=/etc/kubernetes/ssl/ca.pem",
                  "--experimental-cluster-signing-duration=86700h0m0s",
                  "--leader-elect=true",
                  "--controllers=*,tokencleaner,bootstrapsigner",
                  "--feature-gates=RotateKubeletServerCertificate=true",
                  "--horizontal-pod-autoscaler-use-rest-clients",
                  "--horizontal-pod-autoscaler-sync-period=60s",
                  "--node-monitor-grace-period=40s",
                  " --node-monitor-period=5s",
                  "--pod-eviction-timeout=5m0s",
                  "--v=10"
                ],
                "resources": {
                  "requests": {
                    "cpu": "200m"
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
                  },
                  {
                    "name": "plugin",
                    "mountPath": "/usr/libexec/kubernetes/kubelet-plugins"
                  }
                ],
                "livenessProbe": {
                  "httpGet": {
                    "path": "/healthz",
                    "port": 10252,
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

创建kube-scheduler.json 文件

vim kube-scheduler.json

```bash
    {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "kube-scheduler",
        "namespace": "kube-system",
        "creationTimestamp": null,
        "labels": {
          "component": "kube-scheduler",
          "tier": "control-plane"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "kube-scheduler",
            "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0",
            "imagePullPolicy": "Always",
            "command": [
              "/scheduler",
              "--address=127.0.0.1",
              "--master=http://127.0.0.1:8080",
              "--leader-elect=true",
              "--v=2"
            ],
            "resources": {
              "requests": {
                "cpu": "100m"
              }
            },
            "livenessProbe": {
              "httpGet": {
                "path": "/healthz",
                "port": 10251,
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

生成token

kubelet 首次启动时向 kube-apiserver 发送 TLS Bootstrapping 请求，kube-apiserver 验证 kubelet 请求中的 token 是否与它配置的 token 一致，如果一致则自动为 kubelet生成证书和秘钥。

```bash
[root@k8s-116 manifests]# head -c 16 /dev/urandom | od -An -t x | tr -d ' '
5ef39c565fa25df9db0ca2121c612121
```

 创建encryption-config.yaml

在 /etc/kubernetes/  目录创建

```bash
cat >/etc/kubernetes/encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
   - resources:
       - secrets
     providers:
       - aescbc:
           keys:
             name: key1
             secret: 5ef39c565fa25df9db0ca2121c612121
       - identity: {}
EOF拷
#拷贝到其他主机上，kubelet 启动的时候需要
```

创建kubelet 启动加载文件kubelet.config.json 

vim  /etc/kubernetes/kubelet.config.json

```bash
{
          "kind": "KubeletConfiguration",
          "apiVersion": "kubelet.config.k8s.io/v1beta1",
          "authentication": {
            "x509": {
              "clientCAFile": "/etc/kubernetes/ssl/ca.pem"
            },
            "webhook": {
              "enabled": false,
              "cacheTTL": "2m0s"
            },
            "anonymous": {
              "enabled": false
            }
          },
          "authorization": {
            "mode": "AlwaysAllow",
            "webhook": {
              "cacheAuthorizedTTL": "5m0s",
              "cacheUnauthorizedTTL": "30s"
            }
          },
          "address": "10.39.30.116",
          "port": 10250,
          "readOnlyPort": 10255,
          "cgroupDriver": "cgroupfs",
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

 生成kubernetes 配置文件，目录存储在/root/.kube 

配置kubernetes集群

```bash
[root@k8s-116 kubernetes]# kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true --embed-certs=true --server=https://10.39.30.116:6443       #10.39.30.117；10.39.30.118 
```

配置客户端认证

```bash
[root@k8s-116 kubernetes]# kubectl config set-credentials admin   --client-certificate=/etc/kubernetes/ssl/admin.pem   --embed-certs=true   --client-key=/etc/kubernetes/ssl/admin-key.pem

[root@k8s-116 kubernetes]# kubectl config set-context kubernetes    --cluster=kubernetes    --user=admin

[root@k8s-116 ~]# kubectl config use-context kubernetes
```

首次启动kubelet 会去关联注册, 涉及到kubelet授权,详细看以下文档，我需要授权改为所有

[<https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet-authentication-authorization>]()

创建kubelet 启动文件

vim  /etc/systemd/system/kubelet.service

```bash
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/usr/local/bin/kubelet \
          --hostname-override=10.39.30.116 \
          --pod-infra-container-image=harbor.enncloud.cn/enncloud/pause-amd64:3.1 \
          --pod-manifest-path=/etc/kubernetes/manifests  \
          --config=/etc/kubernetes/kubelet.config.json \
          --allow-privileged=true \
          --kube-reserved cpu=500m,memory=512m \
          --image-gc-high-threshold=85  --image-gc-low-threshold=70 \
          --logtostderr=true \
          --enable-controller-attach-detach=true  \
          --v=2 \
          --register-node=false
[Install]
WantedBy=multi-user.target 
```

启动kubelet

```bash
systemctl  status  kubelet 
启动kubelet 会自动读取/etc/kubernetes/manifests/ 目录下的yaml 文件，然后去拉取相应的镜像，来启动，使用docker ps 去查看,是否正常启动
#注意事项: kubelet 的时候检查docker 是否正常启动，并且需要docker login 镜像仓库
```



```bash
systemctl  status  kubelet 
启动kubelet 会自动读取/etc/kubernetes/manifests/ 目录下的yaml 文件，然后去拉取相应的镜像，来启动，使用docker ps 去查看,是否正常启动
#注意事项: kubelet 的时候检查docker 是否正常启动，并且需要docker login 镜像仓库
```

##### #修改kubelet 认证由AlwaysAllow 改变为Webhook

使用kubeadm 生成kubelet 认证

创建token 

```bash
[root@k8s-116 ~]# kubeadm token create --description kubelet-bootstrap-token --groups   system:bootstrappers:k8s-116 --kubeconfig ~/.kube/config 
 hksg7a.lrt1g0o3jcrg8bsy  
[root@k8s-116 ~]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:k8s-117 --kubeconfig ~/.kube/config         
mnnzkk.zaax0qki489afyk1
[root@k8s-116 ~]# kubeadm token create --description kubelet-bootstrap-token --groups system:bootstrappers:k8s-118  --kubeconfig ~/.kube/config                     dpqje5.km320a0uyu9d1o4j 
```

查看token 

```bash
kubeadm token list --kubeconfig ~/.kube/config   
```

生成k8s-116的bootstrap.kubeconfig 

  配置集群参数

```bash
kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true   --server=https://10.39.30.116:6443   --kubeconfig=bootstrap.kubeconfig
```

配置客户端认证

```bash
 kubectl config set-credentials kubelet-bootstrap   --token=hksg7a.lrt1g0o3jcrg8bsy  --kubeconfig=bootstrap.kubeconfig
```

配置关联

```bash
kubectl config set-context default   --cluster=kubernetes   --user=kubelet-bootstrap   --kubeconfig=bootstrap.kubeconfig
```

配置默认关联

```bash
kubectl config use-context default --kubeconfig=bootstrap.kubeconfi
mv  bootstrap.kubeconfig  /etc/kubernetes
```

修改kubelet 启动配置文件

修改kubelet 启动配置文件

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
      --hostname-override=k8s-116 \
      --pod-infra-container-image=harbor.enncloud.cn/enncloud/pause-amd64:3.1 \
      --bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
      --pod-manifest-path=/etc/kubernetes/manifests  \
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
# 备注: --hostname-override=k8s-116 最好和token 认证时的一样 
```

​    vim /etc/kubernetes/kubelet.config.json

```bash
"webhook": {
   "enabled": true
},

  "authorization": {
   "mode": "Webhook" 
```

创建自动批准相关CSR请求ClusterRole

vim /etc/kubernetes/tls-instructs-csr.yaml

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
  rules:
   - apiGroups: ["certificates.k8s.io"]
     resources: ["certificatesigningrequests/selfnodeserver"]
     verbs: ["create"]
```

```bash
kubectl create -f /etc/kubernetes/tls-instructs-csr.yaml
```

查看

```bash
[root@k8s-116 ~]# kubectl describe ClusterRole/system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
    Name:         system:certificates.k8s.io:certificatesigningrequests:selfnodeserver
    Labels:       <none>
    Annotations:  <none>
    PolicyRule:
      Resources                                                      Non-Resource URLs  Resource Names  Verbs
      ---------                                                      -----------------  --------------  -----
      certificatesigningrequests.certificates.k8s.io/selfnodeserver  []                 []              [create]
```

创建csr 请求认证

```bash
#自动批准system:bootstrappers组用户TLS bootstrapping首次申请证书的CSR请求 
kubectl create clusterrolebinding node-client-auto-approve-csr --clusterrole=system:certificates.k8s.io:certificatesigningrequests:nodeclient --group=system:bootstrappers
```

```bash
#自动批准 system:nodes组用户更新kubelet自身与apiserver通讯证书的CSR请求
kubectl create clusterrolebinding node-client-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeclient --group=system:nodes  
```

```bash
#自动批准 system:nodes组用户更新kubelet 10250 api端口证书的CSR请求
kubectl create clusterrolebinding node-server-auto-renew-crt --clusterrole=system:certificates.k8s.io:certificatesigningrequests:selfnodeserver --group=system:nodes
```

创建RBAC

```bash
kubelet 授权 kube-apiserver 的一些操作 exec run logs 等
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes
```

创建bootstrap RBAC 权限

```bash
 kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
```

重启kubelet

```bash
     systemctl    restart   kubelet   
#如果有报错信息：
    failed to run Kubelet: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidd...
     Hint: Some lines were ellipsized, use -l to show in full.
 #故障解决说明 
     kubelet 启动时查找配置的--kubeletconfig 文件是否存在，如果不存在则使用--bootstrap-kubeconfig 向kube-apiserver 发送证书签名请求(csr)
     kube-apiserver 收到csr请求后，对其中的Token进行认证(使用kubeadm 创建的token)，认证通过后将请求的user 设置为system:bootstrap；group设置为system:bootstrappers，这一过程称为 bootstrap token auth 
#默认情况下，这个user 和group 没有创建CSR的权限，kubelet 启动才会失败
#查看
     kubectl   get clusterrolebinding
#如果没有就创建，如果有的话先删除
     kubectl   delete  clusterrolebinding   kubelet-bootstrap 
     kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --group=system:bootstrappers
     systemctl restart kubelet
```

查看集群

```bash
[root@k8s-116 ~]# kubectl get node
NAME      STATUS   ROLES    AGE   VERSION
k8s-116   Ready    <none>   67m   v1.14.0  
#集群打标签，ROLES 是节点标签 
kubectl label nodes k8s-116 node-role.kubernetes.io/master=
[root@k8s-116 ~]# kubectl  get node
NAME      STATUS   ROLES    AGE    VERSION
k8s-116   Ready    master   4h9m   v1.14.0
#不需要调度资源的可以加上如下标签     
kubectl taint nodes k8s-116 node-role.kubernetes.io/master=:NoSchedule
```

##### 添加其他master 节点

生成kubernetes 配置文件，生成证书相关文件存储在/root/.kube 目录中

配置kubernetes集群

```bash
[root@k8s-117 ~]# kubectl config set-cluster kubernetes \
 --certificate-authority=/etc/kubernetes/ssl/ca.pem \
 --embed-certs=true \
 --server=https://10.39.30.116:6443
```



因为要向116去认证，所以写的是116 的

配置客户端认证

```bash
[root@k8s-117 ~]# kubectl config set-credentials admin \
    --client-certificate=/etc/kubernetes/ssl/admin.pem \
    --embed-certs=true \
    --client-key=/etc/kubernetes/ssl/admin-key.pem  

[root@k8s-117 ~]# kubectl config set-context kubernetes \
    --cluster=kubernetes \
    --user=admin
[root@k8s-117 ~]# kubectl config use-context kubernetes
```




这个文件可以直接拷贝116上即可，不需要每台机器都去生成一下

在master-116上查看117 的token，如果没有就重新创建

```bash
[root@k8s-116 ~]# kubeadm token list --kubeconfig ~/.kube/config
```



生成k8s-117 bootstrap.kubeconfig

配置集群参数

```bash
kubectl config set-cluster kubernetes   --certificate-authority=/etc/kubernetes/ssl/ca.pem   --embed-certs=true   --server=https://10.39.30.116:6443   --kubeconfig=bootstrap.kubeconfig
```



配置客户端认证

```bash
 kubectl config set-credentials kubelet-bootstrap   --token=e9u50h.nrden3hosu4yfc9t --kubeconfig=bootstrap.kubeconfig
```



配置关联

```bash
kubectl config set-context default   --cluster=kubernetes   --user=kubelet-bootstrap   --kubeconfig=bootstrap.kubeconfig
```



配置默认关联

```bash
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
mv  bootstrap.kubeconfig  /etc/kubernetes 
vim  /etc/kubernetes/audit-policy.yaml
  apiVersion: audit.k8s.io/v1beta1
  kind: Policy
  rules:
     level: Metadata 
```




创建encryption-config.yaml 文件

  

```bash
cat > /etc/kubernetes/encryption-config.yaml <<EOF
    kind: EncryptionConfig
    apiVersion: v1
    resources:

- resources:
  - secrets
    providers:
    - aescbc:
      keys:
      - name: key1
        secret: 5ef39c565fa25df9db0ca2121c612121
- identity: {}
  F   
```



 创建核心组件json 文件和116 基本一致

 cd  /etc/kubernetes/manifests   

vim kube-apiserver.json  

​     

```yaml
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
              "--advertise-address=10.39.30.117",
              "--allow-privileged=true",
              "--apiserver-count=3",
              "--audit-policy-file=/etc/kubernetes/audit-policy.yaml",
              "--audit-log-maxage=30",
              "--audit-log-maxbackup=3",
              "--audit-log-maxsize=100",
              "--audit-log-path=/var/log/kubernetes/audit.log",
              "--authorization-mode=Node,RBAC",
              "--bind-address=10.39.30.117",
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

vim kube-controller-manager.json

```yml
 {
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "kube-controller-manager",
        "namespace": "kube-system",
        "labels": {
          "component": "kube-controller-manager",
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
          },
          {
            "name": "plugin",
            "hostPath": {
              "path": "/usr/libexec/kubernetes/kubelet-plugins"
            }
          }
        ],
        "containers": [
          {
            "name": "kube-controller-manager",
            "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0",
            "imagePullPolicy": "Always",
            "command": [
              "/controller-manager",
              "--address=127.0.0.1",
              "--master=http://127.0.0.1:8080",
              "--allocate-node-cidrs=false",
              "--service-cluster-ip-range=10.254.0.0/18",
              "--cluster-cidr=10.254.64.0/18",
              "--cluster-name=kubernetes",
              "--cluster-signing-cert-file=/etc/kubernetes/ssl/ca.pem",
              "--cluster-signing-key-file=/etc/kubernetes/ssl/ca-key.pem",
              "--service-account-private-key-file=/etc/kubernetes/ssl/ca-key.pem",
              "--root-ca-file=/etc/kubernetes/ssl/ca.pem",
              "--experimental-cluster-signing-duration=86700h0m0s",
              "--leader-elect=true",
              "--controllers=*,tokencleaner,bootstrapsigner",
              "--feature-gates=RotateKubeletServerCertificate=true",
              "--horizontal-pod-autoscaler-use-rest-clients",
              "--horizontal-pod-autoscaler-sync-period=60s",
              "--node-monitor-grace-period=40s",
              " --node-monitor-period=5s",
              "--pod-eviction-timeout=5m0s",
              "--v=10"
            ],
            "resources": {
              "requests": {
                "cpu": "200m"
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
              },
              {
                "name": "plugin",
                "mountPath": "/usr/libexec/kubernetes/kubelet-plugins"
              }
            ],
            "livenessProbe": {
              "httpGet": {
                "path": "/healthz",
                "port": 10252,
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

 

vi kube-scheduler.json  

​     

```json
{
      "kind": "Pod",
      "apiVersion": "v1",
      "metadata": {
        "name": "kube-scheduler",
        "namespace": "kube-system",
        "creationTimestamp": null,
        "labels": {
          "component": "kube-scheduler",
          "tier": "control-plane"
        }
      },
      "spec": {
        "containers": [
          {
            "name": "kube-scheduler",
            "image": "harbor.enncloud.cn/enncloud/hyperkube-amd64:v1.14.0",
            "imagePullPolicy": "Always",
            "command": [
              "/scheduler",
              "--address=127.0.0.1",
              "--master=http://127.0.0.1:8080",
              "--leader-elect=true",
              "--v=2"
            ],
            "resources": {
              "requests": {
                "cpu": "100m"
              }
            },
            "livenessProbe": {
              "httpGet": {
                "path": "/healthz",
                "port": 10251,
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

vim /etc/kubernetes/kubelet.config.json 

​     

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
      "address": "10.39.30.117",
      "port": 10250,
      "readOnlyPort": 10255,
      "cgroupDriver": "cgroupfs",
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



创建kubelet 启动文件


vim /etc/systemd/system/kubelet.service

​    

```bash
[Unit]
    Description=Kubernetes Kubelet
    Documentation=https://github.com/GoogleCloudPlatform/kubernetes
    After=docker.service
    Requires=docker.service

[Service]
 WorkingDirectory=/var/lib/kubelet
 ExecStart=/usr/local/bin/kubelet \
     --hostname-override=k8s-117 \
     --pod-infra-container-image=harbor.enncloud.cn/enncloud/pause-amd64:3.1 \
     --bootstrap-kubeconfig=/etc/kubernetes/bootstrap.kubeconfig \
     --pod-manifest-path=/etc/kubernetes/manifests  \
     --kubeconfig=/etc/kubernetes/kubelet.config \
     --config=/etc/kubernetes/kubelet.config.json \
     --cert-dir=/etc/kubernetes/pki \
     --allow-privileged=true \
     --kube-reserved cpu=500m,memory=512m \
     --image-gc-high-threshold=85  --image-gc-low-threshold=70 \
     --logtostderr=true \
     --enable-controller-attach-detach=true  \
     --v=4
[Install]
WantedBy=multi-user.target
```



启动kubelet

  

```bash
systemctl restart  kubelet  
```



修改vi /etc/kubernetes/bootstrap.kubeconfig

​    

```bash
  server: https://10.39.30.117:6443
```




修改为117的

  

```bash
    systemctl restart  kubelet
```



查看集群



```bash
kubectl label nodes k8s-117 node-role.kubernetes.io/node=
[root@k8s-116 ~]# kubectl get node
 NAME      STATUS   ROLES    AGE     VERSION
 k8s-116   Ready    master   6h52m   v1.14.0
 k8s-117   Ready    node     80m     v1.14.0
```



其他master 节点增加与117 相同

```bash
[root@k8s-118 kubernetes]# kubectl get node
 NAME      STATUS   ROLES    AGE     VERSION
 k8s-116   Ready    master   7h28m   v1.14.0
 k8s-117   Ready    master   116m    v1.14.0
 k8s-118   Ready    master   11m     v1.14.0
```



组件运行状态

  

```bash
[root@k8s-118 kubernetes]# kubectl get pod -n kube-system
 NAME                              READY   STATUS    RESTARTS   AGE
 kube-apiserver-k8s-116            1/1     Running   12         7h28m
 kube-apiserver-k8s-117            1/1     Running   0          116m
 kube-apiserver-k8s-118            1/1     Running   23         11m
 kube-controller-manager-k8s-116   1/1     Running   2          7h28m
 kube-controller-manager-k8s-117   1/1     Running   0          116m
 kube-controller-manager-k8s-118   1/1     Running   0          11m
 kube-scheduler-k8s-116            1/1     Running   2          7h28m
 kube-scheduler-k8s-117            1/1     Running   0          116m
 kube-scheduler-k8s-118            1/1     Running   0          11m
```



































