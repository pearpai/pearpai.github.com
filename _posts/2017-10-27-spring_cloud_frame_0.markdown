---
layout:     post
title:      "SpringCloud 框架实战学习（0）--- 目录"
date:       UTC2017-10-27 11:37:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - SpringCloud
---
## 目录


## 项目注意事项
  这个学习过程是对springboot 及 springcloud 有了解的基础上进行学习。
  此过程也是我假设的过程，如果有什么不对的地方，欢迎提出指正。
  邮箱地址：wuyunfeng2017@gomail.com。
  各位也可以在文档评论区 进行评论，本人会尽量在第一时间进行回复。
## mvn 结构
- 本人将将项目置于一个父pom.xml文件中的 pom 父pom 文件 将随时更新，我会对新加入依赖等 进行相关说明。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.ridge</groupId>
    <artifactId>spring-cloud-frame</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>spring-cloud-frame</name>
    <description>spring cloud frame</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.8.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.SR4</spring-cloud.version>
    </properties>


    <modules>
        <!-- 发现服务器 -->
        <module>ridge-discovery-server</module>
        <!-- 发现服务器 2 -->
        <module>ridge-discovery-server2</module>

    </modules>


    <dependencies>
        <!-- 基础依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!-- eureka 发现服务的 客户端 依赖 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-eureka</artifactId>
        </dependency>

        <!-- config 配置客户端 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>

        <!-- lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```
