---
title: "Git提交代码报错"
date: 2019-08-13T15:06:38+08:00
draft: false  
categories: [devops]
tags: [devops]

---

git 代码提交报错

<!--more-->

开发人员告知提交不了代码，我测试了一下，报错如下，摘取了部分

error: unpack failed: unable to create temporary object directory 

根据报错，查看是否是因为磁盘空间满了或者权限问题导致，最后查看git 目录确认是目前权限导致引起的报错，修改目录权限之后即可提交



