---
title: "Linux性能优化笔记"
date: 2019-05-31T10:28:18+08:00
draft: false  
categories: [linux]
tags: [linux]

---

linux 性能优化笔记

<!--more-->



linux 性能优化图

![image-20190531104153256](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-05-31-024153.png)

一、 CPU 

系统平均负载

```bash
uptime
 10:49:50 up 149 days, 22:43,  2 users,  load average: 0.00, 0.01, 0.05

10:49:50 //当前时间
up 149 days, 22:43 //系统运行时间
2 users          // 正在登陆用户数

CPU 统计
grep 'model name' /proc/cpuinfo |wc -l
或者
nproc
或者
lscpu 
```

安装stress 和sysstat

```bash
yum  install  http://download-ib01.fedoraproject.org/pub/epel/7/x86_64/Packages/s/stress-1.0.4-16.el7.x86_64.rpm
stress 是一个linux 系统压力测试工具，模拟系统负载较高时场景
     -c 产生n个进程，每个进程都反复不停的计算随机数的平方根
     -i 产生n个进程，每个进程反复sync()将内存上
```

CPU 高使用的工具

负载高的时候可以使用以下工具查看

```bash
mpstat 是一个常用的linux 性能分析工具，用来实时查看每个CPU 的性能指标，以及所有CPU 的平均指标
      -P{|ALL} 表示监控哪个CPU，在0，CPU 个数-1 中取值
      internal:相邻两个采样的间隔时间
      count: 采样的次数，count 只能和delay 一起使用      
mpstat -P ALL 1  间隔为1s 

pidstat 查看进程CPU 使用情况
pidstat -u 5 1  观察

```

平均负载

```bash
简单来说，平均负载是指单位时间内，系统处于可运行状态和不可中断状态的平均进程数，也就是平均活跃进程数，它和CPU使用率并没有直接关系
```



可运行状态的进程

```bash
所谓可运行状态的进程，是指正在使用CPU或者正在等待CPU的进程，也就是我们常用ps 命令看到的，处于R 状态(Running或者Runnable)的进程
```



平均负载与CPU 使用率的关系

```bash
平均负载是指单位时间内处于可运行状态和不可中断状态的进程数。所以它不仅包括了正在使用CPU的进程，好包括等待CPU和等待IO的进程。而CPU使用率，是单位时间内CPU繁忙情况的统计，跟平均负载并不一定完全对应，比如：
​ CPU 密集型进程，使用大量CPU会导致平均负载升高，此时这两者是一致。
​ IO密集型进程，等待IO 也会导致平均负载升高，但CPU使用率不一定很高
​ 大量等待CPU的进程调度也会导致平均负载升高，此时的CPU使用率也会比较高         
```

5. 不可中断状态的进程

```bash
不可中断状态的进程是处于内核态关键流程中的进程，且这些流程是不可打断的，比如常见的是等待硬件设备的IO响应，也就是我们在ps命令中看到的D 状态的进程。比如当一个进程向磁盘写数据时，为了保证数据的一致性，在得到磁盘回复前，它是不能被其他进程中断打断的，这时进程就处于不可中断状态。所以，不可中断状态实际是系统对进程和硬件设备的一种保护机制。
平均负载其实就是平均活跃进程数，平均活跃进程数，直观上的理解就是单位时间内的活跃进程数，但它实际上是活跃进程数的指数衰减平均值。那么最理想的，就是每个CPU上都刚好运行着一个进程，这样每个CPU 都得到充分利用，
```

案例

![image-20190605155323568](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-075324.png)

使用top 查看CPU 消耗 

![image-20190605155526856](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-075526.png)

top  

%CPU 状态信息

```bash
user  用户控件占用CPU 百分比

system  内核空间占用CPU 百分比

nice  改变过优先级的进程占用CPU 的百分比

idle   空闲CPU 百分比

iowait IO等待占用CPU 的百分比

hi  硬中断(Harware IRQ) 占用CPU的百分比

si  软中断(Software interrupts) 占用CPU百分比
```

PID  4872 这个进程可疑 

在这里机器被重启之后进程改变为1258  

```bash
yum install psmisc  -y  安装的是pstree  
```

![image-20190605161931908](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-081932.png)

```bash
top -H -p 1258 查看出线程 
yum install gdb  -y
```

gdb attch  1258

![image-20190605162959643](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-083000.png)

![image-20190605163028512](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-083029.png)

输出线程

![image-20190605163308909](https://xing-blog.oss-cn-beijing.aliyuncs.com/2019-06-05-083309.png)

pstack  显示进程的栈跟踪

```bash
pstack  1258
```

进程和线程的区别

```
进程是资源分配和执行的基本单位;线程是任务调度和运行的基本单位。线程没有资源，进程给指针提供虚拟内存、栈、变量等共享资源，而线程可以共享进程的资源。 
```

查看系统的上下文切换

```bash
vmstat
每隔3s输出一组数据
cs(context  switch) 表示每秒上下文切换次数
in(interrupt) 表示每秒中断的次数
 r(running of runnable)表示就绪队列的长度，也就是正在运行和等待CPU的进程数
 b(Blocked) 表示处于不可中断睡眠状态的进程数
```

故障解决工具使用:

如果top 查看某个进程使用的CPU 高，可以使用pidstat 查看进程使用

pidstat   -p  

如果进程不断重启的话，创建新进程的话使用

pstree  查看父进程

perf 用来分析CPU 性能分析 

perf   record  -g   等待15s  后按Ctrl+C 退出，然后运行perf   report 查看报告

进程的状态

使用top 会看到进程显示的状态

R 是Running 或者Runnable 的缩写，表示进程在CPU 的就绪队列中，正在运行或者正在等待运行

D 是Disk Sleep的缩写，也就是不可中断状态睡眠(Uninterruptible Sleep),一般表示进程正在跟硬件交互，并且交互过程不允许被其他进程或中断打断

Z  是Zombie 的缩写，表示僵尸进程，也就是已经结束了，父进程没有回收它的资源

S 是Interruptible Sleep 的缩写，也就是中断状态睡眠，表示进程因为等待某个事件而被系统挂起，当进程等待事件发生时，它会被唤醒并进入R 状态

I 是ldle的缩写，也就是空闲状态，用在不可中断睡眠的内核线上，



iowait 升高

iowait 升高需要先使用dstat、pidstat 等工具查看CPU和I/O 这两种资源的使用情况 



linux中的软中断

查看软中断和内核线程 

cat /proc/softirqs   查看软中断运行情况

cat /proc/interrupts  查看硬中断的运行情况

```bash
 cat /proc/softirqs 
                    CPU0       CPU1       CPU2       CPU3       
          HI:          0          0          2          0
       TIMER:  246520371  287657513  276025689  471741036
      NET_TX:       5813       7255      10618      11953
      NET_RX:    8457973   11947129   12194105 2090436989
       BLOCK:          0          0          0          0
BLOCK_IOPOLL:          0          0          0          0
     TASKLET:         98        154        163        156
       SCHED:   95782931   95760089   90090419  123126076
     HRTIMER:          0          0          0          0
         RCU:  179145718  200933026  196728001  293878650
```



 



