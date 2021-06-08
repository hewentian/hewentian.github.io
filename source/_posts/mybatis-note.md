---
title: mybatis 学习笔记
date: 2021-04-01 20:29:02
tags: mysql
categories: db
---

### mybatis的xml中的比较符号处理方法
第一种方法：
用转义字符把`>`和`<`替换掉，mapper文件示例代码：
``` xml
<if test="createDateStart != null ">
    AND create_date &gt;= #{createDateStart,jdbcType=DATE} 
</if>
<if test="createDateEnd != null ">
    AND create_date &lt;= #{createDateEnd,jdbcType=DATE} 
</if>
```

XML转义字符表

| 字符     | 符号 | 解释   |
| ----     | ---- | ----   |
| `&lt;`   |  <   | 小于号 |
| `&gt;`   |  >   | 大于号 |
| `&amp;`  |  &   | 和     |
| `&apos;` |  ’   | 单引号 |
| `&quot;` |  "   | 双引号 |


第二种方法：
xml格式中，不允许出现类似`>`这样的字符，但是可以使用`<![CDATA[ ]]>`符号进行说明，将此类符号不进行解析，mapper文件示例代码：
``` xml
<if test="createDateStart != null ">
    AND create_date <![CDATA[ >= ]]> #{createDateStart,jdbcType=DATE} 
</if>
<if test="createDateEnd != null ">
    AND create_date <![CDATA[ <= ]]> #{createDateEnd,jdbcType=DATE} 
</if>
```


