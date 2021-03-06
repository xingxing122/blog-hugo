---
title: "kubernetes 安全机制"
date: 2020-09-09T14:50:05+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

#### kubernetes 安全机制

<!--more-->

#### 授权策略

API server 的授权策略(通过API server的启动参数 "--authorization-node"  设置)

```bash
AlwayDeny 表示拒绝所有请求，一般用于测试
AlwayAllow 允许接收所有请求，kubernetes 默认配置
ABAC(Attribute-Based Access Control): 基于属性的访问控制，定义了一种访问控制的范例，通过使用将属性组合在一起的策略，将访问权限授予用户。策略可以使用任何类型的属性(用户属性，资源属性，对象环境属性等)
RBAC: Role-Based Access Control 基于角色的访问控制是一种企业内个人用户的角色来管理对计算机或网络资源的访问的方法。 
Node: 一种专用模式，根据计划运行的pod 为kubelet 授予权限
Webhook: 是一个HTTP的回调，发生某些事情时调用的HTTP POST；通过HTTP POST 进行简单的事件通知。实现WebHook 的web 应用程序在发生某些事情时将消息发布到URL。

```

#### kubernetes 权限请求过程

```bash
kubernetes 使用API 服务器授权API 请求，它根据所有策略评估所有请求属性来决定允许或拒绝请求。一个API请求的所有部分必须被某些策略允许才能继续。默认情况下拒绝权限。 

配置多个授权模式时，将按顺序检查每个模块。如果任何授权模块批准或拒绝请求，则立即返回该决定，并且不会与其他授权模块协商。如果所有模块对请求没有意见，则拒绝请求。一个拒绝响应返回HTTP状态代码403。 
```

#### kubernetes的权限属性

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



#### RBAC权限

RBAC(Role-Based Access Control,基于角色的访问控制) 

RBAC 优势 

对集群中的资源和非资源权限均有完整的覆盖

整个RBAC 完全由几个API 对象完成，同其他API 对象一样，可以用kubectl 或API进行操作

可以在运行时进行调整，无须重新启动API Server 

要使用RBAC 授权模式，需要在API  Server  的启动参数中加上--authorization-mode=RBAC 

#### RBAC 资源对象

![image-20210311103222520](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-11-023223.png)

```bash
User Account: 用户，这是有外部独立服务进行管理的，管理员进行私钥的分配，用户可以使用KeyStone或者Goolge账号，甚至一个用户名和密码的文件列表也可以，对于用户的管理集群内部没有一个关联的资源对象，所以用户不能通过集群内部API来进行管理 
Group: 组，这是用来关联多个账号的，集群中有一些默认创建的组，比如cluster-admin
Service Account: 服务账号，通过kubernetes API 来管理一些用户账号，和namesapces 进行关联的，适用于集群内部运行的应用程序，需要通过API来完成权限认证，所以在集群内部进行权限操作，我们都需要用到ServiceAccount 
```

创建用户

```bash
创建用户test
#创建私钥
openssl genrsa -out test.key 2048
#
openssl req -new -key test.key  -out test.csr -subj "/CN=test/0=devops"

# 生成证书文件
openssl x509 -req -in test.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key  -CAcreateserial -out test.crt -days 500

#创建凭证
kubectl config set-credentials test --client-certificate=test.crt --client-key=test.key

#为用户设置新的context 
kubectl config set-context test-context --cluster=kubernetes --namespace=kube-system --user=test

使用test 访问
[root@k8s-master-03 Role]# kubectl  get pods --context=test-context
Error from server (Forbidden): pods is forbidden: User "test" cannot list resource "pods" in API group "" in the namespace "kube-system"
就需要创建角色权限
```

![image-20210312093651936](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-12-013652.png)

接下来给test 创建角色

##### 1) Role(角色)

一个角色就是一组权限的集合，这里的权限都是许可形式的，不存在拒绝的规则。在一个命名空间中，可以用角色来定义一个角色，如果是集群级别的，就需要使用ClusterRole了，

角色只能对命名空间内的资源进行授权，例如:  已经创建了test账号了，再创建test 的Role 

test.yaml 

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata: 
  namespace: kube-system   
  name: test 
rules:
- apiGroups: ["","app","extensions"]  #"" 空字符串，表示核心API 群
  resources: ["pods","deployments","replicasets"]
  verbs: ["get","watch","list","create","update","delete"]
  
```

Rules 中的参数说明如下:

apiGroup: 支持的API 组列表

resources: 支持的资源对象列表，例如 pods、 deployment、jobs 等

versb: 对资源对象的操作方法列表，例如 get、awtch、list、delete、replace、patch等 

![image-20210312095428490](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-12-015432.png)

创建role之后，需要test账号与 testRole 进行绑定，就需要创建Rolebinding

##### 2)Rolebinding(角色绑定)

角色绑定或集群绑定用来把一个角色绑定到一个目标上，绑定目标可以是User(用户)、Group(组)或者Service Account。使用RoleBinding为某个命名空间授权，使用ClusterRoleBinding 为集群范围内授权.

例子: 下面的例子中的RoleBinding 会将在前面创建的test 用户进行绑定

```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata: 
  name: test-rolebinding  # rolebinding 名称
  namespace: kube-system  
subjects:
- kind: User
  name: test      # name 是不区分大小写的
  apiGroup: ""
roleRef: 
# "roleRef" 指定与某Role 或ClusterRole 的绑定关系
  kind: Role     # 此字段必须是Role 或者CLusterRole 
  name: test     #  此字段要必须与你要绑定的Role 或者ClusterRole 名称匹配
  apiGroup: ”“
```

![image-20210312100935724](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-12-020938.png)

![image-20210312101039053](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-12-021041.png)

测试访问，test-context 指定的namespace 是kube-system 

![image-20210312101312926](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-12-021315.png)

注意: Role与RoleBinding 对象都是namespace 对象，它们对权限的限制规则仅在它们的namespace 内有效，roleRef 也只能引用当前namespace 里的role 对象

##### 3)ClusterRole(集群角色)

集群角色除了具有和角色一致的命名空间内资源的管理能力，因其集群级别的范围，还可以用于以下特殊元素的授权。

集群范围的资源，例如node 

非资源型路径，例如 ”/healthz“

包含全部命名空间的资源，例如pods(kubectl  get pods  --all-namespaces这样的操作授权)

##### 创建xing 用户

```bash
openssl genrsa -out xing.key 2048
openssl req -new -key xing.key  -out xing.csr -subj "/CN=xing/0=xing"
openssl x509 -req -in xing.csr -CA /etc/kubernetes/pki/ca.crt -CAkey /etc/kubernetes/pki/ca.key  -CAcreateserial -out xing.crt -days 500
kubectl config set-credentials xing --client-certificate=xing.crt --client-key=xing.key
kubectl config set-context xing-context --cluster=kubernetes --namespace=kube-system --user=xing
```

##### 创建Clusterrole

```yaml
cat cluster-xing.yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: xing-clusterrole
rules:

- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get","list","watch"]
```

##### 创建ClusterRoleBinding

```yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: xing-clusterrolebinding
subjects:
- kind: User
  name: xing
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: xing-clusterrole
  apiGroup: rbac.authorization.k8s.io
```

##### 测试

![image-20210317111232440](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-17-031232.png)

##### Service Account 授权管理

默认的RBAC 策略为控制平台组件、节点和控制器授予有限范围的权限，但是出kube-system 外的Service Account 是没有任何权限。 

这就要求用户为Service Account赋予所需的权限。细粒度的角色分配能够提高安全性，但也会提高管理成本。粗放的授权方式可能会给Service Account 多余的权限，但更易于管理。 

首先，需要定义一个Service Account 

##### 创建service account 

```yaml
#serviceaccount.yaml 
apiVersion: v1 
kind: ServiceAccount 
metadata:  
  namespace: kube-system  # serviceaccount 创建在了kube-system 下
  name: abc    # service account 名字
```

##### 创建cluster-role

```yaml
cluster-role.yaml 
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata: 
  name: abc
rules:
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get","list","watch"]
```

##### 创建ClusterRole-namesapce

编写cluster-server.yaml的yaml 文件，为ServiceAccount 分配权限: 

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cr-namespace-abc
rules: 
- apiGroups:
  - ""
  resources: 
  - namespaces/status
  - namespaces 
  verbs: 
  - get 
  - list  
  - watch 
```

```yaml
kubectl apply -f serviceaccount.yaml
kubectl apply -f cluster-role.yaml
kubectl apply -f cluster-server.yaml
```

##### 绑定角色

对sa和集群角色建立绑定关系

```yaml
kubectl create rolebinding abc  --clusterrole=abc --serviceaccount=kube-system:abc --namespace=kube-system
#--clusterrole=abc 名字与上面创建cluster-role 名字一致
#--serviceaccount=kube-system:abc  所在的namespace空间与serviceaccount名字
#--namespace=kube-system 授权的namespace 
```

##### 查看sa与

```yaml
 kubectl  get sa -n kube-system abc -o yaml
 #获取secrets 名字
 kubectl  get  secrets abc-token-fpkl2  -o yaml
 #获取ca与secrets的token 
 token= <token内容>
 echo $token|base64 -d  # 解码
```

##### 客户端config配置文件

config 文件在家目录的.kube/config 

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: https://xxxx:6443
    certificate-authority-data:  #ca认证
  name: abc
users:
- name: "abc"
  user:
    token:  # 这里的token 需要base64 解码
contexts:
- context:
    cluster: abc
    user: "abc"
  name: abc
preferences: {}
current-context: abc
```

![image-20210318145215863](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-18-065218.png)

##### 用户授权

创建用户cluster-sa 的serviceaccount 

```yaml
#cat sa.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  namespace: kube-system
  name: cluster-sa
```

##### 创建role

```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: kube-system
  name: cluster-role
   # 在这里定义role 的名字
rules:
- apiGroups: ["","app","extensions"]
  resources: ["pods","deployments","replicasets","configmaps"]
  verbs: ["get","watch","list","create","update","delete"]
```

##### 绑定roleBinding 

```yaml
cat role-binding.yaml

kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: cluster-rolebinding
  namespace: kube-system
subjects:
- kind: ServiceAccount
  name: cluster-sa
  namespace: kube-system
roleRef:
  kind: Role
  name: cluster-role
  apiGroup: rbac.authorization.k8s.io
```

##### 客户端配置与验证

```yaml
kubectl  get sa -n kube-system  cluster-sa -o yaml
#获取secrets 
 kubectl  get  secrets -n kube-system   cluster-sa-token-wxlwn -o yaml
 token=*****
 echo $token|base64  -d
```

##### config

```yaml
apiVersion: v1
kind: Config
clusters:
- cluster:
    server: https://xxxxx:6443
    certificate-authority-data: # ca证书文件
  name: cluster-sa
users:
- name: "cluster-sa"
  user:
    token: #解压后的token
contexts:
- context:
    cluster: cluster-sa
    user: "cluster-sa"
  name: cluster-sa
preferences: {}
current-context: cluster-sa
```

![image-20210318161146705](https://xing-blog.oss-cn-beijing.aliyuncs.com/2021-03-18-081157.png)