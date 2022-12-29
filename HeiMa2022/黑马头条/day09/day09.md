## Day09 用户行为

### 1)什么是行为

用户行为数据的记录包括了关注、点赞、不喜欢、收藏、阅读等**行为**。

黑马头条项目整个项目开发涉及web展示和大数据分析来给用户推荐文章，如何找出哪些文章是热点文章进行针对性的推荐呢？这个时候需要进行大数据分析的准备工作，埋点。

所谓“**埋点**”，是数据采集领域（尤其是用户行为数据采集领域）的术语，指的是针对特定用户行为或事件进行捕获、处理和发送的相关技术及其实施过程。比如用户某个icon点击次数、阅读文章的时长，观看视频的时长等等。

#### 1.1)前置说明

- 所有的行为数据，都存储到redis中
- 点赞、阅读、不喜欢需要专门创建一个微服务来处理数据，新建模块：heima-leadnews-behavior
- 关注需要在heima-leadnews-user微服务中实现
- 收藏与文章详情数据回显在heima-leadnews-article微服务中实现

#### 1.2)项目搭建

①：找到资料中的 "类\BehaviorConstants.java" 常量类，并导入 heima-leadnews-common 项目中

```java
package com.heima.common.constants;

public class BehaviorConstants {

    public static final String LIKE_BEHAVIOR="LIKE-BEHAVIOR-";
    public static final String UN_LIKE_BEHAVIOR="UNLIKE-BEHAVIOR-";
    public static final String COLLECTION_BEHAVIOR="COLLECTION-BEHAVIOR-";
    public static final String READ_BEHAVIOR="READ-BEHAVIOR-";
    public static final String APUSER_FOLLOW_RELATION="APUSER-FOLLOW-";
    public static final String APUSER_FANS_RELATION="APUSER-FANS-";
}
```

②：找到资料中的 "heima-leadnews-behavior" 导入到 heima-leadnews-service 模块下

③：检查nacos配置文件

```yaml
spring:
  autoconfigure:
    exclude: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://192.168.200.130:3306/leadnews_schedule?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: root
  redis:
    host: 192.168.200.130
    port: 6379
    password: leadnews
# 设置Mapper接口所对应的XML文件位置，如果你在Mapper接口中有自定义方法，需要进行该配置
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 设置别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: com.heima.model.schedule.pojos
```

④：修改 heima-leadnews-app-gateway 的nacos配置文件，添加以下转发规则

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: behavior
          uri: lb://leadnews-behavior
          predicates:
            - Path=/behavior/**
          filters:
            - StripPrefix= 1
```

⑤：修改模板文件

![image-20220720170933160](assets\image-20220720170933160.png)

⑥：重新上传index.js到Minio上

![image-20220720171037288](assets\image-20220720171037288.png)



### 2)关注/取消关注

![image-20210727162600274](用户行为-需求.assets\image-20210727162600274.png)

**如上效果：**

当前登录后的用户可以关注作者，也可以取消关注作者

一个用户关注了作者，作者是由用户实名认证以后开通的作者权限，才有了作者信息，作者肯定是app中的一个用户。

- 从用户的角度出发：一个用户可以关注其他多个作者 —— 我的关注
- 从作者的角度出发：一个用户（同是作者）也可以拥有很多个粉丝 —— 我的粉丝

![image-20210727163038634](用户行为-需求.assets\image-20210727163038634.png)



#### 2.1)接口分析

|          | **说明**                 |
| -------- | ------------------------ |
| 接口路径 | /api/v1/user/user_follow |
| 请求方式 | POST                     |
| 参数     | UserRelationDto          |
| 响应结果 | ResponseResult           |

参数


```javascript
{
	"articleId": 0,   //文章id
	"authorId": 0,    //作者id
	"operation": 0    //0关注   1取消关注
}
```

#### 2.2)逻辑分析

![image-20220719235938088](assets\image-20220719235938088.png)

#### 2.3)环境准备

①：添加ThreadLocal类

```java
package com.heima.user.thread;

import com.heima.model.user.pojos.ApUser;

public class AppThreadLocalUtil {

    private final static ThreadLocal<ApUser> WM_USER_THREAD_LOCAL = new ThreadLocal<>();

    /**
     * 添加用户
     * @param apUser
     */
    public static void  setUser(ApUser apUser){
        WM_USER_THREAD_LOCAL.set(apUser);
    }

    /**
     * 获取用户
     */
    public static ApUser getUser(){
        return WM_USER_THREAD_LOCAL.get();
    }

    /**
     * 清理用户
     */
    public static void clear(){
        WM_USER_THREAD_LOCAL.remove();
    }
}
```

②：添加拦截器类

```java
package com.heima.user.interceptor;

import com.heima.model.user.pojos.ApUser;
import com.heima.user.thread.AppThreadLocalUtil;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AppTokenInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String userId = request.getHeader("userId");
        if(userId != null){
            //存入到当前线程中
            ApUser apUser = new ApUser();
            apUser.setId(Integer.valueOf(userId));
            AppThreadLocalUtil.setUser(apUser);

        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        AppThreadLocalUtil.clear();
    }
}
```

③：添加拦截器配置类

```java
package com.heima.user.config;


import com.heima.user.interceptor.AppTokenInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AppTokenInterceptor()).addPathPatterns("/**");
    }
}
```

#### 2.4)代码实现

在leadnews-heima-user服务中开发接口

①：在 heima-leadnews-model 中导入 UserRelationDto

```java
package com.heima.model.user.dtos;

import com.heima.model.common.annotation.IdEncrypt;
import lombok.Data;

@Data
public class UserRelationDto {

    // 文章作者ID
    Integer authorId;

    // 文章id
    Long articleId;
    /**
     * 操作方式
     * 0  关注
     * 1  取消
     */
    Short operation;
}
```

②：创建 ApUserRelationService 并添加方法

```java
package com.heima.user.service;


import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.user.dtos.UserRelationDto;


public interface ApUserRelationService {
    /**
     * 用户关注/取消关注
     * @param dto
     * @return
     */
    public ResponseResult follow(UserRelationDto dto);
}
```

③：创建 ApUserRelationServiceImpl 并添加方法

```JAVA
package com.heima.user.service.impl;

import com.heima.common.cache.CacheService;
import com.heima.common.constants.BehaviorConstants;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.dtos.UserRelationDto;
import com.heima.model.user.pojos.ApUser;
import com.heima.user.service.ApUserRelationService;
import com.heima.user.thread.AppThreadLocalUtil;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ApUserRelationServiceImpl implements ApUserRelationService {

    @Autowired
    private CacheService cacheService;

    @Override
    public ResponseResult follow(UserRelationDto dto) {
        if (dto == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
        }

        ApUser user = AppThreadLocalUtil.getUser();
        if (user == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.AP_USER_DATA_NOT_EXIST);
        }

        Integer userId = user.getId();
        Integer authorId = dto.getAuthorId();

        Short operation = dto.getOperation();

        // 关注
        if (0 == operation) {
            cacheService.zAdd(BehaviorConstants.APUSER_FOLLOW_RELATION + userId, authorId.toString(), System.currentTimeMillis());
            cacheService.zAdd(BehaviorConstants.APUSER_FANS_RELATION + authorId, userId.toString(), System.currentTimeMillis());
        }
        // 取消关注
        else if (1 == operation) {
            cacheService.zRemove(BehaviorConstants.APUSER_FOLLOW_RELATION + userId, authorId.toString());
            cacheService.zRemove(BehaviorConstants.APUSER_FANS_RELATION + authorId, userId.toString());
        }

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：添加 UserRelationController 类

```java
package com.heima.user.controller.v1;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.user.dtos.UserRelationDto;
import com.heima.user.service.ApUserRelationService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/user")
public class UserRelationController {

    @Autowired
    private ApUserRelationService apUserRelationService;
  
    @PostMapping("/user_follow")
    public ResponseResult follow(@RequestBody UserRelationDto dto){
        return apUserRelationService.follow(dto);
    }
}
```

### 3)文章点赞/取消点赞

- 当前登录的用户点击了”赞“,就要保存当前行为数据
- 可以取消点赞

![image-20210727161838807](用户行为-需求.assets\image-20210727161838807.png)



#### 3.1)接口分析

|          | **说明**               |
| -------- | ---------------------- |
| 接口路径 | /api/v1/likes_behavior |
| 请求方式 | POST                   |
| 参数     | LikesBehaviorDto       |
| 响应结果 | ResponseResult         |

参数

```json
{
	"articleId": 0,   // 文章id
	"operation": 0,   // 0点赞 1取消点赞
	"type": 0         // 0文章 1动态 2评论
}
```

#### 3.2)逻辑分析

![image-20220720000256356](assets\image-20220720000256356.png)

#### 3.3)代码实现

在leadnews-heima-behavior服务中开发接口。

①：在 heima-leadnews-model 中导入 LikesBehaviorDto.java

```java
package com.heima.model.behavior.dtos;

import lombok.Data;

@Data
public class LikesBehaviorDto {


    // 文章、动态、评论等ID
    Long articleId;
    /**
     * 喜欢内容类型
     * 0文章
     * 1动态
     * 2评论
     */
    Short type;

    /**
     * 喜欢操作方式
     * 0 点赞
     * 1 取消点赞
     */
    Short operation;
}
```

②：ApLikesBehaviorService

```java
package com.heima.behavior.service;

import com.heima.model.behavior.dtos.LikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;

public interface ApLikesBehaviorService {

    /**
     * 存储喜欢数据
     * @param dto
     * @return
     */
    ResponseResult like(LikesBehaviorDto dto);
}
```

③：ApLikesBehaviorServiceImpl

```java
package com.heima.behavior.service.impl;

import com.alibaba.fastjson.JSON;
import com.heima.behavior.service.ApLikesBehaviorService;
import com.heima.behavior.thread.AppThreadLocalUtil;
import com.heima.common.cache.CacheService;
import com.heima.common.constants.BehaviorConstants;
import com.heima.model.behavior.dtos.LikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.pojos.ApUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ApLikesBehaviorServiceImpl implements ApLikesBehaviorService {

    @Autowired
    private CacheService cacheService;

    @Override
    public ResponseResult like(LikesBehaviorDto dto) {
        if (dto == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
        }

        ApUser user = AppThreadLocalUtil.getUser();
        if (user == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.AP_USER_DATA_NOT_EXIST);
        }

        Integer userId = user.getId();
        Short operation = dto.getOperation();
        Long articleId = dto.getArticleId();

        // 点赞
        if (0 == operation) {
            Object o = cacheService.hGet(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString());
            if (o != null) {
                return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "已点赞");
            }

            cacheService.hPut(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString(), JSON.toJSONString(dto));
        }
        // 取消点赞
        else if (1 == operation) {
            cacheService.hDelete(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString());
        }

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：ApLikesBehaviorController

```java
package com.heima.behavior.controller.v1;

import com.heima.behavior.service.ApLikesBehaviorService;
import com.heima.model.behavior.dtos.LikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/likes_behavior")
public class ApLikesBehaviorController {

    @Autowired
    private ApLikesBehaviorService apLikesBehaviorService;

    @PostMapping
    public ResponseResult like(@RequestBody LikesBehaviorDto dto) {
        return apLikesBehaviorService.like(dto);
    }
}
```



### 4)阅读

当用户查看了某一篇文章，需要记录当前用户查看的次数，阅读时长（非必要），阅读文章的比例（非必要），加载的时长（非必要）。

#### 4.1)接口分析

|          | **说明**              |
| -------- | --------------------- |
| 接口路径 | /api/v1/read_behavior |
| 请求方式 | POST                  |
| 参数     | ReadBehaviorDto       |
| 响应结果 | ResponseResult        |

#### 4.2)逻辑分析

![image-20220720114630608](assets\image-20220720114630608.png)

#### 4.3)代码实现

在leadnews-heima-behavior服务中开发接口。

①：ReadBehaviorDto

```java
package com.heima.model.behavior.dtos;

import lombok.Data;

@Data
public class ReadBehaviorDto {

    // 文章、动态、评论等ID
    Long articleId;

    /**
     * 阅读次数
     */
    Short count;

    /**
     * 阅读时长（S)
     */
    Integer readDuration;

    /**
     * 阅读百分比
     */
    Short percentage;

    /**
     * 加载时间
     */
    Short loadDuration;

}
```

②：ApReadBehaviorService

```java
package com.heima.behavior.service;

import com.heima.model.behavior.dtos.ReadBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;

public interface ApReadBehaviorService {

    /**
     * 保存阅读行为
     * @param dto
     * @return
     */
    ResponseResult readBehavior(ReadBehaviorDto dto);
}
```

③：ApReadBehaviorServiceImpl

```java
package com.heima.behavior.service.impl;

import com.alibaba.fastjson.JSON;
import com.heima.behavior.service.ApReadBehaviorService;
import com.heima.behavior.thread.AppThreadLocalUtil;
import com.heima.common.cache.CacheService;
import com.heima.common.constants.BehaviorConstants;
import com.heima.model.behavior.dtos.ReadBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.pojos.ApUser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class ApReadBehaviorServiceImpl implements ApReadBehaviorService {

    @Autowired
    private CacheService cacheService;

    @Override
    public ResponseResult read(ReadBehaviorDto dto) {
        if (dto == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
        }

        ApUser user = AppThreadLocalUtil.getUser();
        if (user == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.AP_USER_DATA_NOT_EXIST);
        }

        Long articleId = dto.getArticleId();
        Integer userId = user.getId();

        String o = (String) cacheService.hGet(BehaviorConstants.READ_BEHAVIOR + articleId, userId.toString());
        if (o != null) {
            ReadBehaviorDto readBehaviorDto = JSON.parseObject(o, ReadBehaviorDto.class);
            Short totalCount = readBehaviorDto.getCount();
            Short currentCount = dto.getCount();
            Short newCount = (short) (totalCount + currentCount);
            dto.setCount(newCount);
        }

        cacheService.hPut(BehaviorConstants.READ_BEHAVIOR + articleId, userId.toString(), JSON.toJSONString(dto));

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：ApReadBehaviorController

```java
package com.heima.behavior.controller.v1;

import com.heima.behavior.service.ApReadBehaviorService;
import com.heima.model.behavior.dtos.ReadBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/read_behavior")
public class ApReadBehaviorController {

    @Autowired
    private ApReadBehaviorService apReadBehaviorService;

    @PostMapping
    public ResponseResult readBehavior(@RequestBody ReadBehaviorDto dto) {
        return apReadBehaviorService.readBehavior(dto);
    }
}
```

### 5)文章不喜欢/取消不喜欢

为什么会有不喜欢？

一旦用户点击了不喜欢，不再给当前用户推荐这一类型的文章信息

![image-20210727162221437](用户行为-需求.assets\image-20210727162221437.png)

#### 5.1)接口分析

|          | **说明**                  |
| -------- | ------------------------- |
| 接口路径 | /api/v1/un_likes_behavior |
| 请求方式 | POST                      |
| 参数     | UnLikesBehaviorDto        |
| 响应结果 | ResponseResult            |

参数

```json
{
    "articleId": 123123,  // 文章id
    "type": 0             // 不喜欢操作方式：0 不喜欢 1 取消不喜欢
}
```

#### 5.2)逻辑分析

![image-20220720115032489](assets\image-20220720115032489.png)

#### 5.3)代码实现

在leadnews-heima-behavior服务中开发接口。

①：UnLikesBehaviorDto

```java
package com.heima.model.behavior.dtos;

import lombok.Data;

@Data
public class UnLikesBehaviorDto {
    // 文章ID
    Long articleId;

    /**
     * 不喜欢操作方式
     * 0 不喜欢
     * 1 取消不喜欢
     */
    Short type;
}
```

②：ApUnlikesBehaviorService

```java
package com.heima.behavior.service;

import com.heima.model.behavior.dtos.UnLikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;

/**
 * <p>
 * APP不喜欢行为表 服务类
 * </p>
 *
 * @author itheima
 */
public interface ApUnlikesBehaviorService {

    /**
     * 不喜欢
     * @param dto
     * @return
     */
    ResponseResult unLike(UnLikesBehaviorDto dto);

}
```

③：ApUnlikesBehaviorServiceImpl

```java
package com.heima.behavior.service.impl;

import com.alibaba.fastjson.JSON;
import com.heima.behavior.service.ApUnlikesBehaviorService;
import com.heima.common.constants.BehaviorConstants;
import com.heima.common.redis.CacheService;
import com.heima.model.behavior.dtos.UnLikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.pojos.ApUser;
import com.heima.utils.thread.AppThreadLocalUtil;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * <p>
 * APP不喜欢行为表 服务实现类
 * </p>
 *
 * @author itheima
 */
@Slf4j
@Service
public class ApUnlikesBehaviorServiceImpl implements ApUnlikesBehaviorService {

    @Autowired
    private CacheService cacheService;

    @Override
    public ResponseResult unLike(UnLikesBehaviorDto dto) {

        if (dto.getArticleId() == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
        }

        ApUser user = AppThreadLocalUtil.getUser();
        if (user == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.NEED_LOGIN);
        }

        if (dto.getType() == 0) {
            cacheService.hPut(BehaviorConstants.UN_LIKE_BEHAVIOR + dto.getArticleId().toString(), user.getId().toString(), JSON.toJSONString(dto));
        } else {
            cacheService.hDelete(BehaviorConstants.UN_LIKE_BEHAVIOR + dto.getArticleId().toString(), user.getId().toString());
        }

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：ApUnlikesBehaviorController

```java
package com.heima.behavior.controller.v1;

import com.heima.behavior.service.ApUnlikesBehaviorService;
import com.heima.model.behavior.dtos.UnLikesBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/un_likes_behavior")
public class ApUnlikesBehaviorController {

    @Autowired
    private ApUnlikesBehaviorService apUnlikesBehaviorService;

    @PostMapping
    public ResponseResult unLike(@RequestBody UnLikesBehaviorDto dto) {
        return apUnlikesBehaviorService.unLike(dto);
    }
}
```



### 6)收藏

![image-20210727162332931](用户行为-需求.assets\image-20210727162332931.png)

记录当前登录人收藏的文章

#### 6.1)接口分析

|          | **说明**                    |
| -------- | --------------------------- |
| 接口路径 | /api/v1/collection_behavior |
| 请求方式 | POST                        |
| 参数     | CollectionBehaviorDto       |
| 响应结果 | ResponseResult              |

参数

```json
{
	"entryId": "1302862387124125700",         // 文章id
	"publishedTime": 2022,                    // 发布时间
	"type": 0,                                // 0文章    1动态
	"operation": 0                            // 0收藏    1取消收藏
}
```

#### 6.2)逻辑分析

![image-20220720143435555](assets\image-20220720143435555.png)

#### 6.3)环境准备

①：添加ThreadLocal类

```java
package com.heima.user.thread;

import com.heima.model.user.pojos.ApUser;

public class AppThreadLocalUtil {

    private final static ThreadLocal<ApUser> WM_USER_THREAD_LOCAL = new ThreadLocal<>();

    /**
     * 添加用户
     * @param apUser
     */
    public static void  setUser(ApUser apUser){
        WM_USER_THREAD_LOCAL.set(apUser);
    }

    /**
     * 获取用户
     */
    public static ApUser getUser(){
        return WM_USER_THREAD_LOCAL.get();
    }

    /**
     * 清理用户
     */
    public static void clear(){
        WM_USER_THREAD_LOCAL.remove();
    }
}
```

②：添加拦截器类

```java
package com.heima.article.interceptor;

import com.heima.article.thread.AppThreadLocalUtil;
import com.heima.model.user.pojos.ApUser;
import org.springframework.web.servlet.HandlerInterceptor;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class AppTokenInterceptor implements HandlerInterceptor {

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String userId = request.getHeader("userId");
        if(userId != null){
            //存入到当前线程中
            ApUser apUser = new ApUser();
            apUser.setId(Integer.valueOf(userId));
            AppThreadLocalUtil.setUser(apUser);

        }
        return true;
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        AppThreadLocalUtil.clear();
    }
}
```

③：添加拦截器配置类

```java
package com.heima.article.config;

import com.heima.article.interceptor.AppTokenInterceptor;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new AppTokenInterceptor()).addPathPatterns("/**");
    }
}
```

#### 6.4)代码实现

在 leadnews-heima-article 服务中开发接口。

①：CollectionBehaviorDto

```java
package com.heima.model.article.dtos;

import lombok.Data;

import java.util.Date;

@Data
public class CollectionBehaviorDto {

    // 文章、动态ID
    Long entryId;
    /**
     * 收藏内容类型
     * 0文章
     * 1动态
     */
    Short type;

    /**
     * 操作类型
     * 0收藏
     * 1取消收藏
     */
    Short operation;

    Date publishedTime;
}
```

②：ApCollectionService

```java
package com.heima.article.service;

import com.heima.model.article.dtos.CollectionBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;

public interface ApCollectionService {

    /**
     * 收藏
     * @param dto
     * @return
     */
    ResponseResult collection(CollectionBehaviorDto dto);
}
```

③：ApCollectionServiceImpl

```java
package com.heima.article.service;

import com.alibaba.fastjson.JSON;
import com.heima.article.thread.AppThreadLocalUtil;
import com.heima.common.cache.CacheService;
import com.heima.common.constants.BehaviorConstants;
import com.heima.model.article.dtos.CollectionBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.user.pojos.ApUser;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * @author itheima
 * @since 2022-07-20
 */
@Service
public class ArticleCollectionServiceImpl implements ArticleCollectionService {

    @Autowired
    private CacheService cacheService;

    @Override
    public ResponseResult collection(CollectionBehaviorDto dto) {
        if (dto == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
        }

        ApUser user = AppThreadLocalUtil.getUser();
        if (user == null) {
            return ResponseResult.errorResult(AppHttpCodeEnum.AP_USER_DATA_NOT_EXIST);
        }

        Integer userId = user.getId();

        Short type = dto.getType();
        Long entryId = dto.getEntryId();

        // 收藏
        if (0 == type) {
            Object o = cacheService.hGet(BehaviorConstants.COLLECTION_BEHAVIOR + entryId, userId.toString());
            if (o != null) {
                return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "已收藏");
            }

            cacheService.hPut(BehaviorConstants.COLLECTION_BEHAVIOR + entryId, userId.toString(), JSON.toJSONString(dto));
        }
        // 取消收藏
        else if (1 == type) {
            cacheService.hDelete(BehaviorConstants.COLLECTION_BEHAVIOR + entryId, userId.toString());
        }

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：ApCollectionController

```java
package com.heima.article.controller.v1;

import com.heima.article.service.ApCollectionService;
import com.heima.model.article.dtos.CollectionBehaviorDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/collection_behavior")
public class ApCollectionController {

    @Autowired
    private ApCollectionService apCollectionService;

    @PostMapping
    public ResponseResult collection(@RequestBody CollectionBehaviorDto dto) {
        return apCollectionService.collection(dto);
    }
}
```





### 7)文章详情-行为数据回显

用来展示文章的关系，app端用户必须登录，判断当前用户**是否已经关注该文章的作者、是否收藏了此文章、是否点赞了文章、是否不喜欢该文章等**

例：如果当前用户点赞了该文章，点赞按钮进行高亮，其他功能类似。

![image-20210727162449406](用户行为-需求.assets\image-20210727162449406.png)

#### 7.1)接口分析

|          | **说明**                              |
| -------- | ------------------------------------- |
| 接口路径 | /api/v1/article/load_article_behavior |
| 请求方式 | POST                                  |
| 参数     | ArticleInfoDto                        |
| 响应结果 | ResponseResult                        |

参数


```javascript
{
	"articleId": 0,     // 文章id
	"authorId": 0       // 作者id
}
```

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": { 
		"islike": true,           //是否点赞
		"isunlike": true,         //是否不喜欢
		"iscollection": false,    //是否收藏
		"isfollow": false         //是否关注
	}
}
```

#### 7.2)代码实现

在 leadnews-heima-article 服务中开发接口。

①：ArticleInfoDto

```java
package com.heima.model.article.dtos;

import lombok.Data;

@Data
public class ArticleInfoDto {
    // 设备ID
    Integer equipmentId;
    
    // 文章ID
    Long articleId;
    
    // 作者ID
    Integer authorId;
}
```

②：ApArticleService 中添加方法

```java
package com.heima.article.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.heima.model.article.dtos.ArticleDto;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.article.dtos.ArticleInfoDto;
import com.heima.model.article.pojos.ApArticle;
import com.heima.model.common.dtos.ResponseResult;

import java.io.IOException;

public interface ApArticleService extends IService<ApArticle> {

    /**
     * 加载文章详情 数据回显
     * @param dto
     * @return
     */
    ResponseResult loadArticleBehavior(ArticleInfoDto dto);
}
```

③：ApArticleServiceImpl 中添加方法

```java
@Autowired
private CacheService cacheService;

@Override
public ResponseResult loadArticleBehavior(ArticleInfoDto dto) {
    if (dto == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
    }

    boolean isfollow = false;
    boolean islike = false;
    boolean isunlike = false;
    boolean iscollection = false;

    ApUser user = AppThreadLocalUtil.getUser();
    if(user != null){
        //喜欢行为
        String likeBehaviorJson = (String) cacheService.hGet(BehaviorConstants.LIKE_BEHAVIOR + dto.getArticleId().toString(), user.getId().toString());
        if(StringUtils.isNotBlank(likeBehaviorJson)){
            islike = true;
        }

        //不喜欢的行为
        String unLikeBehaviorJson = (String) cacheService.hGet(BehaviorConstants.UN_LIKE_BEHAVIOR + dto.getArticleId().toString(), user.getId().toString());
        if(StringUtils.isNotBlank(unLikeBehaviorJson)){
            isunlike = true;
        }

        //是否收藏
        String collctionJson = (String) cacheService.hGet(BehaviorConstants.COLLECTION_BEHAVIOR+user.getId(),dto.getArticleId().toString());
        if(StringUtils.isNotBlank(collctionJson)){
            iscollection = true;
        }

        //是否关注
        Double score = cacheService.zScore(BehaviorConstants.APUSER_FOLLOW_RELATION + user.getId(), dto.getAuthorId().toString());
        System.out.println(score);
        if(score != null){
            isfollow = true;
        }

    }

    Map<String, Object> resultMap = new HashMap<>();
    resultMap.put("isfollow", isfollow);
    resultMap.put("islike", islike);
    resultMap.put("isunlike", isunlike);
    resultMap.put("iscollection", iscollection);

    return ResponseResult.okResult(resultMap);
}
```

④：ArticleInfoController

```java
package com.heima.article.controller.v1;

import com.heima.article.service.ApArticleService;
import com.heima.model.article.dtos.ArticleInfoDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/article")
public class ArticleInfoController {

    @Autowired
    private ApArticleService apArticleService;

    @PostMapping("/load_article_behavior")
    public ResponseResult loadArticleBehavior(@RequestBody ArticleInfoDto dto){
        return apArticleService.loadArticleBehavior(dto);
    }
}
```

