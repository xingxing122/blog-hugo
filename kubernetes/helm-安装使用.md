---

title: "Helm 安装使用"
date: 2019-04-16T17:57:23+08:00
draft: false  
categories: [kubernetes]
tags: [kubernetes]

---

helm 安装使用

<!--more-->

helm 可帮助管理kubernetes 应用程序，helm  charts 可帮助定义，安装和升级最复杂的kubernetes 应用

1.  [下载helm](https://github.com/helm/helm/releases)

   ```bash
   wget https://github.com/helm/helm/archive/v2.13.1.tar.gz
   tar -zxvf helm-v2.13.1-linux-amd64.tar.gz 
   mv linux-amd64/helm  /usr/local/bin/helm
   helm version
   Client: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
   Error: could not find tiller
   ```


2. 服务器端安装

   ```bash
   helm init   或者helm init --upgrade  --tiller-image   *****  habror.enncloud.cn/enncloud/tiller:v2.13.1
   $HELM_HOME has been configured at /root/.helm.
   
   Tiller (the Helm server-side component) has been upgraded to the current version.
   Happy Helming	
   ```

3. 查看

   ```bash
   kubectl get pod -n kube-system
   tiller-deploy-54cc7bf8d7-gtvsn             1/1       Running   0          30m
   helm version
   Client: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
   Server: &version.Version{SemVer:"v2.13.1", GitCommit:"618447cbf203d147601b4b9bd7f8c37a5d39fbb4", GitTreeState:"clean"}
   ```

4. helm 绑定k8s 集群admin 权限

   ```yaml
   vim  helm-rabc.yaml
   apiVersion: v1
   kind: ServiceAccount
   metadata:
     name: tiller
     namespace: kube-system
   ---
   apiVersion: rbac.authorization.k8s.io/v1beta1
   kind: ClusterRoleBinding
   metadata:
     name: tiller
   roleRef:
     apiGroup: rbac.authorization.k8s.io
     kind: ClusterRole
     name: cluster-admin
   subjects:
   - name: tiller
     kind: ServiceAccount
     namespace: kube-system
   ```

   创建权限

   ```bash
   kubectl create  -f helm-rabc.yaml
   serviceaccount/tiller created
   clusterrolebinding.rbac.authorization.k8s.io/tiller created
   kubectl patch deploy  --namespace  kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
   ```
5. 使用helm 
    helm 库

   ```shell
   helm repo list
   NAME  	URL
   stable	https://kubernetes-charts.storage.googleapis.com
   local 	http://127.0.0.1:8879/charts!
   
   创建一个nginx 访问
   helm  create hello-helm
   查看一下目录结构
   ```

​       ![](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-05-21-071406.png)

6. 创建nginx 来访问

   ```shell
   修改values.yaml 中的ClusterIP 为NodePort
   helm  install ./hello-helm
   ```

 

​       <img src="https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-05-21-072351.png" alt=" " style="zoom:30%;" />   

​       访问测试 

​       nodeIP:Port   

​        