---
title: "Ansible 使用"
date: 2019-07-08T14:26:46+08:00
draft: false  
categories: [devops]
tags: [devops]

---

# **ansible 日常使用**

<!--more-->

### 运维管理工具比较

| **工具**  | **语言** | **架构**        | **协议**     |
| --------- | -------- | --------------- | ------------ |
| puppet    | ruby     | c/s             | http         |
| chef      | ruby     | c/s             | http         |
| ansible   | python   | 无client        | ssh          |
| saltstack | python   | c/s(可无client) | SSH/ZMQ/RAET |
| fabric    | python   |                 | ssh          |

### ansible 使用

![image-20191218221356777](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-145313.png)

```bash
ansible 优缺点

优点: 
  SSH 高安全性；
  不需要在远程节点上安装任何代理；
  使用YAML 学习简单，Playbook 结构简单；
  简便 ，易用；
  顺序执行顺序；
  
缺点：
  不需要代理，但是需要 SSH 访问和python 解释器

```

#### 工作机制

![image-20191218221612694](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-145321.png)

```bash
Ansible playbooks: 任务剧本，编排定义ansible 任务集的配置文件，由ansible顺序依次执行，通常是json格式的yaml文件。
INVENTORY: ansible 管理主机的清单
MODULES: ansible 执行命令的功能模块，多数为内置的核心模块，也可自定义
PLUGINS: 模块功能的补充，如连续类型插件、循环插件、变量插件、过滤插件等
API: 供第三方程序调用的应用程序编程接口
```

Ansible 命令：

```bash
Ansible 
Ansible-galaxy
Ansible-pull 
Ansible-doc
Ansible-playbook
Ansible-vault
Ansible-console
```

Ansible 执行方式：

```bash
Ad-Hoc(命令模式ansible)
Ansible-playbook
web(收费)
```

Ad-Hoc 命令集由/usr/bin/ansible 实现，其命令用法如下: 

```
Ansible <host-pattern>  [options]
ansible    -i   hosts   kafka    -m   ping 
# 检查服务器的存活
```

![image-2019121822212854](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-145325.png)

 



```bash
ansible 返回结果都非常友好，一般会用3种颜色来表示执行结果: 红色、绿色、
橘黄色。其中红色表示执行过程有异常，一般会中止剩余所有的任务，绿色和、
橘黄色表示执行过程没有异常，橘黄色表示执行结束后由状态的变化
```

**命令模式(ansible)**

```bash
ansible -i hosts k8s -m shell -a “hostname”    
 解释
    -i   指定hosts 文件地址，默认/etc/ansible/ 目录下
    -m  需要执行的模块名称
    -a   需要执行的模块参数
 ansible inventory 配置及详解
     inventory 是ansible 管理主机信息的配置文件，默认存放在/etc/ansible/hosts,  
     inventory 是可以定义变量的，只不过这些变量只能在ansible-ploybook 中使用，而ansible 是不支持的 
```

![image-20191218222418656](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-142418.png)

ansible-doc 是ansible 模块文档说明

```bash
Ansible-doc [options]  [module…….]
Ansible-doc  -l   列出支持的模块
Ansible-doc  ping 模块功能说明    
```

### ansible 命令执行 

<img src="https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-12-18-145330.png" alt="image-20191218222514589" style="width:80%;">





#### 命令常用模块

Ansible 命令常用模块：

```bash
1. copy  实现文件复制和批量文件下发
    ansible -i hosts master -m copy -a “src=etcd.yaml  dest=/opt” -k
2. yum 安装和卸载软件包
    ansible -i hosts master -m yum -a “name=httpd”
3. ping 模块 
    ansible –i  hosts all -m ping or ansible -I hosts 192.168.1.* -m ping
4. setup 查看主机信息
    ansible -i hosts master -m setup
5. Command 执行系统命令
    ansible -i hosts master -m  command -a “df -h ”
6. Shell 
    ansible -i hosts master -m  shell  -a “df -h ”
7. Script 执行脚本
    ansible -i hosts master -m  script –a “bash /opt/script.sh”
```

