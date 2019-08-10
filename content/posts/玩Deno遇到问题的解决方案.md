---
title: "玩Deno遇到问题的解决方案"
date: 2019-05-27T08:00:00.000Z
description: "使用阿里云服务器配置docker"
tags: ["JavaScript"]
---

最近有个新的 Deno 项目是由 node 原作者 ry 发起的，瞬间火爆，star 数飞起。
但是在搭建环境的过程中还是有些小问题，但是在友好的 issue 下都得到了解决！
再此总结，并对那些在 issue 里无私贡献自己解决方案的人点赞！开源社区的和谐需要大家去一起努力。

附上项目地址:
[GitHub - ry/deno: A secure TypeScript runtime on V8](https://github.com/ry/deno)

## Step1

在开始之前请准备好 v*p*n\*。
大家需要去安装 `Go` 环境.并且去 `export` 各种 Go 相关的环境变量.

以下是方正大佬给我提供解决方案，很感谢。
<https://github.com/ry/deno/issues/92>
Mac OS 可以参考以下：

```shell
export GOROOT=/usr/local/go # where your `go` sitting, usually here ( Mac )
export GOPATH=$HOME/go # means `~/go`
export PATH=$PATH:$HOME/go/bin:$GOPATH/bin
```

如果是 Mac 的话，我们还需要去安装`xcode-select`
<http://osxdaily.com/2014/02/12/install-command-line-tools-mac-os-x/>

我们还需要安装 Protobuf 。Ubuntu 下：

```shell
cd ~
wget https://github.com/google/protobuf/releases/download/v3.1.0/protoc-3.1.0-linux-x86_64.zip
unzip protoc-3.1.0-linux-x86_64.zip
export PATH=$HOME/bin:$PATH
```

Mac 下简单粗暴：
`brew install protobuf`
再来装一个`README`中没提及的
`brew install pkg-config`

## Step2

Ok 以上一切正常，没出啥幺蛾子。
我们继续，现在需要 `protoc-gen-go` 和 `go-bindata`:

```shell
go get -u github.com/golang/protobuf/protoc-gen-go
go get -u github.com/jteeuwen/go-bindata/...
```

这步需要等一小伙，记得一定要 v*p*n 啊!

## Step3

现在我们来困难重重的 `v8worker2` 啦。我们需要 get 然后 build 它。大概会花 30min

```shell
go get -u github.com/ry/v8worker2
cd $GOPATH/src/github.com/ry/v8worker2
./build.py --use_ccache
```

接下来大家可能遇到的情况：
![picture](https://blogaaaaxzh.oss-cn-hangzhou.aliyuncs.com/go-get.png)
这种情况说明我们 `clone` 下的 v8 是有损坏的，然后我们需要做以下操作

1

```shell
cd $GOPATH/src/github.com/ry/v8worker2
rm -rf v8
git clone https://github.com/v8/v8.git
cd v8
git checkout fe12316ec4b4a101923e395791ca55442e62f4cc
```

或者

2

```shell
export PATH=$PATH:$GOPATH/src/github.com/ry/v8worker2/depot_tools
cd $GOPATH/src/github.com/ry/v8worker2
rm -rf v8
fetch v8
cd v8
git checkout fe12316
```

因为用第一种方法我发现我的 vpn 不能快速的下载所以就尝试了第二种方法。如果你键入`fetch`发现命令行出现`command not found: fetch`。
你可以尝试

```shell
cd $GOPATH/src/github.com/ry/v8worker2
depot_tools/./fetch v8
```

感谢 [go get v8worker2 Direct fetching of that commit failed? · Issue #92 · ry/deno · GitHub](https://github.com/ry/deno/issues/92) 下面给出解决方案的人@wbgbg ，@qti3e，@ztplz

如果你发现自己的`depot_tools`文件夹下啥都没有。你需要执行以下命令

```shell
cd $GOPATH/src/github.com/ry/v8worker2
git submodule update --init
```

之后再去

```shell
cd $GOPATH/src/github.com/ry/v8worker2
./build.py
```

你看见了以下，那么就恭喜啦！他在编译了
![picture](https://blogaaaaxzh.oss-cn-hangzhou.aliyuncs.com/buildv8woker2.png)

## Step4

最后一步

```shell
go get -u github.com/ry/deno/...
cd $GOPATH/src/github.com/ry/deno
make # 稍等片刻
./deno testdata/001_hello.js # Output: Hello World
```

又是熟悉的 Hello World！
在`go get -u github.com/ry/deno/...`遇到以下问题不要急，直接`make deno`走你！
[deno/dispatch.go:10:26: undefined: BaseMsg · Issue #71 · ry/deno · GitHub](https://github.com/ry/deno/issues/71)

```shell
$ go get  -u github.com/ry/deno/...
# github.com/ry/deno
../deno/dispatch.go:10:26: undefined: BaseMsg
../deno/dispatch.go:30:10: undefined: BaseMsg
../deno/dispatch.go:62:14: undefined: BaseMsg
../deno/dispatch.go:68:34: undefined: Msg
../deno/dispatch.go:119:13: select case must be receive, send or assign recv
../deno/fetch.go:13:11: undefined: Msg
../deno/fetch.go:16:8: undefined: Msg_FETCH_REQ
../deno/fetch.go:29:14: undefined: Msg
../deno/main.go:38:15: undefined: Asset
../deno/main.go:110:19: undefined: Msg
../deno/main.go:110:19: too many errors
```

如果你在这`make deno`遇到了以下问题，说明你第一步方正大佬给出的解决方案还没做。回去乖乖配置吧！
![picture](https://blogaaaaxzh.oss-cn-hangzhou.aliyuncs.com/makedeno.png)

完成！
![picture](https://blogaaaaxzh.oss-cn-hangzhou.aliyuncs.com/denowork.png)
还有别的问题可以看以下 issue
![]([when I run ./build.py —use_ccache ,there was a mistake · Issue #60 · ry/deno · GitHub](https://github.com/ry/deno/issues/60))
