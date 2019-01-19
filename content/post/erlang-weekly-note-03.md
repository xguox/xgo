---
title: "Erlang weekly note 03 - Function 函数"
date: 2017-08-07T16:01:23+08:00
draft: false
tags: ["Erlang"]
categories: ["Erlang"]
---

这是一段 Ruby 方法的定义, 主流的语言语法都差不多.

```ruby
def greet(gender, name)
  case gender
  when :male
    puts("Hello, Mr. #{name}!")
  when :female
    puts("Hello, Mrs. #{name}!")
  else
    print("Hello, #{name}!")
  end
end
```

但是,  如果换做 Erlang 来写的话:

```erlang
greet(male, Name) ->
    io:format("Hello, Mr. ~s!", [Name]);
greet(female, Name) ->
    io:format("Hello, Mrs. ~s!", [Name]);
greet(_, Name) ->
    io:format("Hello, ~s!", [Name]).
```

在 Erlang 中一个很重要的概念就是模式匹配. 调用函数时, Erlang 会自上而下地尝试对子句进行模式匹配. 首先, 用输入参数去匹配第一个子句中的模式；若匹配失败,  就尝试下一个子句(这个例子中, 就是 greet(female, Name))进行匹配, 如果匹配成功, 就运行该匹配对应的语句... 虽然用 Erlang 也能写出类似第一种 Ruby 的写法,  但是,  使用模式匹配同时完成了函数选择和变量绑定两件事情,  也就无需先绑定变量, 然后再去比较它们.

在 Erlang 的这个函数写法中,  每一条函数声明都被称作一个函数子句, 函数子句之间使用分号(;)分隔, 所有的函数子句一起形成一个完整的函数定义, 而整个函数作为一个语句, 需要用 句号(.)结尾. BTW, 每个表达式用逗号分隔, 之前提到过了.

**模式匹配**

```erlang
%% erl

Point = {4,5}.
% {4,5}
{X,Y} = Point.
% {4,5}
X.
% 4
Y.
% 5
{Z,_} = Point.
% {4,5}
Z.
% 4
```

`{4,5}` 是一个元组,  一开始, X和Y没有值, 所以都是未绑定的变量. 当把它们放在=操作符左边的元组{X,Y}中时, = 操作符会去比较两边的值: {X,Y} 和 {4,5}. Erlang 会把值从元组中取出, 并把它们分配给左边的未绑定变量. 接下来, 就变成`{4,5}={4,5}.`的比较了, 显然是成立的. `{Z,_} = Point.` 使用了无需关心型变量 _ .意思是把本来应该放置在这个位置的值丢弃掉, 因为我们不会用到这个值(很多语言其实也都这么干).  _ 在模式匹配中充当通配符的作用. 只要两边元组中元素的个数(元组的长度)相等, 就可以通过模式匹配来提取元组中的元素.

```erlang
{_,_} = {a,b}.
% {a,b}
{_,_} = {a,b,c}.
** exception error: no match of right hand side value {a,b,c}
```

**列表**也可以通过进行模式匹配, 从而获取它的头和尾

```erlang
% functions.erl

-module(functions).
-compile(export_all).
head([H|_]) -> H.

```

上面编写的 `head/1` 和 `erlang:hd/1` 其实一样的,  以一个列表为参数, 返回这个列表的第一个元素. functions:head([7,6,5,4]) 得到的是返回值是 7

如果想得到列表的第二个元素, 可以创建下面的函数:

```erlang
second([_,X|_]) -> X.
```

使用函数和模式匹配, 可以比较出传给函数的两个参数是否完全一样. 为此, 我们创建一个名为`same/2` 的函数, 它有两个参数, 返回两个参数是否一样:

```erlang
same(X,X) ->
    true;
same(_,_) ->
    false.
```

当调用`same(a,a)`时, 第一个 X 未绑定, 所以会自动获取值 a. 然后, 当 Erlang 处理第二个参数时, 发现X已经绑定了. 于是, Erlang 就把 X 已经绑定的值和作为第二个参数传递给函数的 a 进行比较, 检查它们是否匹配. 模式匹配成功, 所以函数返回 true. 否则进入第二个子句返回 false.

----

**高阶函数 higher-order function**

**Erlang** 是一门函数式编程语言, 这类语言的一个显著特征就是可以像处理数据一样处理函数, 也就是说,  函数可以是别的函数的输入或者输出, 可以把函数存在数据结构中供后续使用. 其实,  Ruby 和 JavaScript 这些语言也有相似的 Lambda 概念, 所以, 还是很好理解的.

**作为现有函数别名的 fun 函数**

若要引用当前模块内的函数, 比如 greet/2, 并告知程序的其他部分: "请调用这个函数". 那么可以创建一个这样的 fun 函数

```erlang
fun greet/2
```

和各种其他类型的值一样, 可以将之与变量绑定 `F = fun greet/2`

或者直接传给别的函数 `another_fun(fun greet/2)`

**匿名 fun 函数(Lambda 表达式)**

下面是一个简单的匿名函数, 不接受任何参数, 且总是返回零:
```erlang
fun () -> 0 end
```
以 fun 关键字开头, end 关键字结尾, 之间可以是一个或者多个不带函数名的函数子句.

比如, 再一个稍复杂一点的例子:

```erlang
fun ({circle, Radius}) ->
        Radius * Radius * math:pi();
    ({square, Side}) ->
        Side * Side;
    ({rectangle, Height, Width}) ->
        Height * Width;
end
```

**闭包**

闭包指的就是可以让函数引用到它所携带的某些环境(作用域中的值部分). 换句话说, 当匿名函数、作用域的概念以及可以携带变量的能力结合在一起时, 闭包就出现了.

```erlang
a() ->
    Secret = "pony",
    fun() -> Secret end.

b(F) ->
    "a/0's password is "++F().
```

编译运行

```erlang
c(hhfuns).
% {ok, hhfuns}
hhfuns:b(hhfuns:a()).
% "a/0's password is pony
```

由于定义在 a/0 中的匿名函数继承了 a/0 的作用域, 所以, 根据前面的解释, 当在 b/1 中执行这个匿名函数时, 它仍然携带着 a/0 的作用域.