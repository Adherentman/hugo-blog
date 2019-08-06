---
title: "git Empty Object File"
date: 2019-08-06-T11:02:08+08:00
description: "一次突然断电。。导致之前的commit的没了。。`status`的时候发现全变空了。。解决过程在如下重现"
tags: ["运维"]
---

```shell
~/1mu(1803) » git status

error: object file .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b is empty
error: object file .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b is empty
fatal: loose object 84903eeaf8e0715b6ae944ceffa9677a7b7bab2b (stored in .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b) is corrupt
```

```shell
~/1mu(1803) » git fsck —full

error: object file .git/objects/16/cbec146f596b542e174875e8ad0c8c930b2568 is empty
error: unable to mmap .git/objects/16/cbec146f596b542e174875e8ad0c8c930b2568: 没有那个文件或目录
error: 16cbec146f596b542e174875e8ad0c8c930b2568: object corrupt or missing: .git/objects/16/cbec146f596b542e174875e8ad0c8c930b2568
error: object file .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b is empty
error: unable to mmap .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b: 没有那个文件或目录
error: 84903eeaf8e0715b6ae944ceffa9677a7b7bab2b: object corrupt or missing: .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b
error: object file .git/objects/93/53160448a191e0b2cf58dc4c6d7562df0a1e97 is empty
error: unable to mmap .git/objects/93/53160448a191e0b2cf58dc4c6d7562df0a1e97: 没有那个文件或目录
error: 9353160448a191e0b2cf58dc4c6d7562df0a1e97: object corrupt or missing: .git/objects/93/53160448a191e0b2cf58dc4c6d7562df0a1e97
error: object file .git/objects/af/0735cb070fe8c8e415102f00e9a9d8825cd8a8 is empty
error: unable to mmap .git/objects/af/0735cb070fe8c8e415102f00e9a9d8825cd8a8: 没有那个文件或目录
error: af0735cb070fe8c8e415102f00e9a9d8825cd8a8: object corrupt or missing: .git/objects/af/0735cb070fe8c8e415102f00e9a9d8825cd8a8
error: object file .git/objects/fe/a063e56e69f6739e1648ddccd0c5d55a0d052f is empty
error: unable to mmap .git/objects/fe/a063e56e69f6739e1648ddccd0c5d55a0d052f: 没有那个文件或目录
error: fea063e56e69f6739e1648ddccd0c5d55a0d052f: object corrupt or missing: .git/objects/fe/a063e56e69f6739e1648ddccd0c5d55a0d052f
检查对象目录中: 100% (256/256), 完成.
检查对象中: 100% (26504/26504), 完成.
error: object file .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b is empty
error: object file .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b is empty
fatal: loose object 84903eeaf8e0715b6ae944ceffa9677a7b7bab2b (stored in .git/objects/84/903eeaf8e0715b6ae944ceffa9677a7b7bab2b) is corrupt
```

```shell
~/1mu(1803) » cd .git
~/1mu/.git(1803) » find . -type f -empty -delete -print
执行完后再次

~/1mu/.git(1803) » git fsck —full

检查对象目录中: 100% (256/256), 完成.
检查对象中: 100% (26504/26504), 完成.
error: HEAD: invalid sha1 pointer 84903eeaf8e0715b6ae944ceffa9677a7b7bab2b
error: refs/heads/1803: invalid sha1 pointer 84903eeaf8e0715b6ae944ceffa9677a7b7bab2b
error: 9353160448a191e0b2cf58dc4c6d7562df0a1e97: invalid sha1 pointer in cache-tree
dangling blob 8d24466fd1c78e4210ae30e2281cd15c85162e86
dangling blob 332b8381fc34238dca0a0d181edb679f5c4abfa9
dangling blob de48ce55778be6f22e31d00e7afbed7c7cbc9b20
dangling blob 1a58a089bf85e7115cdda86009c977f9a664844e
dangling blob d55912305163fe7e28e08ead530e50097da2add7
dangling blob 05692eae84da1c4d0882b7b9b681eb41dc61dece
dangling blob 5f69a32eaf9a72ef072e113c60f862acd1a6333c
dangling blob fb69932afa1ec8b6ec09a9131f63422cfec664b7
dangling blob 66706adb33fb8f5365c42e463f399cde6202c675
dangling blob ac99430374779d69a69eba7e0def4c2ceb234eec
dangling blob 63a40e7b1b19c1a148c2ec34de0804dbaec96ff5
dangling blob a4e4e0a5ccb89ecc38b2339b84e84b577228f60d
dangling blob 3fe6af5f741e750da67d9f47dc54310969826429

```

```shell
# 选择自己的分支 就能看见之前自己的commit信息
~/1mu(1803) » tail -n 2 .git/logs/refs/heads/1803

9ac273e691830844014b1f1ff85e071bd5c061a2 54ac2f761953dcd0af684b781513472acf553719 xzh <zihao.xu@1mu.club> 1538212290 +0800  commit: 改变作品的数据结构
```

```shell
# 回到你想回到的分支上
~/1mu(1803) » git update-ref HEAD 54ac2f761953dcd0af684b781513472acf553719
```

```shell
~/1mu(1803*) » git fsck —full                                                                                                                                                              xuzihao@xuzihao
检查对象目录中: 100% (256/256), 完成.
检查对象中: 100% (26504/26504), 完成.
error: 9353160448a191e0b2cf58dc4c6d7562df0a1e97: invalid sha1 pointer in cache-tree
dangling blob 8d24466fd1c78e4210ae30e2281cd15c85162e86
dangling blob 332b8381fc34238dca0a0d181edb679f5c4abfa9
dangling blob de48ce55778be6f22e31d00e7afbed7c7cbc9b20
dangling blob 1a58a089bf85e7115cdda86009c977f9a664844e
dangling blob d55912305163fe7e28e08ead530e50097da2add7
dangling blob 05692eae84da1c4d0882b7b9b681eb41dc61dece
dangling blob 5f69a32eaf9a72ef072e113c60f862acd1a6333c
dangling blob fb69932afa1ec8b6ec09a9131f63422cfec664b7
dangling blob 66706adb33fb8f5365c42e463f399cde6202c675
dangling blob ac99430374779d69a69eba7e0def4c2ceb234eec
dangling blob 63a40e7b1b19c1a148c2ec34de0804dbaec96ff5
dangling blob a4e4e0a5ccb89ecc38b2339b84e84b577228f60d
dangling blob 3fe6af5f741e750da67d9f47dc54310969826429
```

到这步基本上就已经解决了。

```shell

~/1mu(1803*) » rm .git/index
~/1mu(1803*) » git reset

重置后取消暂存的变更：
M interface/www/cms2/index.html
M interface/www/cms2/package.json
M interface/www/cms2/webpack.common.js

~/1mu(1803*) » git status                                                                                                                                                                   xuzihao@xuzihao
位于分支 1803
您的分支领先 ‘origin/1803' 共 2 个提交。
  （使用 “git push" 来发布您的本地提交）

尚未暂存以备提交的变更：
  （使用 “git add <文件>..." 更新要提交的内容）
  （使用 “git checkout -- <文件>..." 丢弃工作区的改动）

  修改：     interface/www/cms2/index.html
  修改：     interface/www/cms2/package.json
  修改：     interface/www/cms2/webpack.common.js

修改尚未加入提交（使用 “git add" 和/或 "git commit -a"）
```

为了放心再次检查下

```shell
~/1mu(1803*) » git fsck —full                                                                                                                                                              xuzihao@xuzihao
检查对象目录中: 100% (256/256), 完成.
检查对象中: 100% (26504/26504), 完成.
dangling blob 8d24466fd1c78e4210ae30e2281cd15c85162e86
dangling blob 332b8381fc34238dca0a0d181edb679f5c4abfa9
dangling blob 2d32a2045e6e9e2c9eac5e94fc7d4105b2fe7bbf
dangling blob 3e408a4fc423ba3d78d4c9a526f70a12ce749c4a
dangling blob de48ce55778be6f22e31d00e7afbed7c7cbc9b20
dangling blob 1a58a089bf85e7115cdda86009c977f9a664844e
dangling blob d55912305163fe7e28e08ead530e50097da2add7
dangling blob 05692eae84da1c4d0882b7b9b681eb41dc61dece
dangling blob 5f69a32eaf9a72ef072e113c60f862acd1a6333c
dangling blob fb69932afa1ec8b6ec09a9131f63422cfec664b7
dangling blob 66706adb33fb8f5365c42e463f399cde6202c675
dangling blob ac99430374779d69a69eba7e0def4c2ceb234eec
dangling blob 63a40e7b1b19c1a148c2ec34de0804dbaec96ff5
dangling blob a4e4e0a5ccb89ecc38b2339b84e84b577228f60d
dangling blob 3fe6af5f741e750da67d9f47dc54310969826429
dangling blob b0e76a232b70b823379a1cfae36dbf443e46fa6e
```
