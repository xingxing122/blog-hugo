---
title: "Mysql 常用命令"
date: 2019-07-26T11:28:30+08:00
draft: false  
categories: [DB]
tags: [DB]

---

mysql 常用命令

<!--more-->

创建用户并授权

```sql
CREATE USER 'username'@'host' IDENTIFIED BY 'password';
GRANT privileges ON databasename.tablename TO 'username'@'host'

```

数据导出

```bash
mysqldump -B -u root -p -h ip  zabbix >  /opt/zabbix.sql
```

数据库导入

```bash
mysql -udevops -p  -h ip < zabbix.sql
```

