---
title: 安装 Hexo
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

## 下面将详细说说安装过程：
### 1. 安装 Git 略

### 2. 安装 Node.js 略

### 3. 安装Hexo

在桌面空白处右键打开Git Bash Here，可以先测试一下Node.js是否安装成功，直接输入node可以看到提示符变成了一个向右
的箭头就表示成功了，然后按ctrl + c退出node模式，出现$符号才表示正常了

输入以下命令
``` bash
$ npm install -g hexo-cli
```
敲完回车可能没有任何提示，请一定要耐心等待
安装成功后，可以输入以下命令测试一下Hexo是否安装成功
``` bash
$ hexo version
```
如果能看到hexo的版本号信息，就表示安装成功了
接下来，进入到我们刚刚创建的文件夹，右键打开Git Bash Here

然后依次输入以下命令
``` bash
$ hexo init  # 注意此命令，要求当前仓库为空，可以先将仓库的文件移走，再执行此命令，然后再移回来
$ npm install
$ hexo g
$ hexo s
```
这时候在浏览器输入http://localhost:4000/ 就可以看到hexo已经成功生成了博客，当然这只是我们本地可以看到的


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

