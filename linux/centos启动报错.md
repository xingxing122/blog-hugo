---
title: "Centos启动报错"
date: 2019-12-18T16:11:32+08:00
draft: false  
categories: [linux]
tags: [linux]

---

es 集群今天突然有一台机器，重启之后启动不起来了

<!--more-->

![image-20191218162514782](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-083853.png)

出现了图中的问题，这个问题看起来好像是文件存储有问题，尝试解决一下 

continnue 中输入机器的密码 

![image-20191218161434402](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-081434.png)

输入完密码进入到了终端 

在卸载模式下进行xfs_repair /dev/vdc 

![image-20191218162243529](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-082244.png)



出现此类问题，需要检查一下/etc/fstab 文件是否正常，另外就是检查和修复文件系统 











