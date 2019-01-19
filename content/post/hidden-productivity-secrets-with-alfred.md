---
title: "Hidden Productivity Secrets With Alfred"
date: 2013-10-28T16:01:23+08:00
draft: false
tags: ["Tools"]
categories: ["Tools"]
---

好的开发人员总是在寻找可以让自己的工作流程更快速,更自动化的方法.
这次,我们带来的是**Alfred的一系列workflows**,它们可以极大的提高你的开发效率,相信你会为之震惊.

### What is Alfred?

对于大多数了解不深的人来说,[Alfred](http://www.alfredapp.com/)只是一个备受赞誉的Mac OS X app, 它可以快速地帮助你查找在线或者本地的文件. 最新的版本 **Alfred 2**更是带来了大量的改进,尤其是其中的[Powerpack](http://www.alfredapp.com/powerpack/),可以让你创建自己的workflows(工作流程).

下面你将会看到这些精心挑选出来的,能够改变你的工作方式的workflows.

## Open With Sublime Text

强大的文件,目录搜索能力是Alfred所最让人喜爱的功能之一. 那如果我们想要利用它使用自己喜爱的编辑器(比如说Sublime Text 3)打开文件或者目录呢?

- [Open With Sublime Text (v3)](https://github.com/franzheidl/alfred-workflows/tree/master/open-with-sublime-text), developed by [@franzheidl](https://github.com/franzheidl/)
- [下载](http://zno.io/RcAe)
- 触发: `subl`,`subl*`

想要其他编辑器的话请猛击 [Extras](https://github.com/zenorocha/alfred-workflows/wiki/Extras#code-editors)


![](https://7nnqba.dm1.livefilestore.com/y2pGNECa5OJFUT2KVW5fKdahDKuCsRFn-r7el4QbFLC7ijAiYxRNxBI_COOWUq8W8-oKBvrS0eU7UyCFn129AtLp9eBjydEPpJzJwHMK8hIqMA/alfred-subl-opt.png?psid=1)

XguoX: 可能更多人还是选择在终端直接用命令敲开.

## Can I Use... Workflow
在HTML5时代,当使用某一个CSS属性或者JavaScript APIs之前,你需要检查知道浏览器是否支持. 当然,你可以打开浏览器,直接浏览[Can I Use...](http://caniuse.com/)这个网站, 然后搜索某个关键字从而看看浏览器的支持情况.除此外,你还可以使用这个Alfred Workflow.

- [Can I Use... Workflow](https://github.com/willfarrell/alfred-caniuse-workflow),developed by [@willfarrell](https://github.com/willfarrell/)
- [下载](http://zno.io/Rcex)
- 触发: `caniuse`

![](https://7nnqba.dm2301.livefilestore.com/y2pLGL9Yh8RK9E6UUkjLPqhxSoA_Wofc3tyJuq-9DOmdK1fjtDTfmvBdegAJmCsO45T_Pf02opsLZ2YJXmhoXWy5HXUTmCw-3u-oJfUH9UERVU/alfred-caniuse-opt.png?psid=1)

XguoX:搜索速度还行

## Dash Workflow
没有人会知道某一门语言或者某一个框架的所有.我们时常会需要查找某个特定的method如何使用.最近,发现了一个非常惊艳的app,[Dash](http://kapeli.com/dash),在本地查找各类APIs文档,完全离线的说. 这还不算啥,通过这个workflow,你可以通过过滤各个语言(框架)关键字来查找相关文档.这个流弊的app可是免费的哦,所以,在使用这个workflow前请先安装好Dash.

- [Dash Workflow](https://github.com/willfarrell/alfred-dash-workflow),developed by [@willfarrell](https://github.com/willfarrell/)
- [下载](http://zno.io/Rc3p)
- 触发:`dash` `html` `css` `gem` `angularjs` `Rails` 基本上常用的语言框架库都有了

![](https://7nnqba.dm1.livefilestore.com/y2p8jXte6unkiz4tr4VJFW9zTtwY5LHw6TPj8Bhx6IOm_bI2eU-6GyzBMx8DdEbI6jsLcPz7ttB9S1FqNDA9q8rU4JsouoEspKKgbjj2k-EQLA/alfred-dash-bs-opt.png?psid=1)
![](https://7nnqba.dm2302.livefilestore.com/y2pxzqD2DRpB4rhywnWViFn5zJMrjn59dI_lKaX4wCj9HTSO40hX1NSuRoCJGObJMCHdRB-bkmuYE9OPQePbPjXcaVli3J3dETsLtKIPWx-aHU/alfred-dash-js-opt.png?psid=1)

XguoX:Dash 真心很流弊!巨赞!!!

## Terminal Finder
一些操作我们可能会希望在终端完成,而另一些则希望在Finder完成.这个workflow可以流畅地在这两者之间转换.在终端(iTerm)中打开当前的Finder窗口,反之亦然.

- [TerminalFinder](https://github.com/LeEnno/alfred-terminalfinder), developed by [@LeEnno](https://github.com/LeEnno/)
- [下载](http://zno.io/RkU2)
- 触发: `ft` `tf` `fi` `if`

![](https://7nnqba.dm2302.livefilestore.com/y2plpdCbBvSP7J7seYaCrTIdd0MPtXmqhhqDa-GPRjiO3nBza38J4jpFTetxVTAWfkafkiiTrqaikp-v3_Q-n2bTiBZj7rc3RvdO8OXgjJIhZU/alfred-fi-opt.png?psid=1)
![](https://7nnqba.dm1.livefilestore.com/y2pKTdNdqK9KacL8BTdEMxozuH9vtTAC0qZTdO7wMa3VFtS0rjV_EXpOMKyYx_eqMAGvs6auzVGPw50UN-qYMJyeUVwZNihpyq8TbyYeyqUFyw/alfred-tf-opt.png?psid=1)

XguoX:又是一个巨实用的workflow

## Package Managers Workflow
代码复用是软件开发的一个重要组成部分,现如今我们有很多的方案来构建我们的代码以及搜索使用第三方软件包. 想要使用某个Node.js module? Grunt task?通过这个workflow,你可以快速简便地在一个地方通过你想要的包管理器查找到你想要的插件或者组件.

- [Package Managers Workflow](https://github.com/willfarrell/alfred-pkgman-workflow), developed by [@willfarrell](https://github.com/willfarrell/)
- [下载](https://github.com/willfarrell/)
- 触发: `bower` `grunt` `npm` `composer` `gems` `pear` `pypi` `cocoa` `brew` `alcatraz` `rpm` `maven` `docker`

![](https://7nnqba.dm1.livefilestore.com/y2ppAHAT45dOTn6bKbLSb4zC8iyK4bl8qlXjz7dsYx3h3lceulR1kHbZdz1wS5XiA5V6pzdJGxrpjKI-gwF6IpD1YNBNZCfYrQjp8YCi1R5fC8/alfred-pkg-npm-opt.png?psid=1)
![](https://7nnqba.dm2302.livefilestore.com/y2pWCd9J95EnTh62aMYZ9zHDatkpfUg4qKScGsMpA-Fhy4FBtFBsJ8PkT-flubEWPpzY36rvmSRkv874m_pllj_80s9B0GaEu5XxWonxf6V02o/alfred-pkg-bower-opt.png?psid=1)
![](https://7nnqba.dm1.livefilestore.com/y2pQTFJVwQuk4cV5yWHSoFYQCOanTC6JCB5UnfAqtUNouF6M38QLKC-k1d97hzFQi5HOGcI44owdzMcMJJUvmhJS-eLWBgwQjKEe4FDjwh6TWo/alfred-pkg-grunt-opt.png?psid=1)

##Colors
不用再每次想要转换某个颜色格式的时候打开Photoshop了.通过这个workflow可以很轻易在HEX, RGB, HSL这些个颜色格式之间转换.

- [Colors](https://github.com/TylerEich/Alfred-Extras/tree/master/Source/Colors), developed by [@TylerEich](https://github.com/TylerEich/)
- [下载](http://zno.io/RcFz)
- 触发: `#` `rgb` `hsl` `c`

![](https://7nnqba.dm2301.livefilestore.com/y2p57G91c-OHhp8vWr5mZDL3mvkawhRy64cCzIWzNEBlqL_EVbapnnvDeZUcVdCVtDJ4ow6hfm0cE1yB5AUViUp2A81StX_YpA7T_5mR_B7Rqk/alfred-colors-opt.png?psid=1)


## Jenkins Workflow
做单元测试固然是好,但是每更改一行代码就手动跑一次测试的话会让人抓狂的. 为了得到更好的代码质量,我们需要跑跟更多的测试,或者至少的自动运行那些我们已经在跑的测试. 这就是为嘛**持续集成系统**那么重要.通过这个workflow,你可以列出[Jenkins](http://jenkins-ci.org/)的所有工作以及它们的状态.

- [Jenkins Workflow for Alfred v2](https://github.com/jeroenseegers/alfred-jenkins-workflow), developed by [@jeroenseegers](https://github.com/jeroenseegers/)
- [下载](https://github.com/jeroenseegers/alfred-jenkins-workflow/raw/master/Jenkins.alfredworkflow)
- 触发: `jenkins status`

![](https://7nnqba.dm2302.livefilestore.com/y2p1OLZbpkCxnoR1-ucaW61I99PIVPvOO_136IItknkfT8GvIGdr9lMaF8P78koNZB9q4_Wxld8CzdfdYegwtcTkOMkdbAtb1A3kPK-8WOlsHw/alfred-jenkins-opt.png?psid=1)

XguoX:好吧,这玩意没接触过

## Open in FileZilla

目前来说传输文件到Web服务器的最流行方式还是使用FTP. 而这个workflow可以帮助你快速地通过[FileZilla](https://filezilla-project.org/)连接到远程服务器端. FileZilla也是一个免费的应用,所以,在用这个workflow之前请记得先安装之.

- [Open in FileZilla](https://github.com/jeffmagill/alfred-open-in-filezilla), developed by [@jeffmagill](https://github.com/jeffmagill/)
- [下载](http://zno.io/RnTx)
- 触发: `fz`

在用其他FTP客户端吗?请猛击 [Extras](https://github.com/zenorocha/alfred-workflows/wiki/Extras#ftp)

![](https://7nnqba.dm1.livefilestore.com/y2pb9b3kwcHV5UkchYzjh4f-Jw_YMi0uHVYkNeM_0A4g4859bfjcc49jLLQcyTpsOu2f-hOlh2D7sDSGAt65humZvmCuy3QFlH9tJHMAhlgLp4/alfred-fz-opt.png?psid=1)

## Domainr Workflow

不想错过一些帅气的域名的话,可以通过[Domainr](https://domai.nr/) APIs快速查找.

- [Domainr Workflow](https://github.com/dingyi/Alfred-Workflows/tree/master/Domainr), developed by [@dingyi](https://github.com/dingyi/)
- [下载](http://zno.io/RctP)
- 触发: `domainr`

![](https://7nnqba.dm2301.livefilestore.com/y2pQl13GHOojZONNNtZnsAeO2IKT6fe1AMAt0uR-SSL5MJQgw-wJ66ZqFNHXS4t7S5X4Osl31-d0jAJ65uishVIPvmNU7L09CfJ19v_Gxi1f7A/alfred-domain-opt.png?psid=1)

## Encode / Decode

有时候, 我们需要把一些UTF-8字符转换成HTML编码,或是解码某个URL. 使用 Encode / Decode , 这些杂碎的事情将不再需要浪费那么多的时间了.

- [Encode / Decode](https://github.com/willfarrell/alfred-encode-decode-workflow), developed by [@willfarrell](https://github.com/willfarrell/)
- [下载](http://zno.io/RcCX)
- 触发: `encode` `decode`

![](https://7nnqba.dm2302.livefilestore.com/y2pKvZ6_kJuc6jkwn90DdrOihAOhM5W7fKS7-AoHkhTC41bHpKjagM54XHIYXBYOSPuxivPJ58mAmETym0mdNXAip6qJBJCkxap46HOKzj8W_E/687474703a2f2f662e636c2e6c792f6974656d732f324a336d3147314-e34363035304930453077336e2f616c667265642d656e636f64652e706e67-opt.png?psid=1)

## Font Awesome Workflow

Font icons很好很强大, 只需简单地输入类似的`<i class="my-icon-name"></i>`. 但问题是,我们经常没能准确地记住我们需要的某个icon的类名,以至于老需要去翻看文档. 现在的话通过这个workflow我们可以很轻易地查找到[Font Awesome](http://fortawesome.github.io/Font-Awesome/)的icon集.

- [Font Awesome Workflow for Alfred 2](https://github.com/ruedap/alfred2-font-awesome-workflow), developed by [@ruedap](https://github.com/ruedap/)
- [下载](http://zno.io/RcJ3)
- 触发: `fonta`

![](https://7nnqba.dm2301.livefilestore.com/y2pHAy4Zg3y3qg293hawHzh1DKDkB1ZUd7Ktbw9BtrL7QxRzvJPGIR-B_OUF-fjyx3wq1sn4OrZ72XCQdlHOys6gHOqyjGGbFnwiElSVCkMNxU/alfred-fonta-opt.png?psid=1)

XguoX:赞!

## Source Tree Workflow
有人习惯在终端使用Git命令, 也有喜欢使用GUI工具. 如果你属于后者,那么[Source Tree](http://www.sourcetreeapp.com/) workflow 可以帮你列出,查找,打开Git仓库. Source Tree 也是需要在使用这个workflow之前先[下载](http://www.sourcetreeapp.com/)安装的应用.

- [Source Tree](https://github.com/zhaocai/alfred2-sourcetree-workflow), developed by [@zhaocai](https://github.com/zhaocai/)
- [下载](http://zno.io/Ro6V)
- 触发: `st`  `stbookmark`

使用其他的Git客户端吗?请猛击 [Extras](https://github.com/zenorocha/alfred-workflows/wiki/Extras#git-client)

![](https://7nnqba.dm2302.livefilestore.com/y2peE5UpatyaBm2USKGJSTP6O-ATHnG5dIkbF_cgAXjVLyUZIJfAjCi-FdRs2E44Me5gKsYygNc6ojyzuCmFfnUnAePfelKmZpB1OmgngYUh1M/alfred-st1-opt.png?psid=1)


## GitHub Workflow
如果你最喜欢的社交网站是Github,那你一定会想要看看这个.简单快速地查找并在浏览器打开Github上的仓库.

- [GitHub Workflow](https://github.com/gharlan/alfred-github-workflow), developed by [@gharlan](https://github.com/gharlan/)
- [下载](http://zno.io/RcPe)
- 触发: `gh`

![](https://7nnqba.dm2302.livefilestore.com/y2p8Kw5ylIUxXvwnWm1HOp4xQcI2AH9fPU0is_zRQvkLws07H-KINpP86ZbEvywJ8F-IbF4umwGc802UCYyAJwHbXsrQjTD5jR9eni4Hn8v-3Y/alfred-gh-1-opt.png?psid=1)
![](https://7nnqba.dm2302.livefilestore.com/y2pAHDVk52bx4FuN7ArITttR0iKhUmXlAenZxif_r6XKjlY7zIdeL57L86L8wTu4a6dHOptenChQ6hdGK12mz4ia1Awc7F1kf42-nF4rKzkbrg/alfred-gh-2-opt.png?psid=1)
![](https://7nnqba.dm2301.livefilestore.com/y2pca8u5ITeeKV3q7PAYkeJ26fM5wfHUmFvSjhjS3vfxHWTx8zXT5PQgeZagvY_b6gIUFTMYMdO3B5n-DnWzI-g3Gdxd7A4Tf-BQh75CSKr4UA/alfred-gh-3-opt.png?psid=1)

##StackOverflow Workflow
在[StackOverflow](http://stackoverflow.com/)搜寻各类编程问题的答案

- [StackOverflow Workflow](https://github.com/xhinking/Alfred), developed by [@xhinking](https://github.com/xhinking/)
- [下载](http://zno.io/RceO)
- 触发: `st`

![](https://7nnqba.dm2302.livefilestore.com/y2pmY5gf1twVGB0cDn-C51fasEXdMo79AFhftB4mJqKzQwshL_a0djKiDcoVu6-NP1QMQ360iBdpNf8Pp7cJ5qxc_XZbcoC_C427S0ty9cdMmI/alfred-st-opt.png?psid=1)

## TimeZones Workflow

现如今,很多的团队的成员纷纷来自全球各地. 那么,我们不会希望在同事的下班时间去打搅人家.所以,在这之前,我们总会先查看一下对方的当地时间.这个workflow可以巨方便地列出世界各地不同城市的当前时间.

- [TimeZones Workflow](http://www.alfredforum.com/topic/491-timezones-a-world-clock-script-filter-updated-to-v161/), developed by [@CarlosNZ](http://www.alfredforum.com/topic/491-timezones-a-world-clock-script-filter-updated-to-v161/)
- [下载](http://zno.io/Rce5)
- 触发: `tz`

![](https://7nnqba.dm2302.livefilestore.com/y2pEw7Fm3pjrusK0c-ihMHdrPo-1oCLOyC6kjipiDXwTsn7OJS3is0zRPDYu6Sy-M80x29f0lVww1ACceNo2LtgksPzQHW-yf0HQ4ZtgilKZVs/alfred-tz-opt.png?psid=1)

## VirtualBox Control
很不幸地,跨浏览器兼容的仍然是开发人员所面临的一大问题. 测试你的网站在不同浏览器 & 不同操作系统 是否运行正常是件无法逃避的事. 使用虚拟机(比如[VirtualBox](https://www.virtualbox.org/))是当下流行的,可以完成这事的一种方式. 好吧,在用这个workflow之前老规矩,先装上VirtualBox.

- [VirtualBox Control](https://github.com/aiyodk/Alfred-Extensions/tree/master/AlfredApp_2.x/VirtualBox-Control), developed by [@aiyodk](https://github.com/aiyodk)
- [下载](http://zno.io/RyOE)
- 触发: `vm`

在用其他的虚拟机客户端吗?请猛击 [Extras](https://github.com/zenorocha/alfred-workflows/wiki/Extras#virtual-machines)

![](https://7nnqba.dm2301.livefilestore.com/y2psvNKnOmlDIHPBDdt55ZjXGXipYEXF1vywXnx7K0QWXLTlFzs34ntkNcga99mClSZGNzkG2ANop67cWYvEwkJzHDDualJDf7xIPTcKdVy3Og/alfred-vb-1-opt.png?psid=1)
![](https://7nnqba.dm2302.livefilestore.com/y2pUUCXsiAOITeTEiZtDeQSoIQdmNUV0CayHgImh1MpczVHPlU-NUC0rIdVZqkztQ3bUBM7EWiu4SAkbl1eL3EMKFSsd2NFTJpjq8kDoCEJfOE/alfred-vb-2-opt.png?psid=1)


## Create Your Own!

所有的这些workflows都非常的赞并对于大多数人来说很有帮助. 但是,每个人的工作方式不尽相同. 所以我们需要创建真正属于自己的workflow. 其实这个也很是简单. 以下的这个例子仅需不到10秒,就可以创建一个workflow来自动搜寻Smashing Magazine.

![](https://7nnqba.dm2301.livefilestore.com/y2p2GeFzQuRTl9E0-rFjNQrAw7qS_a5RAlZpfNQh9tT-P45-le3UY_fNKu7n1K5mIIq9JNdp83flsG9hvRzDHzcfXGuB56sspc6CnmRyDh2ikE/custom-alfred.gif?psid=1)

## Want More?
[这是作者收集的一些](https://github.com/zenorocha/alfred-workflows/)
此外,Alfred的[官方论坛](http://www.alfredforum.com/forum/3-share-your-workflows/)上也有海量的workflows.

## The End?
一堆帮助你自动化工作流程的技巧,很赞对吧!希望能够对你有所帮助.可能这些会激发你的一些灵感,从而创造,分享你的隐藏技能.

如果你喜欢的workflow没有在这列出,可以在下边的评论当中跟我们分享. 如果你觉得上边提到的那些赞到爆的话,也可以告诉我们哦!


文章翻译自 [Hidden Productivity Secrets With Alfred](http://coding.smashingmagazine.com/2013/10/25/hidden-productivity-secrets-with-alfred/)





