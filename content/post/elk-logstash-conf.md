---
title: "Logstash 的日志输出"
date: 2018-11-27T16:01:23+08:00
draft: false
tags: ["Elasticsearch"]
categories: ["Elasticsearch"]
---

Logstash 配置文件的 output 加了一句

```javascript
stdout { codec => rubydebug }
```

本地测试时候看输出用的, 发上去没把这句删掉然后差点把服务器的硬盘撑坏了.

=。 =

去掉这句就不在把正常输出的写进日志, 但是, 还是会有一些 WARN 和 ERROR 的, 在 **/etc/logstash/startup.options** 里边的 `LS_OPTS` 加上 `--quiet` 以后就只剩 ERROR 的.

**--quiet**
Quieter Logstash logging. This causes only errors to be emitted.

**--verbose**
More verbose logging. This causes info level logs to be emitted.

**--debug**
Most verbose logging. This causes debug level logs to be emitted.

这个配置改完还得重装一遍 logstash

**# After changing anything here, you need to re-run $LS_HOME/bin/system-install**

LS_HOME 在 startup.options 这个文件里边有定义