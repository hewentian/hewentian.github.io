---
title: spring 学习笔记
date: 2021-09-25 20:09:40
tags: spring
categories: java
---


### 在filter中无法使用Autowired的问题
解决方案：
https://stackoverflow.com/questions/32494398/unable-to-autowire-the-service-inside-my-authentication-filter-in-spring/32495757

代码片段如下：
``` java
import com.hewentian.service.ISiteConfigService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.WebApplicationContextUtils;
import org.springframework.web.filter.GenericFilterBean;

import javax.servlet.*;
import java.io.IOException;

@Slf4j
public class VerifySignFilter extends GenericFilterBean {
    private String secretSalt;

    @Autowired
    private ISiteConfigService siteConfigService;

    @Override
    protected void initFilterBean() throws ServletException {
        super.initFilterBean();

        FilterConfig filterConfig = getFilterConfig();
        if (null != filterConfig) {
            secretSalt = filterConfig.getInitParameter("secretSalt");
        }

        if (null == siteConfigService) {
            WebApplicationContext webApplicationContext = WebApplicationContextUtils.getWebApplicationContext(getServletContext());
            siteConfigService = webApplicationContext.getBean(ISiteConfigService.class);
        }
    }
}
```

其中，web.xml的配置片段如下：
``` xml
    <filter>
        <filter-name>verifySignFilter</filter-name>
        <filter-class>com.hewentian.study.filter.VerifySignFilter</filter-class>
        <init-param>
            <param-name>secretSalt</param-name>
            <param-value>MH39MmRxziHOC43t</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>verifySignFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
```


### 在filter中处理异常
解决方案：
https://stackoverflow.com/questions/34595605/how-to-manage-exceptions-thrown-in-filters-in-spring

当在filter中抛出异常时，得向前端返回错误信息

代码片段如下：
``` java
import com.fasterxml.jackson.databind.ObjectMapper;

import javax.servlet.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.PrintWriter;

public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {

        try {
            // code that throws exception
        } catch (Exception e) {
            // set the response object
            HttpServletResponse response = (HttpServletResponse) servletResponse;
            response.setContentType("application/json;charset=UTF-8");

            Result result = new Result(); // self-defined result object
            result.setCode(40001);
            result.setMsg(e.getMessage());

            // pass down the actual obj that exception handler normally send
            ObjectMapper mapper = new ObjectMapper();
            PrintWriter out = response.getWriter();
            out.print(mapper.writeValueAsString(result));
            out.flush();

            return;
        }

        // proceed normally otherwise
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
    }
}
```


### 如何多次读输入流InputStream
解决方案：
https://coderanch.com/t/364591/java/read-request-body-filter

代码片段如下：
对 ServletRequest 进行一层包装的类
``` java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.IOUtils;

import javax.servlet.ServletException;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.*;

@Slf4j
public class VerifySignRequestWrapper extends HttpServletRequestWrapper {
    private byte[] payload;

    public VerifySignRequestWrapper(HttpServletRequest request) throws Exception {
        super(request);

        // read the original payload into the payload variable
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();

        try {
            // read the payload into the ByteArrayOutputStream
            InputStream inputStream = request.getInputStream();
            IOUtils.copy(inputStream, byteArrayOutputStream);

            payload = byteArrayOutputStream.toByteArray();
        } catch (Exception ex) {
            log.error("Error reading the request payload", ex);
            throw new ServletException("Error reading the request payload", ex);
        } finally {
            if (byteArrayOutputStream != null) {
                try {
                    byteArrayOutputStream.close();
                } catch (IOException iox) {
                    // ignore
                }
            }
        }
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(payload);

        ServletInputStream inputStream = new ServletInputStream() {
            public int read() throws IOException {
                return byteArrayInputStream.read();
            }
        };

        return inputStream;
    }
}
```

过虑器Filter
``` java
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.io.IOUtils;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import java.io.IOException;
import java.nio.charset.StandardCharsets;

@Slf4j
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)
            throws IOException, ServletException {

        // perform request filtering
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;

        // 先对 ServletRequest 进行一层包装
        VerifySignRequestWrapper verifySignRequestWrapper;

        try {
            verifySignRequestWrapper = new VerifySignRequestWrapper(httpServletRequest);
        } catch (Exception e) {
            log.warn("Unable to wrap the request", e);
            throw new ServletException("Unable to wrap the request", e);
        }

        // 将流读出来，用于验签等操作
        String requestBody = IOUtils.toString(verifySignRequestWrapper.getInputStream(), StandardCharsets.UTF_8);

        // 省略相关操作...

        // 这样，后面的filter，还可以从 request 中读到流
        filterChain.doFilter(verifySignRequestWrapper, servletResponse);
    }

    @Override
    public void destroy() {
    }
}
```



