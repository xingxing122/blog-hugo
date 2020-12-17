---
title: "Zabbix 监控web"
date: 2019-07-25T00:07:49+08:00
draft: false  
categories: [monitor]
tags: [monitor]

---



zabbix配置群组监控web，因为需要监控的业务线比较多，打算在zabbix监控web这块分一下组

<!--more-->

网络监控

需要监控一个网站点gitlab.****.cn，创建一个主机群组，网络组



![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070223.jpg)



添加主机

![image-20190805130547634](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-050548.png)







这里的IP地址写的是gitlab.**.cn解析的地址，如何获得试用nslookup gitlab.**.cn获得

选择模板，根据需要选择HTTPS或者HTTP的模板

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070316.jpg)



打开主机，创建网络场景

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070337.jpg)



场景

![image-20190805130905370](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-050912.png)





步骤

在步骤里面添加需要监控的URL，在步骤里面添加名称，超时时间，还有URL状态码

![image-20190805131001259](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-051004.png)



![image-20190805131124434](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-051128.png)

#### 触发器

在这个主机里面创建触发器，模板中默认带了一个触发器

![image-20190805131258263](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-051301.png)

![image-20190805131427875](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-051431.png)

添加了几个表达式，如果监控项请求的返回值不等于200就报错



### 报警

[报警文档](https://www.zabbix.com/documentation/4.0/zh/manual/config/notifications/media/script)

创建媒体类型

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070827.jpg)



创建媒介类型

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070856.jpg)

这里的脚本名称，需要把脚本放在AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts  目录下，这个目录是在zabbix_server.conf中配置

登陆钉钉创建一个群组，

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070916.jpg)



找到群机器人，创建自定义机器人

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070936.jpg)



添加机器人

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-070956.jpg)



添加机器人

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071014.jpg)



机器人令牌



![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071030.jpg)

在/ usr / local / zabbix / share / zabbix / alertscripts编写web.sh脚本

```
   curl 'https://oapi.dingtalk.com/robot/send?access_token=**'  \
   -H 'Content-Type: application/json' \
   -d '
  {"msgtype": "text",
    "text": {
        "content": "web站点报警测试"
     }
  }'
```

执行脚本

bash web.sh

钉钉测试

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071055.jpg)



#### 报警媒介

创建报警媒介

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071114.jpg)



动作-添加动作

![image-20190805131723865](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-08-05-051726.png)





操作

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071155.jpg)



操作

```bash
发生: {TRIGGER.NAME}故障!
{
监控IP:{HOST.IP}
监控项:{HOST.NAME}
监控取值:{ITEM.LASTVALUE}
告警等级:{TRIGGER.SEVERITY}
当前状态:{TRIGGER.STATUS}
告警信息:{TRIGGER.NAME}
告警时间:{EVENT.DATE} {EVENT.TIME}
}  
```

恢复操作

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-071225.jpg)



恢复操作

```bash
{TRIGGER.NAME}已恢复!
{

恢复IP:{HOST.IP}
恢复项:{HOST.NAME}
监控取值:{ITEM.LASTVALUE}
告警等级:{TRIGGER.SEVERITY}
当前状态:{TRIGGER.STATUS}
告警信息:{TRIGGER.NAME}
告警时间:{EVENT.DATE} {EVENT.TIME}
恢复时间:{EVENT.RECOVERY.DATE} {EVENT.RECOVERY.TIME}
持续时间:{EVENT.AGE}
}    
```



钉钉报警的python脚本

cat dingding.py

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
import requests
import json
import sys
import os

headers = {'Content-Type': 'application/json;charset=utf-8'}
api_url = "https://oapi.dingtalk.com/robot/send?access_token=******"

def msg(text):
    json_text= {
     "msgtype": "text",
        "text": {
            "content": text
        },
    }
    print requests.post(api_url,json.dumps(json_text),headers=headers).content

if __name__ == '__main__':
    text = sys.argv[1]
    msg(text)
```

