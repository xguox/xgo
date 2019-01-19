---
title: "Erlang weekly note 07 - Processes 进程"
date: 2017-08-30T16:01:23+08:00
draft: false
tags: ["Erlang"]
categories: ["Erlang"]
---

为了让几十个任务能同时执行, **Erlang** 采用了 **Actor** 模型, **每个 actor 都是虚拟机中的一个独立进程**. 在 Erlang 中, 并发的基本单位是进程(Actor). 每个进程代表一个持续的活动, 是某段程序代码的执行代理, 与其他按各自的节奏执行自身代码的进程一起并发运行, 进程之间靠消息来通信. **Erlang 的并发是很廉价的, 派生一个进程就跟在 OOP 中分配一个对象的开销差不多.** 更形象一些, 如果你是 Erlang 世界中的一个 actor, 你将会是一个孤独的人, 独自坐在一个没有窗户的黑屋子里, 在你的邮箱旁等待着消息. 当你收到一条消息时, 会用特定的方式来响应这条消息：收到账单就要进行支付；收到生日卡, 就回一封感谢信；对于不理解的消息, 就完全忽略. 人和人之间只能通过写信进行交流, 就是这样.

----

理解进程之前先要说的是 pid 这个特殊的 Erlang 数据类型, Erlang 支持进程编程, **任何代码都需要一个进程作为载体才能执行**, 每个进程都有一个唯一标识符, 也即是 pid. 在 erlang shell 中, 进程的 pid 会以类似<0.62.0>的格式打印,  这个格式只用于调试比较目的.

`self()` 会告诉你当前进程(也就是调用 self() 的那个进程)的 pid.

----

##### PROCESS OPERATIONS

###### 派生进程

派生进程的函数有两个: `spawn/1`, `spawn/3`. 第一个仅有一个参数, 就是用作新进程入口的 fun 函数, 这里要注意的是, 我们无法得到函数F的返回值. 我们只能得到它的 pid, 因为进程不会返回任何东西; 另一个则需要模块名, 函数名, 参数列表三个参数.

```erlang
%% Erlang/OTP 20 [erts-9.0]

%% Eshell V9.0  (abort with ^G)
1> F = fun() -> 2 + 2 end.
#Fun<erl_eval.20.99386804>
2> spawn(F).
<0.63.0>
3> spawn(io, format, ["Hello World!~n"]).
Hello World!
<0.68.0>
```

###### 消息传递

Erlang 的进程之间可以互相使用 `!` 运算符发送消息.

该操作符的左边是一个 pid, 右边可以是任意 Erlang 数据项. 这个数据项会被发送给左边的 pid 所代表的进程, 这个进程就可以访问它了.

```erlang
%% 给当前进程发送一个 hello 原子
1> self() ! hello.
hello
```

发送一些无人会看的消息的用处就和写一些表达自我情绪的诗一样(换句话说, 就是不大有用), 呵呵哒.

因此, 我们还**需要 receive 表达式来接收消息**.

receive 的语法形式如下:
```erlang
receive
     Pattern1 when Guard1 -> Expr1;
     Pattern2 when Guard2 -> Expr2;
     Pattern3 -> Expr3
end
```

举个例子:

```erlang
-module(dolphins).
-compile(export_all).

dolphin1() ->
    receive
        do_a_flip ->
            io:format("How about no?~n");
        fish ->
            io:format("So long and thanks for all the fish!~n");
        _ ->
            io:format("Heh, we're smarter than you humans.~n")
    end.
```

```erlang
1> c(dolphins).
{ok,dolphins}
2> Dolphin = spawn(dolphins, dolphin1, []).
< 0.39.0>
3> Dolphin ! "oh, hello dolphin!".
Heh, we're smarter than you humans.
"oh, hello dolphin!"
4> Dolphin ! fish
fish
```

**函数执行到了 receive 表达式. 因为进程邮箱为空, 所以海豚进程会一直处于等待消息状态.**

收到了消息"oh, hello dolphin!". 函数会先对 do_a_flip 进行模式匹配. 失败了. 然后再尝试模式fish, 也失败了. 最后遇到了匹配一切的子句(_), 匹配成功.

进程打印消息"Heh, we’re smarter than you humans.", 接着, 进程也随之结束. 因此, 再次发送 `fish` 给 `Dolphin` 的时候, 就不会对该消息做任何反应.

----

##### 超时接收

上面提到的, 当进程执行到 receive 表达式以后会一直处于等待消息的状态, 而且如果不做处理就会一直等下去. 出现这样情况的原因有很多, 比如准备发送消息的进程在消息发出之前就崩溃掉了. 为了避免超时问题的出现, 可以在 receive 增加超时设置, 如下:

```erlang
receive
    Match -> Expression1
after Delay ->
    Expression2
end.
```

等待 Delay 毫秒以后如果没有 Match 的消息就会执行 after 中的内容, 在这里就是 `Expression2`

----

##### 注册进程

每个 Erlang 系统都有一个本地进程注册表用于注册进程的简单命名服务. 一个名称一次只能用于一个进程.
可以使用函数 erlang:register(Name, Pid) 为进程命名. 如果进程死亡了, 它会自动失去自己的名字. 也可以使用函数unregister/1手工解除进程的名字注册. 可以调用 `registered/0` 得到所有已注册进程的列表, 或者通过shell命令 `regs()` 得到更详细的信息. 使用内置函数 `whereis/1` 可以查找当前与指定注册名对应的 pid.

假设某个进程崩溃了, 对应的服务被重启, 新的服务进程 pid 将会改变,  此时无需逐个通知给系统中的所有进程, 只要更新进程注册表就可以了.

```erlang

1> registered().
[application_controller,user,standard_error,erl_prim_loader,
 inet_db,init,erts_code_purger,rex,error_logger,user_drv,
 kernel_sup,global_name_server,erl_signal_server,
 standard_error_sup,file_server_2,global_group,
 kernel_safe_sup,code_server]
2> regs().

** Registered procs on node nonode@nohost **
Name                  Pid          Initial Call                      Reds Msgs
application_controlle <0.33.0>     erlang:apply/2                     551    0
code_server           <0.38.0>     erlang:apply/2                  121988    0
erl_prim_loader       <0.6.0>      erlang:apply/2                  139520    0
erl_signal_server     <0.47.0>     gen_event:init_it/6                 51    0
error_logger          <0.32.0>     gen_event:init_it/6                322    0
erts_code_purger      <0.1.0>      erts_code_purger:start/0             4    0
file_server_2         <0.46.0>     file_server:init/1                 113    0
global_group          <0.45.0>     global_group:init/1                 74    0
global_name_server    <0.41.0>     global:init/1                       63    0
inet_db               <0.44.0>     inet_db:init/1                     348    0
init                  <0.0.0>      otp_ring0:start/2                  818    0
kernel_safe_sup       <0.55.0>     supervisor:kernel/1                 83    0
kernel_sup            <0.37.0>     supervisor:kernel/1               2162    0
rex                   <0.40.0>     rpc:init/1                          32    0
standard_error        <0.49.0>     erlang:apply/2                      11    0
standard_error_sup    <0.48.0>     supervisor_bridge:standar           50    0
user                  <0.52.0>     group:server/3                      87    0
user_drv              <0.51.0>     user_drv:server/2                 2703    0

** Registered ports on node nonode@nohost **
Name                  Id              Command
ok
4> whereis(user).
<0.52.0>

```

##### 链接 Link

链接(link)是两个进程之间的一种特殊关系. 当在两个进程间建立了这种关系后, 如果其中一个进程由于意外的抛出, 出错或者退出而死亡时, 另外一个进程也会死亡, 把这两个进程独立的生存期绑定成一个关联在一起的生存期.

Erlang 中有一个原生函数 `link/1`, 用于在两个进程间建立一条链接, 它的参数是进程的pid. 当调用它时, 会在当前进程和参数pid标识的进程之间建立一条链接. 要去除链接, 可以使用 `unlink/1`.

当链接进程中的一个死亡时, 会发送一条特殊的消息, 其中含有死亡原因相关的信息. 如果进程正常死亡了(函数正常执行完毕), 就不会发送这条消息.

注意, `link(spawn(Function))` 或者 `link(spawn(M,F,A))` 并不是一个原子操作. 有时, 进程会在链接建立成功之前死亡, 从而导致不期望的行为. 因此, Erlang 中增加了 `spawn_link/1` 和`spawn_link/3` 函数, 对应 `spawn/1` 和 `spawn/3`, 创建一个进程, 并和它建立链接, 这个操作是原子级的,  也就意味着要么成功, 要么失败.

```erlang
-module(linkmon).
-export([myproc/0]).
myproc() ->
    timer:sleep(5000),
    exit(reason).

1> self().
<0.60.0>
2> c(linkmon).
{ok,linkmon}
3> spawn_link(fun linkmon:myproc/0).
<0.68.0>
** exception error: reason
4> self().
<0.70.0>
```

##### 监控器 Monitor

有时候链接可能不是你想要的, 也许只是想知道目标进程挂了而不想牵连着一起被杀死, 那么这时候就要用监控器(Monitor). 监控器其实是一个单向的链接, 可以让一个进程在不影响目标进程的情况下对目标进程进行监视.

创建监控器的函数是 `erlang:monitor/2`, 它的第一个参数永远是原子 **process**, 第二个参数是进程的 Pid.

```erlang
1> erlang:monitor(process, spawn(fun() -> timer:sleep(500) end)).
#Ref<0.3856117142.833355780.165079>
2> flush().
Shell got {'DOWN',#Ref<0.3856117142.833355780.165079>,process,<0.62.0>,normal}
ok
3>
```

每当被监控的进程死亡时, 监控进程都会收到一条消息, 格式为 `{'DOWN', MonitorReference, process, Pid, Reason}`

其中, `MonitorReference` 可以用来解除对一个进程的监控. 记住, 监控器是可叠加的, 因此会收到多条 DOWN 消息. 引用可以唯一确定一条 DOWN 消息. 类似 `spawn_link` 的也有原子级的 `spawn_monitor`
