---
title: springboot 学习笔记
date: 2020-06-08 15:21:35
tags: springboot
categories: java
---

### springboot前后端分离，跨域问题解决
在java后端启动类中加上跨域处理即可，如下：
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CorsFilter corsFilter() {
        // 设置跨域
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("*");
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        source.registerCorsConfiguration("/**", corsConfiguration);
        return new CorsFilter(source);
    }
}
```

但是，如果前端发请求时，使用了`withCredentials`，如下：
``` javascript
$.ajax({
    type : "POST",
    contentType: "application/json;charset=UTF-8",
    url : "http://www.a.com/admin/user/save",
    async: true,
    xhrFields: {withCredentials: true},
    data : JSON.stringify(list),
    success : function(result) {
        console.log(result);
    },
    error : function(e) {
        console.log(e.status);
        console.log(e.responseText);
    }
});
```

并且前端域名为`http://www.b.com`，则后端需做如下修改（共修改了2行代码）：
``` java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    public CorsFilter corsFilter() {
        // 设置跨域
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration corsConfiguration = new CorsConfiguration();
        corsConfiguration.addAllowedOrigin("http://www.b.com");           // 这里要指定域名
        corsConfiguration.addAllowedHeader("*");
        corsConfiguration.addAllowedMethod("*");
        source.registerCorsConfiguration("/**", corsConfiguration);
        corsConfiguration.setAllowCredentials(true);                      // 增加这行
        return new CorsFilter(source);
    }
}
```

对于附带身份凭证的请求，服务器不得设置`Access-Control-Allow-Origin`的值为`*`。这是因为请求的首部中携带
了`Cookie`信息，如果`Access-Control-Allow-Origin`的值为`*`，请求将会失败。而将`Access-Control-Allow-Origin`
的值设置为`http://www.b.com`，则请求将成功执行。也就是说`Access-Control-Allow-Credentials`设置为true的
情况下`Access-Control-Allow-Origin`不能设置为`*`


### springboot启动时，指定加载的信息
``` bash
nohup java -Dloader.path=/home/root/userinfo/libs \
    -Dlogging.path=/home/root/logs/userinfo \
    -Dspring.config.location=/home/root/userinfo/config/application.yml \
    -jar /home/root/userinfo/userinfo-0.0.1-SNAPSHOT.jar \
    > /dev/null 2>&1 &
```

说明：
1. `loader.path`用于指定加载一些外部的JAR包，指定到目录；
2. `logging.path`指定日志文件存放的目录；
3. `spring.config.location`指定要加载的配置文件。



