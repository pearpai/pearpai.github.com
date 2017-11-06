---
layout:     post
title:      "SpringCloud 框架实战学习（4）--- Feign"
date:       UTC2017-11-06 15:38:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - SpringCloud
---
接着上一篇的内容 feign
## 当前架构图
 ![添加网关架构图](/img/blog/springcloud/zuul-server-1.png)
- feign 提供了 fallback 负载均衡
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

    <artifactId>ridge-client-b</artifactId>
    <packaging>jar</packaging>

    <name>ridge-client-b</name>
    <description>ridge-client-b</description>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>com.ridge</groupId>
            <artifactId>ridge-core</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```
## 配置
```ymal
eureka:
  client:
    service-url:
      defaultZone: http://wuyunfeng:wuyunfeng@server1:8761/eureka/,http://wuyunfeng:wuyunfeng@server1:8762/eureka/

spring:
  application:
    name: ridge-client-b
  cloud:
    config:
      discovery:
        enabled: true
        service-id: ridge-config-service
  rabbitmq:
    host: localhost
    port: 5672

management:
  security:
    enabled: false

feign:
  hystrix:
    enabled: true

# 首次请求超时 设置
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 60000

---
spring:
  profiles: dev

---
spring:
  profiles: dev-2
```

## 启动 @EnableFeignClients
```java
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.feign.EnableFeignClients;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
public class RidgeClientBApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeClientBApplication.class, args);
    }
}
```

## feign 用于service层
### 接口层
```java
package com.ridge.feign;

import com.ridge.dto.request.DemoDto;
import org.springframework.cloud.netflix.feign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

/**
 * Summary: 调用 serviceId：ridge-client 接口
 * Created by wuyunfeng on 26. 十月 2017 上午10:46.
 */
@FeignClient(value = "ridge-client", fallback = SchedualServiceHiHystric.class)
public interface SchedualServiceHi {

    @GetMapping(value = "/hi")
    String sayHiFromClientOne(@RequestParam(value = "name") String name);

    @GetMapping(value = "/demo")
    DemoDto demoDto(@RequestParam(value = "name") String name);
}
```
### fallback
```java
package com.ridge.feign;

import com.ridge.dto.request.DemoDto;
import org.springframework.stereotype.Component;

/**
 * Summary: 熔断 fallback 对错误返回 进行处理
 * Created by wuyunfeng on 26. 十月 2017 上午11:09.
 */
@Component
public class SchedualServiceHiHystric implements SchedualServiceHi {
    @Override
    public String sayHiFromClientOne(String name) {
        return "sorry " + name;
    }

    @Override
    public DemoDto demoDto(String name) {
        return null;
    }
}
```

# ridge-client 服务
## 服务依赖

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

    <artifactId>ridge-client</artifactId>
    <packaging>jar</packaging>

    <name>ridge-client</name>
    <description>ridge-client</description>

    <dependencies>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-bus-amqp</artifactId>
        </dependency>

        <dependency>
            <groupId>com.ridge</groupId>
            <artifactId>ridge-core</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>

    </dependencies>

</project>
```
## 服务配置
```ymal
eureka:
  client:
    service-url:
      defaultZone: http://wuyunfeng:wuyunfeng@server1:8761/eureka/,http://wuyunfeng:wuyunfeng@server1:8762/eureka/

spring:
  application:
    name: ridge-client
  cloud:
    config:
      discovery:
        enabled: true
        service-id: ridge-config-service
  rabbitmq:
    host: localhost
    port: 5672

management:
  security:
    enabled: false

---
spring:
  profiles: dev

---
spring:
  profiles: dev-2

```
## 服务启动
```java
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@SpringBootApplication
@EnableDiscoveryClient
@RefreshScope
public class RidgeClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeClientApplication.class, args);
    }
}
```

## controller

```java
package com.ridge.controller;

import com.ridge.dto.request.DemoDto;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * Summary: 提供一些 简单的demo restful
 * Created by wuyunfeng on 26. 十月 2017 上午10:14.
 */
@RestController
@RefreshScope
public class DemoController {

    @Value("${server.port}")
    String port;

    @Value("${demo.value}")
    String value;

    @RequestMapping("/hi")
    public String home(@RequestParam String name) {
        return "hi " + name + ",i am from port:" + port + " value: " + value;
    }

    @RequestMapping("/demo")
    public DemoDto demo(@RequestParam String name) {
        DemoDto demoDto = new DemoDto();
        demoDto.setName(name);
        demoDto.setNumber(1);
        return demoDto;
    }
}
```

## 请求方式
- localhost:8084/api-c/demo?name=3&token=1
- 返回数据：{"number": 2,  "name": "3"}
