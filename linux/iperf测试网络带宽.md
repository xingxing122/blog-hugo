---
title: "Iperf测试网络带宽"
date: 2019-08-28T16:31:05+08:00
draft: false  
categories: [linux]
tags: [linux]

---

iperf 的基本用法

<!--more-->

iperf 是一个网络性能测试工具，iperf 可以测试TCP 和UDP 带宽质量

```bash
 iperf -c  ip  -u -m -t 60 -i 10 -b 1000M 
------------------------------------------------------------
Client connecting to 10.37.57.104, UDP port 5001
Sending 1470 byte datagrams, IPG target: 11.22 us (kalman adjust)
UDP buffer size:  208 KByte (default)
------------------------------------------------------------
[  3] local 10.39.23.47 port 39241 connected with 10.37.57.104 port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0-10.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] 10.0-20.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] 20.0-30.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] 30.0-40.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] 40.0-50.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] 50.0-60.0 sec  1.22 GBytes  1.05 Gbits/sec
[  3] WARNING: did not receive ack of last datagram after 10 tries.
[  3]  0.0-60.0 sec  7.32 GBytes  1.05 Gbits/sec
[  3] Sent 5349878 datagrams

iperf 用法
-c 以客户端模式运行，并制定服务端的地址
-b 指定客户端通过UDP 协议发送信息的带宽
-d 同时进行双向传输测试
-n 指定传输的字节数
-r 单独进行双向传输测试
-t 指定iperf测试的时间，默认10s 
-i 设置每次报告之间的时间间隔，单位为秒。


TCP 测试
iperf -c host -i 1 -w 1M  #hosts为ip 地址

UDP 测试
iperf -u -c 10.2.1.2 -b 100M  -i 1  -w 1M  -t 60

```

