---
title: "Kubernetes V1.14 etcd 集群安装(二)"
date: 2019-03-27T10:52:20+08:00
draft: false  
categories: ["kubernetes"]
tags: ["kubernetes"]

---

etcd 集群安装

<!--more-->

1. 机器规划

   ```bash
   [etcd]
   10.39.30.116 
   10.39.30.117
   10.39.30.118 
   ```

   

2.  制作证书 

   创建CA 证书 

   mkdir  ssl 

   cd ssl

   vim config.json    

   ```yaml
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
   ```

   生成CA 证书和私钥

   ```bash
   [root@devops ssl]# cfssl gencert -initca csr.json | cfssljson  -bare ca
   [root@devops ssl]# ll
   total 20
   -rw-r--r-- 1 root root 1001 Mar 27 11:02 ca.csr
   -rw------- 1 root root 1679 Mar 27 11:02 ca-key.pem
   -rw-r--r-- 1 root root 1359 Mar 27 11:02 ca.pem
   -rw-r--r-- 1 root root  361 Mar 27 10:59 config.json
   -rw-r--r-- 1 root root  273 Mar 27 11:00 csr.json
   ```

   创建etcd 证书 

    vim etcd-csr.json 

   ```yaml
   {
     "CN": "etcd",
     "hosts": [
       "127.0.0.1",
       "10.39.30.116",
       "10.39.30.117",
       "10.39.30.118"
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

   生成etcd 秘钥

   ```bash
   [root@devops ssl]# cfssl gencert -ca=ca.pem  -ca-key=ca-key.pem  -config=config.json  -profile=kubernetes etcd-csr.json  | cfssljson  -bare etcd
   
   [root@devops ssl]# ll
   total 36
   -rw-r--r-- 1 root root 1001 Mar 27 11:02 ca.csr
   -rw------- 1 root root 1679 Mar 27 11:02 ca-key.pem
   -rw-r--r-- 1 root root 1359 Mar 27 11:02 ca.pem
   -rw-r--r-- 1 root root  361 Mar 27 10:59 config.json
   -rw-r--r-- 1 root root  273 Mar 27 11:00 csr.json
   -rw-r--r-- 1 root root 1062 Mar 27 11:24 etcd.csr
   -rw-r--r-- 1 root root  376 Mar 27 11:09 etcd-csr.json
   -rw------- 1 root root 1675 Mar 27 11:24 etcd-key.pem
   -rw-r--r-- 1 root root 1436 Mar 27 11:24 etcd.pem
   ```

   检查证书 

   ```bash
   [root@devops ssl]# cfssl-certinfo -cert etcd.pem 
   ```

   

   拷贝证书到etcd 服务器上

   ```bash
   [root@devops k8s]# ansible -i hosts etcd -m copy -a 'src=./ssl/  dest=/etc/kubernetes/ssl/' -k
   ```

   验证: 

   ```bash
   [root@devops k8s]# ansible -i hosts etcd -m shell  -a 'ls -l /etc/kubernetes/ssl/' -k 
   ```

   拷贝etcd 二进制到etcd 服务器上

   ```bash
   tar zxvf etcd-v3.2.18-linux-amd64.tar.gz 
   ansible -i hosts etcd -m copy -a "src=./etcd-v3.3.11-linux-amd64/etcdctl  dest=/usr/bin/"  -k
   ansible -i hosts etcd -m copy -a "src=./etcd-v3.3.11-linux-amd64/etcd  dest=/usr/bin/"  -k
   ```

   非root用户，提示证书会没有权限

   ```bash
   ansible -i hosts etcd -m shell -a "chmod 644 /etc/kubernetes/ssl/etcd*" 
   ansible -i hosts etcd -m shell -a "useradd -s /sbin/nologin etcd;mkdir -p /opt/etcd;chown -R etcd:etcd /opt/etcd
   ```

   非root用户，提示证书会没有权限

   ```bash
   ansible -i hosts etcd -m shell -a "chmod 644 /etc/kubernetes/ssl/etcd*" 
   ansible -i hosts etcd -m shell -a "useradd -s /sbin/nologin etcd;mkdir -p /opt/etcd;chown -R etcd:etcd /opt/etcd
   ```

   etcd 部署
   这块使用ansible 进行安装  

   vim  hosts  

   ```bash
   [etcd]
   10.39.30.116
   10.39.30.117
   10.39.30.118
   [master:vars]		 etcd_server="https://10.39.30.116:2379,https://10.39.30.117:2379,https://10.39.30.118:2379"
   [etcd:vars]		 etcd_cluster="10.39.30.116=https://10.39.30.116:2380,10.39.30.117=https://10.39.30.117:2380,10.39.30.118=https://10.39.30.118:2380"
   ```

   vim  templates/etcd.service

   ```bash
   [Unit]
   Description=Etcd Server
   After=network.target
   After=network-online.target
   Wants=network-online.target
   	
   [Service]
   Type=notify
   WorkingDirectory=/opt/etcd
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
   		  --data-dir=/opt/etcd
   		  Restart=on-failure
   		  RestartSec=5
   		  LimitNOFILE=65536
   	
   [Install]
   WantedBy=multi-user.target
   ```

   

   执行ansible 进行安装

   ```bash
   cat etcd.yaml
   
   - hosts: etcd
     tasks:
       -template:
          src: etcd.service
          dest: /etc/systemd/system/
     ansible-playbook -i hosts  etcd.yaml
   ```

   

   修改二进制etcd 文件的权限

   ```bash
   ansible -i hosts etcd -m shell -a "chmod +x /usr/bin/etcd"
   ansible -i hosts etcd -m shell -a "chown -R etcd:etcd /usr/bin/etcd "
   ansible -i hosts etcd -m shell -a "chmod +x /usr/bin/etcdctl"
   ```

   检查   

   ```bash
   cat /opt/etcdstatus.sh	  
   etcdctl --endpoints=https://10.39.30.116:2379,https://10.39.30.117:2379,https://10.39.30.118:2379\
         --cert-file=/etc/kubernetes/ssl/etcd.pem \
       	--ca-file=/etc/kubernetes/ssl/ca.pem \
       	--key-file=/etc/kubernetes/ssl/etcd-key.pem \
         cluster-health	
   ```

   查看

   ```bash
   [root@k8s-116 ~]# sh /opt/etcdstatus.sh
   member 3892f17d530fe3f4 is healthy: got healthy result from https://10.39.30.116:2379
   member 50e8d30f007eab7b is healthy: got healthy result from https://10.39.30.117:2379
   member c7847f34515cfd48 is healthy: got healthy result from https://10.39.30.118:2379
   cluster is healthy
   ```

















