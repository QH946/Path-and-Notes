# Day04 自媒体文章-自动审核

### 1)自媒体文章自动审核流程

![image-20220713134454187](assets\image-20220713134454187.png)

**要做的事：**

- 提取文字
- 提取图片
- 接入第三方审核
- article服务保存文章接口（Feign）
- 回写News状态



### 2)内容安全第三方接口

![image-20220713134424008](assets\image-20220713134424008.png)

#### 2.1)概述

内容安全是识别服务，支持对图片、视频、文本、语音等对象进行多样化场景检测，有效降低内容违规风险。

目前很多平台都支持内容检测，如阿里云、腾讯云、百度AI、网易云等国内大型互联网公司都对外提供了API。

按照性能和收费来看，黑马头条项目使用的就是阿里云的内容安全接口，使用到了图片和文本的审核。

阿里云收费标准：[https://www.aliyun.com/price/product/?spm=a2c4g.11186623.2.10.4146401eg5oeu8#/lvwang/detail](https://www.aliyun.com/price/product/?spm=a2c4g.11186623.2.10.4146401eg5oeu8) 

#### 2.2)准备工作

您在使用内容检测API之前，需要先注册阿里云账号，添加Access Key并签约云盾内容安全。

**操作步骤**

1. 前往[阿里云官网](https://www.aliyun.com/)注册账号。如果已有注册账号，请跳过此步骤。

   进入阿里云首页后，如果没有阿里云的账户需要先进行注册，才可以进行登录。由于注册较为简单，课程和讲义不在进行体现（注册可以使用多种方式，如淘宝账号、支付宝账号、微博账号等...）。

   需要实名认证和活体认证。

2. 打开[云盾内容安全产品试用页面](https://common-buy.aliyun.com/?spm=a2c4g.11186623.0.0.6bc479db1xvRDn&commodityCode=cdi#/open)，单击**立即开通**，正式开通服务。

   ![image-20210504213452171](自媒体文章-自动审核.assets\image-20210504213452171.png)

   内容安全控制台

   ![image-20210504213510896](自媒体文章-自动审核.assets\image-20210504213510896.png)

   

3. ![image-20220713110608843](assets\image-20220713110608843.png)

3. ![image-20220713110632133](assets\image-20220713110632133.png)

3. 在[AccessKey管理页面](https://ak-console.aliyun.com/#/accesskey)管理您的AccessKeyID和AccessKeySecret。

   ![image-20210504213530641](自媒体文章-自动审核.assets\image-20210504213530641.png)

   管理自己的AccessKey,可以新建和删除AccessKey

   ![image-20210504213547219](自媒体文章-自动审核.assets\image-20210504213547219.png)

   查看自己的AccessKey，

   AccessKey默认是隐藏的，第一次申请的时候可以保存AccessKey，点击显示，通过验证手机号后也可以查看

   ![image-20210504213605004](自媒体文章-自动审核.assets\image-20210504213605004.png)

#### 2.3)文本内容审核接口

文本垃圾内容检测：https://help.aliyun.com/document_detail/70439.html?spm=a2c4g.11186623.6.659.35ac3db3l0wV5k 

![image-20210504213640667](自媒体文章-自动审核.assets\image-20210504213640667.png)

文本垃圾内容Java SDK: https://help.aliyun.com/document_detail/53427.html?spm=a2c4g.11186623.6.717.466d7544QbU8Lr 

#### 2.4)图片审核接口

图片垃圾内容检测：https://help.aliyun.com/document_detail/70292.html?spm=a2c4g.11186623.6.616.5d7d1e7f9vDRz4 

![image-20210504213719323](自媒体文章-自动审核.assets\image-20210504213719323.png)

图片垃圾内容Java SDK: https://help.aliyun.com/document_detail/53424.html?spm=a2c4g.11186623.6.715.c8f69b12ey35j4 

#### 2.5)项目集成

##### 2.5.1)封装成插件

①：拷贝“**资料\类\阿里云审核工具类\aliyun**”文件夹中的类到 heima-leadnews-common 模块下；

②：检查并添加依赖

```xml
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.1.1</version>
</dependency>

<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-green</artifactId>
    <version>3.4.1</version>
</dependency>
```

③：添加自动加载配置 META-INF/spring.factories

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.heima.green.aliyun.GreenImageScan,\
  com.heima.green.aliyun.GreenTextScan
```

##### 2.5.2)项目调用

①：在资料中找到 green-demo文件夹，加入到 heima-leadnews-test 模块下

②：添加依赖到 pom.xml

```xml
<dependency>
    <groupId>com.heima</groupId>
    <artifactId>heima-leadnews-common</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

③：添加配置到 application.yml

```yaml
aliyun:
 accessKeyId: LTAI5tCWHCcfvqQzu8k2oKmX
 secret: auoKUFsghimbfVQHpy7gtRyBkoR4vc
#aliyun.scenes=porn,terrorism,ad,qrcode,live,logo
 scenes: terrorism
```

④：创建控制器进行测试

```java
package com.heima.green.controller;

import com.heima.common.aliyun.GreenImageScan;
import com.heima.common.aliyun.GreenTextScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;

/**
 * @author itheima
 * @since 2022-07-11
 */
@RestController
public class TestController {

    @Autowired
    private GreenTextScan greenTextScan;

    @Autowired
    private GreenImageScan greenImageScan;

    @PostMapping("test/test")
    public Map testText(String text) throws Exception {
        return greenTextScan.greeTextScan(text);
    }

    @PostMapping("test/image")
    public Map testText(MultipartFile file) throws Exception {
        byte[] bytes = file.getBytes();
        List<byte[]> imageList = new ArrayList<>();
        imageList.add(bytes);

        return greenImageScan.imageScan(imageList);
    }

}
```

⑤：启动测试

```
POST http://localhost:8882/test/text?text=冰毒哈哈

POST http://localhost:8882/test/image
```

#### 2.6)测试结果

成功

```json
{
	"suggestion": "pass"
}
```

失败

```json
{
	"suggestion": "block"
}
```

不确定

```json
{
	"suggestion": "review"
}
```



### 3)app端文章保存接口

![image-20220713134107604](assets\image-20220713134107604.png)

#### 3.1)表结构说明

ap_article 文章信息表

![image-20210505005300287](自媒体文章-自动审核.assets\image-20210505005300287.png)

ap_article_config  文章配置表

![image-20210505005353286](自媒体文章-自动审核.assets\image-20210505005353286.png)

ap_article_content 文章内容表

![image-20210505005407987](自媒体文章-自动审核.assets\image-20210505005407987.png)

#### 3.2)分布式id

- 什么是分布式id：不重复、可排序、高效生成id值
- 分布式id解决什么问题：解决数据库分表分库后的主键冲突问题
- 如何生成分布式id：雪花算法

随着业务的增长，**文章表可能要占用很大的物理存储空间**，为了解决该问题，后期使用**数据库分片技术**。将一个数据库进行拆分，通过数据库中间件连接。如果数据库中该表选用ID自增策略，则可能产生重复的ID，此时应该使用分布式ID生成策略来生成ID。

![image-20210505005448995](自媒体文章-自动审核.assets\image-20210505005448995.png)

snowflake是Twitter开源的分布式ID生成算法，结果是一个long型的ID。

**每秒可以产生10万个，不重复，有顺序的主键。**

其核心思想是：使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是0

![image-20210505005509258](自媒体文章-自动审核.assets\image-20210505005509258.png)

文章端相关的表都使用雪花算法生成id,包括ap_article、 ap_article_config、 ap_article_content

mybatis-plus已经集成了雪花算法，完成以下两步即可在项目中集成雪花算法

第一：在实体类中的id上加入如下配置，指定类型为id_worker

```java
@TableId(value = "id", type = IdType.ID_WORKER)
private Long id;
```

第二：在application.yml文件中配置数据中心id和机器id

```yaml
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 设置别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: com.heima.model.article.pojos
  global-config:
    datacenter-id: 1
    workerId: 1
```

datacenter-id:数据中心id(取值范围：0-31)

workerId:机器id(取值范围：0-31)

#### 3.3)思路分析

在文章审核成功以后需要在app的article库中新增文章数据

1.保存文章信息 ap_article

2.保存文章配置信息 ap_article_config

3.保存文章内容 ap_article_content

实现思路：

![image-20210505005733405](自媒体文章-自动审核.assets\image-20210505005733405.png)

#### 3.4)环境准备

- 导入接口dto、导入Mapper、创建Feign接口、实现Feign接口

#### 3.5)feign接口

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/article/save |
| 请求方式 | POST                 |
| 参数     | ArticleDto（新）     |
| 响应结果 | ResponseResult       |

ArticleDto

```java
package com.heima.model.article.dtos;

import com.heima.model.article.pojos.ApArticle;
import lombok.Data;

@Data
public class ArticleDto extends ApArticle {
    /**
     * 文章内容
     */
    private String content;
}
```

成功：

```json
{
  "code": 200,
  "errorMessage" : "操作成功",
  "data":"1302864436297442242"
 }
```

失败：

```json
{
  "code":501,
  "errorMessage":"参数失效",
 }
```

```json
{
  "code":501,
  "errorMessage":"文章没有找到",
 }
```

#### 3.6)功能实现

①：在heima-leadnews-feign-api中定义文章端的接口

```json
package com.heima.apis.article;

import com.heima.model.article.dtos.ArticleDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient("leadnews-article")
public interface IArticleClient {

    @PostMapping("/api/v1/article/save")
    ResponseResult saveArticle(@RequestBody ArticleDto dto);
}
```

②：在heima-leadnews-article中实现该方法

```java
package com.heima.article.client;

import com.heima.apis.article.IArticleClient;
import com.heima.article.service.ApArticleService;
import com.heima.model.article.dtos.ArticleDto;
import com.heima.model.common.dtos.ResponseResult;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/article")
public class ArticleClient implements IArticleClient {

    @Autowired
    private ApArticleService apArticleService;

    @PostMapping("/save")
    @Override
    public ResponseResult saveArticle(@RequestBody ArticleDto dto) {
        return apArticleService.saveArticle(dto);
    }
}
```

③：在ApArticleService中新增方法

```java
/**
  * 保存app端相关文章
  * @param dto
  * @return
  */
ResponseResult saveArticle(ArticleDto dto);
```

④：实现类 ApArticleServiceImpl

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult saveArticle(ArticleDto dto) {
    // 1. dto判空
    if (dto == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }

    Long id = dto.getId();
    String content = dto.getContent();

    // 组装article表数据
    ApArticle apArticle = new ApArticle();
    BeanUtils.copyProperties(dto, apArticle);

    // 2. 判断dto里面的id是否存在
    if (id == null) {
        // 3. 不存在 -> 新增

        // 3.1 插入article表
        int insertResult = apArticleMapper.insert(apArticle);
        if (insertResult < 1) {
            return ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR, "文章新增失败");
        }

        // 3.2 插入article_config表
        ApArticleConfig config = new ApArticleConfig();
        config.setIsDown(false);
        config.setIsDelete(false);
        config.setIsForward(true);
        config.setIsComment(true);
        config.setArticleId(apArticle.getId());
        int configInsertResult = apArticleConfigMapper.insert(config);
        if (configInsertResult < 1) {
            log.warn("文章配置表插入失败");
            throw new RuntimeException("文章配置表插入失败");
        }

        // 3.3 插入article_content表
        ApArticleContent articleContent = new ApArticleContent();
        articleContent.setContent(content);
        articleContent.setArticleId(apArticle.getId());
        int contentInsertResult = apArticleContentMapper.insert(articleContent);
        if(contentInsertResult < 1) {
            log.warn("文章内容表插入失败");
            throw new RuntimeException("文章内容表插入失败");
        }

        return ResponseResult.okResult(apArticle.getId());
    } else {
        // 4. id存在 -> 修改

        // 4.1 修改article表
        int updateResult = apArticleMapper.updateById(apArticle);
        if(updateResult < 1) {
            return ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR, "文章修改失败");
        }

        // 4.2 修改article_content表
        ApArticleContent contentUpdate = new ApArticleContent();
        contentUpdate.setContent(content);

        LambdaQueryWrapper<ApArticleContent> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(ApArticleContent::getArticleId, id);

        int contentUpdateResult = apArticleContentMapper.update(contentUpdate, wrapper);
        if(contentUpdateResult < 1) {
            log.warn("文章内容表修改失败");
            throw new RuntimeException("文章内容表修改失败");
        }

        // 5. 将articleId返回回去
        return ResponseResult.okResult(id);
    }
}
```

#### 3.7)接口测试

在 heima-leadnews-wemedia 入口文件添加 @EnableFeignClients(clients = IArticleClient.class) 注解

在 heima-leadnews-wemedia 中编写junit单元测试

```java
package com.heima.wemedia;

import com.heima.apis.article.IArticleClient;
import com.heima.model.article.dtos.ArticleDto;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Date;

/**
 * @author itheima
 * @since 2022-07-11
 */
@SpringBootTest
public class ArticleTest {

    @Autowired
    private IArticleClient articleClient;

    @Test
    public void test() {
        ArticleDto dto = new ArticleDto();
        dto.setTitle("黑马头条项目背景22222222222222");
        dto.setAuthorId(1102L);
        dto.setLayout((short)1);
        dto.setLabels("黑马头条");
        dto.setPublishTime(new Date());
        dto.setImages("http://192.168.200.130:9000/leadnews/2021/04/26/5ddbdb5c68094ce393b08a47860da275.jpg");
        dto.setContent("22222222222222222黑马头条项目背景,黑马头条项目背景,黑马头条项目背景,黑马头条项目背景，黑马头条项目背景");

        articleClient.saveArticle(dto);
    }
}
```

**注意：如果碰到调用超时，可以在wemedia设置一下超时时间**

```yaml
feign:
  client:
    config:
      default:
        #不设置connectTimeout会导致readTimeout设置不生效
        connectTimeout: 3000
        readTimeout: 6000
```



### 4)自媒体文章自动审核功能实现

#### 4.1)表结构说明

wm_news 自媒体文章表

![image-20210505010728156](自媒体文章-自动审核.assets\image-20210505010728156.png)

**status字段：**0 草稿  1 待审核  **2 审核失败  3 人工审核**  4 人工审核通过  8 审核通过（待发布） **9 已发布**

#### 4.2)思路分析

![image-20220713134331750](assets\image-20220713134331750.png)



- 提取文字
- 提取图片
- 调用第三方审核文字
- 调用第三方审核图片
- 调用文章保存
- 回写文章id

#### 4.3)功能实现

①：在heima-leadnews-wemedia中的service新增接口

```java
package com.heima.wemedia.service;

public interface AutoScanService {
    /**
     * 自媒体文章审核
     *
     * @param newsId 自媒体文章id
     */
    void autoScanWmNews(Integer newsId) throws Exception;
}

```

②：实现类

```java
package com.heima.wemedia.service.impl;

import com.alibaba.fastjson.JSON;
import com.baomidou.mybatisplus.core.conditions.query.QueryWrapper;
import com.heima.apis.article.IArticleClient;
import com.heima.common.aliyun.GreenImageScan;
import com.heima.common.aliyun.GreenTextScan;
import com.heima.file.MinIoTemplate;
import com.heima.model.article.dtos.ArticleDto;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.wemedia.pojos.WmChannel;
import com.heima.model.wemedia.pojos.WmNews;
import com.heima.model.wemedia.pojos.WmSensitive;
import com.heima.model.wemedia.pojos.WmUser;
import com.heima.utils.OcrUtil;
import com.heima.utils.SensitiveWordUtil;
import com.heima.wemedia.mapper.WmSensitiveMapper;
import com.heima.wemedia.service.AutoScanService;
import com.heima.wemedia.service.WmChannelService;
import com.heima.wemedia.service.WmNewsService;
import com.heima.wemedia.service.WmUserService;
import lombok.extern.slf4j.Slf4j;
import net.sourceforge.tess4j.TesseractException;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.util.*;
import java.util.stream.Collectors;

/**
 * @author itheima
 * @since 2022-07-17
 */
@Service
@Slf4j
public class AutoScanServiceImpl implements AutoScanService {

    @Autowired
    private WmNewsMapper wmNewsMapper;
    
    @Autowired
    private WmChannelService wmChannelService;

    @Autowired
    private WmUserService wmUserService;

    @Autowired
    private WmSensitiveMapper wmSensitiveMapper;

    @Autowired
    private MinIoTemplate minIoTemplate;

    @Autowired
    private GreenTextScan greenTextScan;

    @Autowired
    private GreenImageScan greenImageScan;

    @Autowired
    private IArticleClient articleClient;
    
    @Override
    @Async
    public void autoScanWmNews(Long newsId) throws Exception {
        // 0. 入参判空
        if (newsId == null) {
            log.warn("newsId为空");
            return;
        }

        // 1. 查询新闻信息
        // 1.1 新闻为空 -> 响应失败
        WmNews wmNews = wmNewsMapper.selectById(newsId);
        if (wmNews == null) {
            log.warn("新闻" + newsId + "为空");
            return;
        }

        // 2. 提取新闻文字

        // 3. 提取新闻图片

        // 4. 调用文字审核接口
        // 4.1 审核失败 -> wmNews修改方法
        // 4.2 审核不确定 -> wmNews修改方法

        // 5. 调用图片审核接口
        // 5.1 审核失败 -> wmNews修改方法
        // 5.2 审核不确定 -> wmNews修改方法

        // 6. 远程调用ArticleClient保存文章
        // 6.1 判断响应结果是不是200


        // 7. 将articleid回写到wmNews表中
    }
}
```

##### 4.3.1)提取文字

①：核心方法

```java
/**
  * 提取新闻文字内容
  *
  * @param title   标题
  * @param content 内容
  * @return 组合后的字符串
  */
private String getText(String title, String content) {
    // 1.入参校验
    if (StringUtils.isBlank(content)) {
        return title;
    }

    // 2.提取content里面的文字内容
    List<Map> maps = JSON.parseArray(content, Map.class);
    if (CollectionUtils.isEmpty(maps)) {
        return title;
    }

    // 3.组装内容文字
    StringBuilder builder = new StringBuilder();
    for (Map item : maps) {
        if (CollectionUtils.isEmpty(item)) {
            continue;
        }

        // 3.1提取文字内容
        Object type = item.get("type");
        if ("text".equals(type)) {
            String value = (String) item.get("value");
            builder.append(value);
        }
    }

    // 4.将content文字内容和title进行组合
    builder.append("_").append(title);

    // 5.返回组合后的结果
    return builder.toString();
}
```

②：在主方法调用

```java
// 2. 提取新闻文字
String title = wmNews.getTitle();
String content = wmNews.getContent();
String text = getText(title, content);
```

##### 4.3.2)提取图片

①：核心方法（因为后续阿里云图片审核需要List<byte[]>结构，所以这里出参是这个类型）

```java
/**
  * 提取图片
  *
  * @param images  封面图
  * @param content 内容
  * @return 图片byte集合
  */
private List<byte[]> getImages(String images, String content) {
    // 1.声明总的图片集合
    List<String> finalList = new ArrayList<>();

    // 2.提取封面图 —> List<String>
    if (StringUtils.isNotBlank(images)) {
        String[] split = images.split(",");
        if (ArrayUtils.isNotEmpty(split)) {
            List<String> list = Arrays.asList(split);

            // 加入到大的集合中
            finalList.addAll(list);
        }
    }

    // 3.提取内容图 -> List<String>
    if (StringUtils.isNotBlank(content)) {
        List<Map> maps = JSON.parseArray(content, Map.class);
        if (!CollectionUtils.isEmpty(maps)) {
            for (Map item : maps) {
                if (CollectionUtils.isEmpty(item)) {
                    continue;
                }

                Object type = item.get("type");
                if ("image".equals(type)) {
                    String value = (String) item.get("value");
                    finalList.add(value);
                }
            }
        }
    }

    // 4.组装最后要输出的结果
    List<byte[]> result = new ArrayList<>();
    // 5.遍历图片集合，转成List<byte[]>
    if (!CollectionUtils.isEmpty(finalList)) {
        for (String url : finalList) {
            if (StringUtils.isEmpty(url)) {
                continue;
            }

            // 使用MinIO的download方法，根据图片地址下载图片byte[]数据
            byte[] bytes = minIoTemplate.downLoadFile(url);
            result.add(bytes);
        }
    }

    // 6.返回集合
    return result;
}
```

②：在主方法调用

```java
// 3. 提取新闻图片
String images = wmNews.getImages();
List<byte[]> imageList = getImages(images, content);
```

##### 4.3.3)阿里云审核

①：更新新闻状态方法

```java
/**
  * 更新新闻状态
  *
  * @param status    状态
  * @param reason    审核结果
  * @param articleId 文章id
  * @param newsId    新闻id（查询条件）
  * @return 是否成功
  */
private Boolean updateWmNews(Integer status, String reason, Long articleId, Integer newsId) {
    // 1.入参校验
    if (newsId == null || status == null) {
        log.warn("更新状态失败，入参缺失");
        return false;
    }
	
    // 2.构建数据
    WmNews wmNews = new WmNews();
    wmNews.setId(newsId);
    wmNews.setStatus(status.shortValue());
    wmNews.setReason(reason);
    if (articleId != null) {
        wmNews.setArticleId(articleId);
    }

    return wmNewsMapper.updateById(wmNews) > 0;
}
```

②：阿里云审核结果处理方法

```java
/**
  * 处理阿里云审核结果
  *
  * @param result    审核结果
  * @param newsId    新闻id
  * @return 是否通过
  */
private Boolean checkResult(Map result, Integer newsId) {
    if (CollectionUtils.isEmpty(result)) {
        log.warn("审核结果为空");
        return true;
    }

    String suggestion = (String) result.get("suggestion");
    if ("pass".equals(suggestion)) {
        return true;
    } else if ("block".equals(suggestion)) {
        // 更新wmNews状态
        updateWmNews(2, "阿里云审核未通过", null, newsId);
        return false;
    } else if ("review".equals(suggestion)) {
        updateWmNews(3, "转人工审核", null, newsId);
        return false;
    }

    return true;
}
```

③：主方法调用

```java
// 4. 调用文字审核接口
// 4.1 审核失败 -> wmNews修改方法
// 4.2 审核不确定 -> wmNews修改方法
Map textScan = greenTextScan.greeTextScan(text);
Boolean textScanResult = checkResult(textScan, newsId.intValue());
if (!textScanResult) {
    log.warn("文字审核失败");
    return;
}

// 5. 调用图片审核接口
// 5.1 审核失败 -> wmNews修改方法
// 5.2 审核不确定 -> wmNews修改方法
Map imageScan = greenImageScan.imageScan(imageList);
Boolean imageScanResult = checkResult(imageScan, newsId.intValue());
if (!imageScanResult) {
    log.warn("图片审核失败");
    return;
}
```

##### 4.3.4)远程调用保存文章

```java
// 6. 远程调用ArticleClient保存文章
// 6.1 判断响应结果是不是200
ArticleDto dto = wmNews2ArticleDto(wmNews);
ResponseResult responseResult = articleClient.saveArticle(dto);
if (responseResult == null || responseResult.getCode() != 200) {
    log.warn("文章保存失败");
    return;
}

Object data = responseResult.getData();
if (data == null) {
    log.warn("文章保存失败，响应内容为空");
    return;
}
```

##### 4.3.5)更新新闻回写文章id

```java
// 7. wmNews回写articleId，并修改状态为“通过”
updateWmNews(9, "审核通过", (Long) data, newsId.intValue());
```



### 5)发布文章提交审核集成

#### 5.1)同步调用与异步调用

同步：就是在发出一个调用时，在没有得到结果之前， 该调用就不返回（实时处理）

异步：调用在发出之后，这个调用就直接返回了，没有返回结果（分时处理）

![image-20210507160912993](自媒体文章-自动审核.assets\image-20210507160912993.png)

异步线程的方式审核文章

#### 5.2)开启异步线程调用

①：在自动审核的方法上加上@Async注解（标明要异步调用）

```java
@Override
@Async  //标明当前方法是一个异步方法
public void autoScanWmNews(Long newsId) throws Exception {
	//代码略
}
```

②：在文章发布成功后调用审核的方法

```java
@Autowired
private AutoScanService autoScanService;

@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult submit(WmNewsDto wmNewsDto) {

    //代码略

    // 8. 调用审核代码
    try {
        autoScanService.autoScanWmNews(wmNews.getId().longValue());
    } catch (Exception e) {
        e.printStackTrace();
    }

    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);

}
```

③：在自媒体引导类中使用@EnableAsync注解开启异步调用

```java
package com.heima.wemedia;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import com.heima.apis.article.IArticleClient;
import com.heima.apis.schedule.IScheduleClient;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.context.annotation.Bean;
import org.springframework.scheduling.annotation.EnableAsync;
import org.springframework.scheduling.annotation.EnableScheduling;

@SpringBootApplication
@EnableDiscoveryClient
@MapperScan("com.heima.wemedia.mapper")
@EnableFeignClients(clients = {IArticleClient.class, IScheduleClient.class})
@EnableAsync        // 开启异步调用
@EnableScheduling
public class WemediaApplication {

    public static void main(String[] args) {
        SpringApplication.run(WemediaApplication.class, args);
    }

    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```



### 6)文章审核功能-综合测试

①：为heima-leadnews-wemedia项目添加阿里云配置（需要改自己的accessKeyId和secret）

```yaml
aliyun:
  accessKeyId: LTAI5tBTSvoqQ7PStWjurZv1
  secret: e2sqr9vP4OInJTpIvXngodwutJRvQ2
  #aliyun.scenes=porn,terrorism,ad,qrcode,live,logo
  scenes: terrorism
```

②：在wemedia入口文件添加 @EnableFeignClients(clients = IArticleClient.class) 注解

③：检查环境是否开启了MiniIO

④：启动项目

- heima-leadnews-wemedia
- heima-leadnews-article
- heima-leadnews-wemedia-gateway

⑤：启动自媒体前端，进行测试

#### 6.1)测试情况列表

1，自媒体前端发布一篇正常的文章

   审核成功后，app端的article相关数据是否可以正常保存，自媒体文章状态和app端文章id是否回显

2，自媒体前端发布一篇包含敏感词的文章

   正常是审核失败， wm_news表中的状态是否改变，成功和失败原因正常保存

3，自媒体前端发布一篇包含敏感图片的文章

   正常是审核失败， wm_news表中的状态是否改变，成功和失败原因正常保存



### 7)文章详情-静态文件生成

#### **7.1)思路分析**

文章端创建app相关文章时，生成文章详情静态页上传到MinIO中

![image-20210709110852966](自媒体文章-自动审核.assets\image-20210709110852966.png)

#### 7.2)实现步骤

前置准备：

①：检查必要的文件

- resources/templates/article.ftl

②：检测配置 bootstrap.yml

```yaml
spring:  
  freemarker:
    cache: false  #关闭模板缓存，方便测试
    settings:
      template_update_delay: 0 #检查模板更新延迟时间，设置为0表示立即检查，如果时间大于0会有缓存不方便进行模板测试
    suffix: .ftl               #指定Freemarker模板文件的后缀名
minio:
  accessKey: minio
  secretKey: minio123
  bucket: leadnews
  endpoint: http://192.168.200.130:9000
  readPath: http://192.168.200.130:9000
```

代码实现：

①：ApArticleServiceImpl类中的saveArticle方法，添加生成HTML方法的调用（共两处）

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult saveArticle(ArticleDto dto) {
    // 1. dto判空

    // 2. 判断dto里面的id是否存在
    if (id == null) {
        // 3. 不存在 -> 新增

        // 3.1 插入article表

        // 3.2 插入article_config表

        // 3.3 插入article_content表
        
        // .. 以上代码略

        // 调用现成的方法，实现文章详情的生成和更新（加这步）
        generate(apArticle.getId());

        return ResponseResult.okResult(apArticle.getId());
    } else {
        // 4. id存在 -> 修改

        // 4.1 修改article表

        // 4.2 修改article_content表
        
        // .. 以上代码略

        // 调用详情生成（加这步）
        generate(id);

        // 5. 将articleId返回回去
        return ResponseResult.okResult(id);
    }
}
```

②：附上generate代码（Day02已经写过这个代码了）

注意添加@Async注解，使用异步方式执行该方法。

```java
@Autowired
private FreemarkerGenerator freemarkerGenerator;

@Autowired
private MinIoTemplate minIoTemplate;

@Override
@Async
public void generate(Long articleId) {
    // 1. 查询ApArticleContent表，获得里面content数据
    LambdaQueryWrapper<ApArticleContent> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(ApArticleContent::getArticleId, articleId);
    ApArticleContent apArticleContent = apArticleContentMapper.selectOne(wrapper);
    if (apArticleContent == null) {
        log.warn("文章内容不存在");
        return;
    }

    // 2. 创建一个数据集合Map，把上一步content数据赋值到map的content字段上
    Map<String, Object> map = new HashMap<>(1);
    map.put("content", JSON.parseArray(apArticleContent.getContent()));

    // 3. 调用Freemarker插件，生成HTML页面
    InputStream is = freemarkerGenerator.generate("article.ftl", map);

    // 4. 调用MinIO插件，上传HTML页面
    String url = minIoTemplate.uploadFile("", articleId + ".html", is, "html");

    ApArticle apArticle = apArticleMapper.selectById(articleId);
    if (apArticle == null) {
        log.warn("文章为空");
        return;
    }

    apArticle.setStaticUrl(url);

    // 5. 更新ApArticle，修改里面static_url字段
    apArticleMapper.updateById(apArticle);
}
```

③：文章微服务开启异步调用 @EnableAsync



### 8)新需求-自管理敏感词

#### 8.1)需求分析

文章审核功能已经交付了，文章也能正常发布审核。突然，产品经理过来说要开会。

会议的内容核心有以下内容：

- 文章审核不能过滤一些敏感词：

  私人侦探、针孔摄象、信用卡提现、广告代理、代开发票、刻章办、出售答案、小额贷款…

需要完成的功能：

需要自己维护一套敏感词，在文章审核的时候，需要验证文章是否包含这些敏感词

#### 8.2)敏感词-过滤

技术选型

| **方案**               | **说明**                     |
| ---------------------- | ---------------------------- |
| 数据库模糊查询         | 效率太低                     |
| String.indexOf("")查找 | 数据库量大的话也是比较慢     |
| 全文检索               | 分词再匹配                   |
| DFA算法                | 确定有穷自动机(一种数据结构) |

#### 8.3)DFA实现原理

DFA全称为：Deterministic Finite Automaton，即确定有穷自动机。

存储：一次性的把所有的敏感词存储到了多个map中，就是下图表示这种结构

敏感词：冰毒、大麻、大坏蛋

![image-20210524160517744](自媒体文章-自动审核.assets\image-20210524160517744.png)

检索的过程

![image-20210524160549596](自媒体文章-自动审核.assets\image-20210524160549596.png)



#### 8.4)自管理敏感词集成到文章审核中

①：创建敏感词表，导入资料中wm_sensitive到leadnews_wemedia库中

![image-20210524160611338](自媒体文章-自动审核.assets\image-20210524160611338.png)

```java
package com.heima.model.wemedia.pojos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

/**
 * <p>
 * 敏感词信息表
 * </p>
 *
 * @author itheima
 */
@Data
@TableName("wm_sensitive")
public class WmSensitive implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    @TableId(value = "id", type = IdType.AUTO)
    private Integer id;

    /**
     * 敏感词
     */
    @TableField("sensitives")
    private String sensitives;

    /**
     * 创建时间
     */
    @TableField("created_time")
    private Date createdTime;

}
```

②：拷贝对应的wm_sensitive的mapper到项目中

```java
package com.heima.wemedia.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.heima.model.wemedia.pojos.WmSensitive;
import org.apache.ibatis.annotations.Mapper;


@Mapper
public interface WmSensitiveMapper extends BaseMapper<WmSensitive> {
}
```

③：从资料中找到 SensitiveWordUtil.java 工具类，导入heima-leadenews-uitls 模块中

④：新增DFA文字检测方法

```java
/**
  * DFA算法检测敏感词
  *
  * @param text 待检测的文字
  * @return 是否通过检测（true为通过，false为不通过）
  */
private Boolean dfaCheck(String text) {
    if (StringUtils.isBlank(text)) {
        return true;
    }

    // 0.1 调用敏感词Mapper，查询所有敏感词
    QueryWrapper<WmSensitive> wrapper = new QueryWrapper<>();
    wrapper.select("sensitives");
    List<WmSensitive> wmSensitives = wmSensitiveMapper.selectList(wrapper);
    if (CollectionUtils.isEmpty(wmSensitives)) {
        return true;
    }

    List<String> words = wmSensitives.stream().filter(Objects::nonNull).map(WmSensitive::getSensitives).collect(Collectors.toList());
    if (CollectionUtils.isEmpty(words)) {
        return true;
    }

    // 0.2 DFA初始化
    SensitiveWordUtil.initMap(words);

    // 0.3 DFA敏感词匹配
    Map<String, Integer> result = SensitiveWordUtil.matchWords(text);

    // 0.4 判断结果
    return CollectionUtils.isEmpty(result);
}
```

⑤：在文章审核的代码中添加自管理敏感词审核（代码加载提取文字内容和提取图片内容的中间）

```java
//2. 提取文字 String getText(String title, String content)
//.....省略

// 使用DFA算法进行自建敏感词处理
Boolean dfaResult = dfaCheck(text);
if (!dfaResult) {
    // DFA检测不通过的话，需要更新一下文章状态
    updateWmNews(2, "DFA审核没通过", null, newsId.intValue());
    return;
}

// 3. 提取图片 List<byte[]> getImage(String images, String content)
//.....省略
```



### 9)新需求-图片识别文字审核敏感词

#### 9.1)需求分析

产品经理召集开会，文章审核功能已经交付了，文章也能正常发布审核。对于上次提出的自管理敏感词也很满意，这次会议核心的内容如下：

- 文章中包含的图片要识别文字，过滤掉图片文字的敏感词

![image-20210524161243572](自媒体文章-自动审核.assets\image-20210524161243572.png)



#### 9.2)图片文字识别

什么是OCR?

OCR （Optical Character Recognition，光学字符识别）是指电子设备（例如扫描仪或数码相机）检查纸上打印的字符，通过检测暗、亮的模式确定其形状，然后用字符识别方法将形状翻译成计算机文字的过程

| **方案**      | **说明**                                            |
| ------------- | --------------------------------------------------- |
| 百度OCR       | 收费                                                |
| Tesseract-OCR | Google维护的开源OCR引擎，支持Java，Python等语言调用 |
| Tess4J        | 封装了Tesseract-OCR  ，支持Java调用                 |

#### 9.3)Tess4j案例

①：在heima-leadnews-utils中，导入tess4j对应的依赖

```xml
<dependency>
    <groupId>net.sourceforge.tess4j</groupId>
    <artifactId>tess4j</artifactId>
    <version>4.1.1</version>
</dependency>
```

②：导入中文字体库， 把资料中的 chi_sim.traineddata 文件拷贝到项目根目录下 

![image-20210524161406081](自媒体文章-自动审核.assets\image-20210524161406081.png)

③：在heima-leadnews-utils中创建工具类，简单封装一下tess4j

```java
package com.heima.utils;

import net.sourceforge.tess4j.ITesseract;
import net.sourceforge.tess4j.Tesseract;
import net.sourceforge.tess4j.TesseractException;

import java.awt.image.BufferedImage;

public class OcrUtil {
    private final static String DATA_PATH = "字体库文件所在目录（注意不用带文件名）";
    private final static String LANGUAGE = "chi_sim";

    public static String doOcr(BufferedImage image) throws TesseractException {
        //创建Tesseract对象
        ITesseract tesseract = new Tesseract();
        //设置字体库路径
        tesseract.setDatapath(DATA_PATH);
        //中文识别
        tesseract.setLanguage(LANGUAGE);
        //执行ocr识别
        String result = tesseract.doOCR(image);
        //替换回车和tal键  使结果为一行
        result = result.replaceAll("[\\r\\n]", "-").replaceAll(" ", "");
        return result;
    }
}
```

④：编写测试接口

```java
import com.heima.utils.common.OcrUtil;
import net.sourceforge.tess4j.TesseractException;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import javax.imageio.ImageIO;
import java.awt.image.BufferedImage;
import java.io.ByteArrayInputStream;
import java.io.IOException;

@RestController
public class TestController {

    @PostMapping("test")
    public String testing(MultipartFile multipartFile) throws IOException, TesseractException {
        byte[] bytes = multipartFile.getBytes();
        ByteArrayInputStream image = new ByteArrayInputStream(bytes);
        BufferedImage read = ImageIO.read(image);

        String s = OcrUtil.doOCR(read);
        System.out.println(s);

        return s;
    }
}
```

#### 9.4)管理敏感词和图片文字识别集成到文章审核

在文章审核的代码中添加OCR文字提取（代码加在提取图内容之后，调用阿里云文字审核之前）

```java
// 使用OCR提取图片中的文字，并和之前提取的文字组合在一起
String textFromOcr = ocrCheck(imageList);
text = text + "_" + textFromOcr;
```

在AutoScanServiceImpl中添加OCR提取文字核心方法

```java
/**
  * 接收图片，提取图片里面的文字，并返回
  *
  * @param images 待检测的图片
  * @return 从图片中提取的文字
  */
private String ocrCheck(List<byte[]> images) throws IOException, TesseractException {
    if (CollectionUtils.isEmpty(images)) {
        return "";
    }

    StringBuilder builder = new StringBuilder();
    for (byte[] image : images) {
        // 调用ocr工具类，提取里面的文字内容
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(image);
        BufferedImage read = ImageIO.read(byteArrayInputStream);

        String text = OcrUtil.doOcr(read);

        // 将文字内容组合成一个字符串，最终返回
        builder.append("_").append(text);
    }

    return builder.toString();
}
```
