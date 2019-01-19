---
title: "Erlang weekly note 02 - Module 模块"
date: 2017-07-30T16:01:23+08:00
draft: false
tags: ["Erlang"]
categories: ["Erlang"]
---

```erlang
% erl
% Eshell V9.0  (abort with ^G)

% 列表中的第一个元素称为头(head)

hd([1,2,3,4]).
1

% 剩余的部分称为尾(tail)

tl([1,2,3,4]).
[2,3,4]

% 列表的长度
length([1,2,3,4]).
4
```

模块(module)是一个具有名字的文件, 其中包含一组函数. 像上面这些 **hd, tl, length** 都是属于 erlang 模块的 BIF(built-in functions 内建函数).

erlang 模块中BIF函数和其他函数不同, 因为在启动Erlang时, 它们会被自动引入. 模块中的其他所有函数都必须用 **Module:Function(Arguments)** 这样的形式进行调用

```erlang
erlang:element(2, {a,b,c}).
b
element(2, {a,b,c}).
b
lists:seq(1,4).
[1,2,3,4]
seq(1,4).
** exception error: undefined shell command seq/2
```

lists 模块中的 seq 函数不会被自动引入, 而 element 函数会被自动引入.

----

**创建模块**

所有模块属性都采用 **-Name(Attribute).** 的形式, 要让你的模块能够编译, 下面的模块属性必须定义:

```erlang
-module(Name).
```

这个属性永远是文件中第一个属性(也是第一条语句).

其中 Name 必须是原子, 且必须和模块的文件名一致. 当调用某个模块中的函数时, 所使用的模块名就是对应的这个 Name.

----

```erlang
% 只要有了这一行就是一个合法的模块
-module(useless).
```

**export**

要决定模块中的哪些函数可以在别的模块中调用,  需要用 export 导出. 作用类似于 OOP 的 public 和 private 方法.

形如

```erlang
-export([Function1/Arity, Function2/Arity,`...`, FunctionN/Arity]).
```

函数的元数 **Arity** 是个整数, 表示这个函数接收的参数个数.  同一个模块中,  即使函数名相同,  只要元数不一样, 就表示不同的函数. 在讨论具体某个函数时一定要说明元素只提供函数名是不够的, 还得用斜杠 / 作为分隔符带上元数.

函数add(X,Y)和add(X,Y,Z)就是不同的函数, 分别表示成add/2和add/3.

```erlang
-module(useless).
-export([add/2, hello/0, greet_and_add_two/1]).

add(A,B) ->
    A + B.

%%  显示欢迎语
%%  io:format/1是标准的文本输出函数
hello() ->
    io:format("Hello, world!~n").

greet_and_add_two(X) ->
    hello(),
    add(X,2).
```

----

有 **export** 自然会想到有 **import**

如果想用和add/2或者其他定义在当前模块中的函数一样的方式去调用io:format/1, 可以在文件的前面增加一个模块属性: -import(io,[format/1]).

-import属性定义的一般形式如下:

```erlang
-import(Module, [Function1/Arity, `...`, FunctionN/Arity]).
```

导入函数带来了方便,  但是也会降低代码的可读性.  要判断出代码中使用的到底是哪个函数,  就要到文件的最前面去查看这个函数是从哪个模块中导入进来的.

通常, 只有 lists 模块中的函数会被引入, 因为 lists 模块中函数的使用频率要远高于其他大多数模块中的函数.

----

**MACRO 宏**

Erlang 的宏和 C 语言的 **#define** 语句类似, 主要用来定义简短的函数和常量.在代码被编译成供虚拟机使用的字节码之前, 它们会被替换成所代表的文本表达式.

定义宏的方式也是用模块属性,
```erlang
-define(MACRO, some_value).
```

然后, 就可以在模块的任意函数中使用宏 `?MACRO` 了, 这个宏在代码编译前会被替换成some_value.

函数'宏的定义方法类似.下面是一个简单的函数宏, 进行两个数的减法:

```erlang
-define(sub(X,Y), X-Y).
```

如果调用 `?sub(23,47)`, 这个调用会被编译器替换成 `23-47`.

Erlang 中有一些预定义的宏, 比如:

- ?MODULE, 会被替换成当前模块的名字, 是一个原子;
- ?FILE, 会被替换成当前文件的名字, 是一个字符串;
- ?LINE, 会被替换成该宏所在的代码行的行号.

使用属性 **-ifdef(MACRO).**, **-else.** 和 **-endif.** 可以检测某个特定的宏是否已在代码中定义.

----

**编译运行**

在命令行中使用 **erl** 打开 Erlang shell, 默认情况下, shell 只会在它的启动目录和标准库中去查找文件. **cd/1** 函数专用于 Erlang shell, 可以更换 shell 的当前目录, 这样寻找文件就会方便一些.

```erlang
% 编译 useless 模块
1> c(useless).
{ok,useless}
2> useless:greet_and_add_two(200).
Hello, world!
202
```

示例来自 **Learn You Some Erlang for Great Good!**