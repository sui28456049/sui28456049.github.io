---
title: Linux常用工具
date: 2018-01-05 18:55:51
tags: Linux
category: Linux
---

# tcpdump抓包工具

运行一个简单的例子
```bash
tcpdump -n -i any tcp port 9501|grep 127.0.0.1
```

 * -n:不进行IP地址到主机名的转换。 
 * -i 参数制定了网卡，any表示所有网卡
 * tcp 指定仅监听TCP协议
 * port 制定监听的端口
 * 需要要看通信的数据内容，可以加 -Xnlps0 参数

运行结果:

```bash
13:29:07.788802 IP localhost.42333 > localhost.9501: Flags [S], seq 828582357, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 0,nop,wscale 7], length 0
13:29:07.788815 IP localhost.9501 > localhost.42333: Flags [S.], seq 1242884615, ack 828582358, win 43690, options [mss 65495,sackOK,TS val 2207513 ecr 2207513,nop,wscale 7], length 0
13:29:07.788830 IP localhost.42333 > localhost.9501: Flags [.], ack 1, win 342, options [nop,nop,TS val 2207513 ecr 2207513], length 0
13:29:10.298686 IP localhost.42333 > localhost.9501: Flags [P.], seq 1:5, ack 1, win 342, options [nop,nop,TS val 2208141 ecr 2207513], length 4
13:29:10.298708 IP localhost.9501 > localhost.42333: Flags [.], ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:10.298795 IP localhost.9501 > localhost.42333: Flags [P.], seq 1:13, ack 5, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 12
13:29:10.298803 IP localhost.42333 > localhost.9501: Flags [.], ack 13, win 342, options [nop,nop,TS val 2208141 ecr 2208141], length 0
13:29:11.563361 IP localhost.42333 > localhost.9501: Flags [F.], seq 5, ack 13, win 342, options [nop,nop,TS val 2208457 ecr 2208141], length 0
13:29:11.563450 IP localhost.9501 > localhost.42333: Flags [F.], seq 13, ack 6, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
13:29:11.563473 IP localhost.42333 > localhost.9501: Flags [.], ack 14, win 342, options [nop,nop,TS val 2208457 ecr 2208457], length 0
```

* 13:29:11.563473 时间带有精确到微妙
* localhost.42333 > localhost.9501 表示通信的流向，42333是客户端，9501是服务器端
* [S] 表示这是一个SYN请求
* [.] 表示这是一个ACK确认包，(client)SYN->(server)SYN->(client)ACK 就是3次握手过程
* [P] 表示这个是一个数据推送，可以是从服务器端向客户端推送，也可以从客户端向服务器端推
* [F] 表示这是一个FIN包，是关闭连接操作，client/server都有可能发起
* [R] 表示这是一个RST包，与F包作用相同，但RST表示连接关闭时，仍然有数据未被处理。可以理解为是强制切断连接
* win 342是指滑动窗口大小
* length 12指数据包的大小

更多用法 `man tcpdump`;

# tcpflow工具(http抓包)

安装:

```
yum -y install tcpflow 
```

# strace

strace可以跟踪系统调用的执行情况，在程序发生问题后，可以用strace分析和跟踪问题。

在Linux世界，进程不能直接访问硬件设备，当进程需要访问硬件设备(比如读取磁盘文件，接收网络数据等等)时，必须由用户态模式切换至内核态模式，通 过系统调用访问硬件设备。strace可以跟踪到一个进程产生的系统调用,包括参数，返回值，执行消耗的时间。


```bash
strace -o /tmp/strace.log -f -p $PID
```

-f 表示跟踪多线程和多进程，如果不加-f参数，无法抓取到子进程和子线程的运行情况
-o 表示将结果输出到一个文件中
-p $PID，指定跟踪的进程ID，通过ps aux可以看到
-tt 打印系统调用发生的时间，精确到微妙
-s 限定字符串打印的长度，如recvfrom系统调用收到的数据，默认只打印32字节
-c 实时统计每个系统调用的耗时
-T 打印每个系统调用的耗时

排查php错误

```
strace -o ./php.strace.log php test.php  -f -c

```

# lsof(一切皆文件)

lsof工具可以查看某个进程打开的文件句柄。可以用于跟踪程序的工作进程所有打开的socket、file、资源。

`lsof -p [进程ID]`

# perf(高级)

perf工具是Linux内核提供一个非常强大的动态跟踪工具，`perf top`指令可用于实时分析正在执行程序的性能问题。与callgrind、xdebug、xhprof等工具不同，perf无需修改代码导出profile结果文件。

使用方法:

```bash
perf top -p [进程ID]
```