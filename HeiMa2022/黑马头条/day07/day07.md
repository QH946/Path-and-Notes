app端文章搜索

## 1) 今日内容介绍

### 1.1)App端搜索-效果图

![image-20210709140539138](app端文章搜索.assets\image-20210709140539138.png)

### 1.2)今日内容

- 文章搜索

  - ElasticSearch环境搭建

  - 索引库创建

  - 文章搜索多条件复合查询

  - 索引数据同步

- 搜索历史记录

  - Mongodb环境搭建

  - 异步保存搜索历史

  - 查看搜索历史列表

  - 删除搜索历史

- 联想词查询

  - 联想词的来源

  - 联想词功能实现

## 2) 搭建ElasticSearch环境

### 2.1) 拉取镜像

```shell
docker pull elasticsearch:7.4.0
```

### 2.2) 创建容器

```shell
docker run -id \
--name elasticsearch \
-d --restart=always \
-d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-p 9200:9200 \
-p 9300:9300 \
-v /home/docker/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
-e "discovery.type=single-node" \
elasticsearch:7.4.0
```

### 2.3) 配置中文分词器 ik

因为在创建elasticsearch容器的时候，映射了目录，所以可以在宿主机上进行配置ik中文分词器

在去选择ik分词器的时候，需要与elasticsearch的版本好对应上

把资料中的`elasticsearch-analysis-ik-7.4.0.zip`上传到服务器上,放到对应目录（plugins）解压

```shell
#切换目录
cd /home/docker/elasticsearch/plugins
#新建目录
mkdir analysis-ik
cd analysis-ik
#root根目录中拷贝文件
mv elasticsearch-analysis-ik-7.4.0.zip /home/docker/elasticsearch/plugins/analysis-ik
#解压文件
cd /home/docker/elasticsearch/plugins/analysis-ik
unzip elasticsearch-analysis-ik-7.4.0.zip
```

### 2.4) 使用ApiPost测试

![image-20220716194902288](assets\image-20220716194902288.png)



POST http://你ip:9200/_analyze

请求参数

```json
{
    "analyzer": "ik_max_word",
    "text": "欢迎来到黑马程序员学习！"
}
```

响应结果（当你执行完请求，得到以下结果后，证明你的ik分词器安装成功）

```json
{
	"tokens": [
		{
			"token": "欢迎",
			"start_offset": 0,
			"end_offset": 2,
			"type": "CN_WORD",
			"position": 0
		},
		{
			"token": "迎来",
			"start_offset": 1,
			"end_offset": 3,
			"type": "CN_WORD",
			"position": 1
		},
		{
			"token": "来到",
			"start_offset": 2,
			"end_offset": 4,
			"type": "CN_WORD",
			"position": 2
		},
		{
			"token": "黑马",
			"start_offset": 4,
			"end_offset": 6,
			"type": "CN_WORD",
			"position": 3
		},
		{
			"token": "程序员",
			"start_offset": 6,
			"end_offset": 9,
			"type": "CN_WORD",
			"position": 4
		},
		{
			"token": "程序",
			"start_offset": 6,
			"end_offset": 8,
			"type": "CN_WORD",
			"position": 5
		},
		{
			"token": "员",
			"start_offset": 8,
			"end_offset": 9,
			"type": "CN_CHAR",
			"position": 6
		},
		{
			"token": "学习",
			"start_offset": 9,
			"end_offset": 11,
			"type": "CN_WORD",
			"position": 7
		}
	]
}
```



## 3)app端文章搜索

### 3.1)需求分析

- 用户输入关键可搜索文章列表

- 关键词高亮显示

- 文章列表展示与home展示一样，当用户点击某一篇文章，可查看文章详情

![image-20210709141502366](app端文章搜索.assets\image-20210709141502366.png)



### 3.2)思路分析

为了加快检索的效率，在查询的时候不会直接从数据库中查询文章，需要在elasticsearch中进行高速检索。

![image-20210709141558811](app端文章搜索.assets\image-20210709141558811.png)

### 3.3)创建索引和映射

使用ApiPost添加映射，app_info_article指的就是ES中的表名（索引名）

put请求 ： http://192.168.200.130:9200/app_info_article

```json
{
    "mappings":{
        "properties":{
            "id":{
                "type":"long"
            },
            "publishTime":{
                "type":"date"
            },
            "layout":{
                "type":"integer"
            },
            "images":{
                "type":"keyword",
                "index": false
            },
            "staticUrl":{
                "type":"keyword",
                "index": false
            },
            "authorId": {
                "type": "long"
            },
            "authorName": {
                "type": "text"
            },
            "title":{
                "type":"text",
                "analyzer":"ik_smart"
            },
            "content":{
                "type":"text",
                "analyzer":"ik_smart"
            }
        }
    }
}
```

GET请求查询映射：http://192.168.200.130:9200/app_info_article

DELETE请求，删除索引及映射：http://192.168.200.130:9200/app_info_article

GET请求，查询所有文档：http://192.168.200.130:9200/app_info_article/_search

### 3.4)数据初始化到索引库

①：导入 "资料\elasticsearch" 下的 "es-demo" 到heima-leadnews-test工程下

![image-20210709142215818](app端文章搜索.assets\image-20210709142215818.png)

②：查询所有的文章信息，批量导入到es索引库中

```java
package com.heima.es;

import com.alibaba.fastjson.JSON;
import com.heima.es.mapper.ApArticleMapper;
import com.heima.es.pojo.SearchArticleVo;
import org.elasticsearch.action.bulk.BulkRequest;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.List;


@SpringBootTest
@RunWith(SpringRunner.class)
public class ApArticleTest {

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private RestHighLevelClient restHighLevelClient;


    /**
     * 注意：数据量的导入，如果数据量过大，需要分页导入
     * @throws Exception
     */
    @Test
    public void init() throws Exception {

        //1.查询所有符合条件的文章数据
        List<SearchArticleVo> searchArticleVos = apArticleMapper.loadArticleList();

        //2.批量导入到es索引库

        BulkRequest bulkRequest = new BulkRequest("app_info_article");

        for (SearchArticleVo searchArticleVo : searchArticleVos) {

            IndexRequest indexRequest = new IndexRequest().id(searchArticleVo.getId().toString())
                    .source(JSON.toJSONString(searchArticleVo), XContentType.JSON);

            //批量添加数据
            bulkRequest.add(indexRequest);

        }
        restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);

    }

}
```

③：ApiPost查询所有的es中数据；

GET请求： http://192.168.200.130:9200/app_info_article/_search

![image-20210709142339535](app端文章搜索.assets\image-20210709142339535.png)



### 3.5)文章搜索功能实现

#### 3.5.1)插件集成

①：将资料中的 "elasticsearch\工具类\ElasticSearchTemplate.java" 文件复制到 heima-leadnews-common 目录下

②：添加依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

③：spring.factories中配置自动装配

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.heima.common.search.ElasticSearchTemplate
```

#### 3.5.2)搭建搜索微服务

①：导入 "资料\elasticsearch\heima-leadnews-search" 到 "heima-leadnews-service" 模块下

![image-20210709142616797](app端文章搜索.assets\image-20210709142616797.png)

②：依赖，因为heima-leadnews-service中已经添加了utils依赖，utils依赖下已经包含了common模块，所以不需要添加额外依赖

③：导入文件：资料中的elasticsearch\实体类

④：nacos配置中心leadnews-search

```yaml
spring:
  autoconfigure:
    exclude: org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
  elasticsearch:
    rest:
      uris: 192.168.200.130:9200
logging:
  level:
    org:
      elasticsearch: debug
```

⑤：app网关添加转发配置

```yaml
#搜索微服务
- id: leadnews-search
 uri: lb://leadnews-search
 predicates:
   - Path=/search/**
 filters:
   - StripPrefix= 1
```

#### 3.5.2)接口分析

|          | **说明**                       |
| -------- | ------------------------------ |
| 接口路径 | /api/v1/article/search/search/ |
| 请求方式 | POST                           |
| 参数     | UserSearchDto                  |
| 响应结果 | ResponseResult                 |

UserSearchDto

```json
{
    "searchWords": "",
    "pageNum": 1,
    "pageSize": 10,
    "minBehotTime": ""
}
```

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": [
		{
			"layout": 1,
			"publishTime": 1655218578000,
			"images": "group1/M00/00/00/wKjIgl9V2CqAZe18AAOoOOsvWPc041.png",
			"authorName": "admin",
			"h_title": "什么是Java<font style='color: red; font-size: inherit;'>语言</font>",
			"id": 1302862387124125698,
			"staticUrl": "http://114.116.122.120:9000/leadnews/2022/07/16/1302862387124125698.html",
			"authorId": 4,
			"title": "什么是Java语言",
			"content": "..."
		},
		{
			"layout": 1,
			"publishTime": 1599490862000,
			"images": "group1/M00/00/00/wKjIgl9V2n6AArZsAAGMmaPdt7w502.png",
			"authorName": "admin",
			"h_title": "Java<font style='color: red; font-size: inherit;'>语言</font>跨平台原理",
			"id": 1302864436297482242,
			"staticUrl": "http://101.35.3.166:9000/news/2022/06/29/1541989067795771394",
			"authorId": 4,
			"title": "Java语言跨平台原理",
			"content": "..."
		}
	]
}
```



#### 3.5.3)前置准备

导入UserSearchDto.java

```java
package com.heima.model.search.dtos;

import lombok.Data;

import java.util.Date;


@Data
public class UserSearchDto {

    /**
    * 搜索关键字
    */
    String searchWords;
    /**
    * 当前页
    */
    int pageNum;
    /**
    * 分页条数
    */
    int pageSize;
    /**
    * 最小时间
    */
    Date minBehotTime;

    public int getFromIndex(){
        if(this.pageNum<1)return 0;
        if(this.pageSize<1) this.pageSize = 10;
        return this.pageSize * (pageNum-1);
    }
}
```

#### 3.5.4)代码实现

①：创建业务层接口：ApArticleSearchService

```java
package com.heima.search.service;

import com.heima.model.search.dtos.UserSearchDto;
import com.heima.model.common.dtos.ResponseResult;

import java.io.IOException;

public interface ArticleSearchService {

    /**
     ES文章分页搜索
     @return
     */
    ResponseResult search(UserSearchDto userSearchDto) throws IOException;
}
```

②：实现类

```java
package com.heima.search.service;

import com.heima.common.search.ElasticSearchTemplate;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.search.dtos.UserSearchDto;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.index.query.*;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.elasticsearch.search.sort.SortOrder;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;

import java.io.IOException;
import java.util.*;

@Service
@Slf4j
public class ArticleSearchServiceImpl implements ArticleSearchService {

    @Autowired
    private ElasticSearchTemplate elasticsearchTemplate;

    @Override
    public ResponseResult search(UserSearchDto dto) throws IOException {
        if (dto == null || StringUtils.isBlank(dto.getSearchWords())) {
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
        }

        // 提取入参
        int pageNum = dto.getPageNum();
        int pageSize = dto.getPageSize();
        String searchWords = dto.getSearchWords();
        Date minBehotTime = dto.getMinBehotTime();

        // 关键字查询
        QueryStringQueryBuilder stringQueryBuilder = elasticsearchTemplate.getStringQueryBuilder(searchWords, Operator.OR, "title", "content");

        // 时间范围查询
        Map<String, Object> rangeCondition = new HashMap<>(1);
        rangeCondition.put("lt", minBehotTime.getTime());
        RangeQueryBuilder rangeQueryBuilder = elasticsearchTemplate.getRangeBuilder("publishTime", rangeCondition);

        // 布尔查询
        Map<String, AbstractQueryBuilder> boolCondition = new HashMap<>();
        boolCondition.put("must", stringQueryBuilder);
        boolCondition.put("filter", rangeQueryBuilder);
        BoolQueryBuilder boolQueryBuilder = elasticsearchTemplate.getBoolQueryBuilder(boolCondition);

        // 高亮
        HighlightBuilder highlightBuilder = elasticsearchTemplate.getHighlightBuilder("title", "<font style='color: red; font-size: inherit;'>", "</font>");

        // 执行查询
        SearchResponse response = elasticsearchTemplate.search("app_info_article", boolQueryBuilder, highlightBuilder, pageNum, pageSize, "publishTime", SortOrder.DESC);

        // 提取结果
        SearchHit[] hits = response.getHits().getHits();

        List<Map<String, Object>> list = new ArrayList<>();
        for (SearchHit hit : hits) {
            Map<String, Object> sourceAsMap = hit.getSourceAsMap();

            Map<String, HighlightField> highlightFields = hit.getHighlightFields();

            // 处理高亮显示
            if (!CollectionUtils.isEmpty(highlightFields)) {
                Text[] titles = highlightFields.get("title").getFragments();
                String join = StringUtils.join(titles, ",");
                sourceAsMap.put("h_title", join);
            } else {
                sourceAsMap.put("h_title", sourceAsMap.get("title"));
            }

            list.add(sourceAsMap);
        }

        return ResponseResult.okResult(list);
    }
}
```

③：新建控制器ArticleSearchController

```java
package com.heima.search.controller.v1;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.search.dtos.UserSearchDto;
import com.heima.search.service.ArticleSearchService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;

@RestController
@RequestMapping("/api/v1/article/search")
public class ArticleSearchController {

    @Autowired
    private ArticleSearchService articleSearchService;

    @PostMapping("/search")
    public ResponseResult search(@RequestBody UserSearchDto dto) throws IOException {
        return articleSearchService.search(dto);
    }
}
```

#### 3.5.5)测试

需要在app的网关中添加搜索微服务的路由配置。



启动项目进行测试，至少要启动文章微服务，用户微服务，搜索微服务，app网关微服务，app前端工程

### 3.6)文章自动审核构建索引

#### 3.6.1)思路分析

![image-20220719101853676](assets\image-20220719101853676.png)

#### 3.6.2)文章微服务发送消息

①：把SearchArticleVo放到model工程下（文件在资料\elasticsearch\实体类）

```java
package com.heima.model.search.vos;

import lombok.Data;

import java.util.Date;

@Data
public class SearchArticleVo {

    // 文章id
    private Long id;
    // 文章标题
    private String title;
    // 文章发布时间
    private Date publishTime;
    // 文章布局
    private Integer layout;
    // 封面
    private String images;
    // 作者id
    private Long authorId;
    // 作者名词
    private String authorName;
    //静态url
    private String staticUrl;
    //文章内容
    private String content;
}
```

②：文章服务中ApArticleServiceImpl中添加创建索引方法

```java
@Autowired
private KafkaTemplate<String,String> kafkaTemplate;

/**
  * 送消息，创建索引
  * @param apArticle
  * @param content
  * @param path
  */
private void createArticleESIndex(ApArticle apArticle, String content, String path) {
    SearchArticleVo vo = new SearchArticleVo();
    BeanUtils.copyProperties(apArticle,vo);
    vo.setContent(content);
    vo.setStaticUrl(path);

    kafkaTemplate.send("article.es.sync.topic", JSON.toJSONString(vo));
}
```

③：文章服务中ApArticleServiceImpl找到generate方法，并添加步骤6，调用上面的createArticleESIndex方法

```java
package com.heima.article.service;

import com.alibaba.fastjson.JSON;
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.heima.article.mapper.ApArticleConfigMapper;
import com.heima.article.mapper.ApArticleContentMapper;
import com.heima.article.mapper.ApArticleMapper;
import com.heima.article.service.ApArticleService;
import com.heima.common.freemarker.FreemarkerGenerator;
import com.heima.file.MinIoTemplate;
import com.heima.model.article.dtos.ArticleDto;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.article.pojos.ApArticle;
import com.heima.model.article.pojos.ApArticleConfig;
import com.heima.model.article.pojos.ApArticleContent;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.io.InputStream;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@Service
public class ApArticleServiceImpl extends ServiceImpl<ApArticleMapper, ApArticle> implements ApArticleService {

    @Autowired
    private ApArticleMapper apArticleMapper;

    @Autowired
    private ApArticleContentMapper apArticleContentMapper;

    @Autowired
    private ApArticleConfigMapper apArticleConfigMapper;

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
        
        // 6. 发送消息，创建索引
        createArticleESIndex(apArticle, content, path);
    }
}
```

④：文章微服务集成kafka发送消息

在文章微服务的nacos的配置中心添加如下配置

```yaml
kafka:
    bootstrap-servers: 192.168.200.130:9092
    producer:
      retries: 10
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

#### 3.6.3)搜索微服务接收消息并创建索引

①：搜索微服务中添加kafka的配置，nacos配置如下

```yaml
spring:
  kafka:
    bootstrap-servers: 192.168.200.130:9092
    consumer:
      group-id: ${spring.application.name}
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

②：定义监听接收消息，保存索引数据

```java
package com.heima.search.listener;

import com.alibaba.fastjson.JSON;
import com.heima.common.constants.ArticleConstants;
import com.heima.model.search.vos.SearchArticleVo;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.elasticsearch.action.index.IndexRequest;
import org.elasticsearch.client.RequestOptions;
import org.elasticsearch.client.RestHighLevelClient;
import org.elasticsearch.common.xcontent.XContentType;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

import java.io.IOException;

@Component
@Slf4j
public class SyncArticleListener {

    @Autowired
    private RestHighLevelClient restHighLevelClient;

    @KafkaListener(topics = "article.es.sync.topic")
    public void onMessage(String message){
        if(StringUtils.isNotBlank(message)){
        	return;
        }
        
        log.info("SyncArticleListener,message={}",message);

        SearchArticleVo searchArticleVo = JSON.parseObject(message, SearchArticleVo.class);
        IndexRequest indexRequest = new IndexRequest("app_info_article");
        indexRequest.id(searchArticleVo.getId().toString());
        indexRequest.source(message, XContentType.JSON);
        try {
            restHighLevelClient.index(indexRequest, RequestOptions.DEFAULT);
        } catch (IOException e) {
            e.printStackTrace();
            log.error("sync es error={}",e);
        }
    }
}
```



## 4)app端搜索-搜索记录

### 4.1)需求分析

- 展示用户的搜索记录10条，按照搜索关键词的时间倒序
- 可以删除搜索记录
- 保存历史记录

![1587366878895](app端文章搜索.assets\1587366878895.png)



### 4.2)数据存储说明

用户的搜索记录，需要给每一个用户都保存一份，数据量较大，要求加载速度快，通常这样的数据存储到mongodb更合适，不建议直接存储到关系型数据库中。

![image-20210709153428259](app端文章搜索.assets\image-20210709153428259.png)

### 4.3)MongoDB安装

检查MongoDB是否启动

```shelll
docker ps

// 查看是否有一个叫做 mongo-service的容器
```



拉取镜像

```
docker pull mongo
```

创建容器

```
docker run -di --name mongo-service --restart=always -p 27017:27017 -v ~/data/mongodata:/data mongo
```

云服务器，记得把白名单打开27017



### 4.4)项目集成

①：导入 "资料中\mongodb\mongo测试类\MongoTest.java" 项目到 "heima-leadnews-search" 的单元测试目录中

②：为 "heima-leadnews-search" 项目添加mongo依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

③：远程nacos配置文件添加mongo依赖

```yaml
spring:
  data:
    mongodb:
      host: 192.168.200.130
      port: 27017
      database: leadnews-history
```

④：导入实体类："资料\mongodb\实体类" 导入到 "heima-leadnews-search" 中

**注意：是导入到"heima-leadnews-search"项目不是"model"**

⑤：改造网关过滤器，添加用户id传递

```java
// 上面的代码省略...

// 获取用户id
Integer id = (Integer) claimsBody.get("id");

// 将userId添加到请求头上
ServerHttpRequest requestNew = request.mutate().headers(new Consumer<HttpHeaders>() {
    @Override
    public void accept(HttpHeaders httpHeaders) {
        httpHeaders.add("userId", id.toString());
    }
}).build();

// 刷新请求头
exchange.mutate().request(requestNew).build();

// 成功 -> 放行
return chain.filter(exchange);
```

⑥：执行单元测试类方法



### 4.4)保存搜索记录



#### 4.4.1)实现思路

用户输入关键字进行搜索的异步记录关键字（在ElasticSearch搜索接口中添加该方法）

- 获得查询关键字
- 检测关键字是否已存在
- 已存在则更新时间
- 不存在则写入

![image-20220719154821788](assets\image-20220719154821788.png)

MongoDB内搜索记录的存储结构：ap_user_search

```json
[
    {
      "userId" : 4,
      "keyword" : "java",
      "createdTime" : {
        "$date" : "2022-07-19T06:26:28.237Z"
      }
    }
]
```

#### 4.4.2)代码实现

①：用户搜索记录对应的集合，对应实体类：

```java
package com.heima.search.pojos;

import lombok.Data;
import org.springframework.data.mongodb.core.mapping.Document;

import java.io.Serializable;
import java.util.Date;

/**
 * <p>
 * APP用户搜索信息表
 * </p>
 * @author itheima
 */
@Data
@Document("ap_user_search")
public class ApUserSearch implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 主键
     */
    private String id;

    /**
     * 用户ID
     */
    private Integer userId;

    /**
     * 搜索词
     */
    private String keyword;

    /**
     * 创建时间
     */
    private Date createdTime;

}
```

②：创建ApUserSearchService新增insert方法

```java
public interface ApUserSearchService {

    /**
     * 保存用户搜索历史记录
     * @param keyword
     * @param userId
     */
    void insert(String keyword,Integer userId);
}
```

③：在ApUserSearchServiceImpl中实现方法

```java
@Service
@Slf4j
public class ApUserSearchServiceImpl implements ApUserSearchService {

    @Autowired
    private MongoTemplate mongoTemplate;
    /**
     * 保存用户搜索历史记录
     * @param keyword
     * @param userId
     */
    @Override
    @Async
    public void insert(String keyword, Integer userId) {
        //1.查询当前用户的搜索关键词
        Query query = Query.query(Criteria.where("userId").is(userId).and("keyword").is(keyword));
        ApUserSearch apUserSearch = mongoTemplate.findOne(query, ApUserSearch.class);

        //2.存在 更新创建时间
        if(apUserSearch != null){
            apUserSearch.setCreatedTime(new Date());
            mongoTemplate.save(apUserSearch);
            return;
        }

        //3.不存在，判断当前历史记录总数量是否超过10
        apUserSearch = new ApUserSearch();
        apUserSearch.setUserId(userId);
        apUserSearch.setKeyword(keyword);
        apUserSearch.setCreatedTime(new Date());

        Query query1 = Query.query(Criteria.where("userId").is(userId));
        query1.with(Sort.by(Sort.Direction.DESC,"createdTime"));
        List<ApUserSearch> apUserSearchList = mongoTemplate.find(query1, ApUserSearch.class);

        if(apUserSearchList == null || apUserSearchList.size() < 10){
            mongoTemplate.save(apUserSearch);
        }else {
            ApUserSearch lastUserSearch = apUserSearchList.get(apUserSearchList.size() - 1);
            mongoTemplate.findAndReplace(Query.query(Criteria.where("id").is(lastUserSearch.getId())),apUserSearch);
        }
    }
}
```

④：在ArticleSearchService的search方法中调用保存历史记录

⑤：保存历史记录中开启异步调用，添加注解@Async

⑥：在搜索微服务引导类上开启异步调用

![image-20210709154841113](app端文章搜索.assets\image-20210709154841113.png)

⑦：测试，搜索后查看结果



### 4.5)加载搜索记录列表

#### 4.5.1)接口分析

按照当前用户，按照时间倒序查询，**注意：只加载最新的10条记录**

|          | **说明**             |
| -------- | -------------------- |
| 接口路径 | /api/v1/history/load |
| 请求方式 | POST                 |
| 参数     | 无                   |
| 响应结果 | ResponseResult       |

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": [
		{
			"id": "62d64e943533b246e7d92b6a",
			"userId": 4,
			"keyword": "java",
			"createdTime": "2022-07-19T06:26:28.237+00:00"
		}
	]
}
```



#### 4.5.2)代码实现

①：在ApUserSearchService中新增方法

```java
/**
  * 查询搜索历史
  *
  * @return ResponseResult
  */
ResponseResult findUserSearch();
```

②：在ApUserSearchServiceImpl中实现方法

```java
@Override
public ResponseResult findUserSearch() {
    ApUser user = AppThreadLocalUtil.getUser();
    if (user == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.NEED_LOGIN);
    }

    Integer id = user.getId();

    Query query = new Query();
    query.addCriteria(Criteria.where("userId").is(id));
    query.with(Sort.by("createTime").descending());
    query.limit(10);
    List<ApUserSearch> records = mongoTemplate.find(query, ApUserSearch.class);

    return ResponseResult.okResult(records);
}
```

③：控制器调用

```java
package com.heima.search.controller.v1;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.search.service.ApUserSearchService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * APP用户搜索信息表 前端控制器
 *
 * @author itheima
 */
@RestController
@RequestMapping("/api/v1/history")
public class ApUserSearchController{

    @Autowired
    private ApUserSearchService apUserSearchService;

    @PostMapping("/load")
    public ResponseResult findUserSearch() {
        return apUserSearchService.findUserSearch();
    }

}
```

#### 4.5.3)测试

打开app的搜索页面，可以查看搜索记录列表



### 4.6)删除搜索记录

#### 4.6.1)接口分析

按照搜索历史id删除

|          | **说明**            |
| -------- | ------------------- |
| 接口路径 | /api/v1/history/del |
| 请求方式 | POST                |
| 参数     | HistorySearchDto    |
| 响应结果 | ResponseResult      |

请求参数

```json
{
    "id": "1231233"
}
```

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": "SUCCESS"
}
```



#### 4.6.2)代码实现

①：HistorySearchDto

```java
@Data
public class HistorySearchDto {
    /**
    * 接收搜索历史记录id
    */
    String id;
}
```

②：在ApUserSearchService中新增方法

```java
/**
  * 删除搜索历史
  * @param historySearchDto
  * @return
  */
ResponseResult delUserSearch(HistorySearchDto historySearchDto);
```

③：在ApUserSearchServiceImpl中实现方法

```java
@Override
public ResponseResult delUserSearch(HistorySearchDto dto) {
    if(dto == null || dto.getId() == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_REQUIRE);
    }
    
    ApUser user = AppThreadLocalUtil.getUser();
    if (user == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.NEED_LOGIN);
    }

    Integer id = user.getId();

    Query query = new Query();
    query.addCriteria(Criteria.where("userId").is(id).and("id").is(dto.getId()));
    mongoTemplate.remove(query, ApUserSearch.class);

    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

④：修改ApUserSearchController，新增方法

```java
@PostMapping("/del")
public ResponseResult delUserSearch(@RequestBody HistorySearchDto historySearchDto) {
    return apUserSearchService.delUserSearch(historySearchDto);
}
```

#### 4.6.3)测试

打开app可以删除搜索记录



## 5)app端搜索-关键字联想词

### 5.1)需求分析

根据用户输入的关键字展示联想词

![1587366921085](app端文章搜索.assets\1587366921085.png)



### 5.2)搜索词-数据来源

通常是网上搜索频率比较高的一些词，通常在企业中有两部分来源：

第一：自己维护搜索词

通过分析用户搜索频率较高的词，按照排名作为搜索词

第二：第三方获取

关键词规划师（百度）、5118、爱站网

![image-20210709160036983](app端文章搜索.assets\image-20210709160036983.png)

### 5.3)导入数据

以MongoDB Compass为例，软件安装包在 "资料\mongodb\mongodb-compass-1.32.4-win32-x64.exe"

①：点击左下角

![image-20220719161848524](assets\image-20220719161848524.png)

②：将 "资料\mongodb\mongo数据脚本\leadnews-history.sql" 中的命令复制进来执行

![image-20220719162149794](assets\image-20220719162149794.png)

③：联想词库的存储结构为 ap_associate_words

```json
{
  "associateWords": "黑马程序员",
  "createdTime": {
    "$date": {
      "$numberLong": "1622999107000"
    }
  }
}
```



### 5.4)功能实现

#### 5.4.1)接口分析

|          | **说明**                 |
| -------- | ------------------------ |
| 接口路径 | /api/v1/associate/search |
| 请求方式 | POST                     |
| 参数     | UserSearchDto            |
| 响应结果 | ResponseResult           |

参数

```json
{
	"searchWords": "java",
	"pageSize": 10
}
```

响应结果

```json
{
	"host": null,
	"code": 200,
	"errorMessage": "操作成功",
	"data": [
		{
			"id": "60bd0043aab4f3022ce4a119",
			"associateWords": "黑马程序员",
			"createdTime": "2021-06-06T17:05:07.000+00:00"
		}
	]
}
```

注意：涉及模糊匹配 `regex(".*?\\关键字.*")`



#### 5.4.2)代码实现

①：对应实体类

```java
package com.heima.search.pojos;

import lombok.Data;
import org.springframework.data.mongodb.core.mapping.Document;

import java.io.Serializable;
import java.util.Date;

/**
 * <p>
 * 联想词表
 * </p>
 *
 * @author itheima
 */
@Data
@Document("ap_associate_words")
public class ApAssociateWords implements Serializable {

    private static final long serialVersionUID = 1L;

    private String id;

    /**
     * 联想词
     */
    private String associateWords;

    /**
     * 创建时间
     */
    private Date createdTime;

}
```

②：在ApAssociateWordsService中添加方法

```java
package com.heima.search.service;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.search.dtos.UserSearchDto;

/**
 * <p>
 * 联想词表 服务类
 * </p>
 *
 * @author itheima
 */
public interface ApAssociateWordsService {

    /**
     联想词
     @param userSearchDto
     @return
     */
    ResponseResult findAssociate(UserSearchDto userSearchDto);

}
```

③：在ApAssociateWordsServiceImpl中实现方法

```java
package com.heima.search.service.impl;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.common.enums.AppHttpCodeEnum;
import com.heima.model.search.dtos.UserSearchDto;
import com.heima.search.pojos.ApAssociateWords;
import com.heima.search.service.ApAssociateWordsService;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.data.mongodb.core.query.Criteria;
import org.springframework.data.mongodb.core.query.Query;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * @Description:
 * @Version: V1.0
 */
@Service
public class ApAssociateWordsServiceImpl implements ApAssociateWordsService {

    @Autowired
    MongoTemplate mongoTemplate;

    /**
     * 联想词
     * @param userSearchDto
     * @return
     */
    @Override
    public ResponseResult findAssociate(UserSearchDto userSearchDto) {
        //1 参数检查
        if(userSearchDto == null || StringUtils.isBlank(userSearchDto.getSearchWords())){
            return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID);
        }
        //分页检查
        if (userSearchDto.getPageSize() > 20) {
            userSearchDto.setPageSize(20);
        }

        //3 执行查询 模糊查询
        Query query = Query.query(Criteria.where("associateWords").regex(".*?\\" + userSearchDto.getSearchWords() + ".*"));
        query.limit(userSearchDto.getPageSize());
        List<ApAssociateWords> wordsList = mongoTemplate.find(query, ApAssociateWords.class);

        return ResponseResult.okResult(wordsList);
    }
}
```

④：新建联想词控制器

```java
package com.heima.search.controller.v1;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.search.dtos.UserSearchDto;
import com.heima.search.service.ApAssociateWordsService;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * <p>
 * 联想词表 前端控制器
 * </p>
 * @author itheima
 */
@Slf4j
@RestController
@RequestMapping("/api/v1/associate")
public class ApAssociateWordsController{

    @Autowired
    private ApAssociateWordsService apAssociateWordsService;

    @PostMapping("/search")
    public ResponseResult findAssociate(@RequestBody UserSearchDto userSearchDto) {
        return apAssociateWordsService.findAssociate(userSearchDto);
    }
}
```

#### 5.3.3)测试

同样，打开前端联调测试效果



