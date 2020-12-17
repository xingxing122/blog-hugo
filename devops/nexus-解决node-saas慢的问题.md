---
title: "Nexus 解决node Saas慢的问题"
date: 2020-08-27T09:53:52+08:00
draft: false  
categories: [devops]
tags: [devops]

---

nexus  解决node-saas 依赖慢的问题

<!--more-->

在nexus 上创建npm私库

[https://help.sonatype.com/repomanager3/formats/npm-registry]:https://help.sonatype.com/repomanager3/formats/npm-registry

创建一个本地的repository  

允许上传

![image-20200827102832919](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-022833.png)

创建proxy repository  

![image-20200827103948103](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-023948.png)

创建group  

![image-20200827104141082](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-024141.png)

当我们设置这些之后，还会发现node 程序在构建的，如果没有设置淘宝源的话还是会去淘宝或者github 上去下载文件

```bash
Downloading binary from https://npm.taobao.org/mirrors/node-sass/v4.14.1/linux-x64-64_binding.node
Download complete
```

每次下载从外网去下载会比较麻烦，也无法进行缓存，导致了构建缓慢，在nexus 如何解决呢

在nexus 中创建一个repository  类型选为raw(proxy)

![image-20200827105524230](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-031113.png)

![image-20200827105445838](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-025446.png)

```
注意，这里的proxy 的代理地址是淘宝源的node-saas 地址  https://npm.taobao.org/mirrors/node-sass/
```

在构建的项目的时候填写如下地址就好 

```bash
npm config set registry=http://nexus.XXX.com/repository/npm/
npm config set sass-binary-site  http://nexus.XXX.cn/repository/node-saas/
```

在去构建的时候，会发现nexus 会缓存这些包 



