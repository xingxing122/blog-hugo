



<!--more-->

### 1. 基础

```bash
kubectl create deployment web --image=nginx 

#使用service 将pod暴露出去
kubectl  expose deployment  web --port=80  --type=NodePort  --target-port=80 --name=web 

#访问应用
http://NodeIP:Port  #端口随机生成，通过get svc获取

```







