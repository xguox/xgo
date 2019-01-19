---
title: "查看 TCP 端口号占用"
date: 2018-11-01T16:01:23+08:00
draft: false
tags: ["Tools", "Linux"]
categories: ["Tools", "Linux"]
---

查看 TCP 端口被哪个进程(PID)占用了

```go
lsof -nP -i4TCP:$PORT | grep LISTEN
```