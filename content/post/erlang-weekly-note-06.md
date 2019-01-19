---
title: "Erlang weekly note 06 - Exceptions 异常"
date: 2017-08-26T16:01:23+08:00
draft: false
tags: ["Erlang"]
categories: ["Erlang"]
---


Let it crash!

Erlang 的异常有三类: 出错(error), 退出(exit), 抛出(throw), 针对每种异常, 都有一个与之对应的用于抛出异常内置函数,  **throw(Exception)**, **exit(Exception)**或者 **error(Exception)** 触发.


- error: 这类是运行时异常, 在发生除零错误, 匹配运算失败, 找不到匹配的函数子句等情况时触发, 这些异常的特点在于一旦它们促使某个进程崩溃, Erlang 的错误日志管理器便会将之记录在案.

```erlang
2 div 0.
** exception error: an error occurred when evaluating an arithmetic expression
    in operator  div/2
        called as 2 div 0

lists:reverse(123).
** exception error: no function clause matching lists:reverse(123) (lists.erl, line 147)
```

- exit: 这类异常用于通报 「进程即将停止」. 它们会在迫使进程崩溃的同时将进程崩溃的原因告知给其他进程,  exit 也在进程正常终止的时候使用. 当你确定想要终止当前进程时就用 `exit(Why)`, 如果这个异常错误没有被捕获到,  信号 `{'EXIT', Pid, Why}` 就会被广播给当前进程链接的所有进程

- throw: 当期望程序员处理所发生的异常时, 可以使用抛出（throw）异常. 与错误异常和退出异常不同, 抛出异常并没有"Let it crash!"的意思, 只是为了改变控制流. 如果进程没能捕获 `throw` 异常便会转变为一个原因为 **nocatch** 的 **error** 异常,  迫使异常终止并记录日志

```erlang
1> throw(permission_denied).
** exception throw: permission_denied
```

抛出异常, 出错异常和退出异常都可以使用`try ... catch`来处理. 跟很多语言类似,  after 相当于 finally, ensure 的.

```erlang
try Expression of
    SuccessfulPattern1 [Guards] ->
        Expression1;
    SuccessfulPattern2 [Guards] ->
        Expression2
catch
    TypeOfError:ExceptionPattern1 ->
        Expression3;
    TypeOfError:ExceptionPattern2 ->
        Expression4
after
    AfterExpr
end.
```

或者

```erlang
try
    Expression1,
    Expression2,
    Expression3
catch
end.
```

----

[Erlang weekly note 07 - Processes 进程](http://xguox.me/erlang-weekly-note-07.html)

##### 并发程序中的异常

设想一个只有单一顺序进程的系统. 如果这个进程挂了, 麻烦可能就大了, 因为没有其他进 程能够帮忙. 出于这个原因, 顺序语言把重点放在故障预防上, 强调进行防御式编程.

而 Erlang 关于构建容错式软件的理念可以总结成两个容易记忆的短句:"让其他进程修复错 误”(link 和 monitor)和"任其崩溃".

在 Erlang 里, 我们有大量的进程可供支配, 因此任何单进程故障都不算特别重要. 通常只需 编写少量的防御性代码, 而把重点放在编写纠正性代码上.

###### 捕捉退出信号

跨进程的错误(崩溃)传播对进程来说和消息传递类似, 不过使用的是一种称为信号(signal)的特殊消息.

Erlang 的进程有两种: **普通进程**和**系统进程**. spawn 创建的是普通进程. 普通进程可以通过执行内置函数 `process_flag(trap_exit, true).` 变成系统进程.

当系统进程收到错误信号时, 该信号会被转换成 `{'EXIT', Pid, Why}` 形式的消息. Pid 是终止进程的标识, Why 是终止原因(有时候被称为退出原因). 如果进程是无错误终止,  Why 就会是原子 **normal**, 否则 Why 会是错误的描述.

当普通进程收到错误信号时, 如果退出原因不是normal, 该进程就会终止. 当它终止时, 同样会向它的连接组广播一个退出信号.

默认情况下, 一旦接收到来自相互链接的其他进程的退出信号, 进程也就会跟着退出. 而变为系统进程以后, 除了无法捕捉的信号(**kill**)以外, 外来的退出信号都会被转换成无害的消息. 系统进程收到摧毁信号(kill signal)时会终止. 摧毁信号是通过调用 `exit(Pid, kill)`生成的. 这种信号会绕过常规的错误信号处理机制, 不会被转换成消息. 摧毁信号只应该 用在其他错误处理机制无法终止的顽固进程上.

任何执行 `exit(Why)` 的进程都会终止(如果代码不是在 catch 或 try 的范围内执行的话),  并向它的连接组广播一个带有原因 Why 的退出信号.
此外, 进程可以通过执行 `exit(Pid, Why)` 来发送一个 **虚假** 的错误信号. 在这种情况下, Pid 会收到一个带有原因 Why 的退出信号. 调用`exit/2`的进程则不会终止.

###### 异常和退出信号捕获

- 异常源：`spawn_link(fun() -> ok end)`

未捕获时的结果：无任何结果

捕获时的结果：`{'EXIT', < 0.61.0>, normal}`

- 异常源：`spawn_link(fun() -> exit(reason) end)`

未捕获时的结果：** exception exit: reason

捕获时的结果：`{'EXIT', < 0.55.0>, reason}`



- 异常源：`spawn_link(fun() -> exit(normal) end)`

未捕获时的结果：无任何结果

捕获时的结果：`{'EXIT', < 0.58.0>, normal}`


- 异常源：`spawn_link(fun() -> 1/0 end)`

未捕获时的结果：`Error in process < 0.44.0> with exit value: {badarith, [{erlang, '/', [1,0]}]}`

捕获时的结果：`{'EXIT', < 0.52.0>, {badarith, [{erlang, '/', [1,0]}]}}`


- 异常源：`spawn_link(fun() -> erlang:error(reason) end)`

未捕获时的结果：`Error in process < 0.47.0> with exit value: {reason, [{erlang, apply, 2}]}`

捕获时的结果：`{'EXIT', < 0.74.0>, {reason, [{erlang, apply, 2}]}}`

- 异常源：`spawn_link(fun() -> throw(rocks) end)`

未捕获时的结果：`Error in process < 0.51.0> with exit value: { {nocatch, rocks},[{erlang, apply, 2}]}`

捕获时的结果：`{'EXIT', < 0.79.0>, { {nocatch, rocks}, [{erlang, apply, 2}]}} `