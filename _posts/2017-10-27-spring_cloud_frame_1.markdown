---
layout:     post
title:      "SpringCloud 框架实战学习（1）--- 高可用服务注册中心"
date:       UTC2017-10-27 13:43:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - SpringCloud
---
使用Eureka进行服务注册与发现，将Eureka Server 集群化，避免单点故障。

## mvn 依赖 spring-cloud-starter-eureka-server 注册发现服务
 **ridge-discover-server 如下：**
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

    <artifactId>ridge-discover-server</artifactId>
    <packaging>jar</packaging>

    <name>ridge-discover-server</name>
    <description>ridge-discover-server</description>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka-server</artifactId>
        </dependency>
    </dependencies>
</project>
```
ridge-discover-server2 的 依赖一致 修改 相关artifactId 等。

## yaml 配置
**ridge-discover-server配置bootstrap.yml：**
```yaml
spring:
  application:
    name: ridge-discovery-server # 服务名称

server:
  port: 8761 #服务监听端口

eureka:
  instance:
    hostname: server1 # 本服务的hostname
    ot-hostname: server2 # ridge-discovery-server 将要注册到的 hostname 即另一个eureka服务
    ot-port: 8762 # 一个eureka服务端口
  client:
    service-url:
      # ridge-discovery-server 注册地址 以及 登录账号
      defaultZone: http://${security.user.name}:${security.user.password}@${eureka.instance.ot-hostname}:${eureka.instance.ot-port}/eureka/

#  安全控制
security:
  basic:
    enabled: true
  user:
    name: wuyunfeng
    password: wuyunfeng
```
**ridge-discover-server配置bootstrap.yml：**
```yaml
spring:
  application:
    name: ridge-discovery-server2

server:
  port: 8762

eureka:
  instance:
    hostname: server2
    ot-hostname: server1
    ot-port: 8761
  client:
    service-url:
      defaultZone: http://${security.user.name}:${security.user.password}@${eureka.instance.ot-hostname}:${eureka.instance.ot-port}/eureka/

security:
  basic:
    enabled: true
  user:
    name: wuyunfeng
    password: wuyunfeng
```
## 启动方法
```java
package com.ridge;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class RidgeDiscoveryServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(RidgeDiscoveryServerApplication.class, args);
    }
}
```
## 启动后界面
![enter image description here](/img/blog/springcloud/discover-server-1.jpg)
- server2 的展示信息 与此 相反

## client

mvn使用父依赖，自依赖没有

```yaml
server:
  port: 8081
spring:
  application:
    name: ridge-client

# defaultZone 可以填写多个注册中心地址，以逗号分隔 这样可以避免单点注册出错
eureka:
  client:
    service-url:
      defaultZone: http://wuyunfeng:wuyunfeng@server1:8761/eureka/,http://wuyunfeng:wuyunfeng@server1:8762/eureka/
```

## 客户端注册后
![enter image description here](/img/blog/springcloud/discover-server-2.jpg)

## 注册中心架构
![enter image description here](/img/blog/springcloud/eureka-uml.jpg)
