---
title: html-css前端相关
date: 2017-11-10 00:37:57
categories: other
---
将图片二进制流显示在html端的img控件中，在获取验证码的时候用得比较多，方法如下
``` html
<img src="data:image/jpeg;base64,fejwiqpofewqlfjewqjfewopfjef...">

在 src 后面加上 data:image/jpeg;base64,二进制码.
```