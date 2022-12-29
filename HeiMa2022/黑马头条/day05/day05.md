## 延迟任务精准发布文章

### 1)文章定时发布

### 2)延迟任务概述

#### 2.1)什么是延迟任务

定时任务：有固定周期的，有明确的触发时间。

![image-20220713191332246](assets\image-20220713191332246.png)

**应用场景：**

场景一：订单下单之后30分钟后，如果用户没有付钱，则系统自动取消订单；如果期间下单成功，任务取消

场景二：定时发布新闻



#### 2.2)技术对比

##### 2.2.1)RabbitMQ实现延迟任务

- TTL：Time To Live (消息存活时间)

- 死信队列：Dead Letter Exchange(死信交换机)，当消息成为Dead message后，可以重新发送另一个交换机（死信交换机）

![image-20210513150319742](延迟任务精准发布文章.assets\image-20210513150319742.png)

##### 2.2.2)redis实现

zset数据类型的去重有序（分数排序）特点进行延迟。例如：时间戳作为score进行排序

**任务：**我把要做什么事，什么时候做相关信息和数据打成一个数据包，存储到延迟队列里面。

![image-20210513150352211](延迟任务精准发布文章.assets\image-20210513150352211.png)



### 3)redis实现延迟任务

**实现思路**

![image-20220722115418671](assets\image-20220722115418671.png)

问题思路

1.为什么任务需要存储在数据库中？

延迟任务是一个通用的服务，任何需要延迟得任务都可以调用该服务，需要考虑数据持久化的问题，存储数据库中是一种数据安全的考虑。

2.为什么redis中使用两种数据类型，list和zset？

效率问题，算法的时间复杂度

3.在添加zset数据的时候，为什么不需要预加载？

任务模块是一个通用的模块，项目中任何需要延迟队列的地方，都可以调用这个接口，要考虑到数据量的问题，如果数据量特别大，为了防止阻塞，只需要把未来几分钟要执行的数据存入缓存即可。



### 4)延迟任务服务实现

#### 4.1)搭建heima-leadnews-schedule模块

leadnews-schedule是一个通用的服务，单独创建模块来管理任何类型的延迟任务

①：导入资料文件夹下的heima-leadnews-schedule模块到heima-leadnews-service下，如下图所示：

![image-20210513151649297](延迟任务精准发布文章.assets\image-20210513151649297.png)

②：添加bootstrap.yml

```yaml
server:
  port: 51701
spring:
  application:
    name: leadnews-schedule
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.130:8848
      config:
        server-addr: 192.168.200.130:8848
        file-extension: yml
```

③：在nacos中添加对应配置，并添加数据库及mybatis-plus的配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    url: jdbc:mysql://localhost:3306/leadnews_schedule?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: root
# 设置Mapper接口所对应的XML文件位置，如果你在Mapper接口中有自定义方法，需要进行该配置
mybatis-plus:
  mapper-locations: classpath*:mapper/*.xml
  # 设置别名包扫描路径，通过该属性可以给包中的类注册别名
  type-aliases-package: com.heima.model.schedule.pojos
```



#### 4.2)数据库准备

导入资料中leadnews_schedule数据库

**taskinfo 任务表**

![image-20220713224713541](assets\image-20220713224713541.png)

**taskinfo_logs 任务日志表**

![image-20220713224905870](assets\image-20220713224905870.png)

实体类Taskinfo.java

```java
package com.heima.model.schedule.pojos;

import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

@Data
@TableName("taskinfo")
public class Taskinfo implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 任务id
     */
    @TableId(type = IdType.ID_WORKER)
    private Long taskId;

    /**
     * 执行时间
     */
    @TableField("execute_time")
    private Date executeTime;

    /**
     * 参数
     */
    @TableField("parameters")
    private byte[] parameters;

}
```

实体类TaskinfoLogs.java

```java
package com.heima.model.schedule.pojos;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;

import java.io.Serializable;
import java.util.Date;

@Data
@TableName("taskinfo_logs")
public class TaskinfoLogs implements Serializable {

    private static final long serialVersionUID = 1L;

    /**
     * 任务id
     */
    @TableId(type = IdType.ID_WORKER)
    private Long taskId;

    /**
     * 执行时间
     */
    @TableField("execute_time")
    private Date executeTime;

    /**
     * 参数
     */
    @TableField("parameters")
    private byte[] parameters;

    /**
     * 状态 0=int 1=EXECUTED 2=CANCELLED
     */
    @TableField("status")
    private Integer status;
}
```



#### 4.3)安装redis

①拉取镜像

```shell
docker pull redis
```

② 创建容器

```shell
docker run -d --name redis --restart=always -p 6379:6379 redis --requirepass "leadnews"
```

③链接测试

 打开资料中的Redis Desktop Manager，输入host、port、password链接测试

![image-20210513152138388](延迟任务精准发布文章.assets\image-20210513152138388.png)

能链接成功，即可

#### 4.4)项目集成redis

① 在heima-leadnews-common项目导入redis相关依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- redis依赖commons-pool 这个依赖一定要添加 -->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

②：将资料中的"类\工具类\CacheService.java"添加到heima-leadnews-common项目中

③：为CacheService配置自动加载 META-INF/spring.factories

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.heima.common.cache.CacheService
```

![image-20210514181214681](assets\image-20210514181214681.png)

④ 在heima-leadnews-schedule中集成添加common模块依赖

```xml
<dependency>
    <groupId>com.heima</groupId>
    <artifactId>heima-leadnews-common</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

⑤：在heima-leadnews-schedule中添加配置

```yaml
spring:
  redis:
    host: 192.168.200.130
    password: leadnews
    port: 6379
```

⑥：测试

```java
package com.heima.schedule.test;


import com.heima.common.redis.CacheService;
import com.heima.schedule.ScheduleApplication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.Set;


@SpringBootTest(classes = ScheduleApplication.class)
@RunWith(SpringRunner.class)
public class RedisTest {

    @Autowired
    private CacheService cacheService;

    @Test
    public void testList(){

        //在list的左边添加元素
//        cacheService.lLeftPush("list_001","hello,redis");

        //在list的右边获取元素，并删除
        String list_001 = cacheService.lRightPop("list_001");
        System.out.println(list_001);
    }

    @Test
    public void testZset(){
        //添加数据到zset中  分值
        /*cacheService.zAdd("zset_key_001","hello zset 001",1000);
        cacheService.zAdd("zset_key_001","hello zset 002",8888);
        cacheService.zAdd("zset_key_001","hello zset 003",7777);
        cacheService.zAdd("zset_key_001","hello zset 004",999999);*/

        //按照分值获取数据
        Set<String> zset_key_001 = cacheService.zRangeByScore("zset_key_001", 0, 8888);
        System.out.println(zset_key_001);
    }
}
```



#### 4.5)添加任务

![image-20220722115107897](assets\image-20220722115107897.png)

①：拷贝mybatis-plus生成的文件，mapper

②：创建task类，用于接收添加任务的参数

```java
package com.heima.model.schedule.dtos;

import lombok.Data;

import java.io.Serializable;

@Data
public class Task implements Serializable {

    /**
     * 任务id
     */
    private Long taskId;

    /**
     * 执行id
     */
    private long executeTime;

    /**
     * task参数
     */
    private byte[] parameters;
    
}
```

ScheduleConstants常量类

```java
package com.heima.common.constants;

/**
 * 定时任务常量
 *
 * @author tongdulong@itcast.cn
 */
public class ScheduleConstants {
    /**
     * 初始化状态
     */
    public static final int SCHEDULED = 0;

    /**
     * 已执行状态
     */
    public static final int EXECUTED = 1;

    /**
     * 已取消状态
     */
    public static final int CANCELLED = 2;

    /**
     * 未来数据key前缀
     */
    public static String FUTURE = "future_";

    /**
     * 当前数据key前缀
     */
    public static String TOPIC = "topic_";
}
```

③：创建TaskService

```java
package com.heima.schedule.service;

import com.heima.model.schedule.dto.Task;

public interface TaskService {
    /**
     * 添加任务
     *
     * @param task 任务对象
     * @return 任务id
     */
    long addTask(Task task);
}
```

④：实现类TaskServiceImpl

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public Long addTask(Task task) {
    // 1.参数校验
    if (task == null) {
        log.warn("入参不能为空");
        return null;
    }

    // 2.插入taskInfo表
    Taskinfo taskInfo = new Taskinfo();
    BeanUtils.copyProperties(task, taskInfo);
    taskInfo.setExecuteTime(new Date(task.getExecuteTime()));
    boolean taskInfoResult = taskinfoService.save(taskInfo);
    if (!taskInfoResult) {
        log.warn("任务插入失败");
        return null;
    }

    // 3.插入taskInfoLog表
    TaskinfoLogs logs = new TaskinfoLogs();
    BeanUtils.copyProperties(task, logs);
    logs.setTaskId(taskInfo.getTaskId());
    logs.setExecuteTime(new Date(task.getExecuteTime()));
    logs.setStatus(0);
    boolean logsResult = taskinfoLogsService.save(logs);
    if (!logsResult) {
        log.warn("日志插入失败");
        throw new RuntimeException("日志插入失败");
    }

    // 4.为Task赋值id
    task.setTaskId(taskInfo.getTaskId());

    // 5.写入缓存
    addTaskToCache(task);

    // 6. 返回任务id
    return taskInfo.getTaskId();
}

public void addTaskToCache(Task task) {
    // 任务执行时间
    long executeTime = task.getExecuteTime();
    // 当前时间
    long currentTime = System.currentTimeMillis();
    // 未来五分钟时间
    long futureTime = currentTime + (5 * 60 * 1000);

    // 判断任务执行时间是否小于当前时间
    if (executeTime <= currentTime) {
        // 加入到当前执行队列
        cacheService.lLeftPush("TOPIC", JSON.toJSONString(task));
    }
    // 判断任务执行时间是否小于未来五分钟
    else if (executeTime <= futureTime) {
        cacheService.zAdd("FUTURE", JSON.toJSONString(task), executeTime);
    }
}
```



#### 4.6)取消任务

![image-20220713210439702](assets\image-20220713210439702.png)

①：在TaskService中添加方法

```java
/**
  * 取消任务
  *
  * @param taskId 任务id
  * @return 取消结果
  */
boolean cancelTask(long taskId);
```

②：TaskServiceImpl中实现方法

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public Boolean cancelTask(Long taskId) {
    if (taskId == null) {
        log.warn("入参不能为空");
        return false;
    }

    // 1. 删除 taskinfo
    boolean taskInfoResult = taskinfoService.removeById(taskId);
    if (!taskInfoResult) {
        log.warn("任务删除失败");
        return false;
    }

    // 2. 更新 taskinfolog
    LambdaQueryWrapper<TaskinfoLogs> wrapper = new LambdaQueryWrapper<>();
    wrapper.eq(TaskinfoLogs::getTaskId, taskId);
    TaskinfoLogs logs = taskinfoLogsService.getOne(wrapper);
    logs.setStatus(2);

    boolean logResult = taskinfoLogsService.updateById(logs);
    if (!logResult) {
        log.warn("日志更新失败");
        throw new RuntimeException("日志更新失败");
    }

    // 3. 组装Task结构
    Task task = new Task();
    BeanUtils.copyProperties(logs, task);
    task.setExecuteTime(logs.getExecuteTime().getTime());

    // 4. 删除redis里面的 list 和 zset结构数据
    cacheService.lRemove("TOPIC", 0, JSON.toJSONString(task));
    cacheService.zRemove("FUTURE", JSON.toJSONString(task));

    return true;
}
```



#### 4.7)消费任务

![image-20220722114250159](assets\image-20220722114250159.png)

在TaskService中添加方法

```java
/**
  * 按照类型和优先级来拉取任务
  *
  * @return Task
  */
Task poll();
```

实现

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public Task pollTask() {
    // 1.从TOPIC队列弹出数据
    String taskString = cacheService.lRightPop("TOPIC");
    if (StringUtils.isEmpty(taskString)) {
        log.warn("没有可执行的任务");
        return null;
    }

    // 2.转成Task对象
    Task task = JSON.parseObject(taskString, Task.class);
    if (task == null) {
        log.warn("没有可执行的任务");
        return null;
    }

    // 3.删除TaskInfo数据
    boolean taskInfoResult = taskinfoService.removeById(task.getTaskId());
    if (!taskInfoResult) {
        log.warn("删除任务失败");
        return null;
    }

    // 4.更新TaskInfoLogs状态
    TaskinfoLogs logs = new TaskinfoLogs();
    BeanUtils.copyProperties(task, logs);
    logs.setExecuteTime(new Date(task.getExecuteTime()));
    logs.setStatus(1);
    boolean logResult = taskinfoLogsService.updateById(logs);
    if (!logResult) {
        log.warn("日志更新失败");
        throw new RuntimeException("日志更新失败");
    }

    // 5.返回Task数据
    return task;
}
```



#### 4.8)未来数据定时刷新

##### 4.8.1)reids管道

![image-20220722114907574](assets\image-20220722114907574.png)

普通redis客户端和服务器交互模式

![image-20210515162537224](延迟任务精准发布文章.assets\image-20210515162537224.png)

Pipeline请求模型

![image-20210515162604410](延迟任务精准发布文章.assets\image-20210515162604410.png)

官方测试结果数据对比

![image-20210515162621928](延迟任务精准发布文章.assets\image-20210515162621928.png)



##### 4.8.2)功能实现

①：在TaskServiceImpl中实现方法（定时为每分钟执行一次）

```java
/**
  * 定时刷新任务
  */
@Scheduled(cron = "0 */1 * * * ?")
public void refresh() {
    System.out.println(System.currentTimeMillis() / 1000 + "执行了定时任务");

    // 获取该组key下当前需要消费的任务数据
    // 获取0~当前时间的所有任务（代表小于当前时间的任务）
    // 任务a，执行时间100000,0 ~ 100020
    // 任务b，执行时间100300
    Set<String> tasks = cacheService.zRangeByScore(ScheduleConstants.FUTURE, 0, System.currentTimeMillis());
    if (!tasks.isEmpty()) {
        //将这些任务数据添加到消费者队列中
        cacheService.refreshWithPipeline(ScheduleConstants.FUTURE, ScheduleConstants.TOPIC, tasks);
        System.out.println("成功的将" + ScheduleConstants.FUTURE + "下的当前需要执行的任务数据刷新到" + ScheduleConstants.TOPIC + "下");
    }
}
```

②：在引导类中添加开启任务调度注解：`@EnableScheduling`



#### 4.9)分布式锁解决集群下的方法抢占执行

##### 4.9.1)问题描述

启动两台heima-leadnews-schedule服务，每台服务都会去执行refresh定时任务方法

![image-20210516112243712](延迟任务精准发布文章.assets\image-20210516112243712.png)

##### 4.9.2)分布式锁

分布式锁：控制分布式系统有序的去对共享资源进行操作，通过互斥来保证数据的一致性。

解决方案：

![image-20210516112457413](延迟任务精准发布文章.assets\image-20210516112457413.png)



##### 4.9.3)redis分布式锁

setnx （SET if Not eXists） 命令在指定的 key 不存在时，为 key 设置指定的值。

- setnx包含两个操作
- 检测key是否存在；
- 如果不存在则写入；
- 如果存在则返回false；
- 以上两个操作是一组原子性操作，不会收到其他线程的影响；



![image-20210516112612399](延迟任务精准发布文章.assets\image-20210516112612399.png)

这种加锁的思路是，如果 key 不存在则为 key 设置 value，如果 key 已存在则 SETNX 命令不做任何操作

- 客户端A请求服务器设置key的值，如果设置成功就表示加锁成功
- 客户端B也去请求服务器设置key的值，如果返回失败，那么就代表加锁失败
- 客户端A执行代码完成，删除锁
- 客户端B在等待一段时间后再去请求设置key的值，设置成功
- 客户端B执行代码完成，删除锁

##### 4.9.4)在工具类CacheService中添加方法

```java
/**
  * 加锁
  *
  * @param name
  * @param expire
  * @return
  */
public String tryLock(String name, long expire) {
    name = name + "_lock";
    String token = UUID.randomUUID().toString();
    Boolean locked = stringRedisTemplate.opsForValue().setIfAbsent(name, token, expire, TimeUnit.MILLISECONDS);
    if (locked == null) {
        return null;
    }

    return locked ? token : null;
}
```

修改未来数据定时刷新的方法，如下：

```java
/**
  * 定时刷新任务
  */
@Scheduled(cron = "0 */1 * * * ?")
public void refresh() {
    String token = cacheService.tryLock("FUTURE_TASK_SYNC", 1000 * 30);
    if(StringUtils.isBlank(token)){
    	return;
    }
    
    System.out.println(System.currentTimeMillis() / 1000 + "执行了定时任务");

    // 获取该组key下当前需要消费的任务数据
    // 获取0~当前时间的所有任务（代表小于当前时间的任务）
    // 任务a，执行时间100000,0 ~ 100020
    // 任务b，执行时间100300
    Set<String> tasks = cacheService.zRangeByScore(ScheduleConstants.FUTURE, 0, System.currentTimeMillis());
    if (!tasks.isEmpty()) {
        //将这些任务数据添加到消费者队列中
        cacheService.refreshWithPipeline(ScheduleConstants.FUTURE, ScheduleConstants.TOPIC, tasks);
        System.out.println("成功的将" + ScheduleConstants.FUTURE + "下的当前需要执行的任务数据刷新到" + ScheduleConstants.TOPIC + "下");
    }
}
```



#### 4.10)数据库定时同步到redis

![image-20220713233944475](assets\image-20220713233944475.png)

```java
@Scheduled(fixedRate = 5 * 60 * 1000)
public void initData() {
    // 清除缓存
    clear();

    List<Taskinfo> list = taskinfoService.list();
    if (CollectionUtils.isEmpty(list)) {
        log.warn("任务为空");
        return;
    }

    for (Taskinfo taskinfo : list) {
        if (taskinfo == null) {
            continue;
        }

        long executeTime = taskinfo.getExecuteTime().getTime();
        long currentTime = System.currentTimeMillis();
        long futureTime = currentTime + (5 * 60 * 1000);

        Task task = new Task();
        BeanUtils.copyProperties(taskinfo, task);
        task.setExecuteTime(executeTime);

        if (executeTime <= currentTime) {
            cacheService.lLeftPush("TOPIC", JSON.toJSONString(task));
        } else if (executeTime <= futureTime) {
            cacheService.zAdd("FUTURE", JSON.toJSONString(task), executeTime);
        }
    }
}

private void clear() {
    cacheService.delete("TOPIC");
    cacheService.delete("FUTURE");
}
```



### 5)延迟队列解决精准时间发布文章

![image-20220722155937412](assets\image-20220722155937412.png)

##### 5.1)延迟队列服务提供对外接口

提供远程的feign接口，在heima-leadnews-feign-api编写类如下：

```java
package com.heima.apis.schedule;

import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.schedule.dto.Task;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient("leadnews-schedule")
public interface IScheduleClient {
    /**
     * 添加任务
     *
     * @param task 任务对象
     * @return 任务id
     */
    @PostMapping("/api/v1/task/add")
    ResponseResult addTask(@RequestBody Task task);

    /**
     * 取消任务
     *
     * @param taskId 任务id
     * @return 取消结果
     */
    @GetMapping("/api/v1/task/cancel/{taskId}")
    ResponseResult cancelTask(@PathVariable("taskId") long taskId);

    /**
     * 按照类型和优先级来拉取任务
     *
     * @return ResponseResult
     */
    @GetMapping("/api/v1/task/poll")
    ResponseResult poll();
}
```

在heima-leadnews-schedule微服务下提供对应的实现

```java
package com.heima.schedule.client;

import com.heima.apis.schedule.IScheduleClient;
import com.heima.model.common.dtos.ResponseResult;
import com.heima.model.schedule.dto.Task;
import com.heima.schedule.service.TaskService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

/**
 * @author itheima
 * @since 2022-07-22
 */
@RestController
public class ScheduleClient implements IScheduleClient {

    @Autowired
    private TaskService taskService;

    /**
     * 添加任务
     *
     * @param task 任务对象
     * @return 任务id
     */
    @PostMapping("/api/v1/task/add")
    @Override
    public ResponseResult addTask(@RequestBody Task task) {
        return ResponseResult.okResult(taskService.addTask(task));
    }

    /**
     * 取消任务
     *
     * @param taskId 任务id
     * @return 取消结果
     */
    @GetMapping("/api/v1/task/cancel/{taskId}")
    @Override
    public ResponseResult cancelTask(@PathVariable("taskId") long taskId) {
        return ResponseResult.okResult(taskService.cancelTask(taskId));
    }

    /**
     * 按照类型和优先级来拉取任务
     *
     * @return ResponseResult
     */
    @GetMapping("/api/v1/task/poll")
    @Override
    public ResponseResult poll() {
        return ResponseResult.okResult(taskService.pollTask());
    }
}
```

##### 5.2)发布文章集成添加延迟队列接口

①：WmNewsServiceImpl中添加方法

方法中使用了ProtostuffUtil工具类，位于heima-leadnews-utils下，直接调用即可；

该工具类需要以下依赖（不需要引入，初始架构里已经准备好了）

```xml
<dependency>
    <groupId>io.protostuff</groupId>
    <artifactId>protostuff-core</artifactId>
    <version>1.6.0</version>
</dependency>

<dependency>
    <groupId>io.protostuff</groupId>
    <artifactId>protostuff-runtime</artifactId>
    <version>1.6.0</version>
</dependency>
```

方法核心代码

```java
private Boolean addToTask(Integer newsId, Long executeTime) {
    // 1.组装Task结构
    Task task = new Task();
    task.setExecuteTime(executeTime);

    // 2.组装News结构，用于存储newsIs，后续消费的时候，需要newsId作为入参
    WmNews wmNews = new WmNews();
    wmNews.setId(newsId);
    
    // 3.序列化并存储（序列化可提高传输效率）
    task.setParameters(ProtostuffUtil.serialize(wmNews));

    // 4.添加任务
    ResponseResult responseResult = scheduleClient.addTask(task);
    if (responseResult != null && responseResult.getCode() == 200) {
        return true;
    }

    return false;
}
```

序列化工具对比

- JdkSerialize：java内置的序列化能将实现了Serilazable接口的对象进行序列化和反序列化， ObjectOutputStream的writeObject()方法可序列化对象生成字节数组
- Protostuff：google开源的protostuff采用更为紧凑的二进制数组，表现更加优异，然后使用protostuff的编译工具生成pojo类

②：修改发布文章代码

把之前的异步调用修改为调用延迟任务

```java
@Override
@Transactional(rollbackFor = RuntimeException.class)
public ResponseResult submit(WmNewsDto wmNewsDto) {

    // 8. 调用审核代码
    // try {
    //    autoScanService.autoScanWmNews(wmNews.getId().longValue());
    // } catch (Exception e) {
    //    e.printStackTrace();
    // }
    
    // 8. 审核文章（修改为任务模式）
    wmNewsTaskService.addNewsToTask(wmNews.getId(),wmNews.getPublishTime());

    return ResponseResult.okResult(AppHttpCodeEnum.SUCCESS);
}
```

##### 5.3)消费任务进行审核文章

①：在WemediaApplication自媒体的引导类中添加开启任务调度注解`@EnableScheduling`

②：AutoScanService中添加方法

```
/**
 * 消费延迟队列数据
 */
void getTask();
```

③：AutoScanServiceImpl中实现方法

```java
@Autowired
private IScheduleClient scheduleClient;

@Scheduled(fixedRate = 1000)
@Override
public void getTask() {
    // 1.试着获得一个任务
    ResponseResult responseResult = scheduleClient.poll();
    if (responseResult == null || responseResult.getCode() != 200) {
        log.warn("消费失败");
        return;
    }

    Object data = responseResult.getData();
    if(data == null) {
        log.warn("没有可消费的任务");
        return;
    }

    // 2.解析任务信息
    String string = JSON.toJSONString(data);
    if(StringUtils.isEmpty(string)) {
        log.warn("没有可消费的任务2");
        return;
    }

    Task task = JSON.parseObject(string, Task.class);
    if(task == null) {
        log.warn("没有可消费的任务");
        return;
    }

    // 3.获得任务中的newsId
    byte[] parameters = task.getParameters();
    WmNews wmNews = ProtostuffUtil.deserialize(parameters, WmNews.class);
    if(wmNews == null) {
        log.warn("参数缺失");
        return;
    }

    // 4.调用审核代码
    try {
        autoScanWmNews(wmNews.getId().longValue());
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```



