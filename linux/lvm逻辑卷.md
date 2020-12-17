---
title: "Lvm逻辑卷"
date: 2020-05-29T10:13:06+08:00
draft: false  
categories: [linux]
tags: [linux]

---

```bash
yum  install lvm2  
```

<!--more-->

创建pv 

```bash
[root@prometheus-tke ~]# pvcreate /dev/vdd  
WARNING: xfs signature detected on /dev/vdd at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/vdd.
  Physical volume "/dev/vdd" successfully created.
[root@prometheus-tke ~]# pvcreate /dev/vde 
WARNING: xfs signature detected on /dev/vde at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/vde.
  Physical volume "/dev/vde" successfully created.
[root@prometheus-tke ~]# vgcreate vgmoint /dev/vdd  /dev/vde  
  Volume group "vgmoint" successfully created
You have mail in /var/spool/mail/root
[root@prometheus-tke ~]# lvcreate -l 100%FREE  vgmoint 
  Logical volume "lvol0" created.
You have mail in /var/spool/mail/root
```

![image-20200529103314704](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-05-29-023315.png)

![image-20200529103336876](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-05-29-023337.png)

![image-20200529103352568](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-05-29-023352.png)

挂载

 

```bash
mount /dev/vgmoint/lvol0  /var/lib/monitoring  
查看UUID 
blkid  
将UUID 写入到/etc/fstab 中即可

```



