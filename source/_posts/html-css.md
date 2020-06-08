---
title: html-css 前端相关
date: 2017-11-10 00:37:57
tags: html
categories: other
---

将图片二进制流显示在html端的img控件中，在获取验证码的时候用得比较多，方法如下
``` html
<img src="data:image/jpeg;base64,fejwiqpofewqlfjewqjfewopfjef...">

在 src 后面加上 data:image/jpeg;base64,二进制码.
```


### HTTPS网页调用HTTP服务
有时候，我们可能会在HTTPS的站点内通过Javascript脚本访问HTTP服务器，这时候，网页可能会报如下错误：
``` html
Mixed Content: The page at 'https://www.a.com' was loaded over HTTPS, but requested an insecure XMLHttpRequest endpoint 'http://api.b.com/data/user/save'. This request has been blocked; the content must be served over HTTPS.
```

这个时候，只要在我们的`http://api.b.com`站点的服务端开放一个测试接口`testApi`：
``` java
import com.b.entity.Result;
import org.apache.commons.lang3.StringUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.servlet.http.HttpServletResponse;

@RestController
@RequestMapping("data/user")
public class UserController {

    @PostMapping(value = "save")
    public Result save(String dataList, HttpServletResponse response) {
        allowOrigin(response);

        if (StringUtils.isBlank(dataList)) {
            return Result.getFalseResult("缺少必要的参数");
        }

        // save handler

        return Result.getSuccResult("保存成功");
    }

    /**
     * 仅用作浏览器在https协议下自动调用一次接口，让浏览器弹出不安全提示
     *
     * @param response
     * @return
     */
    @RequestMapping("testApi")
    public Result testApi(HttpServletResponse response) {
        allowOrigin(response);
        return Result.getSuccResult("ok");
    }

    public void allowOrigin(HttpServletResponse response) {
        response.setCharacterEncoding("UTF-8");
        response.setHeader("Cache-Control", "no-cache, no-store, must-revalidate");
        response.setHeader("Pragma", "no-cache");
        response.setHeader("Access-Control-Allow-Origin", "*");
    }
}
```

然后在前端代码中添加如下脚本：
``` javascript
// 仅用作浏览器在https协议下自动调用一次接口，让浏览器弹出不安全提示
	$.post("http://api.b.com/data/user/testApi", {}, function(){
		//
	});
```

确保在进入目标网页的时候会加载它，在加载它的时候在浏览器的地址栏中，我们点红色的叉会弹出一个对话框，有如下文字：
``` xml
This page is trying to load scripts from unauthenticated sources.

Load unsafe scripts
```

我们点击`Load unsafe scripts`之后，再提原先的请求就可以了。


### 如何刷新清除某个特定网站的缓存（Chrome浏览器）
1. 打开一个网站，如，百度；
2. 按F12，进入开发者模式；
3. 右键浏览器的刷新按钮，会出现三个选项：正常重新加载，硬性重新加载，清空缓存并硬性重新加载。

正常重新加载`Ctrl+R`：正常重新加载。
硬性重新加载`Ctrl+Shift+R`：浅层次的清除历史记录，但不一定完全清除缓存（Ctrl+F5同理）。
清空缓存并硬性重新加载：可以深层次的清除所有的缓存。（建议用这个）



