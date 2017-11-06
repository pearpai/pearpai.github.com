---
layout:     post
title:      "SpringCloud 框架实战学习（3）--- 网关 负载均衡 熔断"
date:       UTC2017-11-02 15:38:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - SpringCloud
---
# 当前架构图
 ![添加网关架构图](/img/blog/springcloud/zuul-server-1.png)

此架构中添加了网关zuul、feign 断路器

## 使用Zuul 可以进行的操作
- 认证
- 洞察
- 压力测试
- 金丝雀测试
- 动态路由
- 服务前移
- 负载脱落
- 安全
- 静态响应处理
- 主动/主动流量管理

Zuul的规则引擎运行基本上写任何JVM语言编写规则和过滤器，内置Java和Groovy。

## 项目依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.ridge</groupId>
        <artifactId>spring-cloud-frame</artifactId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>ridge-getway-service</artifactId>
    <packaging>jar</packaging>

    <name>ridge-getway-service</name>
    <description>ridge-getway-service</description>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>

    </dependencies>

</project>
```
## zuul server 启动
使用@EnableZuulProxy注释Spring Boot主类，并将本地调用转发到相应的服务。

```java
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableDiscoveryClient
@EnableZuulProxy
public class RidgeGetWayServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeGetWayServiceApplication.class, args);
    }
}
```
## zuul server 配置
```ymal
management:
  security:
    enabled: false

eureka:
  client:
    service-url:
      defaultZone: http://wuyunfeng:wuyunfeng@server1:8761/eureka/,http://wuyunfeng:wuyunfeng@server1:8762/eureka/

server:
  port: 8084
spring:
  application:
    name: ridge-getway-service
  cloud:
    config:
      discovery:
        enabled: true
        service-id: ridge-config-service
  rabbitmq:
    host: localhost
    port: 5672

# 网关服务 配置
zuul:
  routes:
    api-a:
      path: /api-a/**
      serviceId: ribbon-service
    api-b:
      path: /api-b/**
      serviceId: ridge-client
    api-c:
      path: /api-c/**
      serviceId: ridge-client-b
  host:
    connect-timeout-millis: 10000
#  ignored-services: '*'

# 首次请求超时 设置
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 60000

ribbon:
  ConnectTimeout: 3000
  ReadTimeout: 60000
```

## zuul server 安全控制
```java
package com.ridge.filter;

import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.context.RequestContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.netflix.zuul.filters.support.FilterConstants;
import org.springframework.stereotype.Component;

import javax.servlet.http.HttpServletRequest;

/**
 * Summary: 此处进行安全过滤
 * Created by wuyunfeng on 26. 十月 2017 下午1:51.
 */
@Component
public class MyFilter extends ZuulFilter {

    private static Logger log = LoggerFactory.getLogger(MyFilter.class);

    /**
     * 过滤类型
     * String ERROR_TYPE = "error"; 处理请求发送错误时被调用
     * String POST_TYPE = "post"; 在 route 和 error 过滤器之后被调用
     * String PRE_TYPE = "pre"; 请求被如有之前调用
     * String ROUTE_TYPE = "route"; 在路由请求时被调用
     */
    @Override
    public String filterType() {
        return FilterConstants.PRE_TYPE;
    }

    // 过滤器排序即执行顺序
    @Override
    public int filterOrder() {
        return FilterConstants.SEND_ERROR_FILTER_ORDER;
    }

    // 是否使用此过滤器
    @Override
    public boolean shouldFilter() {
        // 官方demo
        // RequestContext ctx = RequestContext.getCurrentContext();
        // return !ctx.containsKey(FORWARD_TO_KEY) // a filter has already forwarded
        //        && !ctx.containsKey(SERVICE_ID_KEY); // a filter has already determined serviceId
        return true;
    }

    // 执行过滤方法
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s  >>>>>>>>>>> %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("token");
        //todo 此处获取 对token 进行判断
        if (accessToken == null) {
            log.warn("token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            try {
                ctx.getResponse().getWriter().write("token is empty");
            } catch (Exception ignore) {
            }
            return null;
        }
        return null;
    }
}
```
## Hystrix 断路
```java
package com.ridge.component;

import org.springframework.cloud.netflix.zuul.filters.route.ZuulFallbackProvider;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.stereotype.Component;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;

/**
 * Summary: 断路器
 * Created by wuyunfeng on 02. 十一月 2017 下午11:41.
 */
@Component
public class MyFallbackProvider implements ZuulFallbackProvider {
    /**
     * 此处的返回是service 不是直接路径 如果配置serviceId的 需要进行处理
     * @return serviceID
     */
    @Override
    public String getRoute() {
        // 匹配所有
        // return "*";
        return "ridge-client-b";
    }

    @Override
    public ClientHttpResponse fallbackResponse() {
        return new ClientHttpResponse() {
            @Override
            public HttpStatus getStatusCode() throws IOException {
                return HttpStatus.OK;
            }

            @Override
            public int getRawStatusCode() throws IOException {
                return 200;
            }

            @Override
            public String getStatusText() throws IOException {
                return "OK";
            }

            @Override
            public void close() {

            }

            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream("fallback".getBytes());
            }

            @Override
            public HttpHeaders getHeaders() {
                HttpHeaders headers = new HttpHeaders();
//                headers.setContentType(MediaType.APPLICATION_JSON);
                return headers;
            }
        };
    }
}
```
## 错误路径 返回
```java
package com.ridge.controller;

import org.springframework.boot.autoconfigure.web.ErrorController;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Summary: 错误路径处理方式
 * Created by wuyunfeng on 06. 十一月 2017 下午2:19.
 */
@RestController
public class MyErrorController implements ErrorController {

    private static final String ERROR_PATH = "/error";

    // TODO: 2017/11/6 网关处理返回格式 需要与个服务的处理一直 此处需要后期进行处理
    @RequestMapping("/error")
    public String error(){
        return "error path";
    }

    @Override
    public String getErrorPath() {
        return ERROR_PATH;
    }
}
```
## 后续feign
此节主要内容是对zull 网关的 学习，主要包括断路器、错误返回以及相关配置。下节是对feign的集成：service 层负载均衡以及熔断。
