---
title: "Harbor 2.1.1 安装"
date: 2020-11-18T11:37:14+08:00
draft: false  
categories: [devops]
tags: [devops]

---

### harbor 2.1 安装

<!--more-->

#### 1. 下载harbor

[从这里下载发布的正式版本](https://github.com/goharbor/harbor/releases)

```bash
下载了harbor-offline-installer-v2.1.1.tgz 包 
解压之后就会看到这些文件
```

![image-20201118141755716](https://xing-blog.oss-cn-beijing.aliyuncs.com/2020-11-18-061759.png)

```bash
harbor.yaml 是用来设置系统参数 
参考官网介绍
https://goharbor.io/docs/2.0.0/install-config/configure-yml-file/  
```

#### 2.证书配置

```bash
hostname: reg.mydomain.com
http:
  port: 80
https:
  port: 443
  certificate: /opt/ssl/****.crt
  private_key: /opt/ssl/*****.key
external_url: https://reg.domain.com
harbor_admin_password: Harbor12345

clair:
  updaters_interval: 12
trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false
jobservice:
  max_job_workers: 10
notification:
  webhook_job_max_retry: 10

chart:
  absolute_url: disabled
log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor

_version: 2.0.0
```

#### 3.外接DB设置

```bash
psql -U root  -h 10.39.61.242  -d postgres
#创建harbor 用户，并创建harbor 所涉及的数据库
create user harbor with password 'harbor123';
CREATE DATABASE harbor;
create database harbor_clair;
create database notary_signer;
create database notary_server;
#授权
GRANT ALL PRIVILEGES ON DATABASE harbor to harbor; 
GRANT ALL PRIVILEGES ON DATABASE harbor_clair to harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_signer to harbor;
GRANT ALL PRIVILEGES ON DATABASE notary_server to harbor;

#配置 
external_database:
   harbor:
     host: *****
     port: 5432
     db_name: harbor
     username: harbor
     password: ****
     ssl_mode: disable
     max_idle_conns: 2
     max_open_conns: 0
   clair:
     host: ****
     port: 5432
     db_name: clair
     username: harbor
     password: ***
     ssl_mode: disable
   notary_signer:
     host: *****
     port: 5432
     db_name: notary_signer
     username: ****
     password: ****
     ssl_mode: disable
   notary_server:
     host: *****
     port: 5432
     db_name: notary_server
     username: ****
     password: ****
     ssl_mode: disable
```

#### 4 redis 配置

```bash
 external_redis:
   host: ***:6379
   password: enN12345
   registry_db_index: 1
   jobservice_db_index: 2
   chartmuseum_db_index: 3
   clair_db_index: 4
   trivy_db_index: 5
   idle_timeout_seconds: 30
```

#### 5.存储配置(S3)

registry 存储可以参考官网https://docs.docker.com/registry/configuration/

```bash
storage_service:
  s3:
    accesskey: ****
    secretkey: ****
    region: ****
    regionendpoint: ****
    bucket: harbor
    encrypt: false
    keyid: mykeyid
    secure: true
    v4auth: true
    chunksize: 5242880
    multipartcopychunksize: 33554432
    multipartcopymaxconcurrency: 100
    multipartcopythresholdsize: 33554432
    rootdirectory: /s3/harbor
```































