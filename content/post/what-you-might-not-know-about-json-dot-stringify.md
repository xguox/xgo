---
title: "你可能不知道的JSON.stingify()"
date: 2013-02-01T16:01:23+08:00
draft: false
tags: ["Translation", "JavaScript"]
categories: ["Translation", "JavaScript"]
---

几乎所有开发人员都会花一些时间在JavaScript上,有些时候是在使用JSON.stingify(以及与之相对应的,JSON.parse).JSON - Javascript Object Notation - 已经成为许多开发人员理想的数据交换格式(the go-to data-interchange format)- 并且有许多语言支持序列化为JSON,而不仅仅只有JavaScript本身.如果哪天你半夜起来又无法入睡了,可以查一查关于[JSON](http://en.wikipedia.org/wiki/JSON#History)的历史(tl;dr – Douglas Crockford is the brain behind it)

在写JavaScript的时候,我们用```JSON.stringify```将某个值序列化为一个字符串值来表示一个对象.

```javascript
JSON.stringify({
    name: "Jim Cowart",
    country: "Jimbabwe"
});
// 输出结果:  "{"name":"Jim Cowart","country":"Jimbabwe"}"

JSON.stringify("Oh look, a string!");
// 输出结果: ""Oh look, a string!""

JSON.stringify([1,2,3,4,"open","the","door"]);
// 输出结果: "[1,2,3,4,"open","the","door"]"
```
我不会反复地讨论关于序列化的规则,在[这里](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify)你可以读到更多相关的.但最基本的,你需要知道下面这些:

-  ```undefined``` 值、函数或者XML值会被忽略,除非当...
- 如果你的数组当中含有 ```undefined```值,函数或XML值,该数组中的这些值将会被当成 ```null```

Let's see about that:

```javascript
JSON.stringify({
    doStuff: function() { },
    doThings: [ function() {}, undefined ]
});
// 输出结果:  "{"doThings":[null,null]}"
```

>"太棒了,Jim.我知道了.它是一种数据交换格式.而行为是不会被序列化的.这些天我使用的大多数库都是在底层为我处理序列化(Most of the libraries I use these days handle serializing for me under the hood.).那为什么要专门写一篇post关于 JSON.stringify 呢?"

这个我知道,但是有一些实用的技巧迟早我们会需要用上的.Many of the larger applications I've worked on recently have had debug flags that can be flipped to enable various console logging from different components active on the page.
而我们明显不希望总是通过无数行```console.log```来筛选信息 - 但当这变得有需要的时候,让它的可读性更高是不是更nice呢?

```javascript
// what if this:
'{"name":"Jim Cowart","location":{"city":{"name":"Chattanooga","population":167674},"state":{"name":"Tennessee","abbreviation":"TN","population":6403000}},"company":"appendTo"}'

// could be formatted like this, automagically?
"{
    "name": "Jim Cowart",
    "location": {
        "city": {
            "name": "Chattanooga",
            "population": 167674
        },
        "state": {
            "name": "Tennessee",
            "abbreviation": "TN",
            "population": 6403000
        }
    },
    "company": "appendTo"
}"
```
事实上,`JSON.stringify`可以传入3个参数 `(JSON.stringify(value [, replacer [, space]])`.其中,第三个参数 - "space" - 允许你指定一个字符串字符、或者使用缩进、或是一个数字.如果你传的是一个数字,那相应的空格数(最大为10)会被作为缩进量

```javascript
var person = {
    name: "Jim Cowart",
    location: {
        city: {
            name: "Chattanooga",
            population: 167674
        },
        state: {
            name: "Tennessee",
            abbreviation: "TN",
            population: 6403000
        }
    },
    company: "appendTo"
};
//如果你希望缩进量为2 个空格,
// 你可以这么干:
JSON.stringify(person, null, 2);
/* produces:
"{
  "name": "Jim Cowart",
  "location": {
    "city": {
      "name": "Chattanooga",
      "population": 167674
    },
    "state": {
      "name": "Tennessee",
      "abbreviation": "TN",
      "population": 6403000
    }
  },
  "company": "appendTo"
}"
*/
// 如果你希望使用 tab 缩进,那么
// 你可以传入 \t 作为第三个参数
// 以此来告别空格缩进
JSON.stringify(person, null, "\t");
/*输出结果:
"{
    "name": "Jim Cowart",
    "location": {
        "city": {
            "name": "Chattanooga",
            "population": 167674
        },
        "state": {
            "name": "Tennessee",
            "abbreviation": "TN",
            "population": 6403000
        }
    },
    "company": "appendTo"
}"
*/
```
那么-第二个参数呢?在上面的例子中简单地传了一个```null```.关于 "replacer" 参数-它可以是一个数组或者是一个函数.如果是一个数组,它将只输出你在该数组中所想要包含的keys.

```javascript
// 假定 person对象是上一例子中的那个,
JSON.stringify(person, ["name", "company"], 4);
/* 输出结果:
"{
    "name": "Jim Cowart",
    "company": "appendTo"
}"
*/
```
如果这 "replacer" 参数传入的是一个函数,那么这个函数需要有两个参数:key 和 value :

```javascript
// a bit contrived, but it shows what's possible
// FYI - 被序列化的值是第一个传给replacerFn的东西, 也即是这个对象的每一个key,因此需要检查key的真假.
var replacerFn = function(key, value) {
    if(!key || key === 'name' || key === 'company') {
        return value;
    }
    return; //返回 undefined 并忽略
}
JSON.stringify(person, replacerFn, 4);
/* produces:
"{
    "name": "Jim Cowart",
    "company": "appendTo"
}"
*/
```
你可以使用替换方法来处理得到 ***黑名单***成员的姓名(与此相对应的是,使用***白名单***数组作为"replacer" 参数).我发现这种方法是有用的,尤其当我需要迅速并且有针对性的得到一个DOM元素的值,但我希望阻止一个序列化循环引用的异常(当我drank some antifreeze 或是试图做些荒谬的做法如JSON.stringify(document.body)的时候).Feel free to browse [this fiddle](http://jsfiddle.net/ifandelse/6Yj5h/) for a simple example of using a replacer function to blacklist based on member name(s).

当然,另一个得到自定义某个对象的JSON序列化的选择是在对象中使用"toJSON"方法

```javascript
var person = {
    name: "Jim Cowart",
    location: {
        city: {
            name: "Chattanooga",
            population: 167674
        },
        state: {
            name: "Tennessee",
            abbreviation: "TN",
            population: 6403000
        }
    },
    company: "appendTo",
    toJSON: function () {
        return {
            booyah : this.name + ' , employer: ' + this.company
        };
    }
};

// 上面的toJSON返回的值
// 实际上被字符串化的:
JSON.stringify(person, null, 4);
/* produces:
"{
    "booyah": "Jim Cowart , employer: appendTo"
}"
*/
```
> "说真的,Jim.比起在控制台写更漂亮的JSON我们有更重要的事情要做."

You don't have to convince me, I'm with you 110%. 但我会说,当你为了调试已发布的信息的可视性或者logging websocket payloads而 wire-tapping a message bus 的时候更好看的JSON格式化会让一切都不同.(囧 、o(╯□╰)o)能够通过白名单替换数组(或者黑名单替换函数)来有选择的序列化对象,可以很方便地在DOM元素/事件上帮助你跟踪问题,或者简单地让你在复杂的交互上有更好的可视化,without de-railing you with serialization exceptions.

原文来自 Jim Cowart ,[What You Might Not Know About JSON.stringify()](http://freshbrewedcode.com/jimcowart/2013/01/29/what-you-might-not-know-about-json-stringify/?utm_source=javascriptweekly&utm_medium=email)

XguoX
- Actually,其实这个在Nicholas C.Zakas的《JavaScript高级程序设计》第20章关于JSON的介绍讲的也很详细.比我这蹩脚翻译要好得多.