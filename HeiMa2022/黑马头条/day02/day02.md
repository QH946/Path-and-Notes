# Day02

## 1)JWT

### 1.1)概述

JSON Web token简称JWT， 是用于对应用程序上的用户进行身份验证的标记。也就是说, 使用 JWTS 的应用程序不再需要保存有关其用户的 cookie 或其他session数据。此特性便于可伸缩性, 同时保证应用程序的安全。



### 1.2)内部结构

JWT就是一个字符串，经过加密处理与校验处理的字符串，形式为：A.B.C

- A由JWT头部信息header加密得到：加密算法
- B由JWT用到的身份验证信息json数据加密得到
- C由A和B加密得到，是校验部分：防止第二段内容被篡改



![image-20220711150914433](assets\image-20220711150914433.png)

### 1.3)token校验流程

![image-20220711150809277](assets\image-20220711150809277.png)



![image-20220711151005594](assets\image-20220711151005594.png)



## 2)文章列表加载

### 2.1)需求分析

#### 2.1.1)文章布局展示

![image-20210419151801252](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419151801252.png)

#### 2.1.2)页面加载逻辑

![image-20210419152011931](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419152011931.png)

预览地址：http://heima-app-java.itheima.net

![image-20220703224135764](assets\image-20220703224135764.png)

1 在默认频道展示10条文章信息

2 可以切换频道查看不同种类文章

- `__all__`代表全部频道

3 当用户下拉可以加载最新的文章（分页）本页文章列表中发布时间为最大的时间为依据

- 刷新：查询publishTime > maxBehotTime

4 当用户上拉可以加载更多的文章信息（按照发布时间）本页文章列表中发布时间最小的时间为依据

- 加载更多：查询publishTime < minBehotTime

5 如果是当前频道的首页，前端传递默认参数：

- maxBehotTime：0（毫秒）

- minBehotTime：20000000000000（毫秒）--->2063年



### 2.2)表结构分析

ap_article  文章基本信息表

- layout 布局方式：0无图、1单图、2多图

![image-20210419151839634](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419151839634.png)

ap_article_config  文章配置表

![image-20210419151854868](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419151854868.png)

ap_article_content 文章内容表

![image-20210419151912063](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419151912063.png)

三张表关系分析

![image-20210419151938103](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210419151938103.png)



#### 2.2.1)为什么文章表要拆分成多个表？

![image-20220711152810956](assets\image-20220711152810956.png)





#### 2.2.2)拆分原则

**垂直分表：**将一个表的字段分散到多个表中，每个表存储其中一部分字段。

优势：

- **减少IO争抢，减少锁表的几率**，查看文章概述与文章详情互不影响
- 充分发挥**高频数据**的操作效率，对文章概述数据操作的高效率不会被操作文章详情数据的低效率所拖累。

**拆分规则：**

- 把不常用的字段单独放在一张表
- 把text，blob等大字段拆分出来单独放在一张表
- 经常组合查询的字段单独放在一张表中



### 2.3)接口定义

|          | **加载首页**         | **加载更多**             | **加载最新**            |
| -------- | -------------------- | ------------------------ | ----------------------- |
| 接口路径 | /api/v1/article/load | /api/v1/article/loadmore | /api/v1/article/loadnew |
| 请求方式 | POST                 | POST                     | POST                    |
| 参数     | ArticleHomeDto       | ArticleHomeDto           | ArticleHomeDto          |
| 响应结果 | ResponseResult       | ResponseResult           | ResponseResult          |

ArticleHomeDto

```java
package com.heima.model.article.dtos;

import lombok.Data;

import java.util.Date;

@Data
public class ArticleHomeDto {
    // 最大时间
    Date maxBehotTime;
    // 最小时间
    Date minBehotTime;
    // 分页size
    Integer size;
    // 频道ID
    String tag;
}
```

响应结果

```java
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": [
		{
			"id": 1303156149041758210,
			"title": "全国抗击新冠肺炎疫情表彰大会",
			"authorId": 4,
			"authorName": "admin",
			"channelId": 1,
			"channelName": "java",
			"layout": 1,
			"flag": null,
			"images": "group1/M00/00/00/wKjIgl9W6iOAD2doAAFY4E1K7-g384.png",
			"labels": null,
			"likes": null,
			"collection": null,
			"comment": null,
			"views": null,
			"provinceId": null,
			"cityId": null,
			"countyId": null,
			"createdTime": "2020-09-08T10:20:12.000+00:00",
			"publishTime": "2020-09-08T10:20:12.000+00:00",
			"syncStatus": false,
			"origin": false,
			"staticUrl": null
		}
	]
}
```



### 2.4)导入文章数据库

#### 2.4.1)导入数据库

查看当天资料文件夹，在数据库连接工具中执行leadnews_article.sql

#### 2.4.2)导入对应的实体类（3个）

ap_article文章表对应实体

```java
package com.heima.model.article.pojos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

/**
 * <p>
 * 文章信息表，存储已发布的文章
 * </p>
 *
 * @author itheima
 */

@Data
@TableName("ap_article")
public class ApArticle implements Serializable {

    @TableId(value = "id",type = IdType.ID_WORKER)
    private Long id;


    /**
     * 标题
     */
    private String title;

    /**
     * 作者id
     */
    @TableField("author_id")
    private Long authorId;

    /**
     * 作者名称
     */
    @TableField("author_name")
    private String authorName;

    /**
     * 频道id
     */
    @TableField("channel_id")
    private Integer channelId;

    /**
     * 频道名称
     */
    @TableField("channel_name")
    private String channelName;

    /**
     * 文章布局  0 无图文章   1 单图文章    2 多图文章
     */
    private Short layout;

    /**
     * 文章标记  0 普通文章   1 热点文章   2 置顶文章   3 精品文章   4 大V 文章
     */
    private Byte flag;

    /**
     * 文章封面图片 多张逗号分隔
     */
    private String images;

    /**
     * 标签
     */
    private String labels;

    /**
     * 点赞数量
     */
    private Integer likes;

    /**
     * 收藏数量
     */
    private Integer collection;

    /**
     * 评论数量
     */
    private Integer comment;

    /**
     * 阅读数量
     */
    private Integer views;

    /**
     * 省市
     */
    @TableField("province_id")
    private Integer provinceId;

    /**
     * 市区
     */
    @TableField("city_id")
    private Integer cityId;

    /**
     * 区县
     */
    @TableField("county_id")
    private Integer countyId;

    /**
     * 创建时间
     */
    @TableField("created_time")
    private Date createdTime;

    /**
     * 发布时间
     */
    @TableField("publish_time")
    private Date publishTime;

    /**
     * 同步状态
     */
    @TableField("sync_status")
    private Boolean syncStatus;

    /**
     * 来源
     */
    private Boolean origin;

    /**
     * 静态页面地址
     */
    @TableField("static_url")
    private String staticUrl;
}
```

ap_article_config文章配置对应实体类

```java
package com.heima.model.article.pojos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.io.Serializable;

/**
 * <p>
 * APP已发布文章配置表
 * </p>
 *
 * @author itheima
 */

@Data
@TableName("ap_article_config")
public class ApArticleConfig implements Serializable {

    @TableId(value = "id",type = IdType.ID_WORKER)
    private Long id;

    /**
     * 文章id
     */
    @TableField("article_id")
    private Long articleId;

    /**
     * 是否可评论
     * true: 可以评论   1
     * false: 不可评论  0
     */
    @TableField("is_comment")
    private Boolean isComment;

    /**
     * 是否转发
     * true: 可以转发   1
     * false: 不可转发  0
     */
    @TableField("is_forward")
    private Boolean isForward;

    /**
     * 是否下架
     * true: 下架   1
     * false: 没有下架  0
     */
    @TableField("is_down")
    private Boolean isDown;

    /**
     * 是否已删除
     * true: 删除   1
     * false: 没有删除  0
     */
    @TableField("is_delete")
    private Boolean isDelete;
}
```

ap_article_content 文章内容对应的实体类

```java
package com.heima.model.article.pojos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.io.Serializable;

@Data
@TableName("ap_article_content")
public class ApArticleContent implements Serializable {

    @TableId(value = "id",type = IdType.ID_WORKER)
    private Long id;

    /**
     * 文章id
     */
    @TableField("article_id")
    private Long articleId;

    /**
     * 文章内容
     */
    private String content;
}
```



### 2.5)功能实现

#### 2.5.1) 创建heima-leadnews-article模块

导入heima-leadnews-article微服务，资料在当天的文件夹中。

![image-20210420000326669](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210420000326669.png)

<font color='red'>注意：需要在heima-leadnews-service的pom文件夹中添加子模块信息，如下：</font>

```xml
<modules>
    <module>heima-leadnews-user</module>
    <module>heima-leadnews-article</module>
</modules>
```

在idea中的maven中更新一下，如果工程还是灰色的，需要在重新添加文章微服务的pom文件，操作步骤如下：

![image-20210420001037992](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210420001037992.png)



bootstrap.yml添加以下配置

```yaml
server:
  port: 51802
spring:
  application:
    name: leadnews-article
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.130:8848
      config:
        server-addr: 192.168.200.130:8848
        file-extension: yml
```

需要在nacos中添加对应的配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/leadnews_article?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: root
# 设置Mapper接口所对应的XML文件位置，如果你在Mapper接口中有自定义方法，需要进行该配置
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 设置别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: com.heima.model.article.pojos
```

#### 2.5.2) 定义接口

```java
package com.heima.article.controller.v1;

import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/article")
public class ArticleHomeController {


    @PostMapping("/load")
    public ResponseResult load(@RequestBody ArticleHomeDto dto) {
        return null;
    }

    @PostMapping("/loadmore")
    public ResponseResult loadMore(@RequestBody ArticleHomeDto dto) {
        return null;
    }

    @PostMapping("/loadnew")
    public ResponseResult loadNew(@RequestBody ArticleHomeDto dto) {
        return null;
    }
}
```

#### 2.5.3) 编写mapper文件

```java
package com.heima.article.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.article.pojos.ApArticle;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;

@Mapper
public interface ApArticleMapper extends BaseMapper<ApArticle> {

    public List<ApArticle> loadArticleList(@Param("dto") ArticleHomeDto dto, @Param("type") Short type);

}
```

对应的映射文件

在resources中新建mapper/ApArticleMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.heima.article.mapper.ApArticleMapper">
    
    <select id="loadArticle" resultType="com.heima.model.article.pojos.ApArticle">
        SELECT aa.* FROM ap_article aa
        INNER JOIN ap_article_config aac ON aa.id = aac.article_id
        <where>
            aac.is_delete = 0
            AND aac.is_down = 0
            <if test="loadType == 1">
                AND aa.publish_time <![CDATA[<]]> #{dto.minBehotTime}
            </if>

            <if test="loadType == 2">
                AND aa.publish_time <![CDATA[>]]> #{dto.maxBehotTime}
            </if>

            <if test="dto.tag != null and dto.tag != '__all__'">
                AND aa.channel_id = #{dto.tag}
            </if>
        </where>
        ORDER BY aa.publish_time DESC
        LIMIT #{dto.size}
    </select>

</mapper>
```

#### 2.5.4) 编写业务层代码

```java
package com.heima.article.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.article.pojos.ApArticle;
import com.heima.model.common.dtos.ResponseResult;

import java.io.IOException;

public interface ApArticleService extends IService<ApArticle> {

    /**
     * 根据参数加载文章列表
     * @param loadtype 1为加载更多  2为加载最新
     * @param dto
     * @return
     */
    ResponseResult load(Short loadtype, ArticleHomeDto dto);

}
```

实现类：

```java
package com.heima.article.service;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.heima.article.mapper.ApArticleMapper;
import com.heima.model.article.domain.ApArticle;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class ApArticleServiceImpl extends ServiceImpl<ApArticleMapper, ApArticle> implements ApArticleService {

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Override
    public ResponseResult load(Short loadType, ArticleHomeDto dto) {
        List<ApArticle> articles = apArticleMapper.load(loadType, dto);
        return ResponseResult.okResult(articles);
    }
}

```

定义常量类

```java
package com.heima.common.constants;

public class ArticleConstants {
    public static final Short LOADTYPE_LOAD_MORE = 1;
    public static final Short LOADTYPE_LOAD_NEW = 2;
    public static final String DEFAULT_TAG = "__all__";
}
```

#### 2.5.5) 编写控制器代码

```java
package com.heima.article.controller.v1;

import com.heima.article.service.ApArticleService;
import com.heima.common.constants.ArticleConstants;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/article")
public class ArticleHomeController {

    @Autowired
    private ApArticleService apArticleService;

    @PostMapping("/load")
    public ResponseResult load(@RequestBody ArticleHomeDto dto) {
        return apArticleService.load(ArticleConstants.LOADTYPE_LOAD_MORE, dto);
    }

    @PostMapping("/loadmore")
    public ResponseResult loadMore(@RequestBody ArticleHomeDto dto) {
        return apArticleService.load(ArticleConstants.LOADTYPE_LOAD_MORE,dto);
    }

    @PostMapping("/loadnew")
    public ResponseResult loadNew(@RequestBody ArticleHomeDto dto) {
        return apArticleService.load(ArticleConstants.LOADTYPE_LOAD_NEW,dto);
    }
}
```

#### 2.5.6) 测试

第一：在app网关的微服务的nacos的配置中心添加文章微服务的路由，完整配置如下：

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]': # 匹配所有请求
            allowedOrigins: "*" #跨域处理 允许所有的域
            allowedMethods: # 支持的方法
              - GET
              - POST
              - PUT
              - DELETE
      routes:
        # 用户微服务
        - id: user
          uri: lb://leadnews-user
          predicates:
            - Path=/user/**
          filters:
            - StripPrefix= 1
        # 文章微服务
        - id: article
          uri: lb://leadnews-article
          predicates:
            - Path=/article/**
          filters:
            - StripPrefix= 1
```

第二：启动nginx，直接使用前端项目测试，启动文章微服务，用户微服务、app网关微服务



## 3)freemarker

### 3.1) freemarker介绍

FreeMarker 是一款模板引擎： 即一种基于模板和要改变的数据， 并用来生成输出文本(HTML网页，电子邮件，配置文件，源代码等)的通用工具。 它不是面向最终用户的，而是一个Java类库，是一款程序员可以嵌入他们所开发产品的组件。

模板编写为FreeMarker Template Language (FTL)。它是简单的，专用的语言， *不是* 像PHP那样成熟的编程语言。 那就意味着要准备数据在真实编程语言中来显示，比如数据库查询和业务运算， 之后模板显示已经准备好的数据。在模板中，你可以专注于如何展现数据， 而在模板之外可以专注于要展示什么数据。 

![1528820943975](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\1528820943975.png)



**常用的java模板引擎还有哪些？**

Jsp、Freemarker、Thymeleaf 、Velocity 等。

- Jsp 为 Servlet 专用，不能单独进行使用。
- Thymeleaf 为新技术，功能较为强大，但是执行的效率比较低。
- Velocity从2010年更新完 2.0 版本后，便没有在更新。Spring Boot 官方在 1.4 版本后对此也不在支持，虽然 Velocity 在 2017 年版本得到迭代，但为时已晚。 



**页面渲染过程**

![image-20220703232021468](assets\image-20220703232021468.png)



### 3.2) 环境搭建&&快速入门

freemarker作为springmvc一种视图格式，默认情况下SpringMVC支持freemarker视图格式。

需要创建Spring Boot+Freemarker工程用于测试模板。

#### 3.2.1) 创建测试工程

直接导入资料中的**“\freemarker测试类\freemarker-demo”**目录即可。

pom.xml如下：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>

<!-- apache 对 java io 的封装工具库 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-io</artifactId>
    <version>1.3.2</version>
</dependency>
```

#### 3.2.2) 配置文件

配置application.yml

```yaml
server:
  port: 8881 #服务端口
spring:
  application:
    name: freemarker-demo #指定服务名
  freemarker:
    cache: false  #关闭模板缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
    suffix: .ftl               #指定Freemarker模板文件的后缀名
```

#### 3.2.3) 创建模板

在resources下创建templates，此目录为freemarker的默认模板存放目录。

在templates下创建模板文件 01-basic.ftl ，模板中的插值表达式最终会被freemarker替换成具体的数据。

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
<b>普通文本 String 展示：</b><br><br>
Hello ${name} <br>
<hr>
<b>对象Student中的数据展示：</b><br/>
姓名：${stu.name}<br/>
年龄：${stu.age}
<hr>

<ul>
    <#--判空、条件、集合遍历-->
</ul>

<#--算术运算、比较、逻辑运算-->

<#--内置函数-->

</body>
</html>
```

#### 3.2.4) 创建controller

创建Controller类，向Map中添加name，最后返回模板文件。

```java
package com.heima.demo.freemarker.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.client.RestTemplate;

import java.util.Map;

@Controller
public class HelloController {

    @GetMapping("/basic")
    public String test(Model model) {
        //1.纯文本形式的参数
        model.addAttribute("name", "freemarker");
        //2.实体类相关的参数
        Map<String, Object> student = new HashMap<>();
        student.put("name","小明");
        student.put("age", 18);
        model.addAttribute("stu", student);

        return "01-basic";
    }
}
```

#### 3.2.6) 创建启动类

```java
package com.heima.demo.freemarker;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class FreemarkerDemotApplication {
    public static void main(String[] args) {
        SpringApplication.run(FreemarkerDemotApplication.class,args);
    }
}
```

#### 3.2.7) 测试

请求：http://localhost:8881/basic

![1576129529361](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\1576129529361.png)





### 2.3) freemarker基础

#### 2.3.1) 基础语法种类

  1、注释，即<#--  -->，介于其之间的内容会被freemarker忽略

```velocity
<#--我是一个freemarker注释-->
```

  2、插值（Interpolation）：即 **`${..}`** 部分,freemarker会用真实的值代替**`${..}`**

```velocity
Hello ${name}
```

  3、文本，仅文本信息，这些不是freemarker的注释、插值、FTL指令的内容会被freemarker忽略解析，直接输出内容。

```velocity
<#--freemarker中的普通文本-->
我是一个普通的文本
```



#### 2.3.2) 集合指令

1、数据模型：

在HelloController中修改方法如下：

```java
@GetMapping("/hello")
public String hello(Model model) {
    model.addAttribute("name", "heima");
    model.addAttribute("age", 20);
    model.addAttribute("city", "beijing");

    Map<String, Object> stu = new HashMap<>();
    stu.put("name", "itcast");
    stu.put("age", 18);
    model.addAttribute("stu", stu);

    // 新增数据
    Map<String, Object> stu2 = new HashMap<>();
    stu2.put("name", "itheima");
    stu2.put("age", 16);

    // 新增集合
    List<Map<String, Object>> list = new ArrayList<>();
    list.add(stu);
    list.add(stu2);

    model.addAttribute("stus", list);

    return "01-basic";
}
```



#### 2.3.3) if指令

if 指令即判断指令，是常用的FTL指令，freemarker在解析时遇到if会进行判断，条件为真则输出if中间的内容，否则跳过内容不再输出。

**指令格式**

```html
<#if ></if>
```

**需求：**让大于18岁的记录，用紫色字体显示。

**模板：**

```velocity
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
    <#-- 我是注释 -->
    <b>普通文本 String 展示：</b><br><br>
    Hello ${name}, ${age}, ${city} <br>
    <hr>
    <b>对象Student中的数据展示：</b><br/>
    姓名：${stu.name}<br/>
    年龄：${stu.age}
    <hr>

    我是freemark，我不会被编译，你将在页面上看到我。

    <#-- 集合遍历 -->
    <ul>
        <#list stus as stu>
            <#if (stu.age > 18)>
                <li style="color:rebeccapurple">${stu.name} - ${stu.age}</li>
            <#else>
                <li>${stu.name} - ${stu.age}</li>
            </#if>
        </#list>
    </ul>

</body>
</html>
```



#### 2.3.4)  运算符

**1、算数运算符**

FreeMarker表达式中完全支持算术运算,FreeMarker支持的算术运算符包括:

- 加法： `+`
- 减法： `-`
- 乘法： `*`
- 除法： `/`
- 求模 (求余)： `%`

举例：

```html
<b>算数运算符</b>
<br/><br/>
    100+5 运算：  ${100 + 5 }<br/>
    100 - 5 * 5运算：${100 - 5 * 5}<br/>
    5 / 2运算：${5 / 2}<br/>
    12 % 10运算：${12 % 10}<br/>
<hr>
```

除了 + 运算以外，其他的运算只能和 number 数字类型的计算。



**2、比较运算符**

- **`=`**或者**`==`**:判断两个值是否相等. 

- **`!=`**:判断两个值是否不等. 

- **`>`**或者**`gt`**:判断左边值是否大于右边值 

- **`>=`**或者**`gte`**:判断左边值是否大于等于右边值 

- **`<`**或者**`lt`**:判断左边值是否小于右边值 

- **`<=`**或者**`lte`**:判断左边值是否小于等于右边值 

  

**3、比较运算符注意**

- **`=`**和**`!=`**可以用于字符串、数值和日期来比较是否相等
- **`=`**和**`!=`**两边必须是相同类型的值,否则会产生错误
- 字符串 **`"x"`** 、**`"x "`** 、**`"X"`**比较是不等的.因为FreeMarker是精确比较
- 其它的运行符可以作用于数字和日期,但不能作用于字符串
- 使用**`gt`**等字母运算符代替**`>`**会有更好的效果,因为 FreeMarker会把**`>`**解释成FTL标签的结束字符
- 可以使用括号来避免这种情况,如:**`<#if (x>y)>`**



**3、逻辑运算符**

- 逻辑与:&& 
- 逻辑或:|| 
- 逻辑非:! 

逻辑运算符只能作用于布尔值,否则将产生错误 。



#### 2.3.5)内置函数

- 时间格式
- 统计

控制器修改为

```java
package com.heima.freemarker.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.*;

/**
 * @author itheima
 * @since 2022-07-04
 */
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model) {
        // 新增日期字段
        model.addAttribute("today", new Date());

        return "01-basic";
    }
}
```

模板文件

```java
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
	总数：${stus?size}

    <#-- 内建函数 -->
    ${today?date}
    <br/>
    ${today?time}
    <br/>
    ${today?datetime}
    <br/>
    ${today?string("yyyy年MM月")}
</body>
</html>
```



#### 2.3.6)完整代码

```java
package com.heima.freemarker.demo.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.*;

/**
 * @author itheima
 * @since 2022-07-04
 */
@Controller
public class HelloController {

    @GetMapping("/hello")
    public String hello(Model model) {
        model.addAttribute("name", "heima");
        model.addAttribute("age", 20);
        model.addAttribute("city", "beijing");

        Map<String, Object> stu = new HashMap<>();
        stu.put("name", "itcast");
        stu.put("age", 18);
        model.addAttribute("stu", stu);

        Map<String, Object> stu2 = new HashMap<>();
        stu2.put("name", "itheima");
        stu2.put("age", 16);

        List<Map<String, Object>> list = new ArrayList<>();
        list.add(stu);
        list.add(stu2);

        model.addAttribute("stus", list);
        model.addAttribute("today", new Date());

        return "01-basic";
    }
}
```

模板文件

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Hello World!</title>
</head>
<body>
    <#-- 我是注释 -->
    <b>普通文本 String 展示：</b><br><br>
    Hello ${name}, ${age}, ${city} <br>
    <hr>
    <b>对象Student中的数据展示：</b><br/>
    姓名：${stu.name}<br/>
    年龄：${stu.age}
    <hr>

    我是freemark，我不会被编译，你将在页面上看到我。

    <#-- 集合遍历 -->
    <#-- for(Object stu : stus) -->
    <ul>
        <#if stus??>
            总数：${stus?size}
            <#list stus as stu>
            <#-- 判断 -->
                <#if (stu.age > 18) || (stu.name == 'itheima')>
            <li style="color:rebeccapurple">${stu.name} - ${stu.age + 5}</li>
                <#else>
            <li>${stu.name} - ${stu.age - 5}</li>
                </#if>
            </#list>
        </#if>
    </ul>

    <#-- 数学运算，比较，逻辑运算 -->

    <#-- 空值判断 -->

    <#-- 内建函数 -->
    ${today?date}
    <br/>
    ${today?time}
    <br/>
    ${today?datetime}
    <br/>
    ${today?string("yyyy年MM月")}
</body>
</html>
```



### 2.4) 静态化测试

之前的测试都是SpringMVC将Freemarker作为视图解析器（ViewReporter）来集成到项目中，工作中，有的时候需要使用Freemarker原生Api来生成静态内容，下面一起来学习下原生Api生成文本文件。

#### 2.4.1) 需求分析

使用freemarker原生Api将页面生成html文件，本节测试html文件生成的方法：

![image-20210422163843108](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210422163843108.png)

#### 2.4.2) 静态化测试 

根据模板文件生成html文件

①：修改application.yml文件，添加以下模板存放位置的配置信息，完整配置如下：

```yaml
server:
  port: 8881 #服务端口
spring:
  application:
    name: freemarker-demo #指定服务名
  freemarker:
    cache: false  #关闭模板缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
    suffix: .ftl               #指定Freemarker模板文件的后缀名
    template-loader-path: classpath:/templates   #模板存放位置
```

②：在test下创建测试类

```java
package com.heima.freemarker.test;


import com.heima.freemarker.FreemarkerDemoApplication;
import com.heima.freemarker.entity.Student;
import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.FileWriter;
import java.io.IOException;
import java.util.*;

@SpringBootTest(classes = FreemarkerDemoApplication.class)
@RunWith(SpringRunner.class)
public class FreemarkerTest {

    @Autowired
    private Configuration configuration;

    @Test
    public void test() throws IOException, TemplateException {
        //freemarker的模板对象，获取模板
        Template template = configuration.getTemplate("02-list.ftl");
        Map params = getData();
        //合成
        //第一个参数 数据模型
        //第二个参数  输出流
        template.process(params, new FileWriter("d:/list.html"));
    }

    private Map getData() {
        Map<String, Object> map = new HashMap<>();

        //小强对象模型数据
        Student stu1 = new Student();
        stu1.setName("小强");
        stu1.setAge(18);
        stu1.setMoney(1000.86f);
        stu1.setBirthday(new Date());

        //小红对象模型数据
        Student stu2 = new Student();
        stu2.setName("小红");
        stu2.setMoney(200.1f);
        stu2.setAge(19);

        //将两个对象模型数据存放到List集合中
        List<Student> stus = new ArrayList<>();
        stus.add(stu1);
        stus.add(stu2);

        //向map中存放List集合数据
        map.put("stus", stus);


        //创建Map数据
        HashMap<String, Student> stuMap = new HashMap<>();
        stuMap.put("stu1", stu1);
        stuMap.put("stu2", stu2);
        //向map中存放Map数据
        map.put("stuMap", stuMap);

        //返回Map
        return map;
    }
}
```

### 2.5)封装Freemarker插件

可以直接找到资料里的 “**freemarker插件**”导入到heima-leadnews-common即可，记得添加第二步的自动装配配置。

①：在heima-leadnews-common中添加核心类

```java
package com.heima.common.freemarker;

import freemarker.template.Configuration;
import freemarker.template.Template;
import freemarker.template.TemplateException;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.StringWriter;
import java.util.Map;

/**
 * @author itheima
 * @since 2022-07-13
 */
@Component
@Slf4j
public class FreemarkerGenerator {

    @Autowired
    private Configuration configuration;

    /**
     * 生成静态文件
     *
     * @param templateName 模板文件
     * @param params       模板数据
     * @return 输入流
     */
    public InputStream generate(String templateName, Map params) {
        if (StringUtils.isEmpty(templateName)) {
            log.warn("模板名不能为空");
            return null;
        }

        try {
            Template template = configuration.getTemplate(templateName);
            StringWriter out = new StringWriter();
            template.process(params, out);
            return new ByteArrayInputStream(out.toString().getBytes());
        } catch (IOException | TemplateException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```

②：添加自动装配 resources/META-INF/spring.factories

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.heima.common.freemarker.FreemarkerGenerator
```



## 3) 对象存储服务MinIO 

### 3.1)MinIO简介   

MinIO基于Apache License v2.0开源协议的对象存储服务，可以做为云存储的解决方案用来保存海量的图片，视频，文档。由于采用Golang实现，服务端可以工作在Windows,Linux, OS X和FreeBSD上。配置简单，基本是复制可执行程序，单行命令可以运行起来。

MinIO兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。

**S3 （ Simple Storage Service简单存储服务）**

基本概念

- bucket – 类比于文件系统的目录
- Object – 类比文件系统的文件
- Keys – 类比文件名

官网文档：http://docs.minio.org.cn/docs/

### 3.2)MinIO特点 

- 数据保护

  Minio使用Minio Erasure Code（纠删码）来防止硬件故障。即便损坏一半以上的driver，但是仍然可以从中恢复。

- 高性能

  作为高性能对象存储，在标准硬件条件下它能达到55GB/s的读、35GB/s的写速率

- 可扩容

  不同MinIO集群可以组成联邦，并形成一个全局的命名空间，并跨越多个数据中心

- SDK支持

  基于Minio轻量的特点，它得到类似Java、Python或Go等语言的sdk支持

- 有操作页面

  面向用户友好的简单操作界面，非常方便的管理Bucket及里面的文件资源

- 功能简单

  这一设计原则让MinIO不容易出错、更快启动

- 丰富的API

  支持文件资源的分享连接及分享链接的过期策略、存储桶操作、文件列表访问及文件上传下载的基本功能等。

- 文件变化主动通知

  存储桶（Bucket）如果发生改变,比如上传对象和删除对象，可以使用存储桶事件通知机制进行监控，并通过以下方式发布出去:AMQP、MQTT、Elasticsearch、Redis、NATS、MySQL、Kafka、Webhooks等。


### 3.3)开箱使用 

#### 3.3.1)安装启动   

我们提供的镜像中已经有minio的环境

我们可以使用docker进行环境部署和启动

```shell
docker run -p 9000:9000 -p 9090:9090 \
 --net=host \
 --name minio \
 -d --restart=always \
 -e "MINIO_ACCESS_KEY=minio" \
 -e "MINIO_SECRET_KEY=minio123" \
 -v /home/docker/minio/data:/mydata/minio/data \
 -v /home/docker/minio/config:/mydata/minio/config \
 minio/minio server \
 /data --console-address ":9090" -address ":9000"
```

#### 3.3.2)管理控制台

地址栏输入：http://192.168.200.130:9000/ 即可进入登录界面。

![image-20220618231411762](assets\image-20220618231411762.png)

账号为：minio   ，密码为：minio123，进入系统后可以看到主界面

![image-20220618231500005](assets\image-20220618231500005.png)

点击右上角的“Create Bucket” ，创建一个桶

![image-20220618231537075](assets\image-20220618231537075.png)

输入Bucket Name，点击Create Bucket即可。

![image-20220618231624500](assets\image-20220618231624500.png)

### 3.4)封装MinIO为starter

**可以在资料中找到mino目录，将里面三个文件导入到heima-file-starter中就可以。**

#### 3.4.1)导入依赖

在heima-leadnews-basic下创建 heima-file-starter

```xml
<dependency>
    <groupId>io.minio</groupId>
    <artifactId>minio</artifactId>
    <version>7.1.0</version>
</dependency>
```

#### 3.4.2)配置类

```java
package com.heima.minio;

import io.minio.MinioClient;
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;

@Data
@ConfigurationProperties(prefix = "minio")
public class MinIoProperties {
    private String accessKey;
    private String secretKey;
    private String bucket;
    private String endpoint;
    private String readPath;

    @Bean
    public MinioClient buildMinIoClient(){
        return MinioClient
                .builder()
                .credentials(accessKey, secretKey)
                .endpoint(endpoint)
                .build();
    }
}
```

#### 3.4.3)封装操作minIO类

```java
package com.heima.minio;

import io.minio.GetObjectArgs;
import io.minio.MinioClient;
import io.minio.PutObjectArgs;
import io.minio.RemoveObjectArgs;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.util.StringUtils;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.text.SimpleDateFormat;
import java.util.Date;

@Slf4j
public class MinIoTemplate {

    private final static String separator = "/";

    @Autowired
    private MinioClient minioClient;

    private MinIoProperties minIoProperties;

    public MinIoTemplate(MinIoProperties minIoProperties) {
        this.minIoProperties = minIoProperties;
    }

    /**
     * 构建文件路径
     *
     * @param dirPath  目录
     * @param filename 文件名{yyyy/mm/dd/file.jpg}
     * @return 文件路径
     */
    public String builderFilePath(String dirPath, String filename) {
        StringBuilder stringBuilder = new StringBuilder(50);
        if (!StringUtils.isEmpty(dirPath)) {
            stringBuilder.append(dirPath).append(separator);
        }
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd");
        String todayStr = sdf.format(new Date());
        stringBuilder.append(todayStr).append(separator);
        stringBuilder.append(filename);
        return stringBuilder.toString();
    }

    /**
     * 上传图片文件
     *
     * @param prefix      文件前缀
     * @param filename    文件名
     * @param inputStream 文件流
     * @return 文件全路径
     */
    public String uploadFile(String prefix, String filename, InputStream inputStream, String type) {
        String filePath = builderFilePath(prefix, filename);
        try {
            String contentType = "image/jpg";
            if ("html".equals(type)) {
                contentType = "text/html";
            }

            PutObjectArgs putObjectArgs = PutObjectArgs.builder()
                    .object(filePath)
                    .contentType(contentType)
                    .bucket(minIoProperties.getBucket()).stream(inputStream, inputStream.available(), -1)
                    .build();
            minioClient.putObject(putObjectArgs);
            StringBuilder urlPath = new StringBuilder(minIoProperties.getReadPath());
            urlPath.append(separator + minIoProperties.getBucket());
            urlPath.append(separator);
            urlPath.append(filePath);
            return urlPath.toString();
        } catch (Exception ex) {
            log.error("minio put file error.", ex);
            throw new RuntimeException("上传文件失败");
        }
    }

    /**
     * 删除文件
     *
     * @param pathUrl 文件全路径
     */
    public void delete(String pathUrl) {
        String key = pathUrl.replace(minIoProperties.getEndpoint() + "/", "");
        int index = key.indexOf(separator);
        String bucket = key.substring(0, index);
        String filePath = key.substring(index + 1);
        // 删除Objects
        RemoveObjectArgs removeObjectArgs = RemoveObjectArgs.builder().bucket(bucket).object(filePath).build();
        try {
            minioClient.removeObject(removeObjectArgs);
        } catch (Exception e) {
            log.error("minio remove file error.  pathUrl:{}", pathUrl);
            e.printStackTrace();
        }
    }


    /**
     * 下载文件
     *
     * @param pathUrl 文件全路径
     * @return 文件流
     */
    public byte[] downLoadFile(String pathUrl) {
        String key = pathUrl.replace(minIoProperties.getEndpoint() + "/", "");
        int index = key.indexOf(separator);
        String bucket = key.substring(0, index);
        String filePath = key.substring(index + 1);
        InputStream inputStream = null;
        try {
            inputStream = minioClient.getObject(GetObjectArgs.builder().bucket(bucket).object(filePath).build());
        } catch (Exception e) {
            log.error("minio down file error.  pathUrl:{}", pathUrl);
            e.printStackTrace();
        }

        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        byte[] buff = new byte[100];
        int rc = 0;
        while (true) {
            try {
                if (!((rc = inputStream.read(buff, 0, 100)) > 0)) break;
            } catch (IOException e) {
                e.printStackTrace();
            }
            byteArrayOutputStream.write(buff, 0, rc);
        }
        return byteArrayOutputStream.toByteArray();
    }
}
```

#### 3.4.4)对外加入自动配置

添加自动装配类

```java
package com.heima.minio;

import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;

/**
 * 上传工具入口类
 *
 * @author tongdulong@itcast.cn
 */
@EnableConfigurationProperties({
        MinIoProperties.class
})
public class MinIoConfiguration {
    @Bean
    public MinIoTemplate smsTemplate(MinIoProperties properties) {
        return new MinIoTemplate(properties);
    }
}
```

修改resources中的`META-INF/spring.factories`，将自动装配类设置为自动加载

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.heima.minio.MinIoConfiguration
```



#### 3.4.5)其他微服务使用

第一，在微服务中添加minio所需要的配置

```yaml
minio:
  accessKey: minio
  secretKey: minio123
  bucket: leadnews
  endpoint: http://192.168.200.130:9000
  readPath: http://192.168.200.130:9000
```

第三，在对应使用的业务类中注入MinIoTemplate，样例如下：

```java
package com.heima.minio.test;


import com.heima.file.service.FileStorageService;
import com.heima.minio.MinioApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.FileInputStream;
import java.io.FileNotFoundException;

@SpringBootTest(classes = MinioApplication.class)
@RunWith(SpringRunner.class)
public class MinioTest {

    @Autowired
    private MinIoTemplate minIoTemplate;

    @Test
    public void testUpdateImgFile() {
        try {
            FileInputStream fileInputStream = new FileInputStream("E:\\tmp\\ak47.jpg");
            String filePath = minIoTemplate.uploadFile("", "ak47.jpg", fileInputStream, "image");
            System.out.println(filePath);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```



## 4)文章详情

### 4.1)需求分析

![image-20210602180753705](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210602180753705.png)



### 4.2)实现方案

方案一

用户某一条文章，根据文章的id去查询文章内容表，返回渲染页面

![image-20210602180824202](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210602180824202.png)

方案二

![image-20210602180856833](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210602180856833.png)



### 4.3)实现步骤

#### 4.3.1)上传模板素材

资料中找到模板文件（article.ftl）拷贝到heima-leadnews-artile微服务下

![image-20210602180931839](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210602180931839.png)

资料中找到index.js和index.css两个文件手动上传到MinIO中

![image-20210602180957787](app端文章查看，静态化freemarker,分布式文件系统minIO.assets\image-20210602180957787.png)

#### 4.3.2)调用测试

在heima-leadnews-artile微服务中添加MinIO插件

```xml
<dependency>
    <groupId>com.heima</groupId>
    <artifactId>heima-file-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

新建ApArticleContentMapper

```java
package com.heima.article.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.heima.model.article.pojos.ApArticleContent;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface ApArticleContentMapper extends BaseMapper<ApArticleContent> {
}
```

在artile微服务中新增测试类（后期新增文章的时候创建详情静态页，目前暂时手动生成）

```java
package com.heima.article;

import com.alibaba.fastjson.JSONArray;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.heima.article.mapper.ApArticleContentMapper;
import com.heima.article.mapper.ApArticleMapper;
import com.heima.common.freemarker.FreemarkerGenerator;
import com.heima.file.MinIoTemplate;
import com.heima.model.article.pojos.ApArticle;
import com.heima.model.article.pojos.ApArticleContent;
import org.apache.commons.lang3.StringUtils;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;

@SpringBootTest(classes = ArticleApplication.class)
@RunWith(SpringRunner.class)
public class ArticleFreemarkerTest {

    @Autowired
    private FreemarkerGenerator freemarkerGenerator;

    @Autowired
    private MinIoTemplate minIoTemplate;

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private ApArticleContentMapper apArticleContentMapper;

    @Test
    public void createStaticUrlTest() throws Exception {
        //1.获取文章内容
        LambdaQueryWrapper<ApArticleContent> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(ApArticleContent::getArticleId, 1390536764510310401L);
        ApArticleContent apArticleContent = apArticleContentMapper.selectOne(wrapper);
        if (apArticleContent == null || StringUtils.isEmpty(apArticleContent.getContent())) {
            return;
        }

        //2.文章内容通过freemarker生成html文件
        Map<String, Object> params = new HashMap<>();
        params.put("content", JSONArray.parseArray(apArticleContent.getContent()));
        InputStream generate = freemarkerGenerator.generate("article.ftl", params);

        //3.把html文件上传到minio中
        String path = minIoTemplate.uploadFile("", apArticleContent.getArticleId() + ".html", generate, "html");

        //4.修改ap_article表，保存static_url字段
        ApArticle article = new ApArticle();
        article.setId(apArticleContent.getArticleId());
        article.setStaticUrl(path);
        apArticleMapper.updateById(article);
    }
}
```



