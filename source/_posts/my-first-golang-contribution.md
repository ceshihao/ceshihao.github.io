---
title: 记一次 Golang Contribution
date: 2018-04-19 18:15:13
tags: [golang, gerrit, open-source]
---

源于这个 issue [#24767](https://github.com/golang/go/issues/24767)。正好我最近也在用 `text/template` 这个包来做一些工具（其实主要因为我太弱，这个改动简单……），所以产生了兴趣。

改动很简单，就是有一个 example 和它的描述不相符，补一个符合的 example。但是在代码的提交上花了一些时间。

早就听说 golang 的代码托管在自己的 [Gerrit](https://go.googlesource.com/go) 上，而且提交的流程和一般在 github 上的项目会有些不同。这次终于能自己亲身实践一次。绝大部分步骤都是按照 [Contribute](https://golang.org/doc/contribute.html)上说的来。

# 准备 Contributor 的前期工作

安装 go-contrib-init 工具

```bash
$ go get -u golang.org/x/tools/cmd/go-contrib-init
$ cd /code/to/edit
$ go-contrib-init
```

配置 Gerrit

* 登录 googlesource 并生成 password ，这时在页面上会生成一个脚本。
* 在 shell 里跑这个脚本。
* 在 [Gerrit Review](https://go-review.googlesource.com) 网站上注册自己的账号。

同意 CLA 协议，自己看吧。

# 准备开发环境

安装 git-codereview

```shell
$ go get -u golang.org/x/review/git-codereview
```

配置指令 alias (这一步建议还是配置一下。一开始我没有配，就会导致文档上的指令还要自己脑力转换一下才能跑……)

```
[alias]
	change = codereview change
	gofmt = codereview gofmt
	mail = codereview mail
	pending = codereview pending
	submit = codereview submit
	sync = codereview sync
```

# 正式开始修改代码

这部分还可以参考 git-codereview 的[文档](https://godoc.org/golang.org/x/review/git-codereview)。

下载go的源代码

```shell
$ git clone https://go.googlesource.com/go
$ cd go
```

同步go的主干分支

```shell
$ git checkout master
$ git sync
```

终于可以肆意进行你的改动了

* 提交你的代码

```shell
$ git add/rm/mv <files>
$ git change <branch>
$ git commit
```

这时会默认 `$EDITOR` 指定的编辑器(默认 `vi`)来输入你的 commit message。

# 发送需要review的代码

一条简单的指令

```shell
$ git mail
```

当然也可以稍微复杂一点，指定 reviewer 和 cc

```shell
$ git mail -r joe@golang.org -cc mabel@example.com,math-nuts@swtch.com
```

到这里基本上就大功告成了，等待自己的CL被大牛们review吧。

# 代码审查
由于这个issue Rob Pike之前有过comment，并且之前不一致的example就是他写的。所以他转天很快就在[CL](https://go-review.googlesource.com/c/go/+/107663)上给了comments。

这时，我可有点犯了难。Gerrit 不像 github 上直接在分支上追加 commit 就行了,而是要在 CL 上提交新的 patch set。又马上 google 各种搜索了一通。
用`git commit --amend`解决了问题。重新提交也很快被Rob Pike merge了。

# 总结
虽然这个 contribution 没啥技术含量，但是也算体验了一把 golang contribution 的流程，应该会包含在 go1.11 的 release 中。