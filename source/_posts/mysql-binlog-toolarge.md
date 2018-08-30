---
title: 修改mysql binlog过期时间,及迁移存储路径
date: 2018-04-02 20:17:47
tags: [mysql,binlog]
---
>mysql binlog 的适量产生可以用于容灾修复, 但是大量的log文件同样会给服务器的硬盘带来压力, 以下介绍如何修改binlog的过期时间, 并且为binlog文件重新分区.

<!-- more -->

### 1. 设置过期时间

在mysql中执行该语句 修改mysql配n置 

```bash
mysql > global expire_logs_days=7;

```
刷新配置, 使设置生效, 此操作会立即删除过期日志
```bash
mysql > flush logsset;

```

### 2. 修改binlog存储路径

停止服务
```bash
$ service mysqld stop
```
创建目录
```bash
$ mkdir -p  /mnt/server/mysql/data/
```
拷贝日志文件
```bash
$ cp /alidata/server/mysql/data/mysql-bin.* /mnt/server/mysql/data/
```
修改目录所属用户/用户组
```bash
$ chown -R mysql /mnt/server/mysql/data/
$ chgrp -R mysql /mnt/server/mysql/data/
```
修改my.cnf配置
```bash
$ vi /etc/my.cnf
log-bin=mysql-bin 为 log-bin=/mnt/server/mysql/data/mysql-bin
```
修改binlog中的index路径
```bash
$ vi /data/mysql/data/mysql-bin.index
./mysql-bin.?????? 改为 /data/mysql/data/mysql-bin.??????
```
启动服务
```bash
$ service mysqld stop
如果启动失败 检查binlog新目录的用户组是否是mysql
```



