# 热点文章-实时计算

## 1 今日内容

### 1.1 定时计算与实时计算

![image-20210730201509223](新热文章-实时计算.assets\image-20210730201509223.png)



### 1.2 今日内容

kafkaStream

- 什么是流式计算

- kafkaStream概述

- kafkaStream入门案例

- Springboot集成kafkaStream



实时计算

- 用户行为发送消息

- kafkaStream聚合处理消息

- 更新文章行为数量

- 替换热点文章数据

## 2 实时流式计算

### 2.1 概念

一般流式计算会与批量计算相比较。在流式计算模型中，输入是持续的，可以认为在时间上是无界的，也就意味着，永远拿不到全量数据去做计算。同时，计算结果是持续输出的，也即计算结果在时间上也是无界的。流式计算一般对实时性要求较高，同时一般是先定义目标计算，然后数据到来之后将计算逻辑应用于数据。同时为了提高计算效率，往往尽可能采用增量计算代替全量计算。

![image-20210731090637590](新热文章-实时计算.assets\image-20210731090637590.png)

流式计算就相当于上图的右侧扶梯，是可以源源不断的产生数据，源源不断的接收数据，没有边界。

### 2.2 应用场景

- 日志分析

  网站的用户访问日志进行实时的分析，计算访问量，用户画像，留存率等等，实时的进行数据分析，帮助企业进行决策

- 大屏看板统计

  可以实时的查看网站注册数量，订单数量，购买数量，金额等。

- 公交实时数据

  可以随时更新公交车方位，计算多久到达站牌等

- 实时文章分值计算

  头条类文章的分值计算，通过用户的行为实时文章的分值，分值越高就越被推荐。

### 2.3 技术方案选型

- Hadoop

  ![image-20210731090700382](新热文章-实时计算.assets\image-20210731090700382.png)

- Apche Storm

  Storm 是一个分布式实时大数据处理系统，可以帮助我们方便地处理海量数据，具有高可靠、高容错、高扩展的特点。是流式框架，有很高的数据吞吐能力。

- Kafka Stream 

  可以轻松地将其嵌入任何Java应用程序中，并与用户为其流应用程序所拥有的任何现有打包，部署和操作工具集成。

## 3 Kafka Stream 

### 3.1 概述

Kafka Stream是Apache Kafka从0.10版本引入的一个新Feature。它是提供了对存储于Kafka内的数据进行流式处理和分析的功能。

Kafka Stream的特点如下：

- Kafka Stream提供了一个非常简单而轻量的Library，它可以非常方便地嵌入任意Java应用中，也可以任意方式打包和部署
- 除了Kafka外，无任何外部依赖
- 充分利用Kafka分区机制实现水平扩展和顺序性保证
- 通过可容错的state store实现高效的状态操作（如windowed join和aggregation）
- 支持正好一次处理语义
- 提供记录级的处理能力，从而实现毫秒级的低延迟
- 支持基于事件时间的窗口操作，并且可处理晚到的数据（late arrival of records）
- 同时提供底层的处理原语Processor（类似于Storm的spout和bolt），以及高层抽象的DSL（类似于Spark的map/group/reduce）



![image-20210730201706437](新热文章-实时计算.assets\image-20210730201706437.png)



### 3.2 Kafka Streams的关键概念

- **源处理器（Source Processor）**：源处理器是一个没有任何上游处理器的特殊类型的流处理器。它从一个或多个kafka主题生成输入流。通过消费这些主题的消息并将它们转发到下游处理器。
- **Sink处理器**：sink处理器是一个没有下游流处理器的特殊类型的流处理器。它接收上游流处理器的消息发送到一个指定的**Kafka主题**。

![image-20210731090727323](新热文章-实时计算.assets\image-20210731090727323.png)



### 3.3 KStream

（1）数据结构类似于map,如下图，key-value键值对

![image-20210731090746415](新热文章-实时计算.assets\image-20210731090746415.png)

（2）KStream

![image-20210730201817959](新热文章-实时计算.assets\image-20210730201817959.png)

**KStream**数据流（data stream），即是一段顺序的，可以无限长，不断更新的数据集。
数据流中比较常记录的是事件，这些事件可以是一次鼠标点击（click），一次交易，或是传感器记录的位置数据。

KStream负责抽象的，就是数据流。与Kafka自身topic中的数据一样，类似日志，每一次操作都是**向其中插入（insert）新数据。**

为了说明这一点，让我们想象一下以下两个数据记录正在发送到流中：

（“ alice”，1）->（“” alice“，3）

如果您的流处理应用是要总结每个用户的价值，它将返回`4`了`alice`。为什么？因为第二条数据记录将不被视为先前记录的更新。（insert）新数据

### 3.4 SpringBoot集成Kafka Stream

需求分析，求单词个数（word count）

导入资料中的 "kafka-stream-demo" 项目

![image-20210730201911566](新热文章-实时计算.assets\image-20210730201911566.png)

①：找到Kafka-demo项目，添加依赖

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
</dependency>
```

②：自定配置参数

```java
import lombok.Getter;
import lombok.Setter;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafkaStreams;
import org.springframework.kafka.annotation.KafkaStreamsDefaultConfiguration;
import org.springframework.kafka.config.KafkaStreamsConfiguration;

import java.util.HashMap;
import java.util.Map;

/**
 * 通过重新注册KafkaStreamsConfiguration对象，设置自定配置参数
 */

@Setter
@Getter
@Configuration
@EnableKafkaStreams
@ConfigurationProperties(prefix="kafka")
public class KafkaStreamConfig {
    private static final int MAX_MESSAGE_SIZE = 16* 1024 * 1024;
    private String hosts;
    private String group;
    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration defaultKafkaStreamsConfig() {
        Map<String, Object> props = new HashMap<>(6);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, hosts);
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, this.getGroup()+"_stream_aid");
        props.put(StreamsConfig.CLIENT_ID_CONFIG, this.getGroup()+"_stream_cid");
        props.put(StreamsConfig.RETRIES_CONFIG, 10);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        return new KafkaStreamsConfiguration(props);
    }
}
```

③：修改application.yml文件，在最下方添加自定义配置

```yaml
kafka:
  hosts: ${spring.kafka.bootstrap-servers}
  group: ${spring.kafka.consumer.group-id}
```

**注意：**这两个配置是直接读取之前Kafka的配置，所以原先的kafka配置要保留

```yaml
spring:
  kafka:
    bootstrap-servers: 124.223.167.78:9092
    consumer:
      group-id: ${spring.application.name}-test
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

④：新增配置类，创建KStream对象，进行聚合

```java
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.KStream;
import org.apache.kafka.streams.kstream.TimeWindows;
import org.apache.kafka.streams.kstream.ValueMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;
import java.util.Arrays;

@Configuration
@Slf4j
public class KafkaStreamHelloListener {

    @Bean
    public KStream<String,String> kStream(StreamsBuilder streamsBuilder){
        // 创建kstream对象，同时指定从那个topic中接收消息
        KStream<String, String> stream = streamsBuilder.stream("itcast-topic-input");
        // 流处理
        stream.flatMapValues(new ValueMapper<String, Iterable<String>>() {
            @Override
            public Iterable<String> apply(String value) {
                // 将字符串分割 转为List结构输出
                return Arrays.asList(value.split(" "));
            }
        })
                //根据value进行聚合分组
                .groupBy((key,value)->value)
                //聚合计算时间间隔
                .windowedBy(TimeWindows.of(Duration.ofSeconds(10)))
                //求单词的个数
                .count()
                .toStream()
                //处理后的结果转换为string字符串
                .map((key,value)->{
                    System.out.println("key:"+key+",value:"+value);
                    return new KeyValue<>(key.key().toString(),value.toString());
                })
                // 将处理后的消息转发出来
                .to("itcast-topic-out");
        return stream;
    }
}
```

⑤：修改控制器

```java
package com.heima.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("test")
public class TestController {

    @Autowired
    private KafkaTemplate<String, String> kafkaTemplate;

    @GetMapping("")
    public String test(String message) {
        kafkaTemplate.send("itcast-topic-input", message);
        return "ok";
    }
}
```

⑥：修改监听

```java
package com.heima.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class TestListener {

    @KafkaListener(topics = "itcast-topic-out")
    public void onMessage(String message) {
        System.out.println(message);
    }

}
```

测试：启动微服务，正常发送消息，可以正常接收到消息

频繁发送多条 "黑马程序员" 消息后，10秒得到如下结果：

```
key:[黑马程序员@1656224760000/1656224770000],value:14
```

其中key的结构为["消息内容"@开始时间/结束时间]，value为该消息出现的次数



## 3 app端热点文章计算

### 3.1 思路说明

![image-20210621235620854](新热文章-实时计算.assets\image-20210621235620854.png)

### 3.2 功能实现

![image-20220724202032585](assets\image-20220724202032585.png)

#### 3.2.1 用户行为（阅读量，评论，点赞，收藏）

发送消息，以阅读和点赞为例

①：在 **heima-leadnews-behavior** 微服务中集成kafka生产者配置

修改nacos，新增内容

```yaml
spring:
  application:
    name: leadnews-behavior
  kafka:
    bootstrap-servers: 192.168.200.130:9092
    producer:
      retries: 10
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

②：修改ApLikesBehaviorServiceImpl新增发送消息

定义消息发送封装类：UpdateArticleMess

```java
package com.heima.model.mess;

import lombok.Data;

@Data
public class UpdateArticleMess {

    /**
     * 修改文章的字段类型
      */
    private UpdateArticleType type;
    /**
     * 文章ID
     */
    private Long articleId;
    /**
     * 修改数据的增量，可为正负
     */
    private Integer add;

    public enum UpdateArticleType{
        COLLECTION,COMMENT,LIKES,VIEWS;
    }
}
```

ArticleVisitStreamMess

```java
package com.heima.model.mess.pojos;

import lombok.Data;

@Data
public class ArticleVisitStreamMess {
    /**
     * 文章id
     */
    private Long articleId;
    /**
     * 阅读
     */
    private int view;
    /**
     * 收藏
     */
    private int collect;
    /**
     * 评论
     */
    private int comment;
    /**
     * 点赞
     */
    private int like;
}
```

topic常量类：

```java
package com.heima.common.constants;

public class HotArticleConstants {

    public static final String HOT_ARTICLE_SCORE_TOPIC="hot.article.score.topic";
   
}
```

③：修改ApLikesBehaviorServiceImpl点赞方法 

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
        
        // 组装Kafka消息
        UpdateArticleMess mess = new UpdateArticleMess();
        mess.setArticleId(dto.getArticleId());
        mess.setType(UpdateArticleMess.UpdateArticleType.LIKES);

        // 点赞
        if (0 == operation) {
            Object o = cacheService.hGet(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString());
            if (o != null) {
                return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "已点赞");
            }

            cacheService.hPut(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString(), JSON.toJSONString(dto));
            mess.setAdd(1);
        }
        // 取消点赞
        else if (1 == operation) {
            cacheService.hDelete(BehaviorConstants.LIKE_BEHAVIOR + articleId, userId.toString());
            mess.setAdd(-1);
        }
        
        //发送消息，数据聚合
        kafkaTemplate.send(HotArticleConstants.HOT_ARTICLE_SCORE_TOPIC, JSON.toJSONString(mess));

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```

④：修改阅读行为的类ApReadBehaviorServiceImpl发送消息

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
        
        //发送消息，数据聚合
        UpdateArticleMess mess = new UpdateArticleMess();
        mess.setArticleId(dto.getArticleId());
        mess.setType(UpdateArticleMess.UpdateArticleType.VIEWS);
        mess.setAdd(1);
        kafkaTemplate.send(HotArticleConstants.HOT_ARTICLE_SCORE_TOPIC,JSON.toJSONString(mess));

        return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
    }
}
```



#### 3.2.2 使用kafkaStream实时接收消息，聚合内容

![image-20220728234258514](assets\image-20220728234258514.png)

①：在**leadnews-article**微服务中添加KafkaStream依赖

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-streams</artifactId>
</dependency>
```

②：添加配置类

```xml
import lombok.Getter;
import lombok.Setter;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.Serdes;
import org.apache.kafka.streams.StreamsConfig;
import org.apache.kafka.streams.Topology;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafkaStreams;
import org.springframework.kafka.annotation.KafkaStreamsDefaultConfiguration;
import org.springframework.kafka.config.KafkaStreamsConfiguration;

import java.util.HashMap;
import java.util.Map;

/**
 * 通过重新注册KafkaStreamsConfiguration对象，设置自定配置参数
 */

@Setter
@Getter
@Configuration
@EnableKafkaStreams
@ConfigurationProperties(prefix="kafka")
public class KafkaStreamConfig {
    private static final int MAX_MESSAGE_SIZE = 16* 1024 * 1024;
    private String hosts;
    private String group;
    @Bean(name = KafkaStreamsDefaultConfiguration.DEFAULT_STREAMS_CONFIG_BEAN_NAME)
    public KafkaStreamsConfiguration defaultKafkaStreamsConfig() {
        Map<String, Object> props = new HashMap<>(6);
        props.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, hosts);
        props.put(StreamsConfig.APPLICATION_ID_CONFIG, this.getGroup()+"_stream_aid");
        props.put(StreamsConfig.CLIENT_ID_CONFIG, this.getGroup()+"_stream_cid");
        props.put(StreamsConfig.RETRIES_CONFIG, 10);
        props.put(StreamsConfig.DEFAULT_KEY_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        props.put(StreamsConfig.DEFAULT_VALUE_SERDE_CLASS_CONFIG, Serdes.String().getClass());
        return new KafkaStreamsConfiguration(props);
    }
}
```

③：检查yml配置

```yaml
spring:
  kafka:
    bootstrap-servers: 124.223.167.78:9092
    consumer:
      group-id: ${spring.application.name}-test
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
kafka:
  hosts: ${spring.kafka.bootstrap-servers}
  group: ${spring.kafka.consumer.group-id}
```

④：定义实体类，用于聚合之后的分值封装

```java
package com.heima.model.article.mess;

import lombok.Data;

@Data
public class ArticleVisitStreamMess {
    /**
     * 文章id
     */
    private Long articleId;
    /**
     * 阅读
     */
    private int view;
    /**
     * 收藏
     */
    private int collect;
    /**
     * 评论
     */
    private int comment;
    /**
     * 点赞
     */
    private int like;
}
```

修改常量类：增加常量

```java
package com.heima.common.constans;

public class HotArticleConstants {

    public static final String HOT_ARTICLE_SCORE_TOPIC="hot.article.score.topic";
    public static final String HOT_ARTICLE_INCR_HANDLE_TOPIC="hot.article.incr.handle.topic";
}
```

⑤：编写stream基础类

```java
import com.alibaba.fastjson.JSON;
import com.heima.model.mess.pojos.ArticleVisitStreamMess;
import com.heima.model.mess.pojos.UpdateArticleMess;
import lombok.extern.slf4j.Slf4j;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;
import java.util.Arrays;

@Configuration
@Slf4j
public class KafkaStreamHelloListener {

    @Bean
    public KStream<String, String> kStream(StreamsBuilder streamsBuilder) {
        KStream<String, String> stream = streamsBuilder.stream("itcast-topic-in");
        stream.to("itcast-topic-out");

        return stream;
    }
}
```

⑥：定义stream，接收消息并聚合

```java
package com.heima.article.stream;

import com.alibaba.fastjson.JSON;
import com.heima.model.mess.pojos.ArticleVisitStreamMess;
import com.heima.model.mess.pojos.UpdateArticleMess;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.apache.kafka.streams.KeyValue;
import org.apache.kafka.streams.StreamsBuilder;
import org.apache.kafka.streams.kstream.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Duration;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@Configuration
@Slf4j
public class KafkaStreamHelloListener {

    @Bean
    public KStream<String, String> kStream(StreamsBuilder streamsBuilder) {
        KStream<String, String> stream = streamsBuilder.stream("itcast-topic-in");

        stream.map((key, value) -> {
            if (StringUtils.isEmpty(value)) {
                return new KeyValue<>(key, value);
            }

            UpdateArticleMess mess = JSON.parseObject(value, UpdateArticleMess.class);
            return new KeyValue<>(mess.getArticleId().toString(), mess.getType() + ":" + mess.getAdd());
        })
                .groupBy((key, value) -> key)
                .windowedBy(TimeWindows.of(Duration.ofSeconds(5)))
                .aggregate(
                        new Initializer<String>() {
                            @Override
                            public String apply() {
                                return "COLLECTION:0,COMMENT:0,LIKES:0,VIEWS:0";
                            }
                        },

                        new Aggregator<String, String, String>() {
                            @Override
                            public String apply(String key, String value, String initData) {
                                String[] split = value.split(":");
                                String type = split[0];
                                String add = split[1];

                                Map<String, Integer> map = string2Map(initData);
                                Integer newValue = map.get(type) + Integer.parseInt(add);
                                map.put(type, newValue);

                                return map2String(map);
                            }
                        },

                        Materialized.as("hot-atricle-stream-count-001")
                )
                .toStream()
                .map((key, value) -> {
                    return new KeyValue<>(key.key().toString(), string2JsonString(value, Long.parseLong(key.key())));
                })
                .to("itcast-topic-out");

        return stream;
    }

    /**
     * 字符串转实体类
     *
     * @param str       COLLECTION:0,COMMENT:0,LIKES:0,VIEWS:0
     * @param articleId 文章id
     * @return ArticleVisitStreamMess
     */
    private String string2JsonString(String str, Long articleId) {
        Map<String, Integer> map = string2Map(str);

        ArticleVisitStreamMess mess = new ArticleVisitStreamMess();
        mess.setArticleId(articleId);
        mess.setCollect(map.get("COLLECTION"));
        mess.setComment(map.get("COMMENT"));
        mess.setLike(map.get("LIKES"));
        mess.setView(map.get("VIEWS"));

        return JSON.toJSONString(mess);
    }

    /**
     * Map转字符串
     *
     * @param map map
     * @return COLLECTION:0,COMMENT:0,LIKES:0,VIEWS:0
     */
    private String map2String(Map<String, Integer> map) {
        Set<String> keySet = map.keySet();
        StringBuilder builder = new StringBuilder();
        for (String key : keySet) {
            builder.append(key).append(":").append(map.get(key));
            builder.append(",");
        }

        String result = builder.toString();
        return result.substring(0, result.length() - 1);
    }

    /**
     * 字符串转Map结构
     *
     * @param str COLLECTION:0,COMMENT:0,LIKES:0,VIEWS:0
     * @return Map
     */
    private Map<String, Integer> string2Map(String str) {
        Map<String, Integer> map = new HashMap<>();

        String[] initDataArray = str.split(",");
        for (String data : initDataArray) {

            String[] dataArray = data.split(":");
            String dataKey = dataArray[0];
            String dataValue = dataArray[1];

            map.put(dataKey, Integer.parseInt(dataValue));
        }

        return map;
    }
}
```



#### 3.2.3 重新计算文章的分值，更新到数据库和缓存中

![image-20220730003053175](assets\image-20220730003053175.png)

①在ApArticleService添加方法，用于更新数据库中的文章分值

```java
/**
  * 更新文章的分值  同时更新缓存中的热点文章数据
  * @param mess
  */
void updateScore(ArticleVisitStreamMess mess);
```

实现类方法

```java
@Override
public void updateScore(ArticleVisitStreamMess mess) {
    // 更新数据库
    ApArticle article = updateArticle(mess);
    if (article == null) {
        return;
    }

    // 计算文章分值
    Integer score = compute(article);

    // 替换当前文章对应频道的热点数据
    replaceToRedis(article, score, ArticleConstants.HOT_ARTICLE_FIRST_PAGE + article.getChannelId());

    // 替换推荐频道热点数据
    replaceToRedis(article, score, ArticleConstants.HOT_ARTICLE_FIRST_PAGE + ArticleConstants.DEFAULT_TAG);
}

/**
     * 替换缓存数据
     *
     * @param article 文章内容
     * @param score   分值
     * @param key     缓存键
     */
private void replaceToRedis(ApArticle article, Integer score, String key) {
    // 根据key获取缓存数据
    String cache = cacheService.get(key);
    if (StringUtils.isEmpty(cache)) {
        return;
    }

    List<HotArticleVo> hotArticleVos = JSON.parseArray(cache, HotArticleVo.class);
    if (CollectionUtils.isEmpty(hotArticleVos)) {
        return;
    }

    boolean flag = false;

    // 如果当前内容存在于缓存，则替换缓存分值
    for (HotArticleVo hotArticleVo : hotArticleVos) {
        if (hotArticleVo == null) {
            continue;
        }

        if (hotArticleVo.getId().equals(article.getId())) {
            hotArticleVo.setScore(score);
            flag = true;
            break;
        }
    }

    //如果缓存中不存在，查询缓存中分值最小的一条数据，进行分值的比较，如果当前文章的分值大于缓存中的数据，就替换
    if (flag) {
        if (hotArticleVos.size() >= 30) {
            // 排序
            hotArticleVos = hotArticleVos.stream().sorted(Comparator.comparing(HotArticleVo::getScore).reversed()).collect(Collectors.toList());
            // 取最后一条
            HotArticleVo lastHot = hotArticleVos.get(hotArticleVos.size() - 1);
            // 如果最后一条分值比当前文章分值小，则替换
            if (lastHot.getScore() < score) {
                hotArticleVos.remove(lastHot);
                HotArticleVo hot = new HotArticleVo();
                BeanUtils.copyProperties(article, hot);
                hot.setScore(score);
                hotArticleVos.add(hot);
            }
        } else {
            HotArticleVo hot = new HotArticleVo();
            BeanUtils.copyProperties(article, hot);
            hot.setScore(score);
            hotArticleVos.add(hot);
        }
    }

    // 重新排序
    hotArticleVos = hotArticleVos.stream().sorted(Comparator.comparing(HotArticleVo::getScore).reversed()).collect(Collectors.toList());

    // 缓存到redis
    cacheService.set(key, JSON.toJSONString(hotArticleVos));
}

/**
     * 更新文章MySQL数据
     *
     * @param mess messmess
     * @return ApArticle
     */
private ApArticle updateArticle(ArticleVisitStreamMess mess) {
    ApArticle apArticle = apArticleMapper.selectById(mess.getArticleId());
    if (apArticle == null) {
        log.warn("文章id" + mess.getArticleId() + "不存在");
        return null;
    }

    Integer views = apArticle.getViews();
    Integer likes = apArticle.getLikes();
    Integer comment = apArticle.getComment();
    Integer collection = apArticle.getCollection();

    apArticle.setComment(comment == null ? mess.getComment() : comment + mess.getComment());
    apArticle.setLikes(likes == null ? mess.getLike() : likes + mess.getLike());
    apArticle.setCollection(collection == null ? mess.getCollect() : collection + mess.getCollect());
    apArticle.setViews(views == null ? mess.getView() : views + mess.getView());

    apArticleMapper.updateById(apArticle);

    return apArticle;
}

/**
     * 计算文章评分
     *
     * @param apArticle apArticle
     * @return Integer
     */
private Integer compute(ApArticle apArticle) {
    Integer views = apArticle.getViews();
    Integer likes = apArticle.getLikes();
    Integer comment = apArticle.getComment();
    Integer collection = apArticle.getCollection();

    Integer score = 0;
    if (views != null) {
        score += views;
    }

    if (likes != null) {
        score += likes * 3;
    }

    if (comment != null) {
        score += comment * 5;
    }

    if (collection != null) {
        score += collection * 8;
    }

    return score;
}
```

②定义监听，接收聚合之后的数据，文章的分值重新进行计算

```java
package com.heima.article.listener;

import com.alibaba.fastjson.JSON;
import com.heima.article.service.ApArticleService;
import com.heima.common.constants.HotArticleConstants;
import com.heima.model.mess.ArticleVisitStreamMess;
import lombok.extern.slf4j.Slf4j;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class ArticleIncrHandleListener {

    @Autowired
    private ApArticleService apArticleService;

    @KafkaListener(topics = HotArticleConstants.HOT_ARTICLE_INCR_HANDLE_TOPIC)
    public void onMessage(String mess){
        if(StringUtils.isNotBlank(mess)){
            ArticleVisitStreamMess articleVisitStreamMess = JSON.parseObject(mess, ArticleVisitStreamMess.class);
            apArticleService.updateScore(articleVisitStreamMess);

        }
    }
}
```



