+++
title = "配置阿里云服务器之Docker服务"
date = "2019-07-16T22:04:55+08:00"
description = "使用阿里云服务器配置docker"
tags = ["docker"]
+++

阿里云搜索容器镜像服务。[网址地址](https://cr.console.aliyun.com/)

具体操作看别人的博文:[博文地址](https://yq.aliyun.com/articles/70756)
下面讲一点使用docker注意事项和上传至阿里云

## 保存容器

我们进到容器里一顿`apt update && apt install 巴拉巴拉`
但是只要`exit`一`pull`就发现所有做的都没了。
原来我是没`commit`
命令如下：
`docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]`

OPTIONS:

* --author , -a 作者
* --change , -c 将Docker file指令应用于创建的映像
* --message , -m 提交信息
* --pause , -p 在提交期间暂停容器

## 因为我得是私有镜像仓库可能会有以下情况

![picture](%E9%85%8D%E7%BD%AE%E9%98%BF%E9%87%8C%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B9%8BDocker%E6%9C%8D%E5%8A%A1/dockerpusherror.png)
如果出现这种情况那么我们可以直接pull该镜像即可

## 为什么要有Docker file？

他可以快速创建自定义的Docker镜像。
具体就是能让我们自定义一个镜像不会给别人感觉是黑箱操作。我们任何RUN操作都是大家看的见的。他们知道我们这个镜像里装了什么东西，这不就是我们所想要的透明吗！
Dockerfile官网的最佳实践：[最佳实践](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

## 配置mongo镜像

步骤很简单

1. docker pull mongo
2. docker tag mongo registry.cn-hangzhou.aliyuncs.com/xxxxxxx
3. docker run -t --name database --net networkName -p 49155:27017 -v /data/db:/data/db --log-opt max-size=64m --log-opt max-file=10 registry.cn-hangzhou.aliyuncs.com/xxxxx
当然你可能会问那个`--net` 是个什么玩意儿。
这可是个宝贝哇。
当我们有多个容器互相需要连接时，在以前的`docker` 我们是用 `link` 现在我们只需要`docker create network networkName`
之后我们只要run相互之间有关联的容器加上`--net networkName` 就ok啦！

**当然你有多个容器之间需要互相连接，推荐使用 Docker Compose。**

## 自定义自己的服务端镜像

我们就自定义一个跑后端的吧

1. docker pull node:8.9.4-alpine   //该镜像体积很小
1. docker tag node registry.cn-hangzhou.aliyuncs.com/xxxxxxx
1. 写个Dockerfile吧！

```shell
FROM registry.cn-hangzhou.aliyuncs.com/xxxxx

RUN mkdir -p /usr/src/app

COPY . /usr/src/app

WORKDIR /usr/src/app

CMD npm run start:server

EXPOSE 4040
```

1. 然后写点`docker run balbalbal`啊之类的脚本这样我们就能很方便的使用啦。

## 遇到的一些小问题

![picture](%E9%85%8D%E7%BD%AE%E9%98%BF%E9%87%8C%E4%BA%91%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B9%8BDocker%E6%9C%8D%E5%8A%A1/docker-mongo-url.png)
这种情况我去Stack Overflow上瞅了瞅。就是一天！！！！
发现一个0赞的答案。。。我把。。。。

```javascript
'mongodb://127.0.0.1:27017'
to
'mongodb://database:27017'
// 容器名
```

他居然就work了。。。。我真的是。。。
