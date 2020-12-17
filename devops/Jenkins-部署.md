---
title: "Jenkins 部署"
date: 2020-08-03T16:31:16+08:00
draft: false  
categories: [devops]
tags: [devops]

---

Jenkins 是一个自包含的开源自动化服务器，可用于自动化与构建，测试以及交付或部署软件有关的各种任务。

<!--more-->

1. 安装

   [](https://www.jenkins.io/doc/book/installing/)

   ```bash
   apt-get install openjdk-8-jre
   wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > \
       /etc/apt/sources.list.d/jenkins.list'
   sudo apt-get update
   sudo apt-get install jenkin
   ```

2. 配置jenkins 

   ![image-20200803173237277](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-03-093237.png)

3. 选择自定义插件，先不选择插件进行安装

   ![image-20200803173720483](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-03-093721.png)

4. 进入jenkins 

   ![image-20200803181505751](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-03-101506.png)

5. 修改源为中文社区的源

   系统管理—插件管理—高级 

   https://jenkins-zh.gitee.io/update-center-mirror/tsinghua/update-center.json

   ![image-20200804093731701](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-04-013731.png)

​      修改完毕，点击提交即可

6. 安装插件

   搜索插件pipline 

   ![image-20200804095551463](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-04-015552.png)

7. 添加slave 节点

   ![image-20200804151926598](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-04-071926.png)

     将会看到如下界面   

     ![image-20200804152626439](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-04-072626.png)

```bash
下载agent.jar 到对应的slave 节点，将mv agent.jar /opt/jenkins/ 目录下 
创建/opt/jenkins 目录  
# 注意 slave 节点需要安装jdk环境
使用脚本启动
cat start.sh 
```

  

```bash
#!/bin/bash 
nohup  java -jar agent.jar -jnlpUrl http://10.39.12.9:8080/computer/node/slave-agent.jnlp -secret 58c3f623072f0cb0cf6022974a1c07d7bfec690479bbfdd9b38f5244cb0cbb62 -workDir "/opt/jenkins"  & 
```



   8. slave 节点添加完成 

      ![image-20200804155358738](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-04-075358.png)

​     



