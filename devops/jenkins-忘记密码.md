---
title: "Jenkins 忘记密码"
date: 2020-08-27T15:57:35+08:00
draft: false  
categories: [devops]
tags: [devops]

---

jenkins 忘记密码

<!--more-->

找到config.xml 文件并且备份

find / -name config.xml  

```bash
/var/lib/jenkins/config.xml  
cp /var/lib/jenkins/config.xml   /var/lib/jenkins/config.xml.bak 
```

vim  /var/lib/jenkins/config.xml 

删除以下安全模块

```bash
<useSecurity>true</useSecurity>
  <authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy">
    <denyAnonymousReadAccess>true</denyAnonymousReadAccess>
  </authorizationStrategy>
  <securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
    <disableSignup>true</disableSignup>
    <enableCaptcha>false</enableCaptcha>
  </securityRealm>
```



1. 重启systemctl  restart jenkins  

2. 登录jenins 选择系统管理，打开全局安全设置

![image-20200827160649353](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-080649.png)

​      选择jenkins 专有用户数据库，点击保存



3. 返回界面，选择管理用户，就会看到用户列表，在admin 右边点击进去

![image-20200827160941338](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-080941.png)

4. 这样就会在页面看到登录，点击登录

   ![image-20200827161055675](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-08-27-081056.png)

即可登录jenkins 