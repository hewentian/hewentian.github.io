---
title: git 学习笔记
date: 2017-10-17 10:17:07
tags: git
categories: other
---

### 设置用户名和邮箱
全局设置方式：
``` bash
$ git config --global user.name "hewentian"
$ git config --global user.email "wentian.he@qq.com"
```

单独对某个仓库设置方式：
``` bash
$ git config user.name "hewentian"
$ git config user.email "wentian.he@qq.com"
```

可以通过如下方式查看设置结果：
``` bash
$ git config user.name
$ git config user.email
```


### 创建SSH KEY
``` bash
$ ssh-keygen -t rsa -C "youremail@example.com"
```


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

Git 比较不同版本文件差异的常用命令格式：
``` bash
$ git diff	查看尚未暂存的文件中更新了哪些部分
$ git diff filename	查看尚未暂存的某个文件更新了哪些部分
$ git diff –-cached	查看已经暂存起来的文件中和上次提交的版本之间的差异
$ git diff –-cached filename	查看已经暂存起来的某个文件和上次提交的版本之间的差异
$ git diff <commitId1> <commitId2>	查看某两个版本之间的差异
$ git diff <commitId1>:filename <commitId2>:filename	查看某两个版本的某个文件之间的差异，方式一
$ git diff <commitId1> <commitId2> -- filename	查看某两个版本的某个文件之间的差异，方式二
```

在git中查看历史的命令主要是git log，要查看某个文件的修改历史：
``` bash
$ git log -- begin.txt
```

可以添加不同的选项让输出的内容或格式有所不同。
``` bash
$ git log -p -- begin.txt
```
-p 选项可以输出每次提交中的diff， -p会把输出搞得很长、很乱，不容易找到重点。
还有另外一种方式是：
``` bash
$ git log --pretty=oneline -- filename
```
在log 命令中加入 --pretty=oneline 选项只能看到comments，看不到提交的用户和日期。
这也能够让我们集中注意力快速找到关注的提交记录。

然后使用 git show命令查看完整的提交内容。

当然，除了命令行工具您也可以使用GUI程序查看文件的历史记录：
gitk filename


查看历史中的文件内容

当我们使用 git log 命令找到了某次提交，并且想看看这次提交时文件的完整内容。
这时，我们需要使用 git show 命令：
``` bash
$ git show <commitId>:filename
```
也许此时你并不是看看就算了，你想要使用这个版本的文件更新工作区中的文件。
直接从 git show 命令的输出中拷贝内容是个不错的选择。
但也可以通过组合使用不同的命令来实现：
``` bash
$ git checkout <commitId> --filename
$ git reset filename
```
此时只有工作区被更新了(你也可以把他当做是一次回滚操作)。

### git 将单个文件恢复到历史版本的正确方法如下：
``` bash
$ git log 要恢复的文件路径
$ git reset commit_id 要恢复的文件路径
$ git checkout -- 要恢复的文件路径
```

### git的.gitignore规则不生效的解决办法
有时候定义了规则后发现并未生效，原因是.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。那么解决方法就是先把本地缓存删除（改变成未track状态），然后再提交：

``` bash
$ git rm -r --cached .
$ git add .
$ git commit -m 'update .gitignore'
```

### 在开发的过程中，想知道远程的仓库是否有同事提交代码可以使用如下命令
``` bash
$ cd {your_repo_dir}
$ git fetch
$ git status
```


### git远程删除分支后，本地 git branch -a 依然能看到的解决办法
远程删掉 release/1.1 分支后，本地使用如下命令依然还能看到它，如下：
``` bash
$ git fetch
$ git branch -a
  develop
* developLocal
  master
  remotes/origin/HEAD -> origin/master
  remotes/origin/develop
  remotes/origin/feature/develop_1.1
  remotes/origin/master
  remotes/origin/release/1.1	# 看，它还在
```
此时我们使用如下命令，可以查看remote地址，远程分支，还有本地分支与之相对应关系等信息
``` bash
$ git remote show origin 
* remote origin
  Fetch URL: ssh://git@gitlab.hewentian.com:12022/hexo.git
  Push  URL: ssh://git@gitlab.hewentian.com:12022/hexo.git
  HEAD branch: master
  Remote branches:
    develop                         tracked
    feature/develop_1.1             tracked
    master                          tracked
    refs/remotes/origin/release/1.1 stale (use 'git remote prune' to remove) # 提示该分支已经不存在了
  Local branches configured for 'git pull':
    develop merges with remote develop
    master  merges with remote master
  Local refs configured for 'git push':
    develop pushes to develop (up to date)
    master  pushes to master  (local out of date)
```
根据提示，使用如下命令，删掉本地的远程不存在的分支
``` bash
$ git remote prune origin 
Pruning origin
URL: ssh://git@gitlab.hewentian.com:12022/hexo.git
 * [pruned] origin/release/1.1
```
这样就删除了那些远程仓库不存在的分支


### git status 显示中文和解决中文乱码
在默认设置下，中文文件名在工作区状态输出，中文名不能正确显示，而是显示为八进制的字符编码。
解决方案，在bash提示符下输入：
``` bash
$ git config --global core.quotepath false
```
core.quotepath设为false的话，就不会对0x80以上的字符进行quote，中文显示正常。


### 修复生产问题的方法
在某些情况下，本地可能提交了很多次代码，但此时发现生产有个严重BUG需马上修复，但本地的代码还不能推到生产。此时的解决方法是：
1. 先找到生产上面最新的`commit id`，然后在本地以此`commit id`创建一个新分支；
2. 在新分支修复，然后将此分支推到生产；
3. 在生产构建该分支。

相关操作如下：
``` bash
$ git pull origin master
$ git log
$ git checkout -b fix20191106 469ac6e84d78e84cec59f9c3af8453fb21ce222b
$ git add {修改过的文件}
$ git commit -m "修改原因说明"
$ git push origin fix20191106
```


### 将本地仓库提交到多个远程仓库
一般情况下，在本地有一个gitLab，在生产也有一个gitLab。我们要将本地代码提交到生产的gitLab，方法如下：
将本地仓库和生产的远程仓库绑定：

        cd ~/ProjectD/gitHub/bigdata
        git remote add prod git@github.com:hewentian/bigdata.git

在本地切换到和生产仓库一样的分支，然后提交到生产：

        git checkout develop
        git push prod develop


### git本地分支和远程分支建立追踪关系的三种方式
1. 手动建立追踪关系
        $ git branch --set-upstream-to=<远程主机名>/<远程分支名> <本地分支名>

2. push时建立追踪关系
        $ git push -u <远程主机名> <本地分支名>

加上-u参数，这样push时，本地指定分支就和远程主机的同名分支建立追踪关系。

3. 新建分支时建立跟踪关系
        $ git checkout -b <本地分支名> <远程主机名>/<远程分支名>


### 版本回退
首先使用`git log`找到要回退到的`commit id`，然后在本地`reset`回此`commit id`，最后强制推到远程。

``` bash
$ git log
$ git reset --hard 469ac6e84d78e84cec59f9c3af8453fb21ce222b
$ git push origin prod --force
```


### Change Git Remote Origin
1. Change Git Remote URL
``` bash
$ git remote set-url <remote_name> <remote_url>

eg:
$ git remote -v
$ git remote set-url origin https://git-repo/new-repository.git
```

2. Changing Git Remote to SSH
``` bash
$ git remote set-url <remote_name> <ssh_remote_url>

eg:
$ git remote -v
$ git remote set-url origin git@github.com:user/repository.git
```


### 将本地的prod分支代码，盖掉远程的pre
``` bash
$ git checkout prod                              # 切换到本地的 prod 分支
$ git branch -d pre                              # 删掉本地的 pre 分支
$ git checkout -b pre                            # 以 prod 为基础，切出新的 pre 分支
$ git branch --set-upstream-to=origin/pre pre    # 将本地的 pre 分支，与 origin 建立联系
$ git status                                     # 查看状态
$ git push --force origin pre                    # 加上参数 --force 强推到 origin
```

注意：如果gitLab仓库上面，pre分支是被保护的，则要临时解除保护，否则就算加了 --force 也可能推送失败。


