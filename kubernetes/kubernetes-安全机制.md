---
title: "kubernetes 安全机制"
date: 2020-09-09T14:50:05+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

kubernetes 安全机制

<!--more-->

API server 的授权策略(通过API server的启动参数 "--authorization-node"  设置)

```bash
AlwayDeny 表示拒绝所有请求，一般用于测试
AlwayAllow 允许接收所有请求，kubernetes 默认配置
ABAC(Attribute-Based Access Control): 基于属性的访问控制，定义了一种访问控制的范例，通过使用将属性组合在一起的策略，将访问权限授予用户。策略可以使用任何类型的属性(用户属性，资源属性，对象环境属性等)
RBAC: Role-Based Access Control 基于角色的访问控制是一种企业内个人用户的角色来管理对计算机或网络资源的访问的方法。 
Node: 一种专用模式，根据计划运行的pod 为kubelet 授予权限
Webhook: 是一个HTTP的回调，发生某些事情时调用的HTTP POST；通过HTTP POST 进行简单的事件通知。实现WebHook 的web 应用程序在发生某些事情时将消息发布到URL。

```

kubernetes 权限请求过程

```bash
kubernetes 使用API 服务器授权API 请求，它根据所有策略评估所有请求属性来决定允许或拒绝请求。一个API请求的所有部分必须被某些策略允许才能继续。默认情况下拒绝权限。 

配置多个授权模式时，将按顺序检查每个模块。如果任何授权模块批准或拒绝请求，则立即返回该决定，并且不会与其他授权模块协商。如果所有模块对请求没有意见，则拒绝请求。一个拒绝响应返回HTTP状态代码403。 
```

kubernetes的权限属性

```
user    身份验证期间提供的user 字符串
group   经过身份验证的用户所属的组名列表
extra   由身份验证层提供的任意字符串键到字符串值的映射
API     指示请求是否针对API 资源 
Request path  各种非资源端点的路径， 如/api 
API request verb  API动词  get  list  create  update  patch 等
HTTP request verb    HTTP动词get post put 和delete 用于非资源请求
Resource  正在访问的资源ID或名称(仅限资源请求)  对于使用get update 
```

![image-20200914212534398](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-14-132534.png)



ABAC 

![image-20200914212906681](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-09-14-132906.png)



