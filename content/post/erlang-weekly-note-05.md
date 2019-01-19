---
title: "Erlang weekly note 05 - Records & Maps 记录与键值对"
date: 2017-08-21T16:01:23+08:00
draft: false
tags: ["Erlang"]
categories: ["Erlang"]
---

元组是大部分 Erlang 结构化数据的基石, 而 Erlang 的记录(Record)是元组之上的语法糖, 记录可以让你避免了使用元组时增减字段所带来的麻烦以及必须记住各个字段在元组中的顺序的问题.

使用记录之前首先要定义记录, 记录是以模块属性的形式来声明的, 比如:

```erlang
-record(user, {uuid="<xguox>", address, phone}).
```

这里的 user 相当于记录的名称, 这个记录有三个字段, 分别是: uuid, address, phone, 其中 uuid 具有默认值 `"<xguox>"`.

接着就可以这么使用记录:

```erlang
#user{}

#user{phone="10086"}

#user{uuid="<another>", phone="10086", address="Mars"}

T = #user{phone="10086", address="Earth"}
T#user.address
```

记录可以直接在函数头中对它们进行模式匹配, 譬如:

```erlang
-record(user, {id, name, group, age}).

%% 使用模式匹配进行过滤
admin_panel(#user{name=Name, group=admin}) ->
     Name ++ " is allowed!";
admin_panel(#user{name=Name}) ->
     Name ++ " is not allowed".

%% 可以随意扩展 user 记录, 函数无需修改

adult_section(U = #user{}) when U#user.age >= 18 ->
     allowed;
adult_section(_) ->
     forbidden.
```

编译以后运行:

```erlang
records:admin_panel(#user{id=1, name="ferd", group=admin, age=96}).
%% "ferd is allowed!"
records:admin_panel(#user{id=2, name="you", group=users, age=66}).
%% "you is not allowed"
records:adult_section(#user{id=21, name="Bill", group=users, age=72}).
%% allowed
records:adult_section(#user{id=22, name="Noah", group=users, age=13}).
%% forbidden
```

**更新记录**

因为记录都定义都是在编译时, 而不是运行时, 所以,  要在 erl 中使用记录,  需要用 **rd** 手工来定义一次.

`rd(Name,Definition)`, 定义一个记录, 方式和模块中使用的 `-record(Name,Definition)`类似.

```erlang

U1 = #user{uuid="XguoX", phone="10000", address="Test address"}.
#user{uuid = "XguoX",address = "Test address",phone = "10000"}
U2 = U1#user{uuid="MM",phone="12345"}.
#user{uuid = "MM",address = "Test address",phone = "12345"}
```

**官方长示例**
```erlang
%% File: person.hrl

%%-----------------------------------------------------------
%% Data Type: person
%% where:
%%    name:  A string (default is undefined).
%%    age:   An integer (default is undefined).
%%    phone: A list of integers (default is []).
%%    dict:  A dictionary containing various information
%%           about the person.
%%           A {Key, Value} list (default is the empty list).
%%------------------------------------------------------------
-record(person, {name, age, phone = [], dict = []}).
-module(person).
-include("person.hrl").
-compile(export_all). % For test purposes only.

%% This creates an instance of a person.
%%   Note: The phone number is not supplied so the
%%         default value [] will be used.

make_hacker_without_phone(Name, Age) ->
   #person{name = Name, age = Age,
           dict = [{computer_knowledge, excellent},
                   {drinks, coke}]}.

%% This demonstrates matching in arguments

print(#person{name = Name, age = Age,
              phone = Phone, dict = Dict}) ->
  io:format("Name: ~s, Age: ~w, Phone: ~w ~n"
            "Dictionary: ~w.~n", [Name, Age, Phone, Dict]).

%% Demonstrates type testing, selector, updating.

birthday(P) when record(P, person) ->
   P#person{age = P#person.age + 1}.

register_two_hackers() ->
   Hacker1 = make_hacker_without_phone("Joe", 29),
   OldHacker = birthday(Hacker1),
   % The central_register_server should have
   % an interface function for this.
   central_register_server ! {register_person, Hacker1},
   central_register_server ! {register_person,
             OldHacker#person{name = "Robert",
                              phone = [0,8,3,2,4,5,3,1]}}.
```

----

**键值映射(Maps), 又叫映射组**(_Programming Erlang_). 是从 Erlang R17 开始时新增的一种数据类型.

映射组的语法与记录相似, 不同之处是省略了记路名,  并且键值分隔符是 `=>` 或 `:=`, 此外,  映射组有着明确的顺序,

举个例子, 假设要创建一个包含a, b两个键的映射组.

```erlang
M1 = #{a => 1, b => 10}.
```

映射组在系统内部是作为有序集合存储的, 打印时总是使用各键排序后的顺序, 与映射组的 创建方式无关.譬如:

```erlang
M1 = #{a => 1, b => 10}.
%% #{a => 1,b => 10}
M2 = #{b => 10, a => 1}.
%% #{a => 1,b => 10}
M1 = M2.
%% #{a => 1,b => 10}
```

分隔符号 `=>` 或 `:=` 的区别在于

表达式`K => V`有两种用途, 一种是将现有键 K 的值更新为新值 V, 另一种是给映射组添加一 个全新的 K-V 键值对. 这个操作总是成功的.

表达式`K := V`的作用是将现有键 K 的值更新为新值 V. 如果被更新的映射组不包含键 K, 这个 操作就会失败.