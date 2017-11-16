---
title: java一些常用笔记
date: 2017-11-16 19:58:45
tags: java
categories: java
---
### 想在`eclipse`中新建类的时候，自动在类的头部插入作者、时间等信息
方法是`Window -> Preferences -> Java -> Code Style -> Code Templates`在右则民展开`Code -> New Java files`点`Edit...`并加入如下代码
``` java
${filecomment}
${package_declaration}
/**
 * 
 * <p>
 * <b>${file_name}</b> 是 
 * </p>
 *
 * @author <a href="mailto:wentian.he@qq.com">hewentian</a>
 * @date ${currentDate:date('yyyy-MM-dd')} ${time}
 * @since JDK 1.8
 *
 */
${typecomment}
${type_declaration}
```

