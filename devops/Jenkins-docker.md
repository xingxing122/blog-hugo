---
title: "Jenkins Docker"
date: 2020-08-05T16:59:18+08:00
draft: false  
categories: [devops]
tags: [devops]

---

Jenkins 使用docker 安装

<!--more-->

jenkins 容器安装 

```bash
docker  run --name   jenkins  -itd \
     -p 8081:8080 \
     -p 50000:50000 \ 
     -v ~/jenkins:/var/jenkins_home \ 
     jenkins/jenkins:lts   
docker logs  -f jenkins  #查看日志并获取初始化密码
```

