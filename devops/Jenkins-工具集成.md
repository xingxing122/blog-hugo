---
title: “ jenkins-工具集成 ”
date: 2020-09-01T16:30:14+08:00
draft: false  
categories: [devops]
tags: [devops]

---

maven  npm 等jenkins 工具的集成

<!--more-->

CI 工具构建

安装maven https://maven.apache.org/ 

```bash
maven 解压之后
vim  /etc/profile 
export M2_HOME=/opt/apache-maven-3.6.3
export PATH=${M2_HOME}/bin:$PATH

source /etc/profile
mvn -v
```

 jenkins—全局工具配置 

![image-20200901175535940](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-01-095536.png)



就可以使用mvn构建了



CD 工具













