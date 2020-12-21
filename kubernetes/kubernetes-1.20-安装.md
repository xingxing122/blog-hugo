



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
| Service 网段 | 192.168.0.0/16                         |
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
```

### 3. 安装

##### docker 安装

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

kubeadm 安装

```
将之前编译好的kubeadm二进制文件拷贝到
```

