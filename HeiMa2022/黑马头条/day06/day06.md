## kafka及异步通知文章上下架

### 1)自媒体文章上下架

需求分析

![image-20210525180731705](kafka及异步通知文章上下架.assets\image-20210525180731705.png)

![image-20210525180757907](kafka及异步通知文章上下架.assets\image-20210525180757907.png)



### 2)kafka概述

#### 2.1)消息中间件对比                              

| 特性       | ActiveMQ                               | RabbitMQ                   | RocketMQ                 | Kafka                                    |
| ---------- | -------------------------------------- | -------------------------- | ------------------------ | ---------------------------------------- |
| 开发语言   | java                                   | erlang                     | java                     | scala                                    |
| 单机吞吐量 | 万级                                   | 万级                       | 10万级                   | 100万级                                  |
| 时效性     | ms                                     | us                         | ms                       | ms级以内                                 |
| 可用性     | 高（主从）                             | 高（主从）                 | 非常高（分布式）         | 非常高（分布式）                         |
| 功能特性   | 成熟的产品、较全的文档、各种协议支持好 | 并发能力强、性能好、延迟低 | MQ功能比较完善，扩展性佳 | 只支持主要的MQ功能，主要应用于大数据领域 |

消息中间件对比-选择建议

| **消息中间件** | **建议**                                                     |
| -------------- | ------------------------------------------------------------ |
| Kafka          | 追求高吞吐量，适合产生大量数据的互联网服务的数据收集业务     |
| RocketMQ       | 可靠性要求很高的金融互联网领域,稳定性高，经历了多次阿里双11考验 |
| RabbitMQ       | 性能较好，社区活跃度高，数据量没有那么大，优先选择功能比较完备的RabbitMQ |

#### 2.2)kafka介绍

Kafka 是一个分布式流媒体平台,类似于消息队列或企业消息传递系统。kafka官网：http://kafka.apache.org/  

![image-20210525181028436](kafka及异步通知文章上下架.assets\image-20210525181028436.png)

#### 2.3)名词解释

![image-20210525181100793](kafka及异步通知文章上下架.assets\image-20210525181100793.png)

- producer：发布消息的对象称之为主题生产者（Kafka topic producer）

- topic：Kafka将消息分门别类，每一类的消息称之为一个主题（Topic）

- consumer：订阅消息并处理发布的消息的对象称之为主题消费者（consumers）

- broker：已发布的消息保存在一组服务器中，称之为Kafka集群。集群中的每一个服务器都是一个代理（Broker）。 消费者可以订阅一个或多个主题（topic），并从Broker拉数据，从而消费这些已发布的消息。

### 3)kafka安装配置

Kafka对于zookeeper是强依赖，保存kafka相关的节点数据，所以安装Kafka之前必须先安装zookeeper

①：查看kafka是否正常启动

进入虚拟机后，执行**docker ps**，检查列表是否有kafka和zookeeper容器。如果有，就不需要安装了。

![image-20220723085348254](assets\image-20220723085348254.png)

②：如果看不到以上两条记录，就执行以下命令

```shell
docker start kafka zookeeper

docker ps
```

**安装**

①：Docker安装zookeeper

下载镜像：

```shell
docker pull zookeeper:3.4.14
```

创建容器

```shell
docker run -d --name zookeeper -p 2181:2181 zookeeper:3.4.14
```

②：Docker安装kafka

下载镜像：

```shell
docker pull wurstmeister/kafka:2.12-2.3.1
```

创建容器

```shell
docker run -d --name kafka \
--env KAFKA_ADVERTISED_HOST_NAME=192.168.200.130 \
--env KAFKA_ZOOKEEPER_CONNECT=192.168.200.130:2181 \
--env KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://192.168.200.130:9092 \
--env KAFKA_LISTENERS=PLAINTEXT://0.0.0.0:9092 \
--env KAFKA_HEAP_OPTS="-Xmx256M -Xms256M" \
--net=host wurstmeister/kafka:2.12-2.3.1
```



### 4)springboot集成kafka

#### 4.1)入门案例

导入资料中的“kafka-demo”项目到 heima-leadnews-test

①：导入spring-kafka依赖信息

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- kafka -->
    <dependency>
        <groupId>org.springframework.kafka</groupId>
        <artifactId>spring-kafka</artifactId>
        <exclusions>
            <exclusion>
                <groupId>org.apache.kafka</groupId>
                <artifactId>kafka-clients</artifactId>
            </exclusion>
        </exclusions>
    </dependency>
    <dependency>
        <groupId>org.apache.kafka</groupId>
        <artifactId>kafka-clients</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.54</version>
    </dependency>
</dependencies>
```

②：在resources下创建文件application.yml

```yaml
server:
  port: 9992
spring:
  application:
    name: kafka-demo
  kafka:
    bootstrap-servers: 192.168.200.130:9092
```

③：消息生产者

```java
package com.heima.kafka.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    @GetMapping("/hello")
    public String hello(){
        kafkaTemplate.send("itcast-topic","黑马程序员");
        return "ok";
    }
}
```

④：消息消费者

```java
package com.heima.kafka.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

@Component
public class HelloListener {

    @KafkaListener(topics = "itcast-topic")
    public void onMessage(String message){
        if(!StringUtils.isEmpty(message)){
            System.out.println(message);
        }

    }
}
```

#### 4.2)生产者发送类型及参数配置

##### 4.1.1单消费者（Queue模型）

一个生产者，多个消费者，但只有第一个消费者能收到消息。

![image-20220716152349441](assets\image-20220716152349441.png)

消息生产者

```java
package com.heima.kafka.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    @GetMapping("/hello")
    public String hello(){
        kafkaTemplate.send("itcast-topic","黑马程序员");
        return "ok";
    }
}
```

消息消费者（编写了两个消费者方法，启动测试后发现只有第一个方法能收到消息）

```java
package com.heima.kafka.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;
import org.springframework.util.StringUtils;

@Component
public class HelloListener {

    @KafkaListener(topics = "itcast-topic")
    public void onMessage(String message){
        System.out.println("消费者1");
        System.out.println(message);
    }
    
    @KafkaListener(topics = "itcast-topic")
    public void onMessage2(String message){
        System.out.println("消费者2");
        System.out.println(message);
    }
}
```



##### 4.1.2)多消费者（发布订阅模型）

一个生产者，多个消费者，多个消费者都要收到消息

![image-20220716152416581](assets\image-20220716152416581.png)

多个消费者，只要保证不存在同一个群组中，就可以同时收到生产者发送的消息。

```java
package com.heima.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

@Component
public class TestListener {

    /**
     * 消费者1
     *
     * @param message 收到的消息
     */
    @KafkaListener(topics = "abc-topic", groupId = "group101")
    public void listener(String message) {
        System.out.println(message);
    }

    /**
     * 消费者2
     *
     * @param message 收到的消息
     */
    @KafkaListener(topics = "abc-topic", groupId = "group102")
    public void listener2(String message) {
        System.out.println(message);
    }

}
```

##### 4.2.1)同步发送

将消息成功发送给Kafka后，接收KafKa给的回执；

在消息安全性要求比较高的场景中使用，防止消息丢失；

```java
package com.heima.kafka.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

    @Autowired
    private KafkaTemplate<String,String> kafkaTemplate;

    @GetMapping("/hello")
    public String hello(){
        kafkaTemplate.send("itcast-topic","黑马程序员").get();
        return "ok";
    }
}
```

##### 4.2.3)常用配置

通用配置

| 配置                            | 参考值                                    | 说明                          |
| ------------------------------- | ----------------------------------------- | ----------------------------- |
| spring.kafka.bootstrap-servers  | 192.168.200.130:9200,192.168.200.130:9300 | kafka服务ip (多个地址用,隔开) |
| spring.kafaka.listener.ack-mode | manual                                    | 开启消费者回执功能            |

生产者配置

| 配置                             | 参考值                                                 | 说明                                                         |
| -------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------ |
| spring.producer.retries          | 10                                                     | 失败重试次数                                                 |
| spring.producer.value-serializer | org.apache.kafka.common.serialization.StringSerializer | 消息value序列化                                              |
| spring.producer.key-serializer   | org.apache.kafka.common.serialization.StringSerializer | 消息key序列化                                                |
| spring.producer.compression-type | gzip                                                   | 数据压缩的类型。默认为空（不压缩）。有效的值有 none，gzip，snappy, 或  lz4。 |

| **压缩算法** | **说明**                                                     |
| ------------ | ------------------------------------------------------------ |
| snappy       | 占用较少的  CPU，  却能提供较好的性能和相当可观的压缩比， 如果看重性能和网络带宽，建议采用 |
| lz4          | 占用较少的 CPU， 压缩和解压缩速度较快，压缩比也很客观        |
| gzip         | 占用较多的  CPU，但会提供更高的压缩比，网络带宽有限，可以使用这种算法 |

消费者配置

| 配置                                     | 参考值                                                   | 说明               |
| ---------------------------------------- | -------------------------------------------------------- | ------------------ |
| spring.consumer.group-id                 | ${spring.application.name}-test                          | 默认分组id         |
| spring.consumer.value-deserializer       | org.apache.kafka.common.serialization.StringDeserializer | 消息value反序列化  |
| spring.consumer.key-deserializer         | org.apache.kafka.common.serialization.StringDeserializer | 消息key反序列化    |
| spring.kafka.consumer.enable-auto-commit | true(默认)                                               | 是否自动提交偏移量 |



### 5)消费者的消息特点及偏移量提交方式

#### 5.1)概述

Broker消息是有序的，每条消息有自己的编号；当消费者消费一条消息后，会给Broker发送回执，Broker会记录消息消费进度，这个消费进度就称为**偏移量**。

![image-20220716164553455](assets\image-20220716164553455.png)

#### 5.2)偏移量提交方式

提交偏移量的方式有两种，分别是自动提交偏移量和手动提交

**方式一：自动提交偏移量**

当 spring.kafka.consumer.enable-auto-commit 被设置为true，提交方式就是让消费者自动提交偏移量，每隔5秒消费者会自动把从poll()方法接收的最大偏移量提交上去。

**方式二：手动提交** 

当 spring.kafka.consumer.enable-auto-commit 被设置为false，可以用以下方法进行偏移量提交

①：开启回执模式并关闭自动提交

```java
spring:
  kafka:
    consumer:
      enable-auto-commit: false
    listener:
      ack-mode: manual
```

②：改造消费者

```java
package com.heima.listener;

import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.kafka.support.Acknowledgment;
import org.springframework.stereotype.Component;

@Component
public class TestListener {

    @KafkaListener(topics = "topic-1")
    public void onMessage2(ConsumerRecord<String, String> record, Acknowledgment acknowledgment) {
        System.out.println(record.value());
        acknowledgment.acknowledge();
    }

}
```



### 6)kafka高可用设计

![image-20220722174428080](assets\image-20220722174428080.png)

#### 6.1)集群

- Kafka 的服务器端由被称为 Broker 的服务进程构成，即一个 Kafka 集群由多个 Broker 组成。
- 这样如果集群中某一台机器宕机，其他机器上的 Broker 也依然能够对外提供服务，这其实就是 Kafka 提供高可用的主要手段之一。

![image-20210530223101568](C:\space-bk\项目\黑马头条-上课版\day06\讲义\assets\image-20210530223101568.png)



#### 6.2)备份机制(Replication）

##### 6.2.1)副本的概念

![image-20210530223218580](C:\space-bk\项目\黑马头条-上课版\day06\讲义\assets\image-20210530223218580.png)

Kafka 中消息的备份又叫做**副本**

Kafka 定义了两类副本：

- 领导者副本（Leader Replica）
- 追随者副本（Follower Replica）



##### 6.2.2)副本间的数据同步方式

![image-20210530223316815](C:\space-bk\项目\黑马头条-上课版\day06\讲义\assets\image-20210530223316815.png)

追随者副本分为两种：

- ISR（in-sync replica）：需要同步复制保存的follower
- 普通副本：异步复制保存的follower



##### 6.2.3)副本的选举规则

如果leader失效后，需要选出新的leader，选举的原则如下：

**第一：**选举时优先从ISR中选定，因为这个列表中follower的数据是与leader同步的

**第二：**如果ISR列表中的follower都不行了，就只能从其他follower中选取



极端情况，就是所有副本都失效了，这时有两种方案

**第一：**等待ISR中的一个活过来，选为Leader，数据可靠，但活过来的时间不确定

**第二：**选择第一个活过来的Replication，不一定是ISR中的，选为leader，以最快速度恢复可用性，但数据不一定完整





### 如何保证Kafka数据不丢失？

1. 分析一下哪些环节可能产生数据丢失
2. 生产者发送消息给Broker的时候，可能由于网络波动，或者Broker宕机导致消息丢失
   1. Broker收到消息之后 给我一条回执
   2. 如果生产者收不到回执，就要不断重试
3. 消息存储到Broker时，Broker宕机了，可能导致消息丢失
   1. Broker有自己的持久化方案
   2. 集群：一主二从
4. Broker发送消息给消费者
   1. 由于消费者执行任务报错、消费者宕机、网络波动，导致消息丢失
   2. 消费者成功消费之后，给Broker一个回执

为什么搭一主二从？

- 集群里面有三种副本：leader，ISR，普通follower
- 主机收到消息，会立即同步给ISR（过程是同步的）
- 主机收到消息，会异步的方式发送普通副本
  - leader掉线了，优先选择ISR进行替换；





### 7)自媒体文章上下架功能完成

#### 7.1)需求分析

![image-20210528111736003](kafka及异步通知文章上下架.assets\image-20210528111736003.png)

![image-20210528111853271](kafka及异步通知文章上下架.assets\image-20210528111853271.png)

- 已发表且已上架的文章可以下架

- 已发表且已下架的文章可以上架

#### 7.2)流程说明

![image-20210528111956504](kafka及异步通知文章上下架.assets\image-20210528111956504.png)

#### 7.3)接口定义

|          | **说明**                |
| -------- | ----------------------- |
| 接口路径 | /api/v1/news/down_or_up |
| 请求方式 | POST                    |
| 参数     | DTO                     |
| 响应结果 | ResponseResult          |

使用WmNewsDto（已经存在了），**本接口只用到id和enable两个字段**。

```java
@Data
public class WmNewsDto {
    
    private Integer id;
    /**
    * 是否上架  0 下架  1 上架
    */
    private Short enable;
                       
}
```

ResponseResult  

```json
{
    "code": 501,
    "errorMessage": "文章id不可缺少"
}

{
    "code": 1002,
    "erroeMessage": "文章不存在"
}

{
    "code": 200,
    "errorMessage": "操作成功"
}

{
    "code": 501
    "errorMessage": "当前文章不是发布状态，不能上下架"
}
```



#### 7.4)自媒体文章上下架-功能实现

①：添加依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
</dependency>
```

②：添加配置

```yaml
spring:
	kafka:
    bootstrap-servers: 192.168.200.130:9092
    producer:
      retries: 10
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
```

③：接口定义

在heima-leadnews-wemedia工程下的WmNewsController新增方法

```java
@PostMapping("/down_or_up")
public ResponseResult downOrUp(@RequestBody WmNewsDto wmNewsDto) {
    return wmNewsService.downOrUp(wmNewsDto);
}
```

在WmNewsDto中新增enable属性 ，完整的代码如下：

```java
package com.heima.model.wemedia.dtos;

import lombok.Data;

import java.util.Date;
import java.util.List;

@Data
public class WmNewsDto {
    
    private Integer id;
     /**
     * 标题
     */
    private String title;
     /**
     * 频道id
     */
    private Integer channelId;
     /**
     * 标签
     */
    private String labels;
     /**
     * 发布时间
     */
    private Date publishTime;
     /**
     * 文章内容
     */
    private String content;
     /**
     * 文章封面类型  0 无图 1 单图 3 多图 -1 自动
     */
    private Short type;
     /**
     * 提交时间
     */
    private Date submitedTime; 
     /**
     * 状态 提交为1  草稿为0
     */
    private Short status;
     
     /**
     * 封面图片列表 多张图以逗号隔开
     */
    private List<String> images;

    /**
     * 上下架 0 下架  1 上架
     */
    private Short enable;
}
```

④：业务层编写

在WmNewsService新增方法

```java
/**
 * 文章的上下架
 * @param dto
 * @return
 */
ResponseResult downOrUp(WmNewsDto dto);
```

⑤：实现方法

```java
@Autowired
private KafkaTemplate<String, String> kafkaTemplate;

@Override
public ResponseResult downOrUp(WmNewsDto dto) {

    Integer id = dto.getId();

    // 1. 查询新闻
    WmNews wmNews = wmNewsMapper.selectById(id);
    if (wmNews == null) {
        return ResponseResult.errorResult(AppHttpCodeEnum.DATA_NOT_EXIST, "新闻不存在");
    }

    // 2. 检查状态是否为发布
    Short status = wmNews.getStatus();
    if (status != 9) {
        return ResponseResult.errorResult(AppHttpCodeEnum.PARAM_INVALID, "当前文章不是发布状态，不能上下架");
    }

    // 3. 修改enable
    wmNews.setEnable(dto.getEnable());
    int updateResult = wmNewsMapper.updateById(wmNews);
    if (updateResult < 1) {
        return ResponseResult.errorResult(AppHttpCodeEnum.SERVER_ERROR, "修改文章失败");
    }

    // 4. 消息队列发送给article
    kafkaTemplate.send("news.down.up.topic", JSON.toJSONString(wmNews));

    return ResponseResult.okResult("");
}
```

#### 7.5)消息通知article端文章上下架

①：导入kafka依赖

```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-clients</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
</dependency>
```

②：在自媒体端的nacos配置中心配置kafka的生产者

```yaml
spring:
  kafka:
    bootstrap-servers: 192.168.200.130:9092
    consumer:
      group-id: ${spring.application.name}-test
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

③：在article端编写监听，接收数据

```java
package com.heima.article.listener;

import com.alibaba.fastjson.JSON;
import com.heima.article.mapper.ApArticleConfigMapper;
import com.heima.model.article.pojos.ApArticleConfig;
import com.heima.model.wemedia.pojos.WmNews;
import org.apache.commons.lang3.StringUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Component;

/**
 * @author itheima
 * @since 2022-07-16
 */
@Component
public class ArticleDownListener {

    @Autowired
    private ApArticleConfigMapper apArticleConfigMapper;

    @KafkaListener(topics = "news.down.up.topic")
    public void consumer(String message) {
        if (StringUtils.isBlank(message)) {
            return;
        }

        WmNews wmNews = JSON.parseObject(message, WmNews.class);
        if (wmNews == null) {
            return;
        }

        Long articleId = wmNews.getArticleId();
        ApArticleConfig apArticleConfig = apArticleConfigMapper.selectById(articleId);
        if (apArticleConfig == null) {
            return;
        }

        Short enable = wmNews.getEnable();
        apArticleConfig.setIsDown(enable == 0);
        apArticleConfigMapper.updateById(apArticleConfig);
    }
}
```
