---
title: "gin-gonic/gin Hot Reload"
date: 2018-08-16T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

用 [Gin](https://github.com/gin-gonic/gin) 开始写点 API 的东西,  然后就陷入了不断

`⌘ + c`

`go build .`

`./xxx`

= . =

然后搜索一把发现有不少推荐 [realize](https://github.com/oxequa/realize) 这个工具的, 刚开始看文档半天都不知道这是怎么个玩意, 用法总是说的模棱两可的, 捣腾一番也算能用了. [吐槽文档的其实也不少的](https://www.reddit.com/r/golang/comments/7hogxu/new_realize_v20_the_1_golang_task_runner/).

不过其实要做的功夫也很少, 在项目根目录初始化一个 **.realize.yaml**

```yml

settings:
  legacy:
    force: false
    interval: 0s
schema:
- name: whatever
  path: .
  commands:
    run:
      status: true
  watcher:
    extensions:
    - go
    paths:
    - /
    scripts:
      - type: after
        command: ./your_binary_file_name
        output: true
    ignored_paths:
    - .git
    - .realize
    - vendor
```

就完事了, 讲真那堆参数大部分都是靠蒙的 = . =

启动就跑

`realize start --run`

结果就是酱紫

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1fubuknar1ej227y1cmatb.jpg)

