---
title: "Mac升级之后UltiSnips插件出问题了"
date: 2019-11-21T21:31:37+08:00
draft: false  
categories: [linux]
tags: [linux]

---



<!--more-->





mac  升级到最新版本(macOS  Catalina )之后vim 不能使用了，打开vim 之后报错

![image-20191121214621010](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-11-21-134624.png)

解决

```bash
brew unlink macvim 
brew install vim
```

