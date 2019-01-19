---
title: "NSQ 笔记"
date: 2018-10-22T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

**NSQ** 是一个基于 Go 写的实时分布式消息平台, 打开 [NSQ 的官网](https://nsq.io/)可以看到简单粗暴的排版介绍四大优势, Distributed(分布式), Scalable(可拓展), Ops Friendly(对运维友好), Integrated(易集成).

macOS 上安装 NSQ 用 `brew install nsq`  一句就可以了, 或者到 NSQ 的 [Github Releases](https://github.com/nsqio/nsq/releases) 下载下来把可执行文件复制到 **PATH** 也行.

官方文档的快速使用:

1. 打开第一个 Shell

```
$ nsqlookupd
```

2. 打开第二个 Shell

```
$ nsqd --lookupd-tcp-address=127.0.0.1:4160
```

3. 第三个 Shell

```
$ nsqadmin --lookupd-http-address=127.0.0.1:4161
```

4. 发布一条初始化数据, 并在集群中创建一个 topic(如果不存在):

```
$ curl -d 'hello world 1' 'http://127.0.0.1:4151/pub?topic=test'
```

5. 再开一个 Shell 执行 **nsq_to_file**

```
$ nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161
```

6. 发布更多消息到 **nsqd**

```
$ curl -d 'hello world 2' 'http://127.0.0.1:4151/pub?topic=test'
$ curl -d 'hello world 3' 'http://127.0.0.1:4151/pub?topic=test'
```

7. 打开 `http://127.0.0.1:4171/` 可以看到管理的 UI, 验证刚刚执行的一些数据, 也可以查看 `/tmp` 里边 `test.*.log` 的内容

-----------

#### Topic and Channels

NSQ 的消息传递支持 **multicast(多播)** 和 **load-balanced(负载均衡)** 两种方式组合的消息路由

multicast: 一则消息的发布会被所有订阅者接收到
load-balanced: 一则消息的发布会只会被其中的一个订阅者接收到

当一个 consumer 被创建以后, 订阅的是 **topic/channel** 的组合, 而当 producer 被创建以后, 所发布的消息是到 **topic** 这一层的, 然后再复制到每一个不同的 **channel**.

比如, 有 Consumer1, Consumer2, Consumer3 都订阅了 `a_topic/a_channel`, 当 Producer1 发布消息到 `a_topic` 时, 每一则只会被这三个 Consumer 之中的一个接收到, 当发布三则消息的时候, 每个 Consumer 都收到一个. 这是 **load-balanced(负载均衡)**.

再有, 假设当 Consumer1 订阅了 `a_topic/channel1`, Consumer2 订阅了 `a_topic/channel2`, Consumer3 订阅了 `a_topic/channel3`, 此时, 每次 Producer1 发布消息到 `a_topic`, 这三个 Consumer 都将接收到. 这是 **multicast(多播)**.

组合起来, [官网的例子](https://nsq.io/overview/design.html),

![](https://f.cloud.github.com/assets/187441/1700696/f1434dc8-6029-11e3-8a66-18ca4ea10aca.gif)

- Consumer1 订阅了 `clicks/metrics`
- Consumer2 订阅了 `clicks/metrics`
- Consumer3 订阅了 `clicks/metrics`
- Consumer4 订阅了 `clicks/spam_analytics`
- Consumer5 订阅了 `clicks/archives`

当 Producer 发布一则消息 A, Consumer 1/2/3 之中的一个(动图中的第二个 Consumer)会接收到, 另外 Consumer4 和 Consumer5 也都会收到消息 A. 当发布消息 B 时, Consumer4 和 Consumer5 都会接收到消息 B, 而 Consumer 1/2/3 依然只有一个会接收到消息 B(动图中的第一个 Consumer),

----------

NSQ 自带有一系列的 helper 应用, **nsqlookupd**, 用来管理 nsqd 所发布的 topics 以方便客户端发现与查找并且对感兴趣的 topic 进行订阅. 解耦发布与订阅之间的依赖关系, 各自做好自己的事就够了, 有什么都冲着 nsqlookupd 这个进程去.

可以通过 `nsqlookupd --help` 查询到详细的用法, 各个参数的作用. 最主要的就是 **-http-address**(nsqadmin 各种管理用, 默认 0.0.0.0:4161) 和 **-tcp-address**(nsqd 用, 默认 0.0.0.0:4160)

[**nsqd**](https://nsq.io/components/nsqd.html) 是一个负责处理消息的接收, 排队, 以及投递给客户端的守护进程. 尽管 nsqd 可以独立运行, 但是通常和 nsqlookupd 实例配置再一个集群中. 这样的 **nsqd** 进程会有一个与 **nsqlookupd** 的 TCP 长连接, 间隔定时往 nsqlookupd 推送自己的状态信息, 从而 nsqlookupd 就可以告知用户 nsqd 的地址信息.

nsqd 会默认监听一个 tcp 端口(4150)和一个 http 端口(4151), 和可选的 https 端口.

**nsqadmin**, 简单拿 bootstrap 包装了一下各种管理统计数据的 Web UI.

hello world 例子中的 **nsq_to_file** 创建一个订阅指定 topic 的 Consumer, 并写入指定的 file, 除了这个之外还有 `nsq_to_http`, `nsq_to_nsq`

--------------

#### go-nsq

官方包 [https://github.com/nsqio/go-nsq](https://github.com/nsqio/go-nsq), 上面提到的 nsq_to_file 的, 本身就是拿这个[官方包](https://github.com/nsqio/nsq/tree/master/apps)写的 = 。 = 好吧, nsq 就是 Go 写的.

##### Producer

```go
// NewConfig:
// This must be used to initialize Config structs. The only valid way to create a Config is via NewConfig. Values can be set directly, or through Config.Set()
// c.Set("tls_v1", true)
// c.Set("tls-insecure-skip-verify", true)
// c.Set("tls-min-version", "tls1.2")
// c.Set("local_addr", "1.2.3.4:27015")
// c.Set("dial_timeout", "5s")
config := nsq.NewConfig()

//  After Config is passed into NewProducer the values are no longer mutable (they are copied).
p, err := nsq.NewProducer("127.0.0.1:4150", config)
if err != nil {
    log.Fatal(err)
}

// if err := p.Publish("fuji", []byte("X-T3")); err != nil {
// 	log.Fatal("publish error: " + err.Error())
// }
for {
    // synchronously publishes a message body to the specified topic
    if err := p.Publish("test", []byte("test message")); err != nil {
        log.Fatal("publish error: " + err.Error())
    }
    time.Sleep(1 * time.Second)
}
```

##### Consumer

```go
config := nsq.NewConfig()

// NewConsumer creates a new instance of Consumer for the specified topic/channel
// After Config is passed into NewConsumer the values are no longer mutable (they are copied).
consumer, _ := nsq.NewConsumer("fuji", "channel1", config)

// AddHandler sets the Handler for messages received by this Consumer. This can be called
// multiple times to add additional handlers. Handler will have a 1:1 ratio to message handling goroutines.
consumer.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
    log.Printf("收到: %v", message)
    log.Printf("Body: %v", string(message.Body))
    if string(message.Body) == "X-T3" {
        consumer.Stop()
    }
    return nil
}))

// ConnectToNSQLookupd adds an nsqlookupd address to the list for this Consumer instance.
//
// If it is the first to be added, it initiates an HTTP request to discover nsqd
// producers for the configured topic.
//
// A goroutine is spawned to handle continual polling.

err := consumer.ConnectToNSQLookupd("127.0.0.1:4161")
if err != nil {
    log.Panic("连接失败")
}
// func (r *Consumer) ConnectToNSQLookupds(addresses []string) error {

// ConnectToNSQD takes a nsqd address to connect directly to.
// func (r *Consumer) ConnectToNSQD(addr string) error {

// read from this channel to block until consumer is cleanly stopped
<-consumer.StopChan

// Stop will initiate a graceful stop of the Consumer (permanent)
//
// NOTE: receive on StopChan to block until this process completes
// func (r *Consumer) Stop() {


```

