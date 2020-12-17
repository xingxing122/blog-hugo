---
title: "Zookeeper集群"
date: 2019-08-06T09:37:59+08:00
draft: false  
categories: [DB]
tags: [DB]

---

1. 什么是zookeeper？

<!--more-->

- ```
  ZooKeeper 是一种集中式服务，用户维护配置信息，命名，提供分布式同步和提供组服务。所有这些类型的服务都以分布式应用程序某种形式使用。分布式应用程序可以基于Zookpeer 实现诸如数据发布/订阅、负载均衡、命名空间、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。 
  ```



2. Zookeeper 特点

   ```shell
   顺序一致性: 从同一客户端发起的事务请求，最终将会严格地按照被顺序被应用到zookeeper中去。
   原子性: 所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群中所有机器都成功应用了某一个事务，要么都没有应用
   单一系统映像: 无论客户端连到哪一个zookeeper 服务器上，其看到的服务器数据模型都是一致的
   可靠性: 一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖
   ```

3. ZooKeeper 集群角色介绍

   Leader、Follower、Observer 三种角色

   ![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-06-022100.jpg)

4.  集群部署

   wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.5.5/apache-zookeeper-3.5.5-bin.tar.gz 

   解压完毕之后，配置zookeeper配置文件
   
   cat zoo.cfg
   ```shell
   tickTime=2000   #单个tick的长度，用于调节心跳和超时
   initLimit=10
   syncLimit=5    #zookeeper同步的时间量
   dataDir=/data/zookeeper   
   clientPort=2181   #侦听客户端连接的端口，客户连接使用
   maxClientCnxns=60  #限制ip地址表示的单个客户端可能对zookeeper集合的单个成员进行并发连接数，用于防止DoS攻击，包括文件描述符耗尽，默认值为60，0为不限制
   server.1=node1-ip:2888:3888
   server.2=node2-ip:2888:3888
   server.3=node3-ip:2888:3888
   ```
   
   更多参数可以参考官网
   
   https://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_configuration
   
5. 容错

    

   | 集群台数 | 允许宕机台数 | 备注 |
   | -------- | ------------ | ---- |
   | 3台      | 1台机器宕机  |      |
   | 5        | 2            |      |






