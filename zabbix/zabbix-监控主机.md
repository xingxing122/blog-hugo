---
title: "Zabbix 监控主机"
date: 2019-07-25T00:07:55+08:00
draft: false  
categories: [monitor]
tags: [monitor]

---

zabbix 监控主机的CPU、进程、内存、磁盘  

<!--more-->

1. 安装zabbix_agent 

   ```bash
   yum install https://repo.zabbix.com/zabbix/4.0/rhel/7/x86_64/zabbix-agent-4.0.2-1.el7.x86_64.rpm  -y  
   zabbix_agent.conf 是最zabbix_agent 的配置参数文件
   egrep -v "(^#|^$)" /etc/zabbix/zabbix_agentd.conf
   PidFile=/var/run/zabbix/zabbix_agentd.pid   # pid 文件路径    
   LogFile=/var/log/zabbix/zabbix_agentd.log   #日志文件路径
   LogFileSize=0    #日志切割大小，0 表示不切割
   Server=127.0.0.1   #被动模式，zabbix-server 的ip 地址
   ServerActive=127.0.0.1 #主动模式，zabbix-server 的ip 地址 
   Hostname=Zabbix server  #主机Hostname，使用主动模式则必须配置
   Include=/etc/zabbix/zabbix_agentd.d/*.conf   #启动特殊字符，用于自定义监控
   
   参数含义如下:
   server: 被动模式，允许zabbix_server 服务器连接客户端，此处允许本机访问10050的端口，多个IP之间用逗号分隔。
   ServerActive: 主动模式，向目标zabbix_server 传送数据，这种模式性能较好，建议使用，但需要确保zabbix_agent.conf 的参数Hostname 值与zabbix-web 页面的主机名一致，否则就会报错
   ```

   配置如下
   
   ```bash
   PidFile=/var/run/zabbix/zabbix_agentd.pid
   LogFile=/var/log/zabbix/zabbix_agentd.log
   LogFileSize=0
   Server=zabbix server 主机
   ServerActive=zabbix server 主机
   Hostname=agent 主机
   Include=/etc/zabbix/zabbix_agentd.d/*.conf
   ```
   
   ```bash
   启动zabbix-agent 
   systemctl  restart zabbix-agent
   
   ```
   
   在zabbix-server 测试
   
   ```bash
   zabbix_get -s agent-ip -p  10050 -k "system.uptime"
   ```

2. 监控指标

   监控首先要搞清楚需要监控哪些指标

   |      |      |      |
   | ---- | ---- | ---- |
   |      |      |      |
   |      |      |      |
   |      |      |      |
   
   
   
3. 

4. 

   
