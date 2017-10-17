---
title: git常用操作
date: 2017-10-17 10:17:07
tags: java
---
## 下面记录一些经常会用到的命令：

查看远程分支, 加上 -a 参数可以查看远程分支，远程分支会用红色表示出来（如果你开了颜色支持的话），不加 -a 则查看本地分支：
``` bash
$ git branch -a
  dev
* master
  remotes/origin/HEAD -> origin/master
  remotes/origin/master
  remotes/origin/prod
```
  
删除远程分支和tag, 在Git v1.7.0 之后，可以使用这种语法删除远程分支：
``` bash
$ git push origin --delete <branchName>
```

删除tag这么用：
``` bash
$ git push origin --delete tag <tagname>
```

否则，可以使用这种语法，推送一个空分支到远程分支，其实就相当于删除远程分支：
``` bash
$ git push origin :<branchName>
```
这是删除tag的方法，推送一个空tag到远程tag：
``` bash
$ git tag -d <tagname>
$ git push origin :refs/tags/<tagname>
```
两种语法作用完全相同。

将远程git仓库里的指定分支拉取到本地（本地不存在的分支）
当我想从远程仓库里拉取一条本地不存在的分支时：
``` bash
$ git checkout -b 本地分支名 origin/远程分支名
```
这个将会自动创建一个新的本地分支，并与指定的远程分支关联起来。

例如远程仓库里有个分支dev2,我本地没有该分支，我要把dev2拉到我本地：
``` bash
$ git checkout -b dev2 origin/dev2
```

若成功，将会在本地创建新分支dev2,并自动切到dev2上。
如果出现提示：

    fatal: Cannot update paths and switch to branch 'dev2' at the same time.
    Did you intend to checkout 'origin/dev2' which can not be resolved as commit?

表示拉取不成功。我们需要先执行
``` bash
$ git fetch
```

然后再执行下面的命令即可
``` bash
$ git checkout -b 本地分支名 origin/远程分支名
```
