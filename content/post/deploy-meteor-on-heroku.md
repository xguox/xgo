---
title: "最近在折腾 -- deploy Meteor on heroku"
date: 2013-04-06T16:01:23+08:00
draft: false
tags: ["JavaScript"]
categories: ["JavaScript"]
---

貌似有好一段时间没有在这儿小发一些闷骚或者情感泛滥什么的, 都是用自己蹩脚的英语水平翻译些英文post. 这些天想部署个Meteor app玩一玩, 结果一玩就各种出事了.

vps退货, godaddy的莫名失单, 部署各种卡壳...

因为最近比较多的关注前端的东西,  同时在自己local耍着[SachaG / Telescope](https://github.com/SachaG/Telescope)这个基于Meteor的项目.

Meteor的系统是超级alpha的, 所以很多问题遇上了也没人帮忙, 解决. 比如之前在[Stack Overflow](http://stackoverflow.com/questions/15654637/same-helper-for-different-template). 而部署这一块, 是我比较陌生的. 包括以前搞Rails那会, 也没真正意义的整一整部署. 绝大部分时间都在本地跑啊跑. 所以, 在Meteor, 也是这一块最让我头疼.

从vps到appfog到heroku, 各种转了一圈, 最终还是选定了heroku.  原因之一就是资料相对多, 出错情况也较少.

幸得以前一直搞得是Ruby/Rails. 所以装heroku的时候没啥压力, 不然可能这又得搞上一会. 直接下载了官方的[heroku toolbelt](https://toolbelt.heroku.com/)无压力安装, login, clone...

and then 跟着 [Heroku buildpack for Meteorite](https://github.com/oortcloud/heroku-buildpack-Meteorite) 这个buildpack完成接下来的:

```ruby
heroku create --stack cedar --buildpack https://github.com/oortcloud/heroku-buildpack-Meteorite.git
```

**Please note: 在这如果你想把你的项目名改成想要的比如我改成了[frontn.herokuapp.com](frontn.herokuapp.com), 在heroku的项目页面上修改名字的同时, 记得把repo的git remote 也相应的更改**

否则就会报错

```ruby
heroku !  Resource not found
```

好吧, 很明显我自己傻13了才会知道要这么做的.

接着到了最关键的push了.  当我`git push heroku master`的时候, bla,blah blah...下来还以为这次会真的很顺畅. 结果oops的一嘣

```ruby
-----> Launching... !     Heroku push rejected, Please verify your account to install this add-on
For more information, see http://devcenter.heroku.com/categories/billing
Verify now at https://heroku.com/verify
```

纳尼,  原来这货还要用到一个mongo的数据库插件, 在这里是Mongohq, 原本没注意看以为直接
`heroku addons:add mongohq:sandbox`就OK
(sandbox是免费餐)

结果毫无疑问的继续嘣...

```javascript
Adding mongohq:sandbox on frontn... failed
 !    Please verify your account to install this add-on
 !    For more information, see http://devcenter.heroku.com/categories/billing
 !    Verify now at https://heroku.com/verify
```

Google之才知道原来Heroku在装插件前得通过传说中的credit card卡认证, 我辣时就更嘣了. 对于我等无产阶级而言, credit card仿佛就跟神一般滴遥不可及. 你想买3刀的godaddy域名?不好意思, 请出示credit card; 你想用name.com买域名? 不好意思, 请出示credit card; 你想买linode的vps?有credit card吗? 亲...正当我开始准备放弃heroku这条路时, 忽然无意中发现了虚拟信用卡这货, 顺藤摸瓜发现, 柳暗花明又一村.

**工行国际e卡 + entropay**(怎么觉得我像在拍广告的)

虽然门路是找着了, 不过弄得我好不蛋疼.
因为mac上工行网银不支持插U盾, 于是又启动了以前的那台老爷机, 一卡一顿的, 唉, 为了搞到手就忍忍吧...结果成功申请后唬弄半晌才知道原来要先用软妹b买美刀才能花, 而且至少买50刀. 我现在要50刀搞毛线撒...没法子, 只得照办了. 然后是entropay的虚拟卡>>>
OOXX一轮下来总算是把heroku的验证给通过了, 感动啊...

`heroku addons:add mongohq:sandbox`

` git push heroku master`

好吧, 总算是成功了. 我容易吗...

最后总结, heroku相对的来说除了出错情况较少, 可用资料多之外, 管理上也比自己搭vps要方便些, feel了一下MongoHQ的那个插件还不错...这样, 也就可以花更多的时间的app的开发上. 而不是折腾这出错那无解.  好吧, 尽管咱都爱折腾, 不过, 往你想的地方折腾不更好吗? 其实这些也就都是Paas的优势啦, 专注核心开发, 减少运维成本...

其他的待续吧, 相比Rails, 或许底层它怎么干还没搞透, 但起码的知道它干了什么, 可Meteor我还真不知道它都干了些什么, 比Rails更magic.
