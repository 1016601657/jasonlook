---
title: linux性能分析常用命令
tags:
  - linux
  - 性能
date: 2018-08-30 23:50:18
---


### uptime
描述: 打印当前时间, 系统运行时长, 当前登录用户数, 以及服务器的平均负载
```bash
$ uptime
13:46:04 up 7 days, 20:11,  1 user,  load average: 0.00, 0.01, 0.05
当前时间 | 运行时长 | 当前用户数 | 平均负载: (1分钟, 5分钟, 15分钟)
```
这里的负载表示单位时间段内, cpu等待队列中的平均进程数, 等待数越高代表cpu越忙, 负载越高. 

-----------
### free 
参数: `[-b|-k|-m]` 指定容量的单位
描述: 显示内存及交换分区信息
```bash
$ free -m
             total       used       free     shared    buffers     cached
Mem:          3961       3800        160          0        307       2034
-/+ buffers/cache:       1459       2502
Swap:            0          0          0
```
***linux在开机的时候会预先提取一部分内存, 并划分为buffer与cache, 随时提供给进程使用.***
以上的输出信息中 `Mem`一行的`total`代表内存总容量为3961Mb; `used`代表系统内存的3800Mb划分成了buffer与cache, 也就是buffer与cache的总容量;`free`代表内存总容量减去buffer与cache的总和之后的剩余总容量为160Mb; `buffers`代表当前buffer的剩余容量为307Mb, `cached`代表cache的剩余容量为2034Mb.
第二行`used`代表buffer与cache当前总共使用了1459Mb, `free`代表buffer与cache总剩余加内存未被划分的剩余容量之和, 也就是说 2502Mb = 307Mb + 2034Mb + 160Mb. 这个值就是系统中内存未被使用的实际总容量
第三行为交换分区的使用情况, `total`代表交换分区总容量, `used`代表已经使用的容量, `free`代表剩余交换分区容量.

-----------
### df 
参数: -h (human人性化显示容量信息)
&ensp;&ensp;&ensp;&ensp;&ensp;-i 显示磁盘inode使用量信息
&ensp;&ensp;&ensp;&ensp;&ensp;-T 显示文件系统类型
```bash
$ df -hT
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/vda1      ext4    59G   33G   24G  59% /
tmpfs          tmpfs  1.9G     0  1.9G   0% /dev/shm
/dev/vdb1      ext3    40G   23G   16G  60% /mnt
根分区 | 根分区类型 | 总容量 | 已经使用的容量 | 剩余容量 | 使用率 | 挂载点
```

```bash
$ df -i
Filesystem      Inodes  IUsed   IFree IUse% Mounted on
/dev/vda1      3932160 434775 3497385   12% /
tmpfs           490548      1  490547    1% /dev/shm
/dev/vdb1      2621440 463337 2158103   18% /mnt
根分区 | 根分区的inode总个数 | 已经使用的个数 | 剩余个数 | 使用率 | 挂载点

```
以上信息中, 根分区的inode总个数3932160个 已经使用了434775个, 剩余3497385个,使用率为12%, 这里的inode个数决定了该分区可以创建的文件个数, 有多少个inode节点, 就可以在该分区创建多少个文件. 在上面的显示中, 如果在根分区下在创建3497386个空文件, 则即使系统有剩余空间, 也无法继续创建文件, 因为inode节点已经耗尽.

-----------
### du 
描述: 查看指定目录的占用空间
参数: -s   #仅显示总计，只列出最后所有文件相加的值。
&ensp;&ensp;&ensp;&ensp;&ensp;-h    #人性化显示
```bash
$ du -sh /var/log
56M	/var/log
```

-----------
### ifconfig
描述: 查看网卡接口信息
```bash
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 00:16:3E:01:F1:82
          inet addr:172.31.247.37  Bcast:172.31.255.255  Mask:255.255.240.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:153572723 errors:0 dropped:0 overruns:0 frame:0
          TX packets:150050213 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:39078038946 (36.3 GiB)  TX bytes:104756187251 (97.5 GiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:229489342 errors:0 dropped:0 overruns:0 frame:0
          TX packets:229489342 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:156048407167 (145.3 GiB)  TX bytes:156048407167 (145.3 GiB)
```
在linux中以太网卡一般
 
-----------
### netstat 
描述: 打印网络连接, 路由表, 网络接口统计等信息.
参数: -s   #显示各种协议数据统计信息
&ensp;&ensp;&ensp;&ensp;&ensp;-n   #使用数字星矢的ip, 端口号, 用户id代替主机, 协议, 用户等名称信息.
&ensp;&ensp;&ensp;&ensp;&ensp;-p   #显示进程名称及对应的进程id号.
&ensp;&ensp;&ensp;&ensp;&ensp;-l   #仅显示正在监听的socket接口信息.
&ensp;&ensp;&ensp;&ensp;&ensp;-u   #查看udp连接信息.
&ensp;&ensp;&ensp;&ensp;&ensp;-t   #查看tcp连接信息.
```bash
$ netstat -nutlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name
tcp        0      0 0.0.0.0:8096                0.0.0.0:*                   LISTEN      3796/nginx
tcp        0      0 127.0.0.1:32000             0.0.0.0:*                   LISTEN      1273/java
tcp        0      0 0.0.0.0:8097                0.0.0.0:*                   LISTEN      3796/nginx
tcp        0      0 0.0.0.0:8098                0.0.0.0:*                   LISTEN      3796/nginx
tcp        0      0 0.0.0.0:8099                0.0.0.0:*                   LISTEN      3796/nginx
tcp        0      0 0.0.0.0:2021                0.0.0.0:*                   LISTEN      19047/WorkerMan
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      659/php-fpm
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      13147/mysqld
tcp        0      0 0.0.0.0:6379                0.0.0.0:*                   LISTEN      14342/./redis-serve
```
```bash
$ netstat -s
Ip:
    383023015 total packets received
    0 forwarded
    0 incoming packets discarded
    383023014 incoming packets delivered
    379500022 requests sent out
    2 reassemblies required
    1 packets reassembled ok
Icmp:
    22962 ICMP messages received
    377 input ICMP message failed.
    ICMP input histogram:
        destination unreachable: 18459
        timeout in transit: 873
        echo requests: 3600
省略...
```

-----------
### ps
参数: ps命令版本众多, 有多种语法种类.
描述: 查看当前进程信息.
标准格式语法:
```bash
$ ps -e    # 查看所有的进程信息
$ ps -ef   # 全格式显示进程信息
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 Jun21 ?        00:00:00 /sbin/init
root         2     0  0 Jun21 ?        00:00:00 [kthreadd]
root         3     2  0 Jun21 ?        00:00:08 [migration/0]
root         4     2  0 Jun21 ?        00:00:25 [ksoftirqd/0]
root         5     2  0 Jun21 ?        00:00:00 [stopper/0]
root         6     2  0 Jun21 ?        00:00:04 [watchdog/0]
root         7     2  0 Jun21 ?        00:00:04 [migration/1]
root         8     2  0 Jun21 ?        00:00:00 [stopper/1]
```
BSD语法格式:
```bash
$ ps -ax
$ ps -aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.0  21276   364 ?        Ss   Jun21   0:00 /sbin/init
root         2  0.0  0.0      0     0 ?        S    Jun21   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Jun21   0:08 [migration/0]
root         4  0.0  0.0      0     0 ?        S    Jun21   0:25 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S    Jun21   0:00 [stopper/0]
root         6  0.0  0.0      0     0 ?        S    Jun21   0:04 [watchdog/0]
root         7  0.0  0.0      0     0 ?        S    Jun21   0:04 [migration/1]
root         8  0.0  0.0      0     0 ?        S    Jun21   0:00 [stopper/1]
```
命令的输出信息中, `UID`或`USER`代表进程的执行用户;
`PID`为进程的唯一编号;
`PPID`代表父进程ID编号;
`%CPU`代表进程的cpu占用率;
`%MEM`代表进程的内存占用率;
`VSZ`代表进程所使用的虚拟内存大小(单位为KB);
`RSS`或`START`代表进程启动时间;
`STAT`代表进程状态(D:不可中断的进程, R:正在运行的进程, S:正在睡眠的进程, T:停止或被追踪的进程, X:死掉的进程, Z:僵死进程);
`TIME`代表进程占用CPU的总时间;
`CMD`或`COMMAND`代表进程命令

 
-----------
### top 
描述: 动态查看进程信息
参数: -d #top刷新间隔, 默认为3秒
&ensp;&ensp;&ensp;&ensp;&ensp;-p # 指定查看的PID进程信息 多个pid用,分隔
```bash
$ top -d 1
top - 23:29:10 up 70 days, 11:55,  4 users,  load average: 0.00, 0.00, 0.00
Tasks:   2 total,   0 running,   2 sleeping,   0 stopped,   0 zombie
Cpu(s):  2.5%us,  0.5%sy,  0.0%ni, 97.0%id,  0.0%wa,  0.0%hi,  0.0%si,  0.0%st
Mem:   4056480k total,  3887696k used,   168784k free,   327308k buffers
Swap:        0k total,        0k used,        0k free,  2038372k cached

  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    1 root      20   0 21276  364   84 S  0.0  0.0   0:00.59 init
    2 root      20   0     0    0    0 S  0.0  0.0   0:00.03 kthreadd
    省略...
    
$ top -d 1 -p 1,2
   省略...
```
top默认会按cpu使用率排序, 输入`M`可以按照内存使用率排序, 输入`N`可以按照进程号排序, 输入`z`可以高亮显示颜色.(注意区分大小写)
