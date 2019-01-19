---
title: "Improving UX Through Front-End Performance"
date: 2013-03-20T16:01:23+08:00
draft: false
tags: ["Translation"]
categories: ["Translation"]
---

想象一下,你正在一个马路口等待着过马路. You push the button to call the walk signal, and you take out your phone. 你想要完成某件事: 可能是查看邮件, 可能是给你的to-do列表添加一条item,又或者是阅读你的twitter. 你仅有有限的时间来完成这其中的一件事.

这所要花的长短取决于用户在你的网站上完成他们想做的事情的时长.这很重要.

据Google的一项研究, 用户的搜索结果页面每增加半秒会致使营业额和广告收入减少20%. 另外一篇来自Amazon的[类似报道](http://www.websiteoptimization.com/speed/tweak/psychology-web-performance/)发现, 每增加额外的100毫秒加载时间, 销售额则会减少1%. 用户期望在两秒以内加载完整个页面, 否则第三秒后,高达40%的用户会[选择离开](http://www.gomez.com/pdfs/wp_why_web_performance_matters.pdf).

你能保持这样的速度? 如果你的网站包含有丰富的内容, 许多动态元素, 更大的JavaScript文件和复杂的图形, 那么我们当中的很多人的回答可能会是 "no".

现在,是时候让我们把性能优化作为如何为各种设备设计, 搭建, 测试每一个我们创建的网站的基本组成部分了.

##### Designing for performance

网站的性能优劣从网站设计开始.权衡一个设计选择对你的网页速度的影响 你的网站的转化率. 你真的需要8个深浅不同的蓝色吗?这个宽度为1000px的背景图片还需要加什么值吗? 使用icon font来取代sprite会导致页面变得更重,渲染更慢, 还是会比使用原始图片更快呢?

并非所有的设计决策都是有利于性能的. 我发现一个略微减慢了页面速度的button样式可以提高转化(conversions), 对于仅需要牺牲小小的页面速度来说, 这是值得的.

但有时候, 我们更需要关注的是性能. 我曾经重新设计一个登录页面, 需要在上边为其添加大量的图片, 我不确定性能上变差了是否会对转化带来负面影响, 于是我在一个 [A/B test](http://alistapart.com/article/a-primer-on-a-b-testing) 上为一小部分的用户推出了这个新的设计来看看会有什么样的反响. 新的设计加载时间是之前的两倍, 我看到了一个很高的 exit rate 和更低的转化率, 于是我们沿用了之前的轻量级设计. 犯错了没关系 -- 它能给你一个参考的基准.

在另外一次试验, [dyn.com](http://dyn.com/)主页上着重介绍的一个缩略图部分, 需要在10个图槽内交替显示26张图片.

![dyn.com homepage.](http://d.alistapart.com/371/dyncomhomepage.jpg)

于是我的teammate把这26张的图片放到一个sprite里头, 结果:

- CSS,JavaScript,以及images的体积有所变大, 主页的总大小增加了**60k**
- 请求数减少了**21%**
- 页面总加载时间减少高达**35%**

这也就证明了这个试验是值得的: 我们也不确定这是否能给页面带来提速, 但我们都觉得这是值得学习的试验.

##### Coding for performance

清理你的HTML, 或许其他的一切也会跟着变好.

首先, 重命名你的HTML中的非语义化元素. 这可能会是最艰难的, 但只要你开始依据像`nav`,`article`这样的语义而不是通过图样或者网格来思考主题, 就会有显著的进展. 通常情况下, 我们是通过更复杂的CSS选择器来得到非语义化名称, 这样的话我们就会添加了不必要的ID和元素到HTML当中, 这反而会造成我们的CSS不够简洁, 同时也难以正确的使用某些特性.

其次, 就是整理你的CSS. 第一件事要做的是清除无效的选择器. 我在[writegoodcode.com](http://writegoodcode.com/)中的[一次研究](http://dyn.com/how-we-got-dyndns-com-to-load-faster-and-how-you-can-learn-from-it)发现, 在一个CSS文件中添加一些无效的选择器实际上会致使页面的加载时间增加`5.5%`. 更有效的CSS选择器也因为在样式表中更易阅读且有语义从而更易于将来的重设计和自定义样式. 多用途,可编辑的代码通常可以带来高性能. 在前面提到的那个研究中, 通过整理CSS文件我节省了`39%`的文件大小.

再次, 重点维护你的`div`. 通常你的标记越整洁, 那么你的CSS就会越小, 将来编辑,重设计也就更容易. 这不仅仅是提高了你的页面加载时间, 同时也为你节省了开发的时间.

最后, 专注于开发多用途的代码, 这样可以节省时间且减少CSS和HTML的大小. 更少的CSS和HTML可以明显地提高将来代码的可维护性和可重构性, 更轻的页面对加载速度有着积极的影响.

##### Optimizing requests

请求是指浏览器获取某样东西比如一个文件或者一个DNS记录. 你的标记越简洁, 浏览器也就更少的发出请求, 同时用户花在等待数据在客户端和服务端来回的时间就更少.

除了简洁的标记外, 还要做的是最小化JavaScript请求 -- 只在确实需要的时候才加载之. 如果某一文件不是确实需要, 则不必在每一个页面都加载之. 不要在响应式设计中加载某一个只在大屏幕下才需要用到的JavaScript文件. 例如, 使用[简单的链接来取代使用社交脚本](http://www.zurb.com/article/883/small-painful-buttons-why-social-media-bu). 你也可以异步加载JavaScript从而不会导致所有的内容都被JavaScript给屏蔽掉.

尽管, 通常情况下响应式设计需要更多的CSS和图片(页面更重), 但还是可以通过减少请求来得到更快的页面加载速度.

##### Optimizing images

同样的, 尽你所能的减少图片的请求. 首先, 我们可以[建立sprite](http://alistapart.com/article/sprites/). 在 writegoodcode.com 的 研究中, 我发现一个案例使用icons sprite 后减少了`16.6%`的加载时间. 我喜欢通过创建一个背景重复的sprite来整理图片. 可能你需要创建的是水平重复或者垂直重复的.

接下来, 创建一个no-repeat backgrounds 的透明sprite. 这个sprite可以包含你的logo和icons. 如果你希望更先进一些, 可以使用像[Grunticon](http://filamentgroup.com/lab/grunticon)这样的工具, 它可以使用SVG icon和background images,并基于用户浏览器的兼容性计算得出如何最好的为之服务.

在整理好所有新的图片后, 通过像[ImageOptim](http://imageoptim.com/)这类的优化器运行. 同样的, retina-sized 的图片可以通过使用[extensive compression](http://blog.netvlies.nl/design-interactie/retina-revolution/)使之变小, 不过最终结果不是特别显著.

现在, 看一看哪些图片你可以使用CSS3的渐变来替代. 这样做不仅仅是为了减少加载时间, 而且还可以使得在将来编辑页面变得极其简单, 因为开发人员无须找到原始图片进而编辑,生成,优化等.

最后, 来看看使用[Base64 encode](http://www.greywyvern.com/code/php/binary2base64), 它可以让你在CSS文件中嵌入图片而不是通过调用一个单独的URL. 通常看起来的结果如下:

```css
#nav li:after {
	content:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAI0lEQVQIW2P4//8/w8yZM//DMIjPAGPAMIiPWxCIMQQxzAQAoFpF7lGFr24AAAAASUVORK5CYII=);
}
```

在一个小圆圈内随意添加一些字母和数字在很多地方都可以看到, 包括 dyn.com . 每次当你想要在你的设计中使用这类嵌入的图片, 可以让你节省一次图片请求. 这么嵌入图片会使你的CSS文件变得更大, 因此你需要测试嵌入前后的页面加载时间确保是有意义的.

##### Measuring performance

下面这部分比较有趣一些: 确认你的努力是否得到了回报.

Google的[PageSpeed](https://developers.google.com/speed/pagespeed/)和Yahoo!的[YSlow](http://developer.yahoo.com/yslow/)都而可以为你提供一些可以提高你的页面速度的建议, 包括确定哪个元素阻塞了页面渲染, 页面中不同的组件如HTML以及CSS的大小.

同时, 我也推荐YSlow的拓展 [3PO](http://www.phpied.com/3po/). 它可以检查你的网站所集成的一些流行的第三方脚本比如Facebook, Twitter, Google+等. 这个插件可以给你提高建议来进一步优化你的页面上的这些社交脚本进而提升页面速度.

自从我使用了PageSpeed 和 YSlow的建议来提高页面性能以后, [WebPageTest.org](http://webpagetest.org/)已经成为我首选的基准参考工具. 它非常详细地给出关于请求, 文件大小, 时间等信息, 并且提供不同地方不同浏览器的测试.

确定基准可以在你设计的时候帮助你排除故障. 测量性能并分析结果可以帮助你的页面在大屏幕或者小屏幕上提速. 你可以测试或者定位像[conditional loading of images](http://adactio.com/journal/5414/)这类技术从而在开发中更轻松应对性能问题.

##### The impact of web performance

网页的性能影响着你的用户--这意味着了解, 测量,提高性能问题是每个人的工作. 所有的这些技术都可以提高页面的加载速度. 进而可以显著的提高你的网站的用户体验.

用户更高兴了也就意味着更高的转化率, 无论你是按照什么来衡量, 或是收入, 注册数, 回访数, 还是下载数. 在更快的页面速度下, 用户可以使用你的网站在短时间内完成他们想要做的事--即使可能只是在等待红绿灯.

原文来自Lara Swanson [Improving UX Through Front-End Performance](http://alistapart.com/article/improving-ux-through-front-end-performance)