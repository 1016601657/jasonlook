---
title: nginx负载均衡后 服务器文件双向同步工具:lsync
date: 2018-03-23 22:58:10
tags: [linux 文件]
categories: linux
---


>Lsyncd 是一个简单高效的文件同步工具，通过lua语言封装了 inotify 和 rsync 工具，采用了 Linux 内核（2.6.13 及以后）里的 inotify 触发机制，然后通过rsync去差异同步，达到实时的效果。


<!-- more -->
### 安装过程：

#### 第一步: 安装lsync及其组件

``` bash
$ yum install lua lua-devel
$ yum –y install rsync
$ rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ yum install lsyncd
```


#### 第二步: 修改配置文件
``` bash
$ vi /etc/lsyncd.conf
```

``` bash
$ settings {
       logfile ="/usr/local/lsyncd-2.1.5/var/lsyncd.log",
       statusFile ="/usr/local/lsyncd-2.1.5/var/lsyncd.status",
       inotifyMode = "CloseWrite",
       maxProcesses = 8,
       }
   
   
   -- I. 本地目录同步，direct：cp/rm/mv。 适用：500+万文件，变动不大
   sync {
       default.direct,
       source    = "/tmp/src",
       target    = "/tmp/dest",
       delay = 1
       maxProcesses = 1
       }
   
   -- II. 本地目录同步，rsync模式：rsync
   sync {
       default.rsync,
       source    = "/tmp/src",
       target    = "/tmp/dest1",
       excludeFrom = "/etc/rsyncd.d/rsync_exclude.lst",
       rsync     = {
           binary = "/usr/bin/rsync",
           archive = true,
           compress = true,
           bwlimit   = 2000
           } 
       }
   
   -- III. 远程目录同步，rsync模式 + rsyncd daemon
   sync {
       default.rsync,
       source    = "/tmp/src",
       target    = "syncuser@172.29.88.223::module1",
       delete="running",
       exclude = { ".*", ".tmp" },
       delay = 30,
       init = false,
       rsync     = {
           binary = "/usr/bin/rsync",
           archive = true,
           compress = true,
           verbose   = true,
           password_file = "/etc/rsyncd.d/rsync.pwd",
           _extra    = {"--bwlimit=200"}
           }
       }
   
   -- IV. 远程目录同步，rsync模式 + ssh shell
   sync {
       default.rsync,
       source    = "/tmp/src",
       target    = "172.29.88.223:/tmp/dest",
       -- target    = "root@172.29.88.223:/remote/dest",
       -- 上面target，注意如果是普通用户，必须拥有写权限
       maxDelays = 5,
       delay = 30,
       -- init = true,
       rsync     = {
           binary = "/usr/bin/rsync",
           archive = true,
           compress = true,
           bwlimit   = 2000
           -- rsh = "/usr/bin/ssh -p 22 -o StrictHostKeyChecking=no"
           -- 如果要指定其它端口，请用上面的rsh
           }
       }
   
   -- V. 远程目录同步，rsync模式 + rsyncssh，效果与上面相同
   sync {
       default.rsyncssh,
       source    = "/tmp/src2",
       host      = "172.29.88.223",
       targetdir = "/remote/dir",
       excludeFrom = "/etc/rsyncd.d/rsync_exclude.lst",
       -- maxDelays = 5,
       delay = 0,
       -- init = false,
       rsync    = {
           binary = "/usr/bin/rsync",
           archive = true,
           compress = true,
           verbose   = true,
           _extra = {"--bwlimit=2000"},
           },
       ssh      = {
           port  =  1234
           }
       }
```


主/副服务器添加账号


```bash
主服务器
$ useradd zhuzhanghao
$ passwd zhuzhanghao
$ zhuzhanghaomima


副服务器
$ useradd fuzhanghao
$ passwd fuzhanghao
$ fuzhanghaomima


配置密码:
$ vi /etc/rsyncd.passwd
fuzhanghao:fuzhanghaomima
```