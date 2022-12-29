# xxl-Job分布式任务调度

## 1)今日内容

### 1.1)需求分析

![image-20210729224950851](热点文章-定时计算.assets\image-20210729224950851.png)

目前实现的思路：从数据库直接按照发布时间倒序查询

- 问题1：

  如何访问量较大，直接查询数据库，压力较大

- 问题2：

  新发布的文章会展示在前面，并不是热点文章

### 1.2)实现思路

把热点数据存入redis进行展示

判断文章是否是热点，有几项标准： 点赞数量，评论数量，阅读数量，收藏数量

计算文章热度，有两种方案：

- 定时计算文章热度

- 实时计算文章热度

### 1.3)定时计算

![image-20210729225206299](热点文章-定时计算.assets\image-20210729225206299.png)

- 根据文章的行为（点赞、评论、阅读、收藏）计算文章的分值，利用定时任务每天完成一次计算

- 把分值较大的文章数据存入到redis中

- App端用户查询文章列表的时候，优先从redis中查询热度较高的文章数据



### 1.4)定时任务框架-xxljob

spring传统的定时任务@Scheduled，但是这样存在这一些问题 ：

- 做集群任务的重复执行问题

- cron表达式定义在代码之中，修改不方便

- 定时任务失败了，无法重试也没有统计

- 如果任务量过大，不能有效的分片执行

解决这些问题的方案为：

xxl-job 分布式任务调度框架



## 2)分布式任务调度

### 2.1)什么是分布式任务调度

当前软件的架构已经开始向分布式架构转变，将单体结构拆分为若干服务，服务之间通过网络交互来完成业务处理。在分布式架构下，一个服务往往会部署多个实例来运行我们的业务，如果在这种分布式系统环境下运行任务调度，我们称之为**分布式任务调度**。

![image-20210729230059884](热点文章-定时计算.assets\image-20210729230059884.png)

将任务调度程序分布式构建，这样就可以具有分布式系统的特点，并且提高任务的调度处理能力：

1、并行任务调度

并行任务调度实现靠多线程，如果有大量任务需要调度，此时光靠多线程就会有瓶颈了，因为一台计算机CPU的处理能力是有限的。

如果将任务调度程序分布式部署，每个结点还可以部署为集群，这样就可以让多台计算机共同去完成任务调度，我们可以将任务分割为若干个分片，由不同的实例并行执行，来提高任务调度的处理效率。

2、高可用

若某一个实例宕机，不影响其他实例来执行任务。

3、弹性扩容

当集群中增加实例就可以提高并执行任务的处理效率。

4、任务管理与监测

对系统中存在的所有定时任务进行统一的管理及监测。让开发人员及运维人员能够时刻了解任务执行情况，从而做出快速的应急处理响应。

**分布式任务调度面临的问题：**

当任务调度以集群方式部署，同一个任务调度可能会执行多次，例如：电商系统定期发放优惠券，就可能重复发放优惠券，对公司造成损失，信用卡还款提醒就会重复执行多次，给用户造成烦恼，所以我们需要控制相同的任务在多个运行实例上只执行一次。常见解决方案：

- 分布式锁，多个实例在任务执行前首先需要获取锁，如果获取失败那么就证明有其他服务已经在运行，如果获取成功那么证明没有服务在运行定时任务，那么就可以执行。
- ZooKeeper选举，利用ZooKeeper对Leader实例执行定时任务，执行定时任务的时候判断自己是否是Leader，如果不是则不执行，如果是则执行业务逻辑，这样也能达到目的。

### 2.2)xxl-Job简介

针对分布式任务调度的需求，市场上出现了很多的产品：

1） TBSchedule：淘宝推出的一款非常优秀的高性能分布式调度框架，目前被应用于阿里、京东、支付宝、国美等很多互联网企业的流程调度系统中。但是已经多年未更新，文档缺失严重，缺少维护。

2） XXL-Job：大众点评的分布式任务调度平台，是一个轻量级分布式任务调度平台, 其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

3）Elastic-job：当当网借鉴TBSchedule并基于quartz 二次开发的弹性分布式任务调度系统，功能丰富强大，采用zookeeper实现分布式协调，具有任务高可用以及分片功能。

4）Saturn： 唯品会开源的一个分布式任务调度平台，基于Elastic-job，可以全域统一配置，统一监
控，具有任务高可用以及分片功能。 

XXL-JOB是一个分布式任务调度平台，其核心设计目标是开发迅速、学习简单、轻量级、易扩展。现已开放源代码并接入多家公司线上产品线，开箱即用。

源码地址：https://gitee.com/xuxueli0323/xxl-job

文档地址：https://www.xuxueli.com/xxl-job/

**特性**

- **简单灵活**
  提供Web页面对任务进行管理，管理系统支持用户管理、权限控制；
  支持容器部署；
  支持通过通用HTTP提供跨平台任务调度；
- **丰富的任务管理功能**
  支持页面对任务CRUD操作；
  支持在页面编写脚本任务、命令行任务、Java代码任务并执行；
  支持任务级联编排，父任务执行结束后触发子任务执行；
  支持设置指定任务执行节点路由策略，包括轮询、随机、广播、故障转移、忙碌转移等；
  支持Cron方式、任务依赖、调度中心API接口方式触发任务执行
- **高性能**
  任务调度流程全异步化设计实现，如异步调度、异步运行、异步回调等，有效对密集调度进行流量削峰；
- **高可用**
  任务调度中心、任务执行节点均 集群部署，支持动态扩展、故障转移
  支持任务配置路由故障转移策略，执行器节点不可用是自动转移到其他节点执行
  支持任务超时控制、失败重试配置
  支持任务处理阻塞策略：调度当任务执行节点忙碌时来不及执行任务的处理策略，包括：串行、抛弃、覆盖策略
- **易于监控运维**
  支持设置任务失败邮件告警，预留接口支持短信、钉钉告警；
  支持实时查看任务执行运行数据统计图表、任务进度监控数据、任务完整执行日志；

### 2.3)XXL-Job-环境搭建



#### 2.3.1)初始化“调度数据库”

导入资料中的 "tables_xxl_job.sql" 文件

将创建8张表：

![image-20210730001433997](热点文章-定时计算.assets\image-20210730001433997.png)

```java
- xxl_job_lock：任务调度锁表；
- xxl_job_group：执行器信息表，维护任务执行器信息；
- xxl_job_info：调度扩展信息表： 用于保存XXL-JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等；
- xxl_job_log：调度日志表： 用于保存XXL-JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
- xxl_job_logglue：任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；
- xxl_job_registry：执行器注册表，维护在线的执行器和调度中心机器地址信息；
- xxl_job_user：系统用户表；
```

调度中心支持集群部署，集群情况下各节点务必连接同一个mysql实例;

如果mysql做主从,调度中心集群节点务必强制走主库;



#### 2.3.2)安装调度中心

1.找到资料中的 xxl-job-admin-2.3.0.jar 文件和 application.properties文件，拷贝到你存放软件的目录；

2.修改application.properties中的配置，**将数据库配置改为你的数据库服务器地址**

```yaml
spring.datasource.url=jdbc:mysql://192.168.200.130:3306/xxl_job?Unicode=true&characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=root
server.port=8888
```

3.进入命令行模式，执行以下命令

```shell
java -jar xxl-job-admin-2.3.0.jar --server.port=8888
```

4.访问 http://localhost:8888/xxl-job-admin/toLogin 地址

5.启动调度中心，默认登录账号 “admin/123456”, 登录后运行界面如下图所示。

![image-20210729230630495](热点文章-定时计算.assets\image-20210729230630495.png)



### 2.5)入门案例编写

①：导入资料中的 xxl-demo 项目，到 heima-leadnews-test 模块下。

②：登录调度中心，点击下图所示“新建任务”按钮，新建示例任务

![image-20210729232146585](热点文章-定时计算.assets\image-20210729232146585.png)

③：创建xxljob-demo项目，导入依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!--xxl-job-->
    <dependency>
        <groupId>com.xuxueli</groupId>
        <artifactId>xxl-job-core</artifactId>
        <version>2.3.0</version>
    </dependency>
</dependencies>
```

④：application.yml配置

```yaml
server:
  port: 8881
xxl:
  job:
    admin:
      addresses: http://192.168.200.130:8888/xxl-job-admin
    executor:
      appname: xxl-job-executor-sample
      port: 9999
```

⑤：新建配置类

```java
package com.heima.xxljob.config;

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.port}")
    private int port;


    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setPort(port);
        return xxlJobSpringExecutor;
    }
}
```

⑥：任务代码，重要注解:@XxlJob(“**JobHandler**”)

```java
package com.heima.xxljob.job;

import com.xxl.job.core.handler.annotation.XxlJob;
import org.springframework.stereotype.Component;

@Component
public class HelloJob {


    @XxlJob("demoJobHandler")
    public void helloJob(){
        System.out.println("简单任务执行了。。。。");

    }
}
```

⑦：测试-单节点

- 启动微服务

- 在xxl-job的调度中心中启动任务



### 2.6)任务详解-执行器

- 执行器：任务的绑定的执行器，任务触发调度时将会自动发现注册成功的执行器, 实现任务自动发现功能; 

- 另一方面也可以方便的进行任务分组。每个任务必须绑定一个执行器

![image-20210729232926534](热点文章-定时计算.assets\image-20210729232926534.png)

![image-20210729232825564](热点文章-定时计算.assets\image-20210729232825564.png)

 以下是执行器的属性说明：

| 属性名称 | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| AppName  | 是每个执行器集群的唯一标示AppName, 执行器会周期性以AppName为对象进行自动注册。可通过该配置自动发现注册成功的执行器, 供任务调度时使用; |
| 名称     | 执行器的名称, 因为AppName限制字母数字等组成,可读性不强, 名称为了提高执行器的可读性; |
| 排序     | 执行器的排序, 系统中需要执行器的地方,如任务新增, 将会按照该排序读取可用的执行器列表; |
| 注册方式 | 调度中心获取执行器地址的方式；                               |
| 机器地址 | 注册方式为"手动录入"时有效，支持人工维护执行器的地址信息；   |

自动注册和手动注册的区别和配置

![image-20210729233016355](热点文章-定时计算.assets\image-20210729233016355.png)

### 2.7)任务详解-基础配置

![image-20210729233926457](热点文章-定时计算.assets\image-20210729233926457.png)

**基础配置**

- 执行器：每个任务必须绑定一个执行器, 方便给任务进行分组

- 任务描述：任务的描述信息，便于任务管理；

- 负责人：任务的负责人；

- 报警邮件：任务调度失败时邮件通知的邮箱地址，支持配置多邮箱地址，配置多个邮箱地址时用逗号分隔

![image-20210729234009010](热点文章-定时计算.assets\image-20210729234009010.png)

**调度配置**

- 调度类型：
  - 无：该类型不会主动触发调度；
  - CRON：该类型将会通过CRON，触发任务调度；
  - 固定速度：该类型将会以固定速度，触发任务调度；按照固定的间隔时间，周期性触发；

![image-20210729234114283](热点文章-定时计算.assets\image-20210729234114283.png)

**任务配置**

- 运行模式：

​    BEAN模式：任务以JobHandler方式维护在执行器端；需要结合 "JobHandler" 属性匹配执行器中任务；

- JobHandler：运行模式为 "BEAN模式" 时生效，对应执行器中新开发的JobHandler类“@JobHandler”注解自定义的value值；

- 执行参数：任务执行所需的参数；

![image-20210729234219162](热点文章-定时计算.assets\image-20210729234219162.png)

**阻塞处理策略**

阻塞处理策略：调度过于密集执行器来不及处理时的处理策略；

- 单机串行（默认）：调度请求进入单机执行器后，调度请求进入FIFO(First Input First Output)队列并以串行方式运行；

- 丢弃后续调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，本次请求将会被丢弃并标记为失败；

- 覆盖之前调度：调度请求进入单机执行器后，发现执行器存在运行的调度任务，将会终止运行中的调度任务并清空队列，然后运行本地调度任务；

![image-20210729234256062](热点文章-定时计算.assets\image-20210729234256062.png)

**路由策略**

当执行器集群部署时，提供丰富的路由策略，包括；

- FIRST（第一个）：固定选择第一个机器；

- LAST（最后一个）：固定选择最后一个机器；

- **ROUND（轮询）**

- RANDOM（随机）：随机选择在线的机器；

- CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台机器，且所有任务均匀散列在不同机器上。

- LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的机器优先被选举；

- LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的机器优先被选举；

- FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的机器选定为目标执行器并发起调度；

- BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的机器选定为目标执行器并发起调度；

- SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有机器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务；

![image-20210729234409132](热点文章-定时计算.assets\image-20210729234409132.png)

### 2.8)路由策略(轮询)-案例

①：修改任务为轮询

![image-20210729234513775](热点文章-定时计算.assets\image-20210729234513775.png)

②：启动多个微服务



![image-20210729234536483](热点文章-定时计算.assets\image-20210729234536483.png)

VM options配置如下

```shell
-Dserver.port=8882 -Dxxl.job.executor.port=9998
```

③：启动多个微服务

每个微服务轮询的去执行任务



## 3)热点文章-定时计算

### 3.1)需求分析

需求：为每个频道缓存热度较高的30条文章优先展示

![image-20210729235644605](热点文章-定时计算.assets\image-20210729235644605.png)

判断文章热度较高的标准是什么？

文章：阅读，点赞，评论，收藏

### 3.2)实现思路

![image-20210729235731309](热点文章-定时计算.assets\image-20210729235731309.png)

### 3.3)实现步骤

分值计算不涉及到前端工程，也无需提供api接口，是一个纯后台的功能的开发。

①：导入依赖

```xml
<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.22</version>
</dependency>
```

②：导入vo、修改常量类

```java
package com.heima.model.article.vos;

import com.heima.model.article.pojos.ApArticle;
import lombok.Data;

@Data
public class HotArticleVo extends ApArticle {
    /**
     * 文章分值
     */
    private Integer score;
}
```

修改常量类

```java
package com.heima.common.constants;

public class ArticleConstants {
    public static final Short LOADTYPE_LOAD_MORE = 1;
    public static final Short LOADTYPE_LOAD_NEW = 2;
    public static final String DEFAULT_TAG = "__all__";

    public static final String ARTICLE_ES_SYNC_TOPIC = "article.es.sync.topic";

    public static final Integer HOT_ARTICLE_LIKE_WEIGHT = 3;
    public static final Integer HOT_ARTICLE_COMMENT_WEIGHT = 5;
    public static final Integer HOT_ARTICLE_COLLECTION_WEIGHT = 8;

    public static final String HOT_ARTICLE_FIRST_PAGE = "hot_article_first_page_";
}
```

③：在ApArticleMapper.xml新增方法

```xml
<select id="findArticleListByLast5days" resultMap="resultMap">
    SELECT
    aa.*
    FROM
    `ap_article` aa
    LEFT JOIN ap_article_config aac ON aa.id = aac.article_id
    <where>
        and aac.is_delete != 1
        and aac.is_down != 1
        <if test="dayParam != null">
            and aa.publish_time <![CDATA[>=]]> #{dayParam}
        </if>
    </where>
</select>
```

④：修改ApArticleMapper类

```java
package com.heima.article.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.heima.model.article.dtos.ArticleHomeDto;
import com.heima.model.article.pojos.ApArticle;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.Date;
import java.util.List;

@Mapper
public interface ApArticleMapper extends BaseMapper<ApArticle> {

    /**
     * 加载文章列表
     * @param dto
     * @param type  1  加载更多   2记载最新
     * @return
     */
    List<ApArticle> loadArticleList(ArticleHomeDto dto, Short type);

    /**
     * 加载近五天发布的文章
     */
    List<ApArticle> findArticleListByLast5days(@Param("dayParam") LocalDateTime dayParam);
}
```

⑤：定义业务层接口

```java
package com.heima.article.service;

public interface HotArticleService {

    /**
     * 计算热点文章
     */
    void computeHotArticle();
}
```

⑥：业务层实现类

```java
package com.heima.article.service;

import cn.hutool.core.collection.CollectionUtil;
import com.alibaba.fastjson.JSON;
import com.heima.article.mapper.ApArticleMapper;
import com.heima.common.cache.CacheService;
import com.heima.common.constants.ArticleConstants;
import com.heima.model.article.pojos.ApArticle;
import com.heima.model.article.vos.HotArticleVo;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.util.CollectionUtils;
import org.springframework.util.StringUtils;

import java.time.LocalDateTime;
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class HotArticleServiceImpl implements HotArticleService {

    @Autowired
    private ApArticleMapper apArticleMapper;
    
    @Autowired
    private CacheService cacheService;

    @Override
    public void computeHotArticle() {
        // 1.查询五天内的文章
        LocalDateTime date = LocalDateTime.now().minusDays(5);
        List<ApArticle> articleByLast5days = apArticleMapper.findArticleByLast5days(date);
        if (CollectionUtils.isEmpty(articleByLast5days)) {
            return;
        }

        // 2.计算分值
        List<HotArticleVo> hotArticleVos = computeScore(articleByLast5days);
        if (CollectionUtils.isEmpty(hotArticleVos)) {
            return;
        }

        // 3.分组并缓存
        groupAndCache(hotArticleVos);
    }

    private void groupAndCache(List<HotArticleVo> hotArticleVos) {
        // 3.按频道分组缓存
        List<List<HotArticleVo>> listGroupByChannel = CollectionUtil.groupByField(hotArticleVos, "channelId");
        if (CollectionUtil.isEmpty(listGroupByChannel)) {
            return;
        }

        for (List<HotArticleVo> list : listGroupByChannel) {
            if (CollectionUtil.isEmpty(list)) {
                continue;
            }

            Integer channelId = list.get(0).getChannelId();

            sortedAndCache(list, ArticleConstants.LOADTYPE_LOAD_MORE + channelId.toString());
        }

        // 4.推荐分一组
        sortedAndCache(hotArticleVos, ArticleConstants.LOADTYPE_LOAD_MORE + ArticleConstants.DEFAULT_TAG);
    }

    private void sortedAndCache(List<HotArticleVo> hotArticleVos, String key) {
        if (CollectionUtils.isEmpty(hotArticleVos) || StringUtils.isEmpty(key)) {
            return;
        }

        hotArticleVos = hotArticleVos.stream().sorted(Comparator.comparing(HotArticleVo::getScore).reversed()).collect(Collectors.toList());

        // 截取30条
        if (hotArticleVos.size() > 30) {
            hotArticleVos = hotArticleVos.subList(0, 30);
        }

        cacheService.set(key, JSON.toJSONString(hotArticleVos));
    }

    private List<HotArticleVo> computeScore(List<ApArticle> list) {
        if (CollectionUtils.isEmpty(list)) {
            return new ArrayList<>();
        }

        List<HotArticleVo> voList = new ArrayList<>();
        for (ApArticle apArticle : list) {
            if (apArticle == null) {
                continue;
            }

            // 计算得分
            HotArticleVo vo = compute(apArticle);
            if (vo == null) {
                continue;
            }

            voList.add(vo);
        }

        return voList;
    }

    private HotArticleVo compute(ApArticle article) {
        if (article == null) {
            return null;
        }

        Integer comment = article.getComment();
        Integer likes = article.getLikes();
        Integer views = article.getViews();
        Integer collection = article.getCollection();

        Integer score = 0;

        if (views != null) {
            score += views;
        }

        if (likes != null) {
            score += (ArticleConstants.HOT_ARTICLE_LIKE_WEIGHT * likes);
        }

        if (comment != null) {
            score += (ArticleConstants.HOT_ARTICLE_COMMENT_WEIGHT * comment);
        }

        if (collection != null) {
            score += (ArticleConstants.HOT_ARTICLE_COLLECTION_WEIGHT * collection);
        }

        HotArticleVo vo = new HotArticleVo();
        BeanUtils.copyProperties(article, vo);
        vo.setScore(score);
        return vo;
    }

}
```

⑦：测试

```java
package com.heima.article;

import com.heima.article.service.HotArticleService;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

/**
 * @author itheima
 * @since 2022-07-24
 */
@SpringBootTest
public class ArticleComputeTest {

    @Autowired
    private HotArticleService hotArticleService;

    @Test
    public void test() {
        hotArticleService.computeHotArticle();
    }
}
```

### 3.4)xxl-job定时计算-步骤

①：在heima-leadnews-article中的pom文件中新增依赖

```xml
<!--xxl-job-->
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
    <version>2.3.0</version>
</dependency>
```

②：leadnews-article中集成xxl-job

XxlJobConfig

```java
package com.heima.article.config;

import com.xxl.job.core.executor.impl.XxlJobSpringExecutor;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * xxl-job config
 *
 * @author xuxueli 2017-04-28
 */
@Configuration
public class XxlJobConfig {
    private Logger logger = LoggerFactory.getLogger(XxlJobConfig.class);

    @Value("${xxl.job.admin.addresses}")
    private String adminAddresses;

    @Value("${xxl.job.executor.appname}")
    private String appname;

    @Value("${xxl.job.executor.port}")
    private int port;

    @Bean
    public XxlJobSpringExecutor xxlJobExecutor() {
        logger.info(">>>>>>>>>>> xxl-job config init.");
        XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
        xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
        xxlJobSpringExecutor.setAppname(appname);
        xxlJobSpringExecutor.setPort(port);
        return xxlJobSpringExecutor;
    }
}
```

③：在nacos配置新增配置

```yaml
xxl:
  job:
    admin:
      addresses: http://192.168.200.130:8888/xxl-job-admin
    executor:
      appname: leadnews-hot-article-executor
      port: 9999
```

④：在article微服务中新建任务类

```java
package com.heima.article.job;

import com.heima.article.service.HotArticleService;
import com.xxl.job.core.handler.annotation.XxlJob;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
@Slf4j
public class ComputeHotArticleJob {

    @Autowired
    private HotArticleService hotArticleService;

    @XxlJob("computeHotArticleJob")
    public void handle(){
        log.info("热文章分值计算调度任务开始执行...");
        hotArticleService.computeHotArticle();
        log.info("热文章分值计算调度任务结束...");

    }
}
```

② 在xxl-job-admin中新建执行器和任务

新建执行器：leadnews-hot-article-executor

![image-20210730000549587](热点文章-定时计算.assets\image-20210730000549587.png)

新建任务：路由策略为轮询，Cron表达式：0 0 2 * * ? 

![image-20210730000626824](热点文章-定时计算.assets\image-20210730000626824.png)



## 4)查询文章接口改造

### 4.1)思路分析

![image-20210613110712894](热点文章-定时计算.assets\image-20210613110712894.png)

### 4.2)功能实现

#### 4.2.1)在ApArticleService中新增方法

```java
/**
     * 加载文章列表
     * @param dto
     * @param firstPage  true  是首页  flase 非首页
     * @return
     */
public ResponseResult load2(ArticleHomeDto dto, boolean firstPage);
```

实现方法

```java
/**
     * 加载文章列表
     * @param dto
     * @param firstPage true  是首页  flase 非首页
     * @return
     */
@Override
public ResponseResult load2(ArticleHomeDto dto, boolean firstPage) {
    if(firstPage){
        String jsonStr = cacheService.get(ArticleConstants.HOT_ARTICLE_FIRST_PAGE + dto.getTag());
        if(StringUtils.isNotBlank(jsonStr)){
            List<HotArticleVo> hotArticleVoList = JSON.parseArray(jsonStr, HotArticleVo.class);
            ResponseResult responseResult = ResponseResult.okResult(hotArticleVoList);
            return responseResult;
        }
    }
    return load(dto);
}
```

#### 4.2.2)修改控制器

```java
/**
     * 加载首页
     * @param dto
     * @return
     */
@PostMapping("/load")
public ResponseResult load(@RequestBody ArticleHomeDto dto){
    //        return apArticleService.load(dto, ArticleConstants.LOADTYPE_LOAD_MORE);
    return apArticleService.load2(dto, true);
}
```
