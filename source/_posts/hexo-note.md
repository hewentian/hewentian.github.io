---
title: hexo 学习笔记
date: 2017-12-02 13:59:48
categories: other
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## 下面将详细说说安装过程，基于 Ubuntu 16.04 LTS：
### 1. 安装 Git
在 Ubuntu 下面安裝非常方便，可以先在命令行中輸入`git`，如果没显示命令，则输入如下命令安装即可：
``` bash
$ sudo apt-get install git
```

### 2. 安装 Node.js
先测试一下Node.js是否已安装，在命令行中直接输入`node`可以看到提示符变成了一个向右
的箭头就表示成功了，然后按ctrl + c退出node模式，出现$符号才表示正常了

如果未安裝 node，安裝方法如下：
``` bash
$ sudo apt-get install nodejs-legacy
```

### 3. 安装 Hexo
进入到我们博客的仓库, 输入以下命令
``` bash
$ sudo npm install -g hexo-cli
或者
$ sudo npm install hexo --save  # 或者去掉 sudo
```
敲完回车可能没有任何提示，请一定要耐心等待
安装成功后，可以输入以下命令测试一下Hexo是否安装成功
``` bash
$ hexo version
```
如果能看到hexo的版本号信息，就表示安装成功了
接下来，进入到我们博客的仓库

然后依次输入以下命令
``` bash
$ hexo init  # 注意此命令，要求当前仓库为空，可以先将仓库的文件移走，再执行此命令，然后再将文件移回来。
$ # 可以先在当前目录的父目录创建临时目录hgi
$ mkdir ../hgi
$ mv * ../hgi # 这个命令不会将 .git .gitignore这两个文件夹移走，所以，还要执行如下命令
$ mv .git .gitignore ../hgi/
$ npm install
$ hexo g
$ hexo s
```
这时候在浏览器输入http://localhost:4000/ 就可以看到hexo已经成功生成了博客，当然这只是我们本地可以看到的

最后将博客的仓库目录下除了`node_modules`之外的文件都删掉，然后将 ../hgi 中的除了`node_modules`之外的文件都复制回来(不要忘记.git .gitignore)，大功告成。
重启hexo

配置Hexo到Github

找到我们刚刚创建的文件夹，在里面找到_config.yml文件，用notepad++打开，直接拖到最后，添加如下：

    deploy:
      type: git
      repository: git@github.com:hewentian/hewentian.github.io.git
      branch: master

发布到 github
``` bash
$ hexo clean
$ hexo g
$ hexo d
```
如果出现以下异常

    ERROR Deployer not found: git

尝试输入以下命令，然后重新执行刚刚的两条命令
``` bash
$ npm install hexo-deployer-git --save
```
这时候我们就可以在浏览器输入 https://hewentian.github.io 就可以看到博客已经搭建成功了。
  

使用以下命令验证是否安装成功
``` bash
$ node -v
$ npm -v
$ hexo -v
```
解释一下：
node_modules：是依赖包
public：存放的是生成的页面
scaffolds：命令生成文章等的模板
source：用命令创建的各种文章
themes：主题
_config.yml：整个博客的配置
db.json：source解析所得到的
package.json：项目所需模块项目的配置信息


添加博文
``` bash
hexo new "postName"  #新建博文,其中postName是博文题目
```
博文会自动生成在博客目录下source/_posts/postName.md


注意事项

所有键的冒号后面留一个空格，如 `language: zh-CN`


一些常用命令：
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录, hexo g
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）, hexo s
hexo deploy #将.deploy目录部署到GitHub, hexo d
hexo help # 查看帮助
hexo version #查看Hexo的版本


### ENOSPC Error (Linux)报错的处理
如果启动的时候报如下错误：
``` bash
$ hexo s
INFO  Start processing
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: watch /home/hewentian/ProjectD/gitHub/hewentian.github.io/themes/landscape/ ENOSPC
    at _errnoException (util.js:1022:11)
    at FSWatcher.start (fs.js:1382:19)
    at Object.fs.watch (fs.js:1408:11)
    at createFsWatchInstance (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:37:15)
    at setFsWatchListener (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:80:15)
    at FSWatcher.NodeFsHandler._watchWithNodeFs (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:228:14)
    at FSWatcher.NodeFsHandler._handleDir (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:407:19)
    at FSWatcher.<anonymous> (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:455:19)
    at FSWatcher.<anonymous> (/home/hewentian/ProjectD/gitHub/hewentian.github.io/node_modules/chokidar/lib/nodefs-handler.js:460:16)
    at FSReqWrap.oncomplete (fs.js:153:5)
```

可以尝试通过如下方式解决：
``` bash
$ npm dedupe
$ echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
```

