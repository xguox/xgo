---
title: "Ruby 的并发, 进程, 线程, GIL, EventMachine, Celluloid"
date: 2017-08-31T16:01:23+08:00
draft: false
tags: ["Ruby"]
categories: ["Ruby"]
---

关于并发与并行, 前不久刚好真实发生. 同事一行人去 Family Mart 买午餐, 拿回来公司只有一个微波炉加热, 在 Family Mart 有两个微波炉可以加热. 也就是说, 我们一行人一起去买午餐这是一个并发的程序, 然后在 Family Mart 可以并行加热, 但是, 如果拿回公司的话, 因为只有一个微波炉(单核), 所以是只能一个接一个顺序执行.

# Processes
## 串行执行

给一个 range, 转成 array 以后获取某个特定数字的 index

```ruby
range = 0...100_000_000
number = 99_999_999
puts range.to_a.index(number)

➜ time ruby sequential.rb
99999999
ruby sequential.rb  4.04s user 0.56s system 98% cpu 4.667 total
```

执行这段代码大约耗时 5s, 利用了 99% 的CPU(单核).

## 并行执行

通过分割 range 以及 Ruby 标准库中的 `fork` 方法, 我们可以使用多个进程来执行上面的代码.


```ruby
range1 = 0...50_000_000
range2 = 50_000_000...100_000_000
number = 99_999_999
puts "Parent #{Process.pid}"
fork { puts "Child1 #{Process.pid}: #{range1.to_a.index(number)}" }
fork { puts "Child2 #{Process.pid}: #{range2.to_a.index(number)}" }
Process.wait

➜ time ruby parallel.rb
Parent 5086
Child2 5100: 49999999
Child1 5099:
ruby parallel.rb  3.73s user 0.43s system 192% cpu 2.162 total
```

在多核 CPU 下, 因为是把 range 对半拆开, 所以, 处理时间也快了. (数字有时候会有一些差距). `Process.wait` 是等待所有子进程完成的意思.


因为 GIL(global interpreter lock) 的存在, Ruby MRI 要利用 CPU 的核数唯一方法就是多进程.

[Unicorn](https://bogomips.org/unicorn/) 就是通过 fork Rails 主进程衍生出多个 workers 来处理 HTTP 请求的.

这种多进程的方式最大优点是充分的利用多核 CPU, 且进程间不共享内存, 因此 debug 也会容易一些, 但是因为不共享内存, 也就意味着内存消耗会变大. 不过从Ruby 2.0 开始使用系统的 COW(Copy On Write) 功能可以让 fork 的进程共享内存数据, 只有在数据改变时候才复制数据.

# Threads

Ruby 从 1.9 开始, 线程的实现改为系统的 Native 线程, 线程也由操作系统来完成调度. 但是由于 GIL 的存在, 同一时间一个 Ruby 进程只会有一个线程获取到 GIL 在跑. GIL 的存在意义就是线程安全, 防止发生竞争条件, 比起在数据结构上实现线程安全,  GIL 的实现方式更为容易一些.

然而, 其实, GIL 并不能完全阻止竞争条件的发生 = . =

**Race Condition**

```ruby
# threads.rb
@executed = false
def ensure_executed
  unless @executed
    puts "executing!"
    @executed = true
  end
end
threads = 10.times.map { Thread.new { ensure_executed } }
threads.each(&:join) # 主线程等待子线程执行完毕再继续往后面跑, 比如在后面 p 'done'.

➜ ruby threads.rb
executing!
executing!
executing!
executing!
```

因为所有的线程共享一个 `@executed` 变量, 并且读(`unless @executed`)写(`@executed = true`)操作又不是原子级的, 也就是说当在某个线程中读这个 `@executed` 的值的时候,  可能在别的线程已经把 `@executed` 给改写了.

**GIL and Blocking I/O**

GIL 不能让多个线程同时跑的话那还要多线程来干啥? 其实还是有其高端的地方的. 当线程遇到**阻塞 I/O**时就会释放 GIL 给到其他线程, 比如 HTTP 请求, 数据库查询, 磁盘读写, 甚至 `sleep`  .

```ruby
# sleep.rb
threads = 10.times.map do |i|
  Thread.new { sleep 1 }
end
threads.each(&:join)

➜ time ruby sleep.rb
ruby sleep.rb  0.09s user 0.03s system 9% cpu 1.146 total
```

十个线程执行 `sleep` 并不需要执行 10s. 线程执行到 `sleep` 执行权就会递交出去. 相比起使用进程, 线程更轻量一些, 你甚至可以跑好几千个线程, 处理阻塞I/O 的操作也非常有用. 但是, 得要小心 **race-conditions**. 如果用了互斥锁(Mutex)来避免的话, 又得注意死锁之类的出现. 此外, 线程之间的切换也是会有消耗的, 所以, 太多线程的话, 会把时间都花在切换线程上了.

[Puma](https://github.com/puma/puma) 允许在每个进程中使用多线程, 每个进程都有各自的线程池. 大部分时候不会遇到上面说的竞争问题, 因为每个 HTTP 请求都是在不同的线程处理.

# EventMachine

EventMachine(EM) 是一个基于 Reactor 设计模式提供事件驱动(event-driven) I/O 的 gem.
使用 EventMachine 的一个好处就是当处理大量 IO 操作的时候, 不需要手工处理多线程, EM 可以在一个线程里边处理多个 HTTP 请求.

```ruby
# em.rb
require 'eventmachine'

EM.run do
  EM.add_timer(1) do
    puts 'sleeping...'
    EM.system('sleep 1') { puts "woke up!" }
    puts 'continuing...'
  end
  EM.add_timer(3) { EM.stop }
end

➜ ruby em.rb
sleeping...
continuing...
woke up!
```

`EM.system` 模拟了 I/O 操作, 并传入一个 block 作为回调, 在等待回调期间可以继续响应别的操作(事件). 但是复杂的系统因为要处理成功与失败的回调, 而且可能回调内部嵌套着其他事件和回调, 因此, 很容易就会陷入 **Callback Hell**

# Fiber

用的巨少, 语法掌握了又忘, 语法掌握了又忘 = . =

[Fiber](https://ruby-doc.org/core-2.4.1/Fiber.html) 是 Ruby 1.9 开始新增的轻量级运行单元. 类似线程的暂停, 恢复执行代码, 最大的区别在于, 线程是由于操作系统调度执行, 而 Fiber 是由程序员自己处理调度.

```ruby
# fiber.rb
fiber = Fiber.new do
  # 3. Fiber.yield 交出执行权, 并返回 1 (在这里)
  Fiber.yield 1
  # 5. 执行完毕, 返回 2
  2
end
# 1. fiber 创建以后不会执行里边的代码

# 2. 调用 resume 才会执行
puts fiber.resume

# 4. Go on here...
puts fiber.resume

# 5. fiber 已挂, 报错
puts fiber.resume

➜ ruby fiber.rb
1
2
dead fiber called (FiberError)
```

Fiber 结合 EventMachine 可以避免 **Callback Hell**

```ruby
EventMachine.run do
  page = EM::HttpRequest.new('https://google.ca/').get
  page.errback { puts "Google is down" }
  page.callback {
    url = 'https://google.ca/search?q=universe.com'
    about = EM::HttpRequest.new(url).get
    about.errback  { ... }
    about.callback { ... }
  }
end
```
使用 Fiber 重写

```ruby
EventMachine.run do
  Fiber.new {
    page = http_get('http://www.google.com/')
    if page.response_header.status == 200
      about = http_get('https://google.ca/search?q=universe.com')
      # ...
    else
      puts "Google is down"
    end
  }.resume
end
def http_get(url)
  current_fiber = Fiber.current
  http = EM::HttpRequest.new(url).get
  http.callback { current_fiber.resume(http) }
  http.errback  { current_fiber.resume(http) }
  Fiber.yield
end
```

如果在 Fiber 中执行 I/O 操作, 整个线程将会被这个 fiber 阻塞住, 终归还是 GIL 的原因, 其实并没有真正的解决并发的问题, 而且 Fiber 的语法估计我后天又忘了.

# Celluloid

Celluloid 借鉴了 Erlang 给 Ruby 带来了相似的 Actor 模型. 每个 `include` 了 `Celluloid` 的类都变成了一个拥有自己的执行线程的 Actor.

```ruby
class Sheen
  include Celluloid

  def initialize(name)
    @name = name
  end

  def set_status(status)
    @status = status
  end

  def report
    "#{@name} is #{@status}"
  end
end

irb(main):009:0> charlie = Sheen.new "Charlie Sheen"
=> #<Celluloid::Proxy::Cell(Sheen:0x3ffd8ac62b50) @name="Charlie Sheen">
irb(main):010:0> charlie.set_status "winning!"
=> "winning!"
irb(main):011:0> charlie.report
=> "Charlie Sheen is winning!"
irb(main):012:0> charlie.async.set_status "asynchronously winning!"
=> #<Celluloid::Proxy::Async(Sheen)>
irb(main):013:0> charlie.report
=> "Charlie Sheen is asynchronously winning!"
```

`Celluloid::Proxy::Async` 对象会截获方法的调用, 然后保存到 Actor 并发对象的调用队列中, 程序不必等待响应就可以往下执行(异步). 每个并发对象都有一个自己调用队列, 并且按顺序地一个接一个执行里边的方法调用.

Actor 之间通过发送消息来交流, 而与 Erlang 的 Actor 模型最大的区别就在于 **Erlang 的变量是不可变的**, 而 Ruby 没有这个限制, 所以, 消息(对象)在传递的过程就可能会被修改了, 除非 `freeze`?

----

[https://engineering.universe.com/introduction-to-concurrency-models-with-ruby-part-i-550d0dbb970](https://engineering.universe.com/introduction-to-concurrency-models-with-ruby-part-i-550d0dbb970)

[https://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil](https://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)

[http://merbist.com/2011/02/22/concurrency-in-ruby-explained/](http://merbist.com/2011/02/22/concurrency-in-ruby-explained/)

[https://www.slideshare.net/mperham/actors-and-threads](https://www.slideshare.net/mperham/actors-and-threads)

[http://practicingruby.com/articles/gentle-intro-to-actor-based-concurrency](http://practicingruby.com/articles/gentle-intro-to-actor-based-concurrency)