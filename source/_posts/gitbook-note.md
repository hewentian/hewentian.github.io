---
title: gitbook学习笔记
date: 2018-01-29 13:59:48
tags:
categories: other
---
下面说下如何安装gitbook

### 1. 安装 Node.js
先测试一下Node.js是否已安装，在命令行中直接输入`node`可以看到提示符变成了一个向右
的箭头就表示成功了，然后按ctrl + c退出node模式，出现$符号才表示正常了

如果未安裝 node，安裝方法如下：
``` bash
$ sudo apt-get install nodejs-legacy
```

剩下的安装过程参考[https://yq.aliyun.com/articles/7506](https://yq.aliyun.com/articles/7506)，但是会有一些不同，如下所示：

### 2. 安装gitbook
``` bash
$ sudo npm install -g gitbook-cli
```

### 3. 初始化
在你的文档目录下新建文件 SUMMARY.md，这个文件就是这本书的目录啦：
``` bash
$ cd docs
$ touch SUMMARY.md
```
SUMMARY.md 的格式规范如下：

	# uitest 文档

	- [uitest 是什么](users/index.md)
    	- [如何使用 uitest](users/use.md)
    	- [如何编写自定义的测试用例](users/case.md)
    	- [browserjs API 文档](users/api.md)
	- [uitest 开发者文档](devs/index.md)
    	- [browserjs 开发者文档](devs/browserjs.md)
    	- [utci 文档](devs/utci.md)
    	- [utserver & utclient 文档](devs/utserver.md)
	- [相关文章沉淀](artical.md)
	- [关于 gitbook](gitbook.md)

然后执行`gitbook init`初始化，gitbook 会根据 SUMMARY 的结构生成对应的目录文件：

	├── README.md           // 首页
	├── SUMMARY.md          // 目录
	└── users               // 用户文档
    	└── index.md        // 是什么
    	├── use.md          // 如何使用
    	├── api.md          // browserjs API
    	├── case.md         // 如何写测试用例
	├── devs                // 开发者文档目录
		│   ├── index.md        // 开发者文文档首页
		│   ├── browserjs.md    // browserjs 开发文档
		│   ├── utci.md         // utci 开发文档
		│   └── utserver.md     // utserver 和 utclien 开发文档
	├── artical.md          // 文章沉淀
	├── gitbook.md          // gitbook 相关

### 4. 本地调试
在对应的文档目录下运行`gitbook serve`会启动一个本地的静态服务器：
``` bash
$ cd docs
$ gitbook serve

Live reload server started on port: 35729
Press CTRL+C to quit ...

info: 7 plugins are installed 
info: loading plugin "livereload"... OK 
info: loading plugin "highlight"... OK 
info: loading plugin "search"... OK 
info: loading plugin "lunr"... OK 
info: loading plugin "sharing"... OK 
info: loading plugin "fontsettings"... OK 
info: loading plugin "theme-default"... OK 
info: found 27 pages 
info: found 2 asset files 

```
访问 http://localhost:4000/ 就可以实时的预览啦，并且支持`livereload`, 灰常赞~接下来结合预览的功能编辑对应的文档，完成之后就可以发布啦。

### 5. 发布
在文档目录下执行`gitbook build`会生成一个`_book`的目录，这个目录就是我们的静态网站啦，然后通过 demo 平台或者 github pages 就可以很简单的完成部署了。
