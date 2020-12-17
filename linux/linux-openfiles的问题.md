---
title: "Linux Openfiles的问题"
date: 2019-11-08T15:37:09+08:00
draft: false  
categories: [linux]
tags: [linux]

---

Too many open files

<!--more-->

  用户或系统打开文件限制超过默认设置时，将生成此错误 

  查看操作系统级别的最大打开文件设置，使用以下命令:

```bash
cat /proc/sys/fs/file-max
```

更改系统范围的最大打开文件，需要编辑/etc/sysctl.conf 文件

```bash
fs.nr_open = 2000000
fs.file-max = 6553560 
```

每个用户设置

```bash
echo "* soft nofile 655360" >> /etc/security/limits.conf
echo "* hard nofile 655360" >> /etc/security/limits.conf
```

设置所有用户 

```bash
vim /etc/security/limits.conf 在最后加入 
* soft nofile 655350 
* hard nofile 655350 
```

查看用户最大允许打开的文件数量

```bash
ulimit -a 
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 31200
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 31200
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited

# open files 表示每个用户最大允许打开的文件数量是1024 
```

查看当前系统打开的文件数量

```bash
lsof | wc -l
watch “lsof | wc -l”
```

查看某一个进程打开的文件数量

```bash
lsof -p pid | wc -l
```



soft 与hard 的区分

```bash
ulimit 对资源的限制区分soft 与hard 两类，即同一个资源存在soft 与hard 两个值

在命令上，ulimit 通过-S 与—H 来区分soft 与hard ，如果没有指定-S 与-H，在显示时指的是soft，而在设置时指的是同时设置了soft 与hard 值 
```

