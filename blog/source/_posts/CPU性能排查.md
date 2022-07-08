---
title: CPU性能排查 
categories:
- Linux 
tags:
- Linux
date: 2022-07-07 17:18:19
---

## 简介

在使用计算机的过程，必然会用到计算资源，CPU全称为中央处理器。

其中包含了，控制器，运算器，储存器，三个组成部分。


## 执行过程

* 提取指令
* 解码
* 执行

## 性能指标

* 等待运行队列
* 平均负载
* CPU使用率
* 上下文切换

#### 等待运行队列

vmstat命令中，r和b参数可以进行查询

#### 平均负载 uptime

```
bash-5.0# uptime
 18:42:52 up 174 days,  2:13,  0 users,  load average: 7.81, 6.88, 6.66
```

load average后面三个数值分别代表 1分钟，5分钟，15分钟，CPU使用的百分比

#### CPU使用率

通常我们会关注系统不同部分使用CPU的情况

* 用户进程时间百分比
* 系统进程和中断的时间百分比
* IO等待使用CPU （idle）时间百分比
* 空闲

其中等待IO是一个我们值得关注的值，在IO密集型应用中，等待IO应该越小越好 空闲百分比太低，表示CPU资源紧张

#### top 和 vmsata 命令 查看CPU使用率

top

第一行 展示展示内存相关内容

第二行 CPU使用情况相关情况

* total	进程总数
* running	正在运行的进程数
* sleeping	睡眠的进程数
* stopped	停止的进程数
* zombie	僵尸进程数
* %Cpu(s)
* 0.0 us	用户空间占用CPU百分比
* 0.1 sy	内核空间占用CPU百分比
* 0.0 ni	用户进程空间内改变过优先级的进程占用CPU百分比
* 98.7 id	空闲CPU百分比;
* 0.0 wa	等待输入输出的CPU时间百分比
* 0.0 hi	硬件CPU中断占用百分比
* 0.0 si	软中断占用百分比
* 0.0 st	虚拟机占用百分比 

第三行 展示信息与uptime命令内容相同

#### vmstat

下面是vmsat参数说明

安装命令：apk add procps

```
bash-5.0# vmstat
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 9  0      0 29328352 5430196 74941208    0    0     0    31    0    0 16  9 74  0  0
```

###### Progress进程说明

* r 等待进程数
* b 等待io进程数

###### Swap

* si 每秒从交换区写到内存的大小
* so 每秒写入交换区的内存大小，从内存调入磁盘

如果这两个值长期大于0，系统性能会受到影响，磁盘IO和CPU都会被额外消耗

###### IO

* bi 每秒读取块数
* bo 每秒写入块数

随机磁盘读写中，bi，bo这两个值越大，CPU的IO等待值就越大

###### system 系统

* in ： 每秒中断数
* cs： 每秒上下文切换数

两者值越大，内核消耗CPU时间越大

###### CPU（以百分比表示）

* us：用户进程进行时间百分比
* sy：内核系统进程执行时间百分比
* id：空闲时间百分比
* wa：IO等待时间百分比，磁盘大量随机读写时会偏高
* st：虚拟机占用CPU的百分比

##### pidstat

```bash
bash-5.0# pidstat
Linux 3.10.0-957.el7.x86_64 (watcher-test-59df69f65f-l2dnd)     07/07/22        _x86_64_        (24 CPU)

18:45:13      UID       PID    %usr %system  %guest   %wait    %CPU   CPU  Command
18:45:13        0         1    0.00    0.00    0.00    0.00    0.00     7  supervisord
18:45:13        0         8    0.00    0.00    0.00    0.00    0.00    13  study_admin
18:45:13        0        34    0.00    0.00    0.00    0.00    0.00     8  bash
18:45:13        0       123    0.00    0.00    0.00    0.00    0.00     3  bash
```

* PID：进程ID
* %usr：进程在用户空间占用cpu的百分比
* %system：进程在内核空间占用cpu的百分比
* %guest：进程在虚拟机占用cpu的百分比
* %CPU：进程占用cpu的百分比
* CPU：处理进程的cpu编号
* Command：当前进程对应的命令

#### 上下文切换

发生上下文切换的场景

* 公平调度算法
* 资源不足（内存）
* 主动挂起 sleep
* 优先级
* 硬件中断

安装 apk add sysstat

```
bash-5.0# pidstat -w
Linux 3.10.0-957.el7.x86_64 (watcher-test-59df69f65f-l2dnd)     07/07/22        _x86_64_        (24 CPU)

18:45:57      UID       PID   cswch/s nvcswch/s  Command
18:45:57        0         1      0.00      0.00  supervisord
18:45:57        0         8      0.01      0.00  study_admin
18:45:57        0        34      0.00      0.00  bash
18:45:57        0        63      0.00      0.00  watch
18:45:57        0        65      0.00      0.00  bash
18:45:57        0       123      0.00      0.00  bash
18:45:57        0       137      0.00      0.00  pidstat
```

* cswch 自愿上下文切换 比如IO，内存不足，就会发生切换
* nvcswch 非自愿，可能由于进程时间片用完，被系统强制切换

#### 模拟上下文切换

sysbench

#### 参考链接

https://www.hi917.com/detail/434.html
https://www.brendangregg.com/
https://blog.csdn.net/weixin_44175418/article/details/124986740
https://blog.csdn.net/ligupeng7929/article/details/89888768
https://blog.csdn.net/A642960662/article/details/123030264
https://blog.csdn.net/lin443514407lin/article/details/54342754
https://time.geekbang.org/column/intro/100020901
https://cloud.tencent.com/developer/article/1513544
