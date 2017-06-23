---
layout:     post
title:      "Redis Linux 优化"
date:       UTC2017-06-23 09:50:00
author:     "Pearpai"
header-img: "img/redis-bg.png"
catalog: true
tags:
    - 数据库
    - Blog
    - Redis
---
这两天在学习Redis集群部署，因为没有多余的物理机，所以准备在docker中实现，突发奇想这如果部署到docker中本系统资源就不多，是不是做一些系统上的优化呢？想来就百度了一下，需要优化的地方还是蛮多的，列出了找到的优化方式。
## 查看启动日志
![Redis 启动日志](/img/redis-start-log.png)
### 1. 内核参数：somaxconn
**WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.**
上述为启动日志，提示修改参数128 -> 511
**somaxconn：**系统中每个端口最大的监听队列的长度，这是个全局参数，默认值为128，这个里的数值根据实际情况进行修改。一般修改为1024。
**执行方式：** echo 511 >/proc/sys/net/core/somaxconn

### 2.内存分配策略：vm.overcommit_memory
**WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.**

#### vm.overcommit_memory 分配策略：
- 0：内核将检查是否有足够的可用内存供进程使用；如果有足够的可用内存，内存申请允许，否则内存申请失败，并把错误返回给应用进程。
- 1：内核允许分配所有的物理内存，而不管当前内存状态如何，直到内存用完为止
- 2：表示内核决不过量的(“never overcommit”)使用内存，即系统整个内存地址空间不能超过swap+50%的RAM值，50%是overcommit_ratio默认值，此参数同样支持修改

**执行方式：**
根据日志说明把vm.overcommit_memory设置为1
- echo 1 > /proc/sys/vm/overcommit_memory 或
- sysctl vm.overcommit_memory=1 或
- echo "vm.overcommit_memory=1" >> /etc/sysctl.conf

### 3.Transparent Huge Pages
**WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo never > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled.**
禁用大内存分页，
（引用）Linux kernel在2.6.38内核增加了Transparent Huge Pages (THP)特性 ，支持大内存页(2MB)分配，默认开启。当开启时可以降低fork子进程的速度，但fork之后，每个内存页从原来4KB变为2MB，会大幅增加重写期间父进程内存消耗。同时每次写命令引起的复制内存页单位放大了512倍，会拖慢写操作的执行时间，导致大量写操作慢查询。例如简单的incr命令也会出现在慢查询中。
**设置：** echo never >  /sys/kernel/mm/transparent_hugepage/enabled

## maxmemory
此为设置redis 最大使用内存，当达到最大内存值时会进行一些处理，处理方式通过**maxmemory-policy** 进行设置如下：
```
config set maxmemory-policy volatile-lru
```
 **maxmemory-policy 可选配置：**
- **volatile-lru ->** 只对设置了过期时间的key进行LRU（Least Recently Used 最近最少使用）
- **allkeys-lru ->** 删除lru算法的key   
- **volatile-random ->** 随机删除即将过期key   
- **allkeys-random ->** 随机删除   
- **volatile-ttl ->** 删除即将过期的   
- **noeviction ->** 永不过期，返回错误  

## swappiness
Linux 会使用硬盘的一部分做为SWAP分区，用来进行进程调度--进程是正在运行的程序--把当前不用的进程调成‘等待（standby）‘，甚至‘睡眠（sleep）’，一旦要用，再调成‘活动（active）’，睡眠的进程就躺到SWAP分区睡大觉，把内存空出来让给‘活动’的进程。
　　如果内存够大，应当告诉 linux 不必太多的使用 SWAP 分区， 可以通过修改 swappiness 的数值。swappiness=0的时候表示最大限度使用物理内存，然后才是 swap空间，swappiness＝100的时候表示积极的使用swap分区，并且把内存上的数据及时的搬运到swap空间里面。
　　在linux里面，默认设置swappiness这个值等于60。
**查看：** $ cat /proc/sys/vm/swappiness
**设置：** sudo sysctl vm.swappiness=10 或者 echo 10 > /proc/sys/vm/swappiness

| swapniess      |     策略 |  
| :-------- |:--------|
| 0    |   Linux3.5以及以上：宁愿OOM killer也不用swap<br>Linux3.4以及更早：宁愿swap也不要OOM killer |
| 1    |   Linux3.5以及以上：宁愿swap也不要OOM killer |
| 60    |   默认值 |
| 100    |   	操作系统会主动地使用swap |

本人在阿里云租借了一服务器默认设置为0
```
[xxxx@XXXX ~]# cat /proc/sys/vm/swappiness
0
[xxxx@XXXX ~]# uname -a
Linux iZ94fflcchzZ 3.10.0-123.9.3.el7.x86_64 #1 SMP Thu Nov 6 15:06:03 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux
```
本地docker 最新centos
```
sh-4.2# cat /proc/sys/vm/swappiness
60
sh-4.2# uname -a
Linux redis_master 4.9.8-moby #1 SMP Wed Feb 8 09:56:43 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
sh-4.2#
```
**在设置为0时要确好内核版本**
相关操作指令
系统信息
```
sh-4.2# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 840180  84916 1025452    0    0     0    10   45   59  0  0 100  0  0
 0  0      0 840180  84916 1025464    0    0     0     0  266  402  0  0 100  0  0
 0  0      0 840180  84916 1025464    0    0     0     0  236  357  0  0 100  0  0
```
单个pid信息
```
sh-4.2# cat /proc/{pid}/smaps |grep Swap
```
因为理解有限大家如需深入理解[请点此处](https://www.douban.com/note/349467816/)
