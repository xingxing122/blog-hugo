---
title: "fastdfs部署"
date: 2019-6-10T16:18:06+08:00
draft: false  
categories: ["linux"]
tags: ["linux"]
---

​       

 Fastdfs 是一款开源的轻量级分布式文件系统纯C 实现，支持linux、FreeBSD 等UNIX 系统类google FS，   不是通用的文件系统，只能通过专有API 访问

<!--more-->

##### 1. 部署fastfs

```bash
下载https://github.com/happyfish100/fastdfs
安装依赖包
yum install gcc*   perl-devel   -y  
cd /opt/fastdfs-5.11/ 
./make.sh  
```

![image-20190610170231987](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-10-090254.png)


```
报错，需要安装libfastcommon 
wget  https://github.com/happyfish100/libfastcommon/archive/V1.0.39.tar.gz  
tar xvf  V1.0.39.tar.gz  
cd libfastcommon-1.0.39
./make.sh
./make.sh install 
   
继续安装fastdfs
cd /opt/fastdfs-5.11 
./make.sh 
./make.sh install 
```


  ![image-20190610171917428](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-071048.png)

```bash
安装完成
```

##### 2. 配置fastdfs

```bash
将fastdfs 安装目录下的conf 文件拷贝到/etc/fdfs/  
[root@fastdfs fastdfs-5.11]# cp -r conf/*  /etc/fdfs/
配置trackerd 
vim /etc/fdfs/tracker.conf 
base_path=/data/fastdfs/tracker
http.server_port=80
```

##### 3. 启动tracker

```bash
mkdir -p /data/fastdfs/tracker (/data/ 单独挂载的磁盘)
/usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf restart
```

![image-20190610181617272](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-071142.png)

```bash
日志在 /data/fastdfs/tracker/logs/trackerd.log
```

##### 4. 配置storaged

```bash
vim /etc/fdfs/storage.conf
base_path=/data/fastdfs/storage
store_path0=/data/fastdfs/storage
tracker_server=0.0.0.0:22122
http.server_port=80

mkdir -p /data/fastdfs/storage
/usr/bin/fdfs_storaged /etc/fdfs/storage.conf restart
```

![image-20190610182256767](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-071211.png)

```
日志目录/data/fastdfs/logs/storaged.log
```

##### 5. nginx 安装

```bash
yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel 
wget https://nginx.org/download/nginx-1.16.0.tar.gz
下载fastdfs-nginx-module 模块 
tar xvf nginx-1.16.0.tar.gz
tar xvf  V1.20.tar.gz 
cd nginx-1.16.0
./configure --prefix=/usr/local/nginx --add-module=/opt/fastdfs-nginx-module-1.20/src
make 
报错
```

![image-20190610184915785](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-071319.png)

```bash
需要去找修改config 文件

vim /usr/local/fastdfs-nginx-module/src/config 
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"

再次执行make 
make install 
```

![image-20190610185544150](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-071340.png)

##### 6. 修改nginx 的配置文件

```nginx
vim /usr/local/nginx/conf/nginx.conf
worker_processes  8;
events {
    worker_connections  10240;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

​    gzip  on;

​    server {
​        listen       80;
​        server_name  10.39.33.37;

​        location /group1/M00 {
​           root /data/fastdfs/storage/data;
​           ngx_fastdfs_module;
​        }
​        error_page   500 502 503 504  /50x.html;
​        location = /50x.html {
​            root   html;
​        }

​    }

}
```



##### 7.修改mod_fastdfs.conf

```bash
cp /usr/local/fastdfs-nginx-module/src/mod_fastdfs.conf  /etc/fdfs/
vim /etc/fdfs/mod_fastdfs.conf
base_path=/data/fastdfs/storage
tracker_server=10.39.33.37:22122
# the group name of the local storage server
group_name=group1
# if the url / uri including the group name
# set to false when uri like /M00/00/00/xxx
# set to true when uri like ${group_name}/M00/00/00/xxx, such as group1/M00/xxx
# default value is false
url_have_group_name = true
```

##### 8. 拷贝文件

```bash 
cp /opt/fastdfs-5.11/conf/http.conf  /etc/fdfs/ 
cp /opt/fastdfs-5.11/conf/mime.types /etc/fdfs/
ln -s /data/fastdfs/storage/data  /data/fastdfs/storage/data/M00
```

##### 9. 启动nginx

```bash
/usr/local/nginx/sbin/nginx  -s reload
ngx_http_fastdfs_set pid=4022
```

##### 10.  测试

```bash
vim /etc/fdfs/client.conf 
base_path=/data/fastdfs/storage
tracker_server=10.39.33.37:22122

上传图片和文件测试
/usr/bin/fdfs_test /etc/fdfs/client.conf upload anti-steal.jpg

文件位置
ll /data/fastdfs/storage/data/00/00/CichJVz-bTeASuxcAAAFtNHOfzM63_big.conf 
```

在浏览器中测试

[http://10.39.33.37/group1/M00/00/00/CichJVz-bM-AcO1MAABdrSqbHGQ105_big.jpg](http://10.39.33.37/group1/M00/00/00/CichJVz-bM-AcO1MAABdrSqbHGQ105_big.jpg)

![image-20190611112618542](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-11-032618.png)











