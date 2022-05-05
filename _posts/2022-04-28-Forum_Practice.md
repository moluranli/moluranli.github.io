---
title: Forum_Practice
author: moluranli
date: 2022-04-28
category: Study
layout: post
---

# 简介

## 依赖技术

![image-20220504102821998](https://s2.loli.net/2022/05/04/zUXVM2FS1yWElPb.png)

## 开发环境

![image-20220504102911708](https://s2.loli.net/2022/05/04/fTzDZtREnVH8Kye.png)

# 阶段1:项目环境搭建

## 导入依赖

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

## application.properties文件

```properties
debug=true

logging.level.com.example.forum_practice=debug

spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/forum?characterEncoding=utf-8&useSSL=false&serverTimezone=Hongkong
spring.datasource.username=root
spring.datasource.password=123
```



# 阶段2:论坛首页开发

## 要点

### 思路

1. 根据数据库查出所有的帖子,然后根据每一个帖子查出帖子对应的用户信息
2. 将每一个以Map存储的帖子+用户存入进入List中
3. 创建一个Page类来实验分页,其中包含的属性包括:当前页码,每一页最大行数,总行数并且通过函数以及上述属性计算:起始行,最后一页的页码,并且得到当前页的前两页和后两页
4. 在cotroller层中对page类的属性进行设置
5. 将存储帖子和用户的List以及Page传入前端页面
6. 使用thmeleaf对属性进行读取或设置比如`th:href="@{/(current=${page.current-1})}"`代表点击跳转下一页

### 配置文件

```properties
#将驼峰命名法与属性匹配,不然数据库中含有_的字段查询出来是null
mybatis.configuration.mapUnderscoreToCamelCase=true
#mapper编译后在classes下,不然会显示dao层接口和xml找不到异常
mybatis.mapper-locations=classpath:mapper/*.xml
#实体类放在那里
mybatis.type-aliases-package=com.example.forum_practice.entity
```

### 控制层

使用List<Map<>>结构来存储帖子以及对应用户的集合,先拿出所有的帖子,然后在通过帖子查找每一用户,存入map中

```java
List<Map<String,Object>> discusspostuserlist = new ArrayList<>();
```

`new map()` 需要放在循环中,否则每次都会因为是同一个对象覆盖数据

```java
Map<String,Object> map = new HashMap<>();
map.put("post",discussPost);
User user =userService.findUserById(discussPost.getUserid());
map.put("user",user);
discusspostuserlist.add(map);
```

传出page分页以及贴子和用户的包装

```java
model.addAttribute("page",page);
model.addAttribute("discusspostuserlist",discusspostuserlist);
```

### 页面

[utext和text的区别](https://blog.csdn.net/rongxiang111/article/details/79678765)使用utext可以解析html,比如br

```html
th:utext="${map.post.title}"
```

thymeleaf内嵌函数 

`${#dates.format(map.post.createTime,'yyyy-MM-dd HH:mm:ss')}`  将date格式转换

`${#numbers.sequence(page.upPage,page.downPage)}`  从uppage到downpage(包含边界)之间的数字以数组形式输出

thymeleaf使用有动态和静态的属性处理 使用`|静态 动态|`

```html
th:class="|page-item ${page.current==i?'active':''}|"
```

thymeleaf代码复用

```html
<!--th:fragment 在含有代码的加入,起名header-->
<header class="bg-dark sticky-top" th:fragment="header">
    
<!-- th:replace 表示复用名称为header的代码 -->
<header class="bg-dark sticky-top" th:replace="index::header"></header>
```



## 数据库字段

### discuss_post表

> 文章表

```sql
CREATE TABLE `discuss_post` (
  `id` int(11) NOT NULL AUTO_INCREMENT,#文章id
  `user_id` varchar(45) DEFAULT NULL,#文章所属用户id
  `title` varchar(100) DEFAULT NULL,#标题
  `content` text,#内容
  `type` int(11) DEFAULT NULL COMMENT '0-普通; 1-置顶;',#文章类型
  `status` int(11) DEFAULT NULL COMMENT '0-正常; 1-精华; 2-拉黑;',#文章状态
  `create_time` timestamp NULL DEFAULT NULL,#创建时间
  `comment_count` int(11) DEFAULT NULL,#讨论数量,有专门的表,由此字段可以减少查表
  `score` double DEFAULT NULL,#分数,根据热度排行
  PRIMARY KEY (`id`),
  KEY `index_user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=281 DEFAULT CHARSET=utf8
```

### User表

> 用户表

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(50) DEFAULT NULL,
  `password` varchar(50) DEFAULT NULL,
  `salt` varchar(50) DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  `type` int(11) DEFAULT NULL COMMENT '0-普通用户; 1-超级管理员; 2-版主;',
  `status` int(11) DEFAULT NULL COMMENT '0-未激活; 1-已激活;',
  `activation_code` varchar(100) DEFAULT NULL,
  `header_url` varchar(200) DEFAULT NULL,
  `create_time` timestamp NULL DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `index_username` (`username`(20)),
  KEY `index_email` (`email`(20))
) ENGINE=InnoDB AUTO_INCREMENT=150 DEFAULT CHARSET=utf8
```

## Mapper文件

### 文章mapper

```xml
<mapper namespace="com.example.forum_practice.dao.DiscussPostDao">
    <sql id="select_discusspost">
        id,user_id,title,content,type,status,create_time,comment_count,score
    </sql>

    <select id="findDiscussPost" resultType="DiscussPost">
        select <include refid="select_discusspost"/>
        from discuss_post
        where status != 2
        <if test="userid!=0">
            and user_id = #{userid}
        </if>
        order by type desc, create_time desc
        limit #{offset}, #{limit}
    </select>
</mapper>
```

### 用户mapper

```xml
<mapper namespace="com.example.forum_practice.dao.UserDao">
    <sql id="select_user">
        id,username,password,salt,email,type,status,activation_code,header_url,create_time
    </sql>
    <select id="findUserById" resultType="User">
        select <include refid="select_user"/>
        from user
        where id = #{userid}
    </select>
</mapper>
```

# 阶段3:论坛注册

## 要点

### 
