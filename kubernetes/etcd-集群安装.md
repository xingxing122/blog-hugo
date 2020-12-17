---
title: "Etcd 集群安装"
date: 2020-09-18T10:47:23+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

### etcd 集群的安装

<!--more-->

etcd 集群安装，使用https 模式 

#### 1.制作证书

创建CA 证书

```json
#config.json 文件
mkdir ssl
vim ssl/config.json 

{
     "signing": {
       "default": {
         "expiry": "87600h"
       },
       "profiles": {
         "kubernetes": {
           "usages": [
               "signing",
               "key encipherment",
               "server auth",
               "client auth"
           ],
           "expiry": "87600h"
         }
       }
     }
}

# csr.json 文件
{
  "CN": "kubernetes",
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

#### 2. 生成CA证书和私钥

```bash
./cfssl gencert -initca ./ssl/csr.json | ./cfssljson  -bare ca
 
 #查看证书                                                                           
  rw-------  1  xingxing  staff     1 KiB  Fri Sep 18 14:00:14 2020    ca-key.pem
  rw-r--r--  1  xingxing  staff  1005 B    Fri Sep 18 14:00:14 2020    ca.csr
  rw-r--r--  1  xingxing  staff     1 KiB  Fri Sep 18 14:00:14 2020    ca.pem
  rw-r--r--  1  xingxing  staff   340 B    Fri Sep 18 11:06:02 2020    config.json
  rw-r--r--  1  xingxing  staff   210 B    Fri Sep 18 13:57:40 2020   csr.json
```

#### 3. 创建etcd 证书 

```json
#etcd-csr.json 
../cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes etcd-csr.json  | ../cfssljson -bare etcd
```

![image-20200918142859062](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-18-062859.png)

#### 4. 检查证书

```shell
../cfssl-certinfo -cert etcd.pem
```

##### 4.1将证书copy 到服务器上

```bash
cat hosts                                                                                       [etcd]
10.39.68.22
10.39.68.23
10.39.68.24

[etcd:vars]
ansible_ssh_pass="enN12345"
ansible_ssh_user="root"

# 创建存放证书的目录
ansible -i hosts etcd -m shell -a "mkdir  /etc/kubernetes/ssl/ -p"

# copy 证书到服务器
ansible -i hosts etcd -m copy  -a "src=../ssl/  dest=/etc/kubernetes/ssl"
ansible -i hosts etcd -m shell   -a "ls -l /etc/kubernetes/ssl/ " 

#copy etcd  etcdctl的二进制到服务器上
ansible -i hosts etcd -m copy -a "src=./etcd-v3.4.13-linux-amd64/etcdctl   dest=/usr/bin/"
ansible -i hosts etcd -m copy -a "src=./etcd-v3.4.13-linux-amd64/etcd   dest=/usr/bin/"


#修改etcd证书权限，非root用户，提示证书没有权限
ansible -i hosts etcd -m shell -a "chmod 644 /etc/kubernetes/ssl/etcd*" 
ansible -i hosts etcd -m shell -a "useradd -s /sbin/nologin etcd;mkdir -p /data/etcd;chown -R etcd:etcd /data/etcd"
```



#### 5.安装etcd 

vim templates/etcd.service

```bash
Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/data/etcd
User=etcd

# set GOMAXPROCS to number of processors

ExecStart=/usr/bin/etcd \
       --name={{ ansible_default_ipv4.address }} \
       --cert-file=/etc/kubernetes/ssl/etcd.pem \
       --key-file=/etc/kubernetes/ssl/etcd-key.pem \
       --peer-cert-file=/etc/kubernetes/ssl/etcd.pem \
       --peer-key-file=/etc/kubernetes/ssl/etcd-key.pem \
       --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
       --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
       --initial-advertise-peer-urls=https://{{ ansible_default_ipv4.address }}:2380 \
       --listen-peer-urls=https://{{ ansible_default_ipv4.address }}:2380 \
       --listen-client-urls=https://{{ ansible_default_ipv4.address }}:2379,http://127.0.0.1:2379 \
       --advertise-client-urls=https://{{ ansible_default_ipv4.address }}:2379 \
       --initial-cluster-token=kubernetes-etcd-cluster \
       --initial-cluster={{ etcd_cluster }} \
       --initial-cluster-state=new \
       --data-dir=/data/etcd
       Restart=on-failure
       RestartSec=5
       LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```

ansible  文件

```yaml
# hosts 文件
[etcd]
10.39.68.22
10.39.68.23
10.39.68.24



[etcd:vars]
ansible_ssh_pass="enN12345"
ansible_ssh_user="root"
etcd_cluster="10.39.68.22=https://10.39.68.22:2380,10.39.68.23=https://10.39.68.23:2380,10.39.68.24=https://10.39.68.24:2380"

[master:vars]		 etcd_server="https://10.39.68.22:2379,https://10.39.68.23:2379,https://10.39.68.24:2379"



#etcd yaml 文件
---
- hosts: etcd
  tasks:
    - name: copy etcd.service
      template:
        src: etcd.service
        dest: /etc/systemd/system/
        
# 执行命令
ansible-playbook -i hosts etcd.yaml

# 修改etcd 文件权限
ansible -i hosts etcd -m shell -a "chmod +x /usr/bin/etcd;chown -R etcd:etcd /usr/bin/etcd;chmod +x /usr/bin/etcdctl"

# 打开最大连接数
 ansible -i hosts etcd -m shell -a "ulimit 65535 "
 
# 创建etcd 数据存储位置
ansible -i hosts etcd -m shell -a "mkdir /data/etcd -p"

#启动etcd 
ansible -i hosts etcd -m shell -a "systemctl restart etcd " 
```

#### 6. etcd 集群的测试

vim etcdstatus.sh

```bash
# v3 和v2 的命令有所不同，查看集群状态就两条命令 endpoint health  endpoint status
etcdctl --endpoints=https://10.39.68.22:2379,https://10.39.68.23:2379,https://10.39.68.24:2379\
      --cert=/etc/kubernetes/ssl/etcd.pem \
     --cacert=/etc/kubernetes/ssl/ca.pem \
     --key=/etc/kubernetes/ssl/etcd-key.pem \
      endpoint health
```

![image-20200924140214556](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-24-060215.png)

