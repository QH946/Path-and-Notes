[TOC]



# 大型互联网项目架构目标

- 高性能：提供快速的访问体验
- 高可用：网站服务一直可以正常访问

- 可伸缩：通过硬件增加/减少，提高/降低处理能力

- 高可扩展：系统间耦合低，方便的通过新增/移除方式，增加/减少新的功能/模块

- 安全性：提供网站安全访问和数据加密，安全存储等策略

- 敏捷性：随需应变，快速响应



## 衡量网站的性能指标：

	- 响应时间：一个请求从开始到最后收到响应数据所花费的总体时间
	- 并发数：同时能处理的请求数量
	- 并发连接数：每秒钟服务器连接的总TCP数量【一个TCP连接可以有多个请求】
	- 请求数：也称为QPS(Query Per Second) 指**每秒多少请求**
 - 并发用户数：**单位时间**内有多少用户
 - 吞吐量：指**单位时间**内系统能处理的**请求数量**
   ​	QPS：Query Per Second 每秒查询数
   ​	TPS：Transactions Per Second 每秒事务数 【**QPS >= 并发连接数 >= TPS**】
   	- 一个事务是指一个客户机向服务器发送请求然后服务器做出反应的过程。

	- 客户机在发送请求时开始计时，收到服务器响应后结束计时，以此来计算使用的时间和完成的事务个数

- 一个页面的一次访问，只会形成一个TPS；但一次页面请求，可能产生多次对服务器的请求，就会有多个QPS



# 集群和分布式

## 集群

一个业务模块，部署在多台服务器上。 【很多“人”一起 ，**干一样的事**】



### 分布式

很多“人”一起，干**不一样的事**。这些不一样的事，**合起来是一件大事**

一个大的业务系统，拆分为小的业务模块，分别部署在不同的机器上



- 单机

---

![image-20220815234928092](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220815234928092.png)

---



- 集群

---

![image-20220518143237733](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518143237733.png)

---



- 集群分布式

---

![image-20220519151429078](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220519151429078.png)

---











# 架构演进



## 单体架构

简单：开发部署都很方便，小型项目首选



## 垂直架构

两个独立的项目之间不能进行通行，所以会有重复的模块在不同的项目中

---

![image-20220518144401203](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518144401203.png)

---







## 分布式架构

分布式架构是指在垂直架构的基础上，**将公共业务模块抽取出来，作为独立的服务**，**供其他调用者消费**，以==实现服务的共享和重用==

​		RPC： Remote Procedure Call 远程过程调用**【简称：远程调用】**

有非常多的协议和技术来都实现了RPC的过程：

​		比如：HTTP REST风格，Java RMI规范【JavaEE13个规范】、WebService SOAP协议、Hession等等。



垂直架构存在的问题：重复功能太多

分布式架构存在的问题：服务提供方一旦产生变更，所有消费方都需要变更。

---

![image-20220518144736707](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518144736707.png)

---



## SOA架构

SOA：（Service-Oriented Architecture，面**向服务的架构**）是一个组件模型

​			它将应用程序的不同功能单元（称为服务）进行拆分，并通过这些服务之间定义良好的接口和契约联系起来。



ESB：(Enterparise Servce Bus) 企业服务总线，**服务中介**  ==主要是服务交互dubbo，服务治理zookeeper==

​	主要是提供了一个服务于服务之间的交互。

​	ESB 包含的功能如：负载均衡，流量控制，加密处理，服务的监控，异常处理，监控告急等等**【主要是服务交互dubbo，服务治理zookeeper】**



分布式架构存在的问题：
服务提供方一旦产生变更，所有消费方都需要变更



---

![image-20220518145411698](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518145411698.png)

---





## 微服务架构

微服务架构是在 SOA 上做的升华，微服务架构强调的一个重点是“**业务需要彻底的组件化和服务化**”

原有的单个业务系统会拆分为多个可以独立开发、设计、运行的小应用。这些小应用之间通过服务完成交互和集成。

微服务架构 = 80%的SOA服务架构思想 + 100%的组件化架构思想 + 80%的领域建模思想



特点：

- **服务实现组件化**：不同服务的开发者可以选择不同的开发技术。也不需要协调其他团队

- **服务之间交互一般使用REST API**

- 去中心化：**每个微服务有自己私有的数据库持久化业务数据**

- 自动化部署：把应用拆分成为一个一个独立的单个服务，方便自动化部署、测试、运维





---

![image-20220518145543698](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518145543698.png)

---





## 架构演进



---

![image-20220518145757291](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518145757291.png)

---



dubbo 是 SOA时代的产物，SpringCloud 是微服务时代的产物







# Dubbo

## 概念

Dubbo是阿里巴巴公司开源的一个高性能、轻量级的 Java RPC 框架

致力于提供高性能和透明化的 **RPC** 远程==服务调用==方案，以及 SOA ==服务治理==方案

官网：http://dubbo.apache.org





## Dubbo架构

---

![image-20220518150130914](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518150130914.png)

---

节点角色说明：
Provider：暴露服务的服务提供方
Container：服务运行容器
Consumer：调用远程服务的服务消费方
Registry：服务注册与发现的注册中心
Monitor：统计服务的调用次数和调用时间的监控中心

 



# Dubbo快速入门





## 在服务器上安装zookeeper

**1、环境准备**

ZooKeeper服务器是用Java创建的，它运行在JVM之上。需要安装JDK 7或更高版本。



**2、上传**

将下载的ZooKeeper放到/opt/ZooKeeper目录下

```shell
#上传zookeeper alt+p
put f:/setup/apache-zookeeper-3.5.6-bin.tar.gz
#打开 opt目录
cd /opt
#创建zooKeeper目录
mkdir  zooKeeper
#将zookeeper安装包移动到 /opt/zooKeeper
mv apache-zookeeper-3.5.6-bin.tar.gz /opt/zookeeper/
```

**3、解压**

将tar包解压到/opt/zookeeper目录下

```shell
tar -zxvf apache-ZooKeeper-3.5.6-bin.tar.gz 
```

## 1.2 配置启动

**1、配置zoo.cfg**

进入到conf目录拷贝一个zoo_sample.cfg并完成配置

```shell
#进入到conf目录
cd /opt/zooKeeper/apache-zooKeeper-3.5.6-bin/conf/
#拷贝
cp  zoo_sample.cfg  zoo.cfg
```



修改zoo.cfg

```shell
#打开目录
cd /opt/zooKeeper/
#创建zooKeeper存储目录
mkdir  zkdata
#修改zoo.cfg
vim /opt/zooKeeper/apache-zooKeeper-3.5.6-bin/conf/zoo.cfg
```

---

![1577548250377](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1577548250377.png)

---



修改存储目录：dataDir=/opt/zookeeper/zkdata

**2、启动ZooKeeper**

```shell
cd /opt/zooKeeper/apache-zooKeeper-3.5.6-bin/bin/
#启动
 ./zkServer.sh  start
```

---

![1577548052037](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1577548052037.png)

---



看到上图表示ZooKeeper成功启动

**3、查看ZooKeeper状态**

```shell
./zkServer.sh status
```

zookeeper启动成功。standalone代表zk没有搭建集群，现在是单节点

---

![1577548175232](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1577548175232.png)

---



zookeeper没有启动

---

![1577548112773](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1577548112773.png)

---





## 服务提供者

采用SSM【没有SpringMVC 没有Mybatis】



- 业务逻辑层

~~~java
package com.itheima.service.impl;

import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


@Service//这是Dubbo的注解而不是springframework的 import org.apache.dubbo.config.annotation.Service;
public class UserServiceImpl implements UserService {

    public String sayHello() {
        return "hello dubbo!~";
    }
}

~~~





- spring核心配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

	<!--不需要spring来扫包 通过dubbo扫包-->
    <!--	<context:component-scan base-package="com.itheima.service" />-->

    
    <!--配置dubbo-->
    
    <!--配置该某块名字-->
    <dubbo:application name="dubbo-service"/>

    <!--配置注册中心地址-->
    <dubbo:registry address="zookeeper://192.168.175.130:2181"/>

    <!--配置dubbo的包扫描-->
    <dubbo:annotation package="com.itheima.service.impl"/>

</beans>

~~~



- resource下的log4j.properties文件

~~~properties
# DEBUG < INFO < WARN < ERROR < FATAL
# Global logging configuration
log4j.rootLogger=info, stdout,file
# My logging configuration...
#log4j.logger.com.tocersoft.school=DEBUG
#log4j.logger.net.sf.hibernate.cache=debug
## Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p %d %C: %m%n

log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=../logs/iask.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss}  %l  %m%n
~~~



- web文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">


	<!--将spring核心配置文件配置为tomcat的上下文参数--> 
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:spring/applicationContext*.xml</param-value>
    </context-param>
    
       <!-- 在Tomcat启动时，根据配置文件加载spring容器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

</web-app>

~~~



- pom

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.itheima</groupId>
    <artifactId>dubbo-service</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>war</packaging>


    <properties>
        <spring.version>5.1.9.RELEASE</spring.version>
        <dubbo.version>2.7.4.1</dubbo.version>
        <zookeeper.version>4.0.0</zookeeper.version>

    </properties>

    <dependencies>
      
        <!-- servlet3.0规范的坐标 -->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
       
        <!--spring的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>${spring.version}</version>
        </dependency>
        
        <!--springmvc的坐标-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>${spring.version}</version>
        </dependency>

        <!--日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.21</version>
        </dependency>
       
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.21</version>
        </dependency>


        <!--Dubbo的起步依赖，版本2.7之后统一为rg.apache.dubb -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>${dubbo.version}</version>
        </dependency>
        
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-framework</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>
        
        <!--ZooKeeper客户端实现 -->
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>${zookeeper.version}</version>
        </dependency>


    </dependencies>
    <build>
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>9000</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

~~~



- maven内嵌tomcat插件

~~~xml
<build>
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>9000</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
~~~



## 服务消费者



- 界面层

~~~java
package com.itheima.controller;

import com.itheima.service.UserService;

import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class UserController {

    //@Autowired 本地注入
    
    /**
     *	 @Reference注解三个作用： 
     * 		1. 从zookeeper注册中心获取userService的访问url
     * 		2. 进行远程调用RPC
     * 		3.将结果封装成一个代理对象
     */
    @Reference//远程注入业务层
    private UserService userService;


    @RequestMapping("/sayHello")
    public String sayHello() {
        //return "sssss";
        return userService.sayHello();
    }

}


~~~



- 为通过编译 提供业务层接口

~~~java
package com.itheima.service;

/**
 * @Author fgcy
 * @Date 2022/5/18
 */
public interface UserService {
    String sayHello();
}

~~~



- spring核心配置文件

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans 
                           http://www.springframework.org/schema/beans/spring-beans.xsd
       					   http://www.springframework.org/schema/mvc 
                           http://www.springframework.org/schema/mvc/spring-mvc.xsd
         					http://dubbo.apache.org/schema/dubbo 
                           http://dubbo.apache.org/schema/dubbo/dubbo.xsd
                           http://www.springframework.org/schema/context
                           https://www.springframework.org/schema/context/spring-context.xsd">

	<!--返回json格式 @ResponseBody-->
    <mvc:annotation-driven/>
    
	<!--要扫界面层-->
    <context:component-scan base-package="com.itheima.controller"/>
    <!--配置该某块名字-->
    <dubbo:application name="dubbo-web"/>

    <!-- 配置注册中心地址-->
    <dubbo:registry address="zookeeper://192.168.175.130:2181"/>

    <!--配置包扫描-->
    <dubbo:annotation package="com.itheima.controller"/>

</beans>

~~~



- 日志配置

~~~properties
# DEBUG < INFO < WARN < ERROR < FATAL
# Global logging configuration
log4j.rootLogger=info, stdout,file
# My logging configuration...
#log4j.logger.com.tocersoft.school=DEBUG
#log4j.logger.net.sf.hibernate.cache=debug
## Console output...
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%5p %d %C: %m%n

log4j.appender.file=org.apache.log4j.FileAppender
log4j.appender.file.File=../logs/iask.log
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss}  %l  %m%n
~~~





- web

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
         version="2.5">



	<!-- Springmvc -->
    <!--将所有请求都交给DispaterServlet处理-->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        
        <!-- 指定加载的配置文件 ，通过参数contextConfigLocation加载-->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring/springmvc.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>

</web-app>

~~~



注意：

请求java小程序要加后缀 .do





## 公共接口

写一个项目专门用来存放接口



将服务调用者的业务层删掉

在服务调用者的pom文件中引入公共接口依赖



~~~java
package com.itheima.service;

public interface UserService {
    public String sayHello();
}
~~~



# Dubbo高级特性





## 入门dubbo-admin

dubbo-admin 管理平台，是图形化的服务管理页面

从注册中心中获取到所有的提供者 / 消费者进行配置管理

路由规则、动态配置、服务降级、访问控制、权重调整、负载均衡等管理功能

 dubbo-admin 是一个前后端分离的项目。前端使用vue，后端使用springboot

安装 dubbo-admin 其实就是部署该项目





使用maven对dubbo-admin经行打包

要有jdk和node.js环境





## 一、dubbo-admin安装

**1、环境准备**

dubbo-admin 是一个前后端分离的项目。前端使用vue，后端使用springboot，安装 dubbo-admin 其实就是部署该项目。我们将dubbo-admin安装到开发环境上。要保证开发环境有jdk，maven，nodejs

安装node**(如果当前机器已经安装请忽略)**

因为前端工程是用vue开发的，所以需要安装node.js，node.js中自带了npm，后面我们会通过npm启动

下载地址

```
https://nodejs.org/en/
```

---

![1578298201398](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578298201398.png)

---





**2、下载 Dubbo-Admin**

进入github，搜索dubbo-admin

```
https://github.com/apache/dubbo-admin
```

下载：

---

![1578297063167](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578297063167.png)

---



**3、把下载的zip包解压到指定文件夹(解压到那个文件夹随意)**

----

![1578297477356](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578297477356.png)

----





**4、修改配置文件**

解压后我们进入…\dubbo-admin-develop\dubbo-admin-server\src\main\resources目录，找到 **application.properties** 配置文件 进行配置修改

---

![1578297603008](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578297603008.png)

---



修改zookeeper地址

---

![1578297758655](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578297758655.png)

---



```shell
# centers in dubbo2.7
admin.registry.address=zookeeper://192.168.149.135:2181
admin.config-center=zookeeper://192.168.149.135:2181
admin.metadata-report.address=zookeeper://192.168.149.135:2181

```

admin.registry.address注册中心
admin.config-center 配置中心
admin.metadata-report.address元数据中心

**5、打包项目**

在 dubbo-admin-develop 目录执行打包命令

```shell
mvn clean install package '-Dmaven.test.skip=true'
```

---

![1578300464726](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578300464726.png)

---



**6、启动后端**

切换到目录

```shell
dubbo-Admin-develop\dubbo-admin-distribution\target>
```

执行下面的命令启动 dubbo-admin，dubbo-admin后台由SpringBoot构建。

```shell
java -jar .\dubbo-admin-0.1.jar
```

![1578300551892](F:\Dubbo\dubbo\资料\images\1578300551892.png)

**7、前台后端**

dubbo-admin-ui 目录下执行命令

```shell
npm run dev
```

---

![1578300677041](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578300677041.png)

----



**8、访问**

浏览器输入。用户名密码都是root

```
http://localhost:8081/
```

---

![1578300774258](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578300774258.png)

---



## 二、dubbo-admin简单使用



注意:Dubbo Admin【服务Mock】【服务统计】将在后续版本发布....

在上面的步骤中，我们已经进入了Dubbo-Admin的主界面，在【快速入门】章节中，我们定义了服务生产者、和服务消费者，下面我们从Dubbo-Admin管理界面找到这个两个服务

**1、点击服务查询**

**2、查询结果**

---

![1578301528363](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/1578301528363.png)

---





A:输入的查询条件com.itheima.service.UserService

B:搜索类型，主要分为【按服务名】【按IP地址】【按应用】三种类型查询

C:搜索结果

**3.1.4 dubo-admin查看详情**

我们查看com.itheima.service.UserService （服务提供者）的具体详细信息，包含【元数据信息】

我们需要打开我们的**生产者spring配置文件**加入下面配置

```xml
    <!-- 元数据配置 -->
    <dubbo:metadata-report address="zookeeper://192.168.149.135:2181" />
```

重新启动生产者，再次打开Dubbo-Admin

这样我们的元数据信息就出来了

---

![image-20220518212532184](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518212532184.png)

---



## 序列化

dubbo 内部已经将序列化和反序列化的过程内部封装了
我们只需要在定义pojo类时实现Serializable接口即可 implements Serializable



一般会定义一个公共的pojo模块，让生产者和消费者都依赖该模块



---

![image-20220518213407998](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518213407998.png)

---







## 地址缓存

- 问题

注册中心挂了，服务是否可以正常访问？



可以，因为dubbo服务消费者在第一次调用时，会将**服务提供方地址缓存到本地**，以后在调用则不会访问注册中心
**当服务提供者地址发生变化时，注册中心会通知服务消费者**





## 超时与重试



- 问题

服务消费者在调用服务提供者的时候发生了阻塞、等待的情形，这个时候，服务消费者会一直等待下去。
在某个峰值时刻，大量的请求都在同时请求服务消费者，会造成线程的大量堆积，势必会造成雪崩。
dubbo 利用超时机制来解决这个问题，设置一个超时时间，在这个时间段内，无法完成服务访问，则自动断开连接。
使用timeout属性配置超时时间，**默认值1000，单位毫秒**。**【默认1秒】**





- 解决

设置了超时时间，在这个时间段内，无法完成服务访问，则自动断开连接。
如果出现网络抖动，则这一次请求就会失败。
Dubbo 提供**重试机制来**避免类似问题的发生。
通过 retries  属性来设置重试次数。**默认为 2 次**。



- 重试

设置了超时时间，在这个时间段内，无法完成服务访问，则自动断开连接。
如果出现网络抖动，则这一次请求就会失败。
Dubbo 提供重试机制来避免类似问题的发生。
通过 retries  属性来设置重试次数。默认为 2 次。





- 服务提供者

~~~java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义


//将这个类提供的方法（服务）对外发布【服务暴露】。将访问的地址 ip，端口，路径注册到注册中心中
@Service(timeout = 3000,retries = 2)//当前服务3秒超时,重试2次，一共访问3次 默认超时一秒；重试两次；
public class UserServiceImpl implements UserService {

    int i = 1;
    public String sayHello() {
        return "hello dubbo hello!~";
    }

	//重试验证
    public User findUserById(int id) {
        System.out.println("服务被调用了："+i++);
        //查询User对象
        User user = new User(1,"zhangsan","123");
        
        //数据库查询很慢，查了5秒
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return user;
    }
}

~~~





- 服务调用者

~~~JAVA
package com.itheima.controller;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    //注入Service
    //@Autowired//本地注入

    /*
        1. 从zookeeper注册中心获取userService的访问url
        2. 进行远程调用RPC
        3. 将结果封装为一个代理对象。给变量赋值

     */

    @Reference(timeout = 1000)//远程注入【调用者的超时时间可以覆盖提供者定义的】
    private UserService userService;


    @RequestMapping("/sayHello")
    public String sayHello(){
        return userService.sayHello();
    }

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */
    int i = 1;
    @RequestMapping("/find")
    public User find(int id){
		//异步计时 一秒一次
        new Thread(new Runnable() {
            public void run() {
                while (true){
                    System.out.println(i++);
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();

        return userService.findUserById(id);
    }
}

~~~



**建议超时时间 配在服务提供者一端**







## 多版本



灰度发布：当出现新功能时，会让一部分用户先使用新功能，用户反馈没问题时，再将所有用户迁移到新功能

dubbo 中使用version 属性来设置和调用同一个接口的不同版本



---

![image-20220518220024867](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518220024867.png)

---



- 版本一服务提供者

~~~java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service(version = "v1.0")
public class UserServiceImpl implements UserService {

    public String sayHello() {
        return "hello dubbo hello!~";
    }


    public User findUserById(int id) {
        System.out.println("old...");
        //查询User对象
        User user = new User(1,"zhangsan","123");

        return user;
    }
}
~~~



- 版本二服务提供者

~~~java
package com.itheima.service.impl;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Service;


//@Service//将该类的对象创建出来，放到Spring的IOC容器中  bean定义
//将这个类提供的方法（服务）对外发布。将访问的地址 ip，端口，路径注册到注册中心中
@Service(version = "v2.0")
public class UserServiceImpl2 implements UserService {

    public String sayHello() {
        return "hello dubbo hello!~";
    }


    public User findUserById(int id) {
        System.out.println("new....");
        //查询User对象
        User user = new User(1,"zhangsan","123");

        return user;
    }
}

~~~







- 服务调用者

~~~java
package com.itheima.controller;

import com.itheima.pojo.User;
import com.itheima.service.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/user")
public class UserController {

    //注入Service
    //@Autowired//本地注入

    /*
        1. 从zookeeper注册中心获取userService的访问url
        2. 进行远程调用RPC
        3. 将结果封装为一个代理对象。给变量赋值

     */

    @Reference(version = "v2.0")//远程注入
    private UserService userService;


    @RequestMapping("/sayHello")
    public String sayHello(){
        return userService.sayHello();
    }

    /**
     * 根据id查询用户信息
     * @param id
     * @return
     */

    @RequestMapping("/find")
    public User find(int id){
        return userService.findUserById(id);
    }
}
~~~







## 负载均衡





- Random ：按权重随机，默认值。按权重设置随机概率。

---

![image-20220815235256216](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220815235256216.png)

---

1/4  1/2 1/4



- RoundRobin ：按权重轮询

---

![image-20220815235322915](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220815235322915.png)

---

​				来四个请求：1分到一个；2分到两个；3分到一个





- LeastActive：最少活跃调用数，相同活跃数的随机。

---

![image-20220815235346035](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220815235346035.png)

---

活跃数指调用前后计数差，数量少的说明处理的次数少；使慢的 Provider 收到更少请求





- ConsistentHash：一致性 Hash，相同参数的请求总是发到同一提供者【ipHash】

---

![image-20220518230959751](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518230959751.png)

---





注意：配置端口



- tomcat端口

~~~xml
  <build>
        <plugins>
            <!--tomcat插件-->
            <plugin>
                <groupId>org.apache.tomcat.maven</groupId>
                <artifactId>tomcat7-maven-plugin</artifactId>
                <version>2.1</version>
                <configuration>
                    <port>9002</port>
                    <path>/</path>
                </configuration>
            </plugin>
        </plugins>
    </build>
~~~



- dubbo端口

~~~xml
	<dubbo:protocol port="20882" />

	<dubbo:application name="dubbo-service">
		<dubbo:parameter key="qos.port" value="65533"/>
	</dubbo:application>
~~~



- 配置权重

~~~java
@Service(weight = 300)
~~~









---

![image-20220518231033566](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518231033566.png)

---





## 集群容错





---

![image-20220518234037870](https://cdn.jsdelivr.net/gh/fgcy-333/gitnote-images/image-20220518234037870.png)

---

集群容错模式：
Failover Cluster：失败重试。默认值。当出现失败，重试其它服务器 ，默认重试2次，使用 retries 配置。一般用于读操作

Failfast Cluster ：快速失败，只发起一次调用，失败立即报错。通常用于写操作

Failsafe Cluster ：失败安全，出现异常时，直接忽略。返回一个空结果【可用于日志】

Failback Cluster ：失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作【必须调用某个服务】

Forking Cluster ：并行调用多个服务器，只要一个成功即返回

Broadcast  Cluster ：广播调用所有提供者，逐个调用，任意一台报错则报错 【多个集群都需要同步的重要数据】



- 消费者

~~~java
    @Reference(cluster = "failover")//失败重试 
    private UserService userService;
~~~



## 服务降级

服务降级方式:

- mock=force:return null 

​		表示消费方对该服务的方法调用都直接返回null值,不发起远程调用。**用来屏蔽不重要服务不可用时对调用方的影响**

~~~java
	@Reference(mock = "force:return null")//服务降级 屏蔽userService的所有服务
    private UserService userService;
~~~





- mock=fail:return null 

​		表示消费方对该服务的方法调用在失败后,再返回null值,不抛异常。用来容忍不重要服务不稳定时对调用方的影响

~~~java
	@Reference(mock = "fail:return null")//服务降级 屏蔽userService的所有服务
    private UserService userService;
~~~



调用服务三次返回null，重试两次



