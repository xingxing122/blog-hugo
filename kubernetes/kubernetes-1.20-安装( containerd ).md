



<!--more-->

### 1. 节点规划



| 主机名        | IP地址       | 角色   | 配置      |
| :------------ | ------------ | ------ | :-------- |
| k8s-master-03 | 10.39.60.157 | Master | 4C8G/20GB |
| k8s-master-02 | 10.39.60.154 | Master | 4c8G/20GB |
| k8s-master-01 | 10.39.60.175 | master | 4c8G/20GB |
|               |              |        |           |

| 信息         | 备注                                        |
| ------------ | ------------------------------------------- |
| 系统版本     | debian  Linux k8s-master-01 4.19.0-12-amd64 |
| Docker版本   | 无                                          |
| k8s 版本     | 1.20                                        |
| Pod 网段     | 172.168.0.0/16                              |
| Service 网段 | 192.168.0.0/12                              |
| LB           | 用云端的alb                                 |

### 2.基本信息配置

#### 2.1 所有节点配置hosts 

```bash
10.39.60.157     k8s-master-03
10.39.60.154     k8s-master-02
10.39.60.175     k8s-master-01
```

#### 2.2 安装软件包

```bash

```

#### 2.3 关闭swap分区

```bash
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

#### 2.4 时钟同步

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime 
echo 'Asia/Shanghai' >/etc/timezone 
ntpdate time2.aliyun.com

# 加入到crontab 
*/5 * * * * ntpdate time2.aliyun.com
```

#### 2.5 设置limit

```bash
# /etc/security/limits.conf
* soft nofile 655360
* hard nofile 131072
* soft nproc 655350
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
```

#### 2.6 master01 节点免秘钥登录其他节点

```bash
ssh-keygen -t rsa
for i in k8s-master-02  k8s-master-03 k8s-master-01; do ssh-copy-id -i .ssh/id_rsa.pub $i;done

```

#### 2.7 内核配置

##### 内核参数

```bash
cat > /etc/sysctl.d/kubernetes.conf <<EOF 
#将桥接的IPv4流量传递到iptables 的链
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
system --system # 生效
```

### 3. 安装

https://kubernetes.io/zh/docs/setup/production-environment/container-runtimes/#containerd

#### 3.1 containerd 安装

```bash
安装和配置 
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# 设置必需的 sysctl 参数，这些参数在重新启动后仍然存在。
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# 应用 sysctl 参数而无需重新启动

#安装containerd 
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  sudo apt-get install containerd.io
 
#配置
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml

#重新启动containerd 
sudo systemctl restart containerd

使用 systemd cgroup 驱动程序
结合 runc 使用 systemd cgroup 驱动，在 /etc/containerd/config.toml 中设置
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
#重启
vim  /etc/containerd/config.toml   

sudo systemctl restart containerd

```

#### 3.2 kubeadm 安装

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configure-cgroup-driver-used-by-kubelet-on-control-plane-node

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
apt-get update && apt-get install -y apt-transport-https
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

#指定版本安装
apt-get install  kubelet=1.20.0-00  kubeadm=1.20.0-00  kubectl=1.20.0-00  

```

#### 3.3 启动kubelet

```bash
systemctl daemon-reload
systemctl enable --now kubelet
## 初始化成功之后kubelet 会自动启动
```

#### 3.4 打印初始化文件

```bash
kubeadm config print init-defaults
kubeadm config print init-defaults --component-configs KubeletConfiguration
kubeadm config print init-defaults --component-configs KubeProxyConfiguration
### 根据打印的文件可以修改为自己的

也可以使用
kubeadm init \
  --apiserver-advertise-address=10.39.60.175 \
  --image-repository registry.aliyuncs.com/google_containers \
  --kubernetes-version v1.20.0 \
  --service-cidr=10.96.0.0/12 \
  --pod-network-cidr=10.244.0.0/16 \
  --control-plane-endpoint=10.39.60.221:6443 \
  --ignore-preflight-errors=all
  
  
  # --control-plane-endpoint 负载均衡的ip，如果不是集群可不写，
  集群的配置信息会生成在kubectl -n kube-system get   cm kubeadm-config -oyaml 这里，方便查看,如果涉及到修改可能需要kubeadm  reset 重置集群 
```

#### 3.4 输出信息

```bash

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join 10.39.60.221:6443 --token 05iqjt.mk4i4jijsq3t89mu \
    --discovery-token-ca-cert-hash sha256:745f7ea4bad5e2f9a62157411a4c81e1eb3dccbcfca2a4e9547e8a8016f3824d \
    --control-plane 

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.39.60.221:6443 --token 05iqjt.mk4i4jijsq3t89mu \
    --discovery-token-ca-cert-hash sha256:745f7ea4bad5e2f9a62157411a4c81e1eb3dccbcfca2a4e9547e8a8016f3824d 
```

```bash
#--control-plane 为加入Master 节点 
#token具有实效性，如果失效，可以创建
kubeadm token create –print-join-command 创建新的 join token
#故障解决
由于各种原因会导致的失败
kubeadm   reset 重新初始化  
```

#### 3.8 master1 上操作

```bash
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
export KUBECONFIG=/etc/kubernetes/admin.conf
# kubectl  get node   查看集群信息
```

#### 3.9 安装calico

https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

```bash
curl https://docs.projectcalico.org/manifests/calico-typha.yaml -o calico.yaml 
#需要修改calico.yaml 中
CALICO_IPV4POOL_CIDR 与kubeadm init 指定的  --pod-network-cidr 一致 
kubectl apply -f calico.yaml 
```

####  3.10 加入其他Master 节点

```bash
#

# 根据提示执行  
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 3.11 遇到的故障

```bash
#故障一 
root@k8s-master-02:~# kubeadm join 10.39.60.175:6443 --token  yjqd42.mimqzh2bczx4ck9w    --discovery-token-ca-cert-hash sha256:c7b8645c2263f4557a65eb53f9c076a6cf84dbdd4e94fa9a89ea07f671f9d53a     --control-plane --certificate-key a4a4df512944b2505a0f10320d7ea2d0857e335860977b57e0a1de48331a5e8f 
[preflight] Running pre-flight checks
        [WARNING SystemVerification]: missing optional cgroups: hugetlb
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
error execution phase preflight: 
One or more conditions for hosting a new control plane instance is not satisfied.

unable to add a new control plane instance a cluster that doesn't have a stable controlPlaneEndpoint address

Please ensure that:

* The cluster has a stable controlPlaneEndpoint address.
* The certificates that must be shared among control plane instances are provided.


To see the stack trace of this error execute with --v=5 or higher

# 解决
# 在k8s-master01 上
kubectl -n kube-system get cm kubeadm-config -o yaml 
```




#### 3.12 配置Master to node 

```bash
#这里主要是让master直接可以运行pods 
#执行命令
kubectl taint node node-name node-role.kubernetes.io/master-
#禁止master运行pod 
kubectl taint nodes node-name node-role.kubernetes.io/master=:NoSchedule
#增加 ROLES 标签: 
kubectl label nodes localhost node-role.kubernetes.io/node=
#删除 ROLES 标签
kubectl label nodes localhost node-role.kubernetes.io/node-

#cordns 可以根据节点数进行调整
kubectl scale deploy/coredns --replicas=3 -n kube-system  
#增加副本数
```

#### 3.15 Metrics Server部署

[官网文档参考](https://github.com/kubernetes-sigs/metrics-server)

```bash
#下载到本地
wget  https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml

vim  components.yaml
......................
containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        image: k8s.gcr.io/metrics-server:v0.5.0   #修改为阿里云或者自己本地拉取的
        imagePullPolicy: IfNotPresent
        ### 添加下面几行
        command:
          - /metrics-server
          - --kubelet-preferred-address-types=InternalIP,Hostname,Internaldns,ExternalDNS,ExternalIP
          - --kubelet-insecure-tls   # 关闭kubelet认证 
 #开启认证
          - --requestheader-client-ca-file 
#因为镜像需要科学上网拉取，可以先把镜像pull下来，然后修改镜像地址 
[root@k8s-master-01 opt]# kubectl  apply -f components.yaml
serviceaccount/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
service/metrics-server created
deployment.apps/metrics-server created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
```

##### 验证

```bash
root@k8s-master-01:~# kubectl  top  node
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
i-konvehyo      49m          1%     703Mi           12%       
i-mz9wlzx3      45m          1%     832Mi           14%       
k8s-master-01   243m         6%     1245Mi          21%       
k8s-master-02   109m         2%     1192Mi          20%       
k8s-master-03   103m         2%     1153Mi          19%     

#查看pod 的资源使用
kubectl top pods -n kube-system
```

部署kubernetes-dashboard 

```bash
# 修改recommended.yaml 
## 注释掉Dashboard  Secret，不然后面访问网页不安全，证书过期 
#apiVersion: v1
#kind: Secret
#metadata:
#  labels:
#    k8s-app: kubernetes-dashboard
#  name: kubernetes-dashboard-certs
#  namespace: kubernetes-dashboard
#type: Opaque

#将type: targetPort  修改为 type: NodePort  38000

root@k8s-master-01:~# kubectl  apply -f  recommended.yaml 
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

创建service  account 并绑定默认的cluster-admin 管理员集群角色

```bash
kubectl create serviceaccount dashboard-admin -n kube-system

kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin

kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

