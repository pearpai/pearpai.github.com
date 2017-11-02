---
layout:     post
title:      "SpringCloud 框架实战学习（2）--- 高可用配置中心、消息总线"
date:       UTC2017-10-27 15:38:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - SpringCloud
---
# 概要

 - 服务器外部配置提供了基于资源的HTTP。服务器使用@EnableConfigServer注解嵌入到SpringBoot程序中。
 - 高可用配置中心依赖于注册中心，通过配置中服务ID向注册中心获取配置中心集群地址集。
 - 配置信息存入git中。
 - Spring Cloud Bus 将分布式系统的节点与轻量级消息代理链接。用于广播状态的更改如配置的更改或其他指令。总线就像一个分布式执行器，用于扩展Spring Boo应用程序，但也可以作为应用程序直接通信通道。目前唯一的实现是使用AMQP代理作传输。

# 当前架构图
 ![enter image description here](/img/blog/springcloud/config-server-2.png)

# 配置服务

## 依赖包

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


    <artifactId>ridge-config-service</artifactId>
    <packaging>jar</packaging>

    <name>ridge-config-service</name>
    <description>ridge-config-service</description>

    <dependencies>
        <!-- 集成配置服务 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
    </dependencies>

</project>

```

## ymal 配置

```ymal

# 服务端口
server:
  port: 8888

# 配置git服务地址
spring:
  application:
    name: ridge-config-service
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/yunfengxc/spring-cloud-config.git
          username: ***
          password: ***
      label: master

# 向注册中心注册
eureka:
  client:
    service-url:
      defaultZone: http://wuyunfeng:wuyunfeng@server1:8761/eureka/,http://wuyunfeng:wuyunfeng@server1:8762/eureka/

```

## 启动main 方法
```java8
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
// 向注册中心注册
@EnableDiscoveryClient
// 配置服务
@EnableConfigServer
public class RidgeConfigServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeConfigServiceApplication.class, args);
    }
}
```

## git中Client service配置信息

```ymal
demo:
  value: http_bbbbxxx
---
server:
  port: 9082
spring:
  profiles: dev-2

---
server:
  port: 9081
spring:
  profiles: dev
```
- 共用的放在最上面。
- 通过"---"来分割不同的环境配置
- profiles：环境变量

配置服务比较容比较容易，主要提供了配置相关的功能。

# 消息总线
此处使用消息总线进行配置的刷新功能

## 总线依赖
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

    </dependencies>

</project>

```

## 总线配置

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

在此配置需要连接的注册中心， 通过注册中心获取配置中心当前地址，可以获取多配置中心地址，取得其中的一个地址来获取配置。以及消息总线mq的地址，此时为进行密码登录，需要添加 management:security:enabled: false 配置。

## 启动main 方法

```java8
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.context.config.annotation.RefreshScope;

@SpringBootApplication
@EnableDiscoveryClient
public class RidgeClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeClientApplication.class, args);
    }
}
```
这里的启动main 与一般的client 没有区别 只有搞一个注册功能

## 配置刷新demo

```java8
package com.ridge.controller;

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

    @RequestMapping("/hello")
    public String home(@RequestParam String name) {
        return "hello " + name + ",i am from port:" + port + " value: " + value;
    }

}
```
此处需要注意的是@RefreshScope 在需要信息配置刷新的地方都要加入此注解，如果没有此注解的，将不进行刷新。


## 刷新功能
刷新功能需要post 提交：http://localhost:9081/bus/refresh 刷新。<br/>
总线当前支持向所有节点发送消息，用于特定服务的所有节点（由Eureka定义）。未来可能会添加更多的选择器标准（即，仅数据中心Y中的服务X节点等）。/bus/*执行器命名空间下还有一些http端点。目前有两个实施。第一个/bus/env发送密钥/值对来更新每个节点的Spring环境。第二个，/bus/refresh，将重新加载每个应用程序的配置，就好像他们在他们的/refresh端点上都被ping过。

## 注册中心 注册信息图
![enter image description here](/img/blog/springcloud/config-server-1.png)
