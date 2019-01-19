---
title: "Golang, Delve & Visual Studio Code"
date: 2018-07-23T16:01:23+08:00
draft: false
tags: ["Go"]
categories: ["Go"]
---

从 Ruby 切换到 Go 一个不顺手的地方是没有 REPL 随手实验, 虽说有 [Go Playground](https://play.golang.org/), 但还是喜欢 Local 多一些, 而且有一些操作线上很难甚至完成不了的. 另外一个不顺手地方是「没有」 pry 这样的 debug 利器. 其实不是没有, 只是我对 Go 的生态没了解多少. 所以今天扯一点 [Delve](https://github.com/derekparker/delve), 其实官方有 GDB, 但是貌似对 goroutine 不友好, 好吧, 官方自己说的, 所以, 直奔了 **Delve**.

作为测试用直接拿之前看的[一篇短文](https://schier.co/blog/2015/04/26/a-simple-web-scraper-in-go.html) 源码, 提取出来放到 [gist](https://gist.github.com/xguox/2dc50f58e7c9149cb9d8c9d1e59b1181) 了.

```go
go get -u github.com/derekparker/delve/cmd/dlv
```

如无意外, `which dlv` 在 **$GOPATH/bin** 下就会多了一个 **dlv** 的二进制可执行文件, 执行 `dlv` 就有挺详细的文档了. 再深入一些还可以执行类似 `dlv attach --help`

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1ftlbqw0n8dj22801e0wqa.jpg)

最简单的用法,

```go
➜ dlv debug scraper.go
Type 'help' for list of commands.
(dlv)
```

这提示信息比 pry 要详细多得多, 毕竟只是看 help 文档描述功能也多不少, 还有一大票 alias, 瞬间感动.

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1ftlbwn1muaj22801e0n9k.jpg)

```go
(dlv) break main.main
Breakpoint 1 set at 0x12c42eb for main.main() ./scraper.go:76
(dlv) continue
> main.main() ./scraper.go:76 (hits goroutine(1):1 total:1) (PC: 0x12c42eb)
    71:				}
    72:			}
    73:		}
    74:	}
    75:
=>  76:	func main() {
    77:		foundUrls := make(map[string]bool)
    78:		seedUrls := os.Args[1:]
    79:		fmt.Println(seedUrls)
    80:		// Channels
    81:		chUrls := make(chan string)
(dlv)
```

到了这里很经常习惯性就 enter 下一步, 然后其实就退出了 = . =, 这里还得多一步 **next** 操作先.

或者直接定位到哪一行,不过还是得先 next, 只要 next 了一次以后就可以直接一直 enter...这个倒是很方便的呢

```go
(dlv) break scraper.go:78
Breakpoint 1 set at 0x12c439d for main.main() ./scraper.go:78
(dlv) continue
> main.main() ./scraper.go:78 (hits goroutine(1):1 total:1) (PC: 0x12c439d)
    73:		}
    74:	}
    75:
    76:	func main() {
    77:		foundUrls := make(map[string]bool)
=>  78:		seedUrls := os.Args[1:]
    79:		fmt.Println(seedUrls)
    80:		// Channels
    81:		chUrls := make(chan string)
    82:		chFinished := make(chan bool)
    83:
```

习惯于 pry 的我到这里很自然就输入变量名获取值,然后就歇菜了,得用 **print 或者 p**

```go
(dlv) foundUrls
Command failed: command not available
(dlv) p foundUrls
map[string]bool []
```

进入某个方法的 **step 或者缩写 s**, 甚至是输出所有局部变量的 **locals**, 总之在 Go 里边找到了 pry 更好的感觉了.

----------------------

然后是 Visual Studio Code, 在 VS Code 里边写 Go 的话基本都会装 M$ 的官方 go 拓展吧, 点 **调试 > 打开配置** 说是要配置啥不过我这里默认的就可行了.

![](http://wx2.sinaimg.cn/large/62fdd4d5gy1ftlbqy3m9vj22801e04cv.jpg)

![](http://wx4.sinaimg.cn/large/62fdd4d5gy1ftlbqysm64j22801e0tln.jpg)

在行号前面打个断点, 然后启动调试, 或者 Fn + F5, 之后如果想查看某个变量的值 直接 hover 到变量名就可以了.

![](http://wx1.sinaimg.cn/large/62fdd4d5gy1ftlbqzl0tij22801e0nd1.jpg)

不过这里要注意的是启动调试得把编辑器 focus 在 `main.go` 才可以, 断点还是可以打在任意地方的, 不然就会报 **Failed to continue: Check the debug console for details. Can not debug non-main package Process exiting with code: 1**