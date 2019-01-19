---
title: "go build timeout in docker"
date: 2018-12-12T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

尝试用 docker 部署个简单的 go 应用, 然后发现 build 不起来. 因为一些神秘力量, 会出现类似下面的一些错误信息.

**unrecognized import path "cloud.google.com/go"**

**unrecognized import path "golang.org/x/sync"**

**i/o timeout**

想着总不能在服务器上用神秘的方式去解决这股力量吧 = 。 =

幸好 **go 1.11** 的 modules 功能可以启用 vendor 模式

**go mod vendor**

这样, 本地拉好相关依赖就可以了. **Dockerfile** 的 `RUN` 命令后面加 **-mod vendor**

```go
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -mod vendor
```

---------

或者还有一种办法, 比如:

如果报错

```go
go: golang.org/x/sys@v0.0.0-20180903190138-2b024373dcd9: unrecognized import path "golang.org/x/sys" (https fetch: Get https://golang.org/x/sys?go-get=1: dial tcp 216.239.37.1:443: i/o timeout)
```

可以用 **go mod edit** 把对应的源换成 github 的,

```go
go mod edit -replace=golang.org/x/sys@v0.0.0-20180903190138-2b024373dcd9=github.com/golang/sys@v0.0.0-20180903190138-2b024373dcd9
```