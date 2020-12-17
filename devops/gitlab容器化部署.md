---
title: "Gitlab容器化部署"
date: 2019-08-07T21:38:01+08:00
draft: false  
categories: [Name]
tags: [Name]

---

gitlab 容器化部署

<!--more-->

```bash
docker run -d \
    -p 80:80 \
    -p 443:443 \
    -p 22:22 \
    --name gitlab \
    --restart unless-stopped \
    -v /data/gitlab/config:/etc/gitlab \
    -v /data/gitlab/logs:/var/log/gitlab \
    -v /data/gitlab/data:/var/opt/gitlab \
    gitlab-ce:10.7.3-ce
```

