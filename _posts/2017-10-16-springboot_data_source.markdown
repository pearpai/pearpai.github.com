---
layout:     post
title:      "springboot mybatis 多数据源 事务 连接池"
date:       UTC2017-10-16 13:06:00
author:     "Pearpai"
header-img: "img/starry_sky.jpeg"
catalog: true
tags:
    - springboot
    - mybatis
    - 数据库
---
[mybatis 文档](http://www.mybatis.org/mybatis-3/zh/getting-started.html)

## 功能说明
- 多数据源切换
- 事务的处理
- sql语句的存放方式
- 阿里的druid

## 依赖包
``` xml
 <!--spring 切面-->
 <dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-aop</artifactId>
 </dependency>

 <!--springboot mybatis-->
 <dependency>
     <groupId>org.mybatis.spring.boot</groupId>
     <artifactId>mybatis-spring-boot-starter</artifactId>
     <version>1.3.1</version>
 </dependency>
  <!--数据库连接驱动-->
 <dependency>
     <groupId>mysql</groupId>
     <artifactId>mysql-connector-java</artifactId>
     <version>5.1.30</version>
 </dependency>

 <!--连接池-->
 <dependency>
     <groupId>com.alibaba</groupId>
     <artifactId>druid-spring-boot-starter</artifactId>
     <version>1.1.0</version>
 </dependency>
```

## 数据库配置
数据库连接地址需要自己配置
```yaml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl

spring:
  datasource:
    master:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://*****:3306/codeNut?autoReconnect=true&characterEncoding=utf8&useUnicode=true&zeroDateTimeBehavior=convertToNull&noAccessToProcedureBodies=true
      username: ***
      password: ***
      type: com.alibaba.druid.pool.DruidDataSource
      initialSize: 10
      maxActive: 100
      maxWait: 60000
      minIdle: 5
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 'x'
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      maxPoolPreparedStatementPerConnectionSize: 20
    slave:
      driver-class-name: com.mysql.jdbc.Driver
      url: jdbc:mysql://***:3306/codeNutTest?autoReconnect=true&characterEncoding=utf8&useUnicode=true&zeroDateTimeBehavior=convertToNull&noAccessToProcedureBodies=true
      username: ***
      password: ***
      type: com.alibaba.druid.pool.DruidDataSource
      initialSize: 10
      maxActive: 100
      maxWait: 60000
      minIdle: 5
      timeBetweenEvictionRunsMillis: 60000
      minEvictableIdleTimeMillis: 300000
      validationQuery: SELECT 'x'
      testWhileIdle: true
      testOnBorrow: false
      testOnReturn: false
      poolPreparedStatements: true
      maxPoolPreparedStatementPerConnectionSize: 20

logging:
  level:
    com: info
    mybatis: debug
  file: ./logs/core.log
```

## 整合多数据源
```java

package com.ridge.config;

import com.ridge.utils.Constant;
import org.springframework.boot.autoconfigure.jdbc.DataSourceBuilder;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;
import java.util.HashMap;
import java.util.Map;

/**
 * Summary: 数据源连接配置
 * Created by wuyunfeng on 10. 十月 2017 下午10:17.
 */
@Configuration
public class DataSourceConfig {

    @Bean(name = Constant.DS_MASTER)
    @ConfigurationProperties(prefix = "spring.datasource.master")
    public DataSource masterSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean(name = Constant.DS_SALVE)
    @ConfigurationProperties(prefix = "spring.datasource.slave")
    public DataSource slaveSource() {
        return DataSourceBuilder.create().build();
    }

	/**
     * 动态数据源集合创建
     * @return DataSource
     */
    @Bean(name = "dynamicDS")
    public DataSource dataSource() {
        DynamicDataSource dynamicDataSource = new DynamicDataSource();
        // 默认数据源
        dynamicDataSource.setDefaultTargetDataSource(masterSource());

        // 配置多数据源
        Map<Object, Object> dsMap = new HashMap<>();

        dsMap.put(Constant.DS_MASTER, masterSource());
        dsMap.put(Constant.DS_SALVE, slaveSource());

        dynamicDataSource.setTargetDataSources(dsMap);

        return dynamicDataSource;
    }
}
```

## 设置当前数据源名称
```java
package com.ridge.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Summary: 设置当前数据源名称
 * Created by wuyunfeng on 09. 十月 2017 下午3:36.
 */
public class DataSourceContextHolder {

    public static final Logger log = LoggerFactory.getLogger(DataSourceContextHolder.class);

    public static final String DEFAULT_DS = "master";

    private static final ThreadLocal<String> contextHolder = new ThreadLocal<>();

    // 设置数据源名
    public static void setDB(String dbType) {
        log.debug("切换到{}数据源", dbType);
        contextHolder.set(dbType);
    }

    // 获取数据源名
    public static String getDB() {
        return contextHolder.get();
    }

    // 清除数据源名
    public static void clearDB() {
        contextHolder.remove();
    }


}
```

## javax.sql.DataSource 接口实现
```java
package com.ridge.config;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.jdbc.datasource.lookup.AbstractRoutingDataSource;

/**
 * Summary: 获取当前返回的数据源基础信息实现
 * Created by wuyunfeng on 09. 十月 2017 下午3:45.
 */
public class DynamicDataSource extends AbstractRoutingDataSource {

    private static final Logger log = LoggerFactory.getLogger(DynamicDataSource.class);

    @Override
    protected Object determineCurrentLookupKey() {
        return DataSourceContextHolder.getDB();
    }
}
```

## 自定义注解
```java
package com.ridge.annotation;

import com.ridge.utils.Constant;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * Summary: 自定义 mybatis 数据源注解
 * Created by wuyunfeng on 09. 十月 2017 下午3:58.
 */
@Retention(RetentionPolicy.RUNTIME)
@Target({
        ElementType.METHOD
})
public @interface DS {

    String value() default Constant.DS_MASTER;

}
```

## 切面实现
``` java
package com.ridge.aspect;


import com.ridge.annotation.DS;
import com.ridge.config.DataSourceContextHolder;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

/**
 * Summary: mybatis 数据源数据源切换 切面实现
 * Created by wuyunfeng on 09. 十月 2017 下午4:00.
 */
@Aspect
@Component
@Order(-1)
public class DynamicDataSourceAspect {

    @Before(value = "@annotation(ds)", argNames = "point, ds")
    public void beforeSwitchDS(JoinPoint point, DS ds) {


        String methodName = point.getSignature().getName();
        //得到方法的参数的类型
        String dataSource = DataSourceContextHolder.DEFAULT_DS;
        try {
            // 取出注解中的数据源名
            dataSource = ds.value();

        } catch (Exception e) {
            e.printStackTrace();
        }
        // 切换数据源
        DataSourceContextHolder.setDB(dataSource);
    }


    @Before(value = "@annotation(com.ridge.annotation.DS)")
    public void afterSwitchDS(JoinPoint point) {

        DataSourceContextHolder.clearDB();

    }
}
```

## 具体实现
``` java
package com.ridge.mapper;

import com.ridge.entity.UserInfo;
import org.apache.ibatis.annotations.Select;

import java.util.List;
import java.util.Map;

/**
 * Summary:
 * Created by wuyunfeng on 09. 十月 2017 下午1:13.
 */
public interface UserMapper {


    @Select("SELECT * FROM userInfo")
    List<UserInfo> getAll();

    List<UserInfo> selectByPrimaryKey(Map<String, Object> map);

    int update(Map<String, Object> map);


}
```

## 事务处理说明
```java
package com.ridge.service;

import com.ridge.annotation.DS;
import com.ridge.entity.UserInfo;
import com.ridge.mapper.UserMapper;
import com.ridge.utils.Constant;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * Summary:
 * Created by wuyunfeng on 09. 十月 2017 下午1:18.
 */
@Service
public class UserInfoService {

    @SuppressWarnings("all")
    @Autowired
    UserMapper userMapper;

//    @DS
    public List<UserInfo> getall() {
        return userMapper.getAll();
    }

    @DS(Constant.DS_SALVE)
    public List<UserInfo> selectByPrimaryKey(Integer id){
        Map<String, Object> map = new HashMap<>();
        map.put("id", id);
        return userMapper.selectByPrimaryKey(map);
    }


	/**
	* 事务处理 需要在同一个service 方法中执行
	*/
    @DS(Constant.DS_SALVE)
    @Transactional
    public int update() throws Exception {
        Map<String, Object> map = new HashMap<>();
        map.put("userId", "U2017041321123200003");
        map.put("newPassword", "abc");
        int count = userMapper.update(map);
        System.out.println( "----->  " +count);
        throw new RuntimeException("error interface");
//        return count;
    }
}
```
