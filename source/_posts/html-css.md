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


### 网页请求不带cookie问题
为什么ajax请求没有带上cookie呢？然后去查了一下资料，原来ajax默认只会带上同源的cookie。如果只是这样还没什么，
重点是localhost和本机的Ip地址（我的是192.168.1.112）不是一个域，因为我在浏览器输入网址打的是`localhost:8761`
（也就是说当前域是localhost:8761），但为ajax的请求地址是写`192.168.1.112:8761`，所以ajax请求就没带上`localhost:8761`
下的cookie。补充一下：当然不仅ajax，比如直接在浏览器上输入地址或者通过表单提交都是默认只带上同源的cookie。


### js发送post请求下载文件
JQuery的ajax函数的返回类型只有xml、text、json、html等类型，没有“流”类型，所以我们要实现ajax下载，
不能够使用相应的ajax函数进行文件下载。但可以用js生成一个form，用这个form提交参数，并返回“流”类型
的数据。在实现过程中，页面也没有进行刷新。
``` javascript
function toExport() {
    let formData = getFormData();

    if (formData) {
        var $iframe = $('<iframe id="down-file-iframe" />');
        var $form = $('<form target="down-file-iframe" method="post" />');
        $form.attr('action', 'http://study.hewentian.com/userInfo/download');

        for (var key in formData) {
            $form.append('<input type="hidden" name="' + key + '" value="' + formData[key] + '" />');
        }

        $iframe.append($form);
        $(document.body).append($iframe);
        $form[0].submit();
        $iframe.remove();
    }
}

function getFormData() {
    let userName = $('#userName').val();

    let startTime = $('#startTime').val();
    let endTime = $('#endTime').val();

    if (isNull(startTime) || isNull(endTime)) {
        layer.alert("请选择日期范围");
        return;
    }

    return {
        'userName': userName,
        'startTime': startTime,
        'endTime': endTime
    };
}
```

``` java
@RestController
@RequestMapping("/web/userInfo")
@Slf4j
public class UserInfoController {
    @Autowired
    private IUserInfoService userInfoService;

    @PostMapping("/download")
    public void download(String userName, String startTime, String endTime, HttpServletResponse response) {
        XSSFWorkbook xssfWorkbook = userInfoService.getXSSFWorkbook(userName, startTime, endTime);

        try {
            String filename = "用户信息-" + DateFormatUtils.format(new Date(), "yyyy-MM-dd") + ".xlsx";
            response.setContentType("application/vnd.ms-excel;charset=UTF-8");
            response.addHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(filename, "UTF-8"));

            response.setCharacterEncoding("UTF-8");
            OutputStream outputStream = response.getOutputStream();

            xssfWorkbook.write(outputStream);

            outputStream.flush();
            outputStream.close();
        } catch (Exception e) {
            log.error(e.getMessage(), e);
        }
    }
}
```


### JAVA后台接收前端参数
如果JAVA后台想以`@RequestBody`方式，用实体类来接收参数，那么前端就不能以form方式提交表单。
        @PostMapping("/search")
        public ResponseEntity search(@RequestBody UserDTO dto) {

此时前端必须要使用`contentType: 'application/json'`指定参数类型。
``` javascript
$.ajax({
    url: url,
    type: method,
    async: async,
    xhrFields: {withCredentials: true},
    contentType: 'application/json',
    dataType: 'json',
    data: data ? method === 'GET' ? data : JSON.stringify(data) : null,
    success: function (data) {
        let code = data.code;
        if (code === 4003){
            layer.confirm(data.message, function (index) {
                $.removeCookie('username');
                top.location.href = LOGIN_URI;
            });
        } else {
            layer.msg(data.message);
        }
    },
    error: function () {
        layer.msg("客户端繁忙...");
    }
});
```


### javascript中的字符串替换函数replace
字符串 stringObject 的 replace() 方法执行的是查找并替换的操作。它将在 stringObject 中查找与 regexp 相
匹配的子字符串，然后用 replacement 来替换这些子串。如果 regexp 具有全局标志 g，那么 replace() 方法将替
换所有匹配的子串。否则，它只替换第一个匹配子串。

``` javascript
var str = "abcdbe";
var str1 = str.replace('b','B');  // aBcdbe
var str2 = str.replace(/b/g,'B'); // aBcdBe
```


### javascript中字符串与数组的互转
``` javascript
var str = "a,b,c,d";
var strArray = str.split(","); // 字符串转数组

var str1 = strArray.join(","); // 数组转字符串，用指定的分隔符分隔
var str2 = String(strArray);   // 数组转字符串，默认的分隔符为逗号
```


### javascript函数的参数中有逗号的情况
当往函数中传递的参数中包含有逗号的时候，要用单引号将其括起来。

``` javascript
var username = "scott, tiger";
var age = 20;

printUserinfo("'" + username + "'", age);

function printUserinfo(username, age) {
    console.log(username + ", " + age)
}
```


### javascript将字符串转为JSON对象
`JSON.parse()`方法用于将一个`JSON`字符串转换为对象。
        var str = '[{"name":"scott","age":20},{"name":"tiger","age":21}]'; // str是一个字符串

        var jsonArray = JSON.parse(str); // 转换为json对象

        for(var i = 0; i < jsonArray.length; i++) {
            var jsonObj = jsonArray[i];
            console.log(jsonObj.name + ", " + jsonObj.age); // 取json中的值
        }


### javascript将JSON对象转为字符串
`JSON.stringify()`方法用于将`javaScript`值转换为`JSON`字符串，一般用于将服务器端返回的对象转成JSON字符串。
``` javascript
var objArray = [{"name":"scott","age":20},{"name":"tiger","age":21}]; // objArray是一个数组对象
var jsonStr = JSON.stringify(objArray);
console.log(jsonStr);

var jsonStr2 = JSON.stringify(objArray, null, 4);
console.log(jsonStr2);
```

输出结果分别如下，第二个会使用4个空格格式化输出：
``` javascript
[{"name":"scott","age":20},{"name":"tiger","age":21}]

[
    {
        "name": "scott",
        "age": 20
    },
    {
        "name": "tiger",
        "age": 21
    }
]
```


