---
title: Forum_Practice
author: moluranli
date: 2022-04-28
category: Study
layout: post
---

# Forum_Practice论坛项目

## 简介

略

## 阶段1:项目环境搭建

### 导入依赖

```xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.2.2</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
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
```

### application.properties文件

```properties
debug=true

logging.level.com.example.forum_practice=debug

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/forum?characterEncoding=utf-8&useSSL=false&serverTimezone=Hongkong
spring.datasource.username=root
spring.datasource.password=123
```



## 阶段2:论坛首页开发

### 数据库字段

#### discuss_post表

> 文章表

```sql
CREATE TABLE `discuss_post` (
  `id` int(11) NOT NULL AUTO_INCREMENT,//文章id
  `user_id` varchar(45) DEFAULT NULL,//文章所属用户id
  `title` varchar(100) DEFAULT NULL,//标题
  `content` text,//内容
  `type` int(11) DEFAULT NULL COMMENT '0-普通; 1-置顶;',//文章类型
  `status` int(11) DEFAULT NULL COMMENT '0-正常; 1-精华; 2-拉黑;',//文章状态
  `create_time` timestamp NULL DEFAULT NULL,//创建时间
  `comment_count` int(11) DEFAULT NULL,//讨论数量,有专门的表,由此字段可以减少查表
  `score` double DEFAULT NULL,//分数,根据热度排行
  PRIMARY KEY (`id`),
  KEY `index_user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=281 DEFAULT CHARSET=utf8
```

#### 等等

