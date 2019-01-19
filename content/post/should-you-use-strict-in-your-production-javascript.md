---
title: "你应该在生产环境下使用JavaScript的严格模式(strict mode)吗?"
date: 2013-02-08T16:01:23+08:00
draft: false
tags: ["Translation", "Ruby"]
categories: ["Translation", "Ruby"]
---

ECMAScript 5 中引入了一种定义,可以切换让你的JavaScript代码是否在所谓的 **严格模式(`strict mode`)**下执行.因为在strict mode下,当浏览器遇到一些脚本中包含有不好的代码做法时,即使没有语法上的错误也会抛出runtime错误,因而常常被吹捧为一种显著有益于捕获程序员错误的方式.

##### 在strict mode下运行被认为可以更少的出错

举个栗子,在定义一个变量时,缺少关键字"var",如果在non-strict mode下会默认为该变量为全局变量,而在 strict mode下则会抛出一个runtime错误.

Non-Strict Mode:

```javascript
(function () {
    x = 1;
    console.log(x);
}());

//=> 1
```

Strict Mode:

```javascript
(function () {
    "use strict";
    x = 1;
    console.log(x);
}());

//=> ReferenceError: x is not defined
```

##### 但strict mode太爱出风头了

有些时候这样的严格会是一把双刃剑.我们都希望知道在我们的代码中的这些无声的bug,但绝对不会希望看到这些bug在我们运营上线的网站上抛出runtime错误给了我们的最终用户.尤其是,当这些代码实际上都按我们的期望正常运行着的时候,只是这些代码有一些"bug"

##### 可以同时使用两种方式吗?

你不会得到许多的同情,而应该开始抱怨JavaScript暴露了一个bug给你的用户而你自己却不知道,除此之外,你可能会想这可以有两全其美的方法-在测试代码下使用strict mode,而生产环境下不使用.但这个想法有几个问题,首先,这很可能会使你的生产代码运行起来比测试代码要慢的多(见下文),然后,这两种代码可能跑起来会有所不同(见下文).

##### Non-Strict Mode 下代码可能跑得要慢一些

或许这会跟你的直觉相悖,可能你会认为更多的错误检测和一些额外的"事情"一定会使得引擎做得更多从而你的代码跑得更慢.但[Chris Heilmann](https://hacks.mozilla.org/2011/01/ecmascript-5-strict-mode-in-firefox-4/)总结出了使用strict mode跑的更快的原因.他的解释如下:

> __Strict mode__ 修复了那些误区使得JavaScript引擎性能得到优化: 相较于非strict mode而言,strict mode代码能跑的更快. Firefox 4 还未优化strict mode ,但其后续版本会.

也就是说是否去掉strict mode 在生产环境意义并不是那么的明显, 但如果没有令人信服的考虑,去掉strict mode的话你实际上是改变了你的代码的执行方式.下面是几个例子.

##### 是否为Strict Mode,函数的上下文会有所不同
在non-strict mode,任意全局函数中,保留字"`this`"默认指向全局对象(对于浏览器而言,这个对象会是 **window对象**).而在strict mode,"`this`"默认则为`undefined`
Non-strict Mode:

```javascript
(function () {
    console.log(this);
}());
//=> Window
```
Strict Mode:

```javascript
(function () {
    "use strict";
    console.log(this);
}());
//=> undefined
```

在一个函数中使用 `call()` 你可以设定 `this` 的指代,但在这两种不同的模式下,你用来充当上下文的值会巧妙地变得有些不一样. 在 Non-strict Mode,如果上下文的值是原始的(primitive),比如数字 `1`,那它将会被转换成一个对象,一个Number对象.而在 strict mode ,这个原始的值 `1`, 将会被保留.
Non-strict Mode:

```javascript
console.log(function () {
    return this;
}.call(1));

//=> Number {}
```
Strict Mode:

```javascript
console.log(function () {
    "use strict";
    return this;
}.call(1));

//=> 1
```
##### 是否为Strict Mode,arguments对象的行为有所不同

 `arguments` 对象包含了引用函数的形参.在 non-strict mode下,修改arguments对象的值同时也会修改对应的形参的值,而在Strict Mode之下,两者是独立的.
Non-strict Mode:

```javascript
(function (a, b) {
    arguments[0] = arguments[1];

    console.log(a, b);
}('one', 'two'));

//=> two two
```
Strict Mode:

```javascript
(function (a, b) {
    "use strict";
    arguments[0] = arguments[1];

    console.log(a, b);
}('one', 'two'));

//=> one two
```

##### 为什么你无论如何也要移除严格模式

考虑到在你的代码中的所有可能的变更行为,你可能会认为有明确的理由不去除"use strict",毕竟,你也不希望你的做法会改变代码原本的运行方式.对不?但事实上,你确实会遇上这些问题,即使你保留了"use strict".这是因为,你的"use strict"在一些旧版本的浏览器下会直接被无视,[它们不认识这句声明](http://caniuse.com/use-strict).因此,对于你的其中一些用户,你还不如把"use strict"去掉更好.这样,他们也将得到同样的而在你想象的现代浏览器中不会得到的不严格行为.如果希望使用严格模式的同时支持旧版本的浏览器,你需要明白所有的严格模式下是否有效的副作用,然后避开对它们的依赖.

你可能会觉得兼容性胜利了以及最好从来都没有出现过什么"use strict"的效果.又或者,你可能 觉得你了解了所有的副作用影响并坚持使用"use strict",但还有另外一个问题需要考虑到的.如果通过连接和压缩你在生产环境下的代码来优化了HTTP请求,你可能会遇到一个与"use strict"相关的问题.当在函数中声明了 "use strict" , 它只适用于该函数的词法作用域(lexical scope ).但如果在函数外声明,则将会作用于整一个文件.How could this come back to bite you?让我举个栗子...

在我的一些工作项目中,我们有时候会引入第三方的实用工具库到我们的应用程序中.而且很有可能会引入多个.当然,我们自己的代码是 concatenation-friendly ,因为我们只使用了函数范围内的 严格模式,但假设一下如果某一个第三方库在文件顶部--不在任何函数域内声明了"use strict".现在也想象一下另外一个第三方库不是严格的安全.通过串联这两个库你几乎可以肯定会得到不同的行为以及一些以往不会抛出的错误.

当然也有一些方法可以缓解这些问题:避免使用任何在函数域外声明"use strict"的库,或者你可以放弃连接使用这些库并且 take the HTTP request hit to performance.这些个看起来都有些局限性,不那么吸引人.

因此,不顾所有可能的移除严格模式副作用,在特定情况下,你可能会决定选择在生产环境下移除"use strict".

原文来自Michael Mathews,[Should You "Use Strict" in Your Production JavaScript?](http://scriptogr.am/micmath/post/should-you-use-strict-in-your-production-javascript?utm_source=javascriptweekly&utm_medium=email)

