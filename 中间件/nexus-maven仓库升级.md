---
title: "Nexus Maven仓库升级"
date: 2020-02-05T10:34:10+08:00
draft: false  
categories: [devops]
tags: [devops]

---

nexus 2.14 升级到3.20版本

<!--more-->

nexus 2.x 升级到2.x 版本的话，只需要将sonatype-work 目录同步下，同步之后密码将是老的nexus 的密码，而不是最新的nexus2.x 的密码

nexus 2.x   版本升级到3.x 版本  

需要将nexus 2.x 升级到指定的指定升级版本才可以升级到nexus 3.x 版本

https://help.sonatype.com/repomanager3/upgrading?_ga=2.196178220.1873667372.1580866793-823368280.1575427960#Upgrading-Upgrading2.xto3.y



下载版本

https://help.sonatype.com/repomanager3/download/download-archives---repository-manager-3

https://help.sonatype.com/repomanager2/download/download-archives---repository-manager-2

目前将nexus 升级到2.14.16  

![image-20200206103809140](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-023809.png)

![image-20200206104031620](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-024031.png)

access token   123456 简单点

创建完成

![image-20200206104145090](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-024145.png)

在nexus-3.20 版本上登录

选择设置-升级

![image-20200327235504559](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-03-27-155505.png)



![image-20200327235641837](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-03-27-155642.png)

![image-20200331003043728](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-03-30-163043.png)





![image-20200206104731528](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-024731.png)

选择需要升级的repo 

![image-20200206104810902](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-024810.png)

选择

![image-20200206104838916](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-024839.png)



在升级测试发现，2.14.16.01升级到最新版本是没有问题的3.20.1 版本是ok的 

![image-20200206155624518](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-02-06-075624.png)

等升级完成即可 

