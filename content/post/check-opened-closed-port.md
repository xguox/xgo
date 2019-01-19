---
title: "查看 VPS 上打开/关闭的端口号"
date: 2015-11-23T16:01:23+08:00
draft: false
tags: ["Linux"]
categories: ["Linux"]
---

Mark 一下一个有用的命令, 运维比较渣没有系统研究过, 唯有见一个标一个 = . =

```shell

netstat -lpn  list current enable port

```

输出一般如下:

```shell

Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:9300          0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      -
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -
udp        0      0 0.0.0.0:54328           0.0.0.0:*                           -
udp        0      0 127.0.0.1:123           0.0.0.0:*                           -
udp        0      0 0.0.0.0:123             0.0.0.0:*                           -
udp6       0      0 :::123                  :::*                                -
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   PID/Program name    Path
unix  2      [ ACC ]     SEQPACKET  LISTENING     7466     -                   /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     9525     -                   /var/run/nscd/socket
unix  2      [ ACC ]     STREAM     LISTENING     7126     -                   @/com/ubuntu/upstart
unix  2      [ ACC ]     STREAM     LISTENING     11561    -

```
