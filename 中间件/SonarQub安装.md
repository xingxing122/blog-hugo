---
title: "SonarQub安装"
date: 2019-04-02T10:56:02+08:00
draft: false  
categories: [devops]
tags: [devops]

---

SonarQube 部署安装

<!--more-->


1. 安装jdk   

   ```bash
   su -c "yum install java-1.8.0-openjdk"  
   ```

   

2. 创建mysql 数据库

   ```mysql
     mysql> CREATE DATABASE sonar CHARACTER SET utf8 COLLATE utf8_general_ci; 
   	mysql> CREATE USER 'sonar' IDENTIFIED BY 'WN%yT$z9D3';
   	mysql> GRANT ALL ON sonar.* TO 'sonar'@'%' IDENTIFIED BY 'sonar';
   	mysql> GRANT ALL ON sonar.* TO 'sonar'@'localhost' IDENTIFIED BY 'sonar';
   	mysql> FLUSH PRIVILEGES;   
   ```

   ​	

3. 