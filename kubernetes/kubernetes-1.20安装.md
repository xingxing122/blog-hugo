



<!--more-->

### 1. 节点规划



| 主机名        | IP地址       | 角色   | 配置            |
| :------------ | ------------ | ------ | :-------------- |
| k8s-master-03 | 10.39.60.153 | Master | 4C8G/20GB       |
| k8s-master-02 | 10.39.60.154 | Master | 4c8G/20GB       |
| k8s-master-01 | 10.39.60.157 | master | 4c8G/20GB       |
| K8s-node1     | 10.39.60.175 | Node   | 4c8G/20GB/150GB |

| 信息         | 备注                                   |
| ------------ | -------------------------------------- |
| 系统版本     | Centos  7.9 3.10.0-1160.6.1.el7.x86_64 |
| Docker版本   |                                        |
| k8s 版本     | 1.20                                   |
| Pod 网段     | 172.168.0.0/16                         |
| Service 网段 | 192.168.0.0/12                         |
| LB           | 用云端的alb                            |

### 2.基本信息配置

#### 2.1 所有节点配置hosts 

```bash
10.39.60.153     k8s-master-03
10.39.60.154     k8s-master-02
10.39.60.157     k8s-master-01
10.39.60.175     K8s-node1
```

#### 2.2 安装软件包

```bash
yum install wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git -y
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

##### 安装ipvsadm

```bash
yum install ipvsadm ipset sysstat conntrack libseccomp -y
```

##### 所有节点配置ipvs

```bash
vim /etc/modules-load.d/ipvs.conf 
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack_ipv4
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip

### 加载内核配置
systemctl enable --now systemd-modules-load.service
#查看加载的模块
lsmod | grep -e ip_vs -e nf_conntrack_ipv4 
```

##### 内核参数

```bash
vim /etc/sysctl.d/kubernetes.conf 
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
# sysctl --system
```

### 3. 安装

#### 3.1 docker 安装

```bash
 yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
 
 yum install -y yum-utils 
 yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
    
 yum install docker-ce docker-ce-cli containerd.io -y 
 #### 启动docker 
 systemctl daemon-reload && systemctl enable --now docker 
```

#### 3.2 kubeadm 安装

```bash
使用yum 安装 
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
yum install -y yum-utils device-mapper-persistent-data lvm2
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo

yum list kubeadm.x86_64 --showduplicates
yum list kubeadm.x86_64 --showduplicates | sort -r 
yum install kubeadm -y 
#自带安装一些依赖包
```

![image-20201222105650271](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-22-025650.png)

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
```

 kubeadm-init.yaml

```yaml
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.39.60.157
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8s-master-01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  # api vip地址
  - 10.39.60.221
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
#apiserver 的高可用vip地址
controlPlaneEndpoint: 10.39.60.221:6443
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
   #etcd 存储的目录
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
networking:
  dnsDomain: cluster.local
  podSubnet: 172.168.0.0/16
  serviceSubnet: 192.168.0.0/12
#默认调度
scheduler: {} 
```

#### 3.5 下载镜像

```bash
kubeadm config images pull --config /root/kubeadm-init.yaml
```

#### 3.6 初始化集群

```bash
kubeadm init --config /root/kubeadm-init.yaml  --upload-certs
### --upload-certs 会在加入master 节点的时候自动拷贝证书
```

#### 3.7 输出信息

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
You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 10.39.60.221:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:aa12ea3319ceb1882f4ff7a6ef6fd71806d4a150d4e77db01dc3b9564f98e4db \
    --control-plane --certificate-key 5abd57dba90cdafd7e7c457efc7fbb5593a8a276ecbdc65fb80fcf92cae2141b

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.39.60.221:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:aa12ea3319ceb1882f4ff7a6ef6fd71806d4a150d4e77db01dc3b9564f98e4db 
    
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

![image-20201222151223082](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-22-071223.png)

#### 3.9 查看组件

```bash
初始化安装的组件都在kube-system 空间
```

![image-20201222151409707](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-22-071410.png)

####  3.10 加入其他Master 节点

```bash
#k8s-master-02 与k8s-master-03执行 
kubeadm join 10.39.60.221:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:aa12ea3319ceb1882f4ff7a6ef6fd71806d4a150d4e77db01dc3b9564f98e4db \
    --control-plane --certificate-key 5abd57dba90cdafd7e7c457efc7fbb5593a8a276ecbdc65fb80fcf92cae2141b
# 根据提示执行  
        mkdir -p $HOME/.kube
        sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
        sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 3.11 验证Master 节点

![image-20201222161132036](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-22-081135.png)

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

#### 3.13 部署node 节点

```bash
# k8s-node1 上执行
kubeadm join 10.39.60.221:6443 --token abcdef.0123456789abcdef     --discovery-token-ca-cert-hash sha256:aa12ea3319ceb1882f4ff7a6ef6fd71806d4a150d4e77db01dc3b9564f98e4db 
```

#### 3.14 网络组件安装

kubernetes 常见的网络组件有calico   flannel,  我选择calico 网络组件  

[calico 官网安装文档](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-etcd-datastore)

****calico 文件修改, 使用calico 3.17 版本

```bash
etcd-ca: `cat /etc/kubernetes/pki/etcd/ca.crt | base64 | tr -d '\n'`
etcd-cert: `cat /etc/kubernetes/pki/etcd/server.crt | base64 | tr -d '\n'`
etcd-key: `cat /etc/kubernetes/pki/etcd/server.key | base64 | tr -d '\n'`

etcd_endpoints: "https://10.39.60.153:2379,https://10.39.60.154:2379,https://10.39.60.157:2379"
  
etcd_ca: "/calico-secrets/etcd-ca"
etcd_cert: "/calico-secrets/etcd-cert"
etcd_key: "/calico-secrets/etcd-key"

veth_mtu: "1340"
#青云的网络需要配置MTU为1340，其他网络根据情况默认即可
   - name: CALICO_IPV4POOL_CIDR
     value: "172.168.0.0/16"  #pod 的网段
```

##### 故障解决

```bash
2020-12-24 03:24:27.407 [INFO][1] main.go 88: Loaded configuration from environment config=&config.Config{LogLevel:"info", WorkloadEndpointWorkers:1, ProfileWorkers:1, PolicyWorkers:1, NodeWorkers:1, Kubeconfig:"", DatastoreType:"etcdv3"}
2020-12-24 03:24:27.408 [FATAL][1] main.go 101: Failed to start error=failed to build Calico client: could not initialize etcdv3 client: open /calico-secrets/etcd-cert: permission denied
```

##### calico 3.17.1 版本最低权限是0040，而不是0400 

```bash
  volumes:
    # Mount in the etcd TLS secrets with mode 400.
    # See https://kubernetes.io/docs/concepts/configuration/secret/
    - name: etcd-certs
      secret:
        secretName: calico-etcd-secrets
        defaultMode: 0040 #默认是0400，修改为0040即可，问题就解决了
```
##### 验证

![image-20201224150817511](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-24-070822.png)

#### 3.15 Metrics Server部署

[官网文档参考](https://github.com/kubernetes-sigs/metrics-server)

```bash
#下载到本地
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml 

vim  components.yaml
......................
containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        image: k8s.gcr.io/metrics-server:v0.4.1   #修改为阿里云或者自己本地拉取的
        imagePullPolicy: IfNotPresent
        ### 添加下面几行
        command:
          - /metrics-server
          - --kubelet-preferred-address-types=InternalIP,Hostname,Internaldns,ExternalDNS,ExternalIP
          - --kubelet-insecure-tls
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
[root@k8s-master-01 opt]# kubectl  top  node
NAME            CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
k8s-master-01   157m         3%     2188Mi          38%
k8s-master-02   195m         4%     1910Mi          33%
k8s-master-03   142m         3%     1985Mi          34%
k8s-node1       98m          2%     1044Mi          13%

#查看pod 的资源使用
kubectl top pods -n kube-system
```

#### 3.16 部署dashboard 

[参考文档](https://github.com/kubernetes/dashboard)

```bash
wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.1.0/aio/deploy/recommended.yaml  
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

[root@k8s-master-01 opt]# kubectl  apply -f recommended.yaml
namespace/kubernetes-dashboard unchanged
serviceaccount/kubernetes-dashboard unchanged
service/kubernetes-dashboard unchanged
secret/kubernetes-dashboard-csrf unchanged
secret/kubernetes-dashboard-key-holder unchanged
configmap/kubernetes-dashboard-settings unchanged
role.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard unchanged
deployment.apps/kubernetes-dashboard configured
service/dashboard-metrics-scraper unchanged
deployment.apps/dashboard-metrics-scraper unchanged
```

![image-20201225152931570](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-12-25-072935.png)

###### 生成新的secret 

```bash
mkdir key && cd key
kubectl create  namespace kubernetes-dashboard
openssl genrsa -out dashboard.key 2048
openssl req -new -out dashboard.csr -key dashboard.key -subj '/CN=10.39.60.221'
openssl x509 -req -in dashboard.csr -signkey dashboard.key -out dashboard.crt
kubectl create secret generic kubernetes-dashboard-certs --from-file=dashboard.key --from-file=dashboard.crt -n kubernetes-dashboard
```

##### 设置权限文件

[官网文档](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md)

****admin-user.yaml

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```

****admin-user-role-binding.yaml

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```

****部署权限文件

```bash
kubectl create -f admin-user.yaml  
kubectl create -f admin-user-role-binding.yaml
```

