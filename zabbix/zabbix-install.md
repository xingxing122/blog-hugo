---
title: "Zabbix Install"
date: 2019-07-25T00:06:52+08:00
draft: false  
categories: [monitor]
tags: [monitor]

---

zabbix 的安装

<!--more-->

概述

zabbix是由alexei Vladishev，目前是由zabbix SIA在持续开发和提供支持，

zabbix是一种企业级的分布式开源监控解决方案。

zabbix是一款能够监控众多网络参数和服务器的健康度和完整性的软件.zabbix使用灵活的通知机制，允许用户为几乎任何事件配置基于邮件的警报。这样可以快速相应服务器问题，zabbix基于存储的数据提供出色的报告和数据可视化，这些功能使得zabbix成为容量规划的理想选择



功能

数据采集

```bash
  可用性和性能采集
  支持SNMP(包括主动轮询和被动捕获)、IPMI JMX VMWARE 监控 
  自定义检查
  按照自定义的时间间隔采集需要的数据
  通过Server/Proxy 和Agents 来执行数据采集
```

灵活的阈值定义

```bash
  可以定义非常灵活的告警阈值，称之为触发器 
```

实时图形

```bash
  使用内置图形可以将监控项绘制成图形
```

权限管理系统

```bash
  安全的用户身份验证
  将特定用户限制于访问特定的视图 
```



组件

服务器

```bash
Zabbix server  是Zabbix agent 向其报告可用性、系统完整性信息和统计信息的核心组件.是存储所有配置信息、统计信息和操作信息的核心存储库
```

数据库

```bash
 所有配置信息以及zabbix 收集到数据都被存储在数据库中
```

网络界面

```bash
  为了从任何地方和任何平台轻松访问zabbix，提供了一个基于web的界面，一般情况下和zabbix server 运行在同一台物理机上
```

代理

```bash
  zabbix proxy 可以替zabbix server 收集性能和可用性数据，zabbix proxy 是zabbix 环境部署的可选部分，然而，它对于单个zabbix 负载的负担是非常有益的
```

代理人

```bash
   zabbix agent 部署在被监控目标上，用于主动监控本地资源和应用程序，并收集数据发送给zabbix server  
```

安装PHP

```bash
yum -y install libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel curl curl-devel openssl openssl-devel openldap openldap-devel  libzip  libzip-devel    
```

[php7.3下载](https://downloads.php.net/~cmb/php-7.3.0.tar.gz)

```bash
tar xvf php-7.3.0.tar.gz
cd php-7.3.0
./configure  --prefix=/usr/local/php7  --enable-fpm  --with-fpm-user=www  --with-fpm-group=www  --with-pdo-mysql=mysqlnd  --with-mysqli=mysqlnd  --with-zlib  --with-curl  --with-gd  --with-gettext --enable-bcmath --enable-sockets  --with-ldap --with-jpeg-dir  --with-png-dir  --with-freetype-dir  --with-openssl  --enable-mbstring  --enable-xml  --enable-session  --enable-ftp  --enable-pdo -enable-tokenizer  --enable-zip  --with-libdir=lib64
```

报错配置：错误：请重新安装libzip发行版

解决方法

```bash
wget https://libzip.org/download/libzip-1.5.1.tar.gz
```

libzip需要cmake 3来进行编译安装

```bash
 yum -y install epel-release
 yum  -y install cmake3     
 ln -s /usr/bin/cmake3 /usr/bin/cmake

cd libzip-1.5.1 
mkdir build
cd build
cmake3 ..
make
make test
make install    

cd /opt/php-7.3.0 
 ./configure  --prefix=/usr/local/php7  --enable-fpm    --with-pdo-mysql=mysqlnd  --with-mysqli=mysqlnd  --with-zlib  --with-curl  --with-gd  --with-gettext --enable-bcmath --enable-sockets  --with-ldap --with-jpeg-dir  --with-png-dir  --with-freetype-dir  --with-openssl  --enable-mbstring  --enable-xml  --enable-session  --enable-ftp  --enable-pdo -enable-tokenizer  --enable-zip  --with-libdir=lib64
 make  
 make install 

 cp php.ini-production  /usr/local/php7/lib/php.ini     
 cp sapi/fpm/php-fpm.service /usr/lib/systemd/system/      
```

PHP优化

vim /usr/local/php7/lib/php.ini

```bash
post_max_size = 16M
 max_execution_time = 300
memory_limit = 128M
max_input_time = 300
date.timezone = Asia/Shanghai
```

lnmp配置

```bash
  yum install -y nginx  
```

vi /etc/nginx/conf.d/zabbix.conf

```bash
  server {
    listen       80 default_server;
    server_name  10.3.1.5;

    root         /var/www/;


    location / {
        index index.php index.html index.htm;
    }

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }

    location ~ \.php$ {
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

修改php-fpm配置文件，启用php-fpm默认配置，执行如下操作

```bash
 cd /usr/local/php7/etc/
 cp php-fpm.conf.default  php-fpm.conf
 cp php-fpm.d/www.conf.default php-fpm.d/www.conf     
```

启动

```bash
 systemctl   restart  php-fpm 
```

报错

```
Dec 07 14:26:43 zabbix php-fpm[18853]: [07-Dec-2018 14:26:43] ERROR: [pool www] cannot get uid for user 'www'
Dec 07 14:26:43 zabbix php-fpm[18853]: [07-Dec-2018 14:26:43] ERROR: FPM initialization failed  
```

解决

vim /usr/local/php7/etc/php-fpm.d/www.conf

```bash
user = zabbix
group = zabbix    

systemctl  restart php-fpm 
```

状态检查

```bash
systemctl  status  php-fpm
● php-fpm.service - The PHP FastCGI Process Manager
   Loaded: loaded (/usr/lib/systemd/system/php-fpm.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-12-07 14:37:13 CST; 17s ago
 Main PID: 19980 (php-fpm)
   CGroup: /system.slice/php-fpm.service
           ├─19980 php-fpm: master process (/usr/local/php7/etc/php-fpm.conf)
           ├─19981 php-fpm: pool www
           └─19982 php-fpm: pool www

Dec 07 14:37:13 zabbix systemd[1]: Started The PHP FastCGI Process Manager     


zabbix server 安装
```

[zabbix4.0官网安装文档](https://www.zabbix.com/documentation/4.0/zh/manual/installation/install)

创建用户账户

```bash
 groupadd zabbix
 useradd -g zabbix zabbix
```

[zabbix4.0下载](https://jaist.dl.sourceforge.net/project/zabbix/ZABBIX Latest Stable/4.0.2/zabbix-4.0.2.tar.gz)

安装依赖包

```bash
 yum -y install  libxml2-devel net-snmp-devel libcurl-devel  mysql-devel  libevent-devel 
 tar xvf zabbix-4.0.2.tar.gz 
 cd zabbix-4.0.2  
 ./configure  --prefix=/usr/local/zabbix    --enable-server  --with-mysql  --with-net-snmp --with-libcurl --with-libxml2
 make  install 

 默认安装/usr/local/
```

配置zabbix下的mysql环境

[zabbix4.0数据库配置](https://www.zabbix.com/documentation/4.0/manual/appendix/install/db_scripts)

创建数据库

```bash
create database zabbix character set utf8 collate utf8_bin;
grant all privileges on zabbix.* to zabbix@"%" identified by '***';
grant all  on zabbix.* to zabbix@"%" identified by 'password' with grant option;
flush privileges;
```

导入数据

```bash
mysql -uzabbix -h 10.3.1.2  -ppassword zabbix < schema.sql
mysql -uzabbix -h 10.3.1.2  -ppassword zabbix < images.sql
mysql -uzabbix -h 10.3.1.2  -ppassword zabbix < data.sql
```

配置zabbix服务器

vim /usr/local/zabbix/etc/zabbix_server.conf

```bash
LogFile=/tmp/zabbix_server.log
DBHost=10.3.1.2
DBName=zabbix
DBUser=zabbix
DBPassword=ZTQoq4NTQiUo
Timeout=4
AlertScriptsPath=/usr/local/zabbix/share/zabbix/alertscripts
LogSlowQueries=3000
```

zabbix网站

zabbix web程序是用php写的，只需要将web程序拷贝到web目录app下即可

```bash
cp -rp /opt/zabbix-4.0.2/frontends/php/*  /app/
```

启动服务

```bash
cd /opt/zabbix-4.0.2 
cp  misc/init.d/fedora/core/zabbix_server  /etc/init.d/zabbix_server 
chmod +x /etc/init.d/zabbix_server
systemctl restart  zabbix_server
 systemctl status zabbix_server
systemctl restart nginx
systemctl status  nginx
```

浏览在器输入侧[http://10.3.1.5](http://10.3.1.5/)

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065200.jpg)

php模块检查

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065228.jpg)



数据库配置

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065246.jpg)



zabbix服务器信息配置

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065305.jpg)



信息核实

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065329.jpg)



配置好的信息组成一个配置文件，放到zabbix的目录，点击下载到本地，然后在/ app / conf /目录下创建zabbix.conf.php文件即可



![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065347.jpg)

安装完成

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065510.jpg)

点击完成

将配置的文件拷贝到zabbix web目录下

```bash
cat /app/conf/zabbix.conf.php
<?php
// Zabbix GUI configuration file.
global $DB;

$DB['TYPE']     = 'MYSQL';
$DB['SERVER']   = '10.3.1.2';
$DB['PORT']     = '3306';
$DB['DATABASE'] = 'zabbix';
$DB['USER']     = 'zabbix';
$DB['PASSWORD'] = 'paswoord';

// Schema name. Used for IBM DB2 and PostgreSQL.
$DB['SCHEMA'] = '';

$ZBX_SERVER      = '10.3.1.5';
$ZBX_SERVER_PORT = '10051';
$ZBX_SERVER_NAME = '10.3.1.5';

$IMAGE_FORMAT_DEFAULT = IMAGE_FORMAT_PNG;
```

zabbix默认的用户名为Admin密码zabbix

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065541.jpg)



登陆ZABBIX，修改英文为中文

![img](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-07-29-065600.jpg)



修改乱码问题

zabbix 4.0.2自带的字体为DejaVuSans.ttf但是修改为中英文之后存在乱码情况，zabbix设置成中文之后有乱码问题解决在win的机器上拷贝字体

```
C:\Windows\Fonts\   
拷贝字体到
sudo cp simsunb.ttf  /app/fonts/  
```

修改web前端加载的字体

```
vim /app/include/defines.inc.php 

修改前
define('ZBX_GRAPH_FONT_NAME',           'DejaVuSans'); // font file name
define('ZBX_FONT_NAME', 'DejaVuSans');

修改后
define('ZBX_GRAPH_FONT_NAME',           'simsunb'); // font file name
define('ZBX_FONT_NAME', 'simsunb');
```

如果是编译的php需要去掉参数-enable-gd-jis-conv，php编译启用enable-gd-jis-conv选项的话，那么非ASCII字符（例如汉字，拼音，希腊文和箭头）会被当成EUC- JP编码（phpinfo中美其名曰“支持JIS编码的字体”），从而导致乱码（由于西文字体没有假名或汉字，一般表现为全部是方框）

到这里zabbix基本上就安装完成！



