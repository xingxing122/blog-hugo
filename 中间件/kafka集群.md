---
title: "Kafka集群"
date: 2019-08-01T14:26:08+08:00
draft: false  
categories: [DB]
tags: [DB]

---

kafka 集群安装

<!--more-->

1. 安装zookeeper  

   wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz 

   tickTime=2000
   initLimit=10
   syncLimit=5
   dataDir=/data/zookeeper
   dataLogDir=/var/lib/log
   clientPort=2181
   server.1=10.39.33.38:2888:3888
   server.2=10.39.33.50:2888:3888
   server.3=10.39.33.51:2888:3888

   

