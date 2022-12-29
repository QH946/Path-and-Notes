# 项目部署_持续集成

## 1 今日内容介绍

### 1.1 什么是持续集成

持续集成（ Continuous integration ， 简称 CI ）指的是，频繁地（一天多次）将代码集成到主干

![image-20210802000658790](项目部署_持续集成.assets\image-20210802000658790.png)

**持续集成的组成要素**

一个自动构建过程， 从检出代码、 编译构建、 运行测试、 结果记录、 测试统计等都是自动完成的， 无需人工干预。

一个代码存储库，即需要版本控制软件来保障代码的可维护性，同时作为构建过程的素材库，一般使用SVN或Git。

一个持续集成服务器， Jenkins 就是一个配置简单和使用方便的持续集成服务器。

### 1.2 持续集成的好处

1、降低风险，由于持续集成不断去构建，编译和测试，可以很早期发现问题，所以修复的代价就少；
2、对系统健康持续检查，减少发布风险带来的问题；
3、减少重复性工作；
4、持续部署，提供可部署单元包；
5、持续交付可供使用的版本；
6、增强团队信心；

### 1.3 今日内容

![image-20210802000829722](项目部署_持续集成.assets\image-20210802000829722.png)



## 2 软件开发模式

### 2.1 软件开发生命周期

软件开发生命周期又叫做SDLC（Software Development Life Cycle），它是集合了计划、开发、测试和部署过程的集合。如下图所示 ：

![image-20210802011508487](项目部署_持续集成.assets\image-20210802011508487.png)

- 需求分析

  这是生命周期的第一阶段，根据项目需求，团队执行一个可行性计划的分析。项目需求可能是公司内部或者客户提出的。这阶段主要是对信息的收集，也有可能是对现有项目的改善和重新做一个新的项目。还要分析项目的预算多长，可以从哪方面受益及布局，这也是项目创建的目标。

- 设计

  第二阶段就是设计阶段，系统架构和满意状态（就是要做成什么样子，有什么功能），和创建一个项目计划。计划可以使用图表，布局设计或者文字的方式呈现。

- 实现

  第三阶段就是实现阶段，项目经理创建和分配工作给开者，开发者根据任务和在设计阶段定义的目标进行开发代码。依据项目的大小和复杂程度，可以需要数月或更长时间才能完成。

- 测试

  测试人员进行代码测试 ，包括功能测试、代码测试、压力测试等。

- 进化

  最后进阶段就是对产品不断的进化改进和维护阶段，根据用户的使用情况，可能需要对某功能进行修改，bug修复，功能增加等。

### 2.2 软件开发瀑布模型

瀑布模型是最著名和最常使用的软件开发模型。瀑布模型就是一系列的软件开发过程。它是由制造业繁衍出来的。一个高度化的结构流程在一个方向上流动，有点像生产线一样。在瀑布模型创建之初，没有其它开发的模型，有很多东西全靠开发人员去猜测，去开发。这样的模型仅适用于那些简单的软件开发， 但是已经不适合现在的开发了。

下图对软件开发模型的一个阐述。

![image-20210802011525024](项目部署_持续集成.assets\image-20210802011525024.png)

| 优势                                       | 劣势                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| 简单易用和理解                             | 各个阶段的划分完全固定，阶段之间产生大量的文档，极大地增加了工作量。 |
| 当前一阶段完成后，您只需要去关注后续阶段。 | 由于开发模型是线性的，用户只有等到整个过程的末期才能见到开发成果，从而增加了开发风险。 |
| 为项目提供了按阶段划分的检查节点           | 瀑布模型的突出缺点是不适应用户需求的变化。                   |

### 2.3 软件的敏捷开发

- 什么是敏捷开发？

  敏捷开发（Agile Development） 的核心是迭代开发（Iterative Development） 与 增量开发（Incremental Development）。

- 何为迭代开发？

  对于大型软件项目，传统的开发方式是采用一个大周期（比如一年）进行开发，整个过程就是一次"大开发"；迭代开发的方式则不一样，它将开发过程拆分成多个小周期，即一次"大开发"变成多次"小开发"，每次小开发都是同样的流程，所以看上去就好像重复在做同样的步骤。

  举例来说，SpaceX 公司想造一个大推力火箭，将人类送到火星。但是，它不是一开始就造大火箭，而是先造一个最简陋的小火箭 Falcon 1。结果，第一次发射就爆炸了，直到第四次发射，才成功进入轨道。然后，开发了中型火箭 Falcon 9，九年中发射了70次。最后，才开发 Falcon 重型火箭。如果SpaceX 不采用迭代开发，它可能直到现在还无法上天。

- 何为增量开发？

  软件的每个版本，都会新增一个用户可以感知的完整功能。也就是说，按照新增功能来划分迭代。

  举例来说，房产公司开发一个10栋楼的小区。如果采用增量开发的模式，该公司第一个迭代就是交付一号楼，第二个迭代交付二号楼......每个迭代都是完成一栋完整的楼。而不是第一个迭代挖好10栋楼的地基，第二个迭代建好每栋楼的骨架，第三个迭代架设屋顶......

- 敏捷开发如何迭代？

  虽然敏捷开发将软件开发分成多个迭代，但是也要求，每次迭代都是一个完整的软件开发周期，必须按照软件工程的方法论，进行正规的流程管理。

![image-20210802011540379](项目部署_持续集成.assets\image-20210802011540379.png)

- 敏捷开发有什么好处？

  - 早期交付

    敏捷开发的第一个好处，就是早期交付，从而大大降低成本。 还是以上一节的房产公司为例，如果按照传统的"瀑布开发模式"，先挖10栋楼的地基、再盖骨架、然后架设屋顶，每个阶段都等到前一个阶段完成后开始，可能需要两年才能一次性交付10栋楼。也就是说，如果不考虑预售，该项目必须等到两年后才能回款。 敏捷开发是六个月后交付一号楼，后面每两个月交付一栋楼。因此，半年就能回款10%，后面每个月都会有现金流，资金压力就大大减轻了。

  - 降低风险

    敏捷开发的第二个好处是，及时了解市场需求，降低产品不适用的风险。 请想一想，哪一种情况损失比较小：10栋楼都造好以后，才发现卖不出去，还是造好第一栋楼，就发现卖不出去，从而改进或停建后面9栋楼？

## 3 Jenkins安装配置

### 3.1 Jenkins介绍

![image-20210802011553923](项目部署_持续集成.assets\image-20210802011553923.png)

Jenkins  是一款流行的开源持续集成（Continuous Integration）工具，广泛用于项目开发，具有自动化构建、测试和部署等功能。官网：  http://jenkins-ci.org/。

Jenkins的特征：

- 开源的 Java语言开发持续集成工具，支持持续集成，持续部署。
- 易于安装部署配置：可通过 yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理。
- 消息通知及测试报告：集成 RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告。
- 分布式构建：支持 Jenkins能够让多台计算机一起构建/测试。
- 文件识别： Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等。
- 丰富的插件支持：支持扩展插件，你可以开发适合自己团队使用的工具，如 git，svn，maven，docker等。



Jenkins安装和持续集成环境配置

![image-20210802011607894](项目部署_持续集成.assets\image-20210802011607894.png)

1 ）首先，开发人员每天进行代码提交，提交到Git仓库

2）然后，Jenkins作为持续集成工具，使用Git工具到Git仓库拉取代码到集成服务器，再配合JDK，Maven等软件完成代码编译，代码测试与审查，测试，打包等工作，在这个过程中每一步出错，都重新再执行一次整个流程。

3）最后，Jenkins把生成的jar或war包分发到测试服务器或者生产服务器，测试人员或用户就可以访问应用。



### 3.2 Docker安装

```shell
# 1、yum 包更新到最新 
yum update
# 看到下面信息后，输入y敲回车
```

![image-20220730233724622](assets\image-20220730233724622.png)

![image-20220730233932320](assets\image-20220730233932320.png)

```
# 2、安装需要的软件包， yum-util 提供yum-config-manager功能，另外两个是devicemapper驱动依赖的 
yum install -y yum-utils device-mapper-persistent-data lvm2
```

![image-20220730233950452](assets\image-20220730233950452.png)

```
# 3、 设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

![image-20220730234012270](assets\image-20220730234012270.png)

```
# 4、 安装docker，出现输入的界面都按 y 
yum install -y docker-ce
```

启动

设置开机启动：

```sh
systemctl enable docker
```

启动docker

```sh
systemctl start docker
```



### 3.3 Maven安装

**步骤一：解压安装包**

找到资料中的apache-maven-3.6.1.zip，上传到/usr/local/maven/目录

```shell
cd /usr/local
unzip -o apache-maven-3.6.1.zip 
```

**步骤二：环境变量配置**

```sh
vi /etc/profile
```

增加：

```sh
export MAVEN_HOME=/usr/local/apache-maven-3.6.1
export PATH=$PATH:$MAVEN_HOME/bin
```

如果权限不够，则需要增加当前目录的权限

```shell
chmod 777 /usr/local/apache-maven-3.6.1/bin/mvn
```

刷新配置

```shell
source /etc/profile
```

**步骤三：检测安装结果**

```shell
mvn -v
```

看到如下信息代表安装成功

```shell
Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
Maven home: /usr/local/apache-maven-3.6.1
Java version: 1.8.0_332, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.332.b09-1.el7_9.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.45.1.el7.x86_64", arch: "amd64", family: "unix"
```

**步骤四：修改镜像仓库配置**

```sh
vi /usr/local/apache-maven-3.6.1/conf/settings.xml
```

**需要把本机的仓库打包上传到服务器上（不上传会自动下载）**

然后指定上传后的仓库配置

```xml
<localRepository>/usr/local/apache-maven-3.6.1/repository_new</localRepository>
```

**步骤五：修改阿里云镜像**

找到mirrors标签，添加配置

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

**步骤六：**上传依赖仓库

```shell
 cd /usr/local/apache-maven-3.6.1/
 
 选择本地的repository_new.zip文件，并上传
 
 unzip -o repository_new.zip
```



### 3.4 Jenkins安装

```shell
docker run -d \
--privileged=true -u=root \
--name jenkins \
-p 9999:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/local/apache-maven-3.6.1:/usr/local/apache-maven-3.6.1 \
-v /home/docker/jenkins/jenkins_home:/var/jenkins_home \
-v /usr/bin/docker:/usr/bin/docker \
-v /etc/localtime:/etc/localtime:ro \
-e JAVA_OPTS="-Xms256m -Xmx512m -Xss1024K -XX:PermSize=128m -XX:MaxPermSize=256m" \
jenkins/jenkins:lts-jdk11
```

**注意**

如果使用阿里云/腾讯云/百度云，记得开启 9999、50000端口的访问白名单

### 3.5 Jenkins配置

**步骤一：**

访问http://你的ip:9999，等待Jenkins初始化（这里等待比较久，可能需要5~10分钟）

![image-20220625020252910](assets\image-20220625020252910.png)

**步骤二：**输入密码

在linux中输入 docker logs jenkins，得到以下信息 4d343b563b1a4ca29a805eaa3ffe3e0e 就是密码

```shell
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

4d343b563b1a4ca29a805eaa3ffe3e0e

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************

```

输入密码

![image-20210802011625800](项目部署_持续集成.assets\image-20210802011625800.png)

**步骤三：**安装推荐插件

![2](assets\2.png)按默认设置，把建议的插件都安装上

![image-20210802011638639](项目部署_持续集成.assets\image-20210802011638639.png)

这一步等待时间较长， 耐心等待。

**步骤四：**创建管理员用户（你自己设置，账号密码别忘记了）

![image-20210802011653454](项目部署_持续集成.assets\image-20210802011653454.png)

**步骤五：**配置访问地址

![image-20210802011707013](项目部署_持续集成.assets\image-20210802011707013.png)



![6](assets\6.png)



**步骤六：**配置完成之后， 会进行重启， 之后可以看到管理后台

![7](assets\7.png)



### 3.6 Jenkins插件安装

在实现持续集成之前， 需要确保以下插件安装成功。

- Maven Integration plugin： Maven 集成管理插件。
- Docker plugin： Docker集成插件。
- GitLab Plugin： GitLab集成插件。
- Publish Over SSH：远程文件发布插件。
- SSH: 远程脚本执行插件。

**安装方法：**

**步骤一：**进入【系统管理】-【插件管理】

**步骤二：**点击标签页的【可选插件】，在过滤框中搜索插件名称

![image-20210802011740056](项目部署_持续集成.assets\image-20210802011740056.png)

**步骤三：**勾选插件， 点击直接安装即可。

>注意，如果没有安装按钮，需要更改配置
>
>在安装插件的高级配置中，修改升级站点的连接为：http://updates.jenkins.io/update-center.json   保存
>
>![image-20210802011758588](项目部署_持续集成.assets\image-20210802011758588.png)
>
>



### 3.7 Jenkins工具配置

**步骤一：**进入【系统管理】--> 【全局工具配置】

![image-20210802011944005](项目部署_持续集成.assets\image-20210802011944005.png)

**步骤二：**指定MAVEN 目录

- 选择新增Maven
- 把自动安装点掉
- 在Name输入3.6.1
- 在MAVEN_HOME里输入 /usr/local/apache-maven-3.6.1

![image-20220625150315544](assets\image-20220625150315544.png)

**步骤三：**指定DOCKER目录

- 新增Docker
- 点掉自动安装
- Name输入least
- Installation root输入/usr/bin

![image-20220625150401852](assets\image-20220625150401852.png)

**步骤四：**如果git没有配置，就按照下图配置即可

![image-20220801102100597](assets\image-20220801102100597.png)


## 4 后端项目部署

### 4.1 多环境切换

在项目开发部署的过程中，一般都会有三套项目环境

- Development ：开发环境

- Production ：生产环境

- Test ：测试环境

例如：开发环境的mysql连接的是本地，生产环境需要连接线上的mysql环境



### 4.2 多环境切换-微服务中多环境配置

1.在微服务中的bootstrap.yml中新增配置(spring.profiles.active:dev)

```yaml
server:
  port: 51801
spring:
  application:
    name: leadnews-user
  cloud:
    nacos:
      discovery:
        server-addr: 192.168.200.130:8848
      config:
        server-addr: 192.168.200.130:8848
        file-extension: yml
  profiles:
    active: dev
```

2.在nacos的配置中心中新增各个环境的配置文件，例如user微服务中新增

修改bootstrap.yml 添加内容

```properties
spring:
  profiles:
    active: dev
```

创建对应的nacos的多环境配置：

- leadnews-user-dev.yml
- leadnews-user-prod.yml

 

![image-20210623143417530](项目部署_持续集成.assets\image-20210623143417530.png)

 ![image-20210623143557710](项目部署_持续集成.assets\image-20210623143557710.png)

注意事项：

其中DataID属性命名有规范：

- prefix，默认使用${spring.application.name}，也可以通过spring.cloud.nacos.config.prefix来配置。
- spring.profile.active，即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}
- file-exetension，为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。

### 4.3 整体思路

目标：把黑马头条的app端相关的微服务部署到192.168.200.100这台服务器上

![image-20210802003955971](项目部署_持续集成.assets\image-20210802003955971.png)

![image-20210802004007699](项目部署_持续集成.assets\image-20210802004007699.png)

### 4.4 服务集成Docker配置

目标：部署的每一个微服务都是先创建docker镜像后创建对应容器启动

方式一：本地微服务打包以后上传到服务器，编写Dockerfile文件完成。

方式二：使用dockerfile-maven-plugin插件，可以直接把微服务创建为镜像使用（更省事）



**服务集成Docker配置**

![image-20210802004133439](项目部署_持续集成.assets\image-20210802004133439.png)

每个微服务都引入该依赖,以heima-leadnews-user微服务为例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>heima-leadnews-service</artifactId>
        <groupId>com.heima</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>heima-leadnews-user</artifactId>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <docker.image>docker_storage</docker.image>
    </properties>

    <build>
        <finalName>heima-leadnews-user</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <executions>
                    <execution>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.7.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.3.6</version>
                <configuration>
                    <repository>${docker.image}/${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

**服务集成Dockerfile文件**

```dockerfile
# 设置JAVA版本
FROM java:8
# 指定存储卷, 任何向/tmp写入的信息都不会记录到容器存储层
VOLUME /tmp
# 拷贝运行JAR包
ARG JAR_FILE
COPY ${JAR_FILE} app.jar
# 设置JVM运行参数， 这里限定下内存大小，减少开销
ENV JAVA_OPTS="\
-server \
-Xms256m \
-Xmx512m \
-XX:MetaspaceSize=256m \
-XX:MaxMetaspaceSize=512m"
#空参数，方便创建容器时传参
ENV PARAMS=""
# 入口点， 执行JAVA运行命令
ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar $PARAMS"]
```

**提交GIT**

完成以上操作之后，需要记得把所有改动提交到你的git仓库。



### 4.5 jenkins基础依赖打包配置

在微服务运行之前需要在本地仓库中先去install所依赖的jar包，所以第一步应该是从git中拉取代码，并且把基础的依赖部分安装到仓库中

1，父工程heima-leadnews

![image-20210802004744531](项目部署_持续集成.assets\image-20210802004744531.png)

2，找到自己指定的git仓库，设置用户名和密码

![image-20210802004803711](项目部署_持续集成.assets\image-20210802004803711.png)

3，把基础依赖信息安装到服务器上的本地仓库

![image-20210802004818581](项目部署_持续集成.assets\image-20210802004818581.png)

4，执行

执行日志，部分截图，下面是从git中拉取代码

```shell
Started by user root
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/heima-leadnews
The recommended git tool is: NONE
using credential c5b6865e-8234-4ada-86a6-8b8ed43b9fd2
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/heima-leadnews/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://gitee.com/tongdl/leadnews.git # timeout=10
Fetching upstream changes from https://gitee.com/tongdl/leadnews.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_ASKPASS to set credentials 
 > git fetch --tags --force --progress -- https://gitee.com/tongdl/leadnews.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
Checking out Revision 81f6f75abdbe2a1e3b812cd5762d34831875f935 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 81f6f75abdbe2a1e3b812cd5762d34831875f935 # timeout=10
Commit message: "删除文件 heima-leadnews-service/heima-leadnews-schedule/target"
 > git rev-list --no-walk f0e77c09790422c79f5c74e1e682f65a6068896f # timeout=10
```

执行日志，部分截图，编译打包

```shell
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ heima-freemarker-starter ---
[INFO] Deleting /var/jenkins_home/workspace/heima-leadnews/heima-leadnews-basic/heima-freemarker-starter/target
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ heima-freemarker-starter ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /var/jenkins_home/workspace/heima-leadnews/heima-leadnews-basic/heima-freemarker-starter/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.6.1:compile (default-compile) @ heima-freemarker-starter ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 3 source files to /var/jenkins_home/workspace/heima-leadnews/heima-leadnews-basic/heima-freemarker-starter/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ heima-freemarker-starter ---
[INFO] Not copying test resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.6.1:testCompile (default-testCompile) @ heima-freemarker-starter ---
[INFO] Not compiling test sources
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ heima-freemarker-starter ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ heima-freemarker-starter ---
```

部分执行日志如下：

```shell
[INFO] heima-leadnews-common .............................. SUCCESS [  2.890 s]
[INFO] heima-leadnews-utils ............................... SUCCESS [  0.998 s]
[INFO] heima-leadnews-gateway ............................. SUCCESS [  0.006 s]
[INFO] heima-leadnews-app-gateway ......................... SUCCESS [  1.276 s]
[INFO] heima-leadnews-wemedia-gateway ..................... SUCCESS [  0.664 s]
[INFO] heima-leadnews-admin-gateway ....................... SUCCESS [  0.160 s]
[INFO] heima-leadnews-model ............................... SUCCESS [  2.104 s]
[INFO] heima-leadnews-feign-api ........................... SUCCESS [  0.521 s]
[INFO] heima-leadnews-service ............................. SUCCESS [  0.009 s]
[INFO] heima-leadnews-user ................................ SUCCESS [  1.352 s]
[INFO] heima-leadnews-basic ............................... SUCCESS [  0.006 s]
[INFO] heima-file-starter ................................. SUCCESS [  0.567 s]
[INFO] heima-leadnews-article ............................. SUCCESS [  1.002 s]
[INFO] heima-aliyun-green-starter ......................... SUCCESS [  0.733 s]
[INFO] heima-ocr-starter .................................. SUCCESS [  0.515 s]
[INFO] heima-leadnews-wemedia ............................. SUCCESS [  1.243 s]
[INFO] heima-leadnews-schedule ............................ SUCCESS [  0.590 s]
[INFO] heima-leadnews-search .............................. SUCCESS [ 18.255 s]
[INFO] heima-freemarker-starter ........................... SUCCESS [  0.292 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  35.858 s
[INFO] Finished at: 2022-06-25T07:25:24Z
[INFO] ------------------------------------------------------------------------
Finished: SUCCESS
```



### 4.6 jenkins微服务打包配置

所有微服务打包的方式类似，以heima-leadnews-user微服务为例

**步骤一：**新建任务

![image-20220625153227827](assets\image-20220625153227827.png)



**步骤二：**找到自己指定的git仓库，设置用户名和密码

<img src="assets\image-20220625153343180.png" style="float:left" />



**步骤三：**执行maven命令

![image-20220625153440985](assets\image-20220625153440985.png)



输入构建命令：

```java
clean install -Dmaven.test.skip=true  dockerfile:build -f heima-leadnews-service/heima-leadnews-user/pom.xml
```

![image-20220625221415745](assets\image-20220625221415745.png)

<font color='red'>注意：根据自己的实际代码路径配置</font>

-Dmaven.test.skip=true  跳过测试

dockerfile:build 启动dockerfile插件构建容器

-f heima-leadnews-user/pom.xml 指定需要构建的文件（必须是pom）



**步骤四：**并执行shell脚本

![image-20220625153005390](assets\image-20220625153005390.png)

加入如下代码

```java
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]
 then
 #删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 
 # 启动docker服务
docker run -d --net=host -e PARAMS="--spring.profiles.active=prod"  --name $JOB_NAME docker_storage/$JOB_NAME
```

**这里要注意了：**$JOB_NAME 是 jenkins任务的名称，这个地方要保证和项目的maven名称一致。

<img src="assets\image-20220625153051564.png" style="float:left" />



**步骤五：**执行日志

拉取代码

```shell
Started by user root
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/heima-leadnews
The recommended git tool is: NONE
using credential c5b6865e-8234-4ada-86a6-8b8ed43b9fd2
 > git rev-parse --resolve-git-dir /var/jenkins_home/workspace/heima-leadnews/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://gitee.com/tongdl/leadnews.git # timeout=10
Fetching upstream changes from https://gitee.com/tongdl/leadnews.git
 > git --version # timeout=10
 > git --version # 'git version 2.30.2'
using GIT_ASKPASS to set credentials 
 > git fetch --tags --force --progress -- https://gitee.com/tongdl/leadnews.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
Checking out Revision 81f6f75abdbe2a1e3b812cd5762d34831875f935 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 81f6f75abdbe2a1e3b812cd5762d34831875f935 # timeout=10
Commit message: "删除文件 heima-leadnews-service/heima-leadnews-schedule/target"
 > git rev-list --no-walk f0e77c09790422c79f5c74e1e682f65a6068896f # timeout=10
```

编译打包

```shell
[INFO] -------------------< com.heima:heima-leadnews-user >--------------------
[INFO] Building heima-leadnews-user 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-clean-plugin:3.1.0:clean (default-clean) @ heima-leadnews-user ---
[INFO] Deleting /var/jenkins_home/workspace/heima-leadnews-user/heima-leadnews-service/heima-leadnews-user/target
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:resources (default-resources) @ heima-leadnews-user ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] Copying 1 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ heima-leadnews-user ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 5 source files to /var/jenkins_home/workspace/heima-leadnews-user/heima-leadnews-service/heima-leadnews-user/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.1.0:testResources (default-testResources) @ heima-leadnews-user ---
[INFO] Not copying test resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ heima-leadnews-user ---
[INFO] Not compiling test sources
[INFO] 
[INFO] --- maven-surefire-plugin:2.22.2:test (default-test) @ heima-leadnews-user ---
[INFO] Tests are skipped.
[INFO] 
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ heima-leadnews-user ---
```

构建镜像

```shell
[INFO] Step 2/7 : VOLUME /tmp
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 297dbf54e4bf
[INFO] Step 3/7 : ARG JAR_FILE
[INFO] 
[INFO]  ---> Using cache
[INFO]  ---> 3b4012fe0068
[INFO] Step 4/7 : COPY ${JAR_FILE} app.jar
[INFO] 
[INFO]  ---> b4d2770dc33c
[INFO] Step 5/7 : ENV JAVA_OPTS="-server -Xms256m -Xmx512m -XX:MetaspaceSize=256m -XX:MaxMetaspaceSize=512m"
[INFO] 
[INFO]  ---> Running in 1e456d206361
[INFO] Removing intermediate container 1e456d206361
[INFO]  ---> a48d1f64fe49
[INFO] Step 6/7 : ENV PARAMS=""
[INFO] 
[INFO]  ---> Running in cad0f239d595
[INFO] Removing intermediate container cad0f239d595
[INFO]  ---> e7306d259d8d
[INFO] Step 7/7 : ENTRYPOINT ["sh","-c","java -jar $JAVA_OPTS /app.jar $PARAMS"]
[INFO] 
[INFO]  ---> Running in 7ad436232865
[INFO] Removing intermediate container 7ad436232865
[INFO]  ---> 094ac319b9a1
[INFO] Successfully built 094ac319b9a1
[INFO] Successfully tagged docker_storage/heima-leadnews-user:latest
[INFO] 
[INFO] Detected build of image with id 094ac319b9a1
[INFO] Building jar: /var/jenkins_home/workspace/heima-leadnews-user/heima-leadnews-service/heima-leadnews-user/target/heima-leadnews-user-docker-info.jar
[INFO] Successfully built docker_storage/heima-leadnews-user:latest
```

清理容器，创建新的容器

```shell
+ docker ps -a -f name=heima-leadnews-user --format {{.ID}}
+ [ -n 7c1cc90d5b76 ]
+ docker ps -a -f name=heima-leadnews-user --format {{.ID}}
+ docker rm -f 7c1cc90d5b76
7c1cc90d5b76
+ docker image prune -f
Deleted Images:
deleted: sha256:06667391f9c81b82edd69603527572c41f7d017cd2fdd76ddbca8efe776cfd8f
deleted: sha256:1426082be7f54596091f425333eb0fc95acbd772e9d991bd51d9b5afdc138eb9
deleted: sha256:3957ad6b1a9b226c6a38522a45ff9f196e33768bef5024759a70315c404c6825
deleted: sha256:3eb062d088cc4fb8f43e24ea44dfd7cf19f56dfbfeabba1f0a6a9f7356731c42
deleted: sha256:744e7ccc7d896604d0d8c55b72e3889de980056a8a9afe88a3a8c4e3c1dd56df

Total reclaimed space: 54.76MB
+ docker run -d --net=host -e PARAMS=--spring.profiles.active=prod --name heima-leadnews-user docker_storage/heima-leadnews-user
05b220d8b085e3561acd860975e72b31133aec0c1e0b2986b584f3f1a994617b
Finished: SUCCESS
```





### 4.7 部署服务到远程服务器上

目标：使用jenkins（192.168.200.100）把微服务打包部署到192.168.200.130服务器上

![image-20210802005538404](项目部署_持续集成.assets\image-20210802005538404.png)

#### 4.7.1 安装配置私有仓库

对于持续集成环境的配置，Jenkins会发布大量的微服务， 要与多台机器进行交互， 可以采用docker镜像的保存与导出功能结合SSH实现， 但这样交互繁琐，稳定性差， 而且不便管理， 这里我们通过搭建Docker的私有仓库来实现， 这个有点类似GIT仓库， 集中统一管理资源， 由客户端拉取或更新。

**步骤一：**下载最新Registry镜像

```shell
docker pull registry:latest
```

**步骤二：**启动Registry镜像服务

映射5000端口； -v是将Registry内的镜像数据卷与本地文件关联， 便于管理和维护Registry内的数据。

```shell
docker run -d \
-p 5000:5000 \
--name registry \
-v /usr/local/docker/registry:/var/lib/registry \
registry:latest
```

**步骤三：**查看仓库资源

访问地址：http://192.168.200.100:5000/v2/_catalog

![image-20210802005839314](assets\image-20210802005839314.png)

启动正常， 可以看到返回：（目前并没有上传镜像， 显示空数据）

```json
{"repositories":[]}
```

**步骤四：**配置Docker客户端

正常生产环境中使用， 要配置HTTPS服务， 确保安全，内部开发或测试集成的局域网环境，可以采用简便的方式， 不做安全控制。

先确保持续集成环境的机器已安装好Docker客户端， 然后做以下修改：

```shell
vi /lib/systemd/system/docker.service
```

修改内容：（13行左右）

```shell
// 原来的内容
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

// 修改后的内容
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --insecure-registry 192.168.200.130:5000
```

指向安装Registry的服务IP与端口。

**步骤五：**重启生效：

```shell
systemctl daemon-reload
systemctl restart docker.service
```

#### 4.7.2 jenkins系统配置远程服务器链接

需要添加凭证

位置：Manage Jenkins（系统管理）-->Manage CreDentials

![image-20220801110030731](assets\image-20220801110030731.png)



添加链接到服务器的用户名和密码（如果你用的是虚拟机，那么账号是root、密码是itcast）

![image-20220801110109724](assets\image-20220801110109724.png)



![image-20220625224341578](assets\image-20220625224341578.png)



![image-20210802010201146](项目部署_持续集成.assets\image-20210802010201146.png)



**配置远程服务器登录信息**

①：位置：Manage Jenkins（系统管理）-->Configure System（系统配置）

![image-20220801110558546](assets\image-20220801110558546.png)

②：往下翻找到SSH remote hosts（如果找不到这项，证明没有安装SSH插件）

<img src="assets\image-20220801112315138.png" style="float:left" />

③：添加远程登录信息

**Hostname：**输入你用来发布代码的服务器（虚拟机用192.168.200.130）

**Port：**22

**Credentials：**选择你上一步设置的凭证

![image-20220801112532641](assets\image-20220801112532641.png)



#### 4.7.4 jenkins项目创建与其他微服务相同

创建项目参考之前创建过的用户微服务

#### 4.7.5 设置参数

![image-20220625224717300](assets\image-20220625224717300.png)



添加字符参数

**名称：**docker_registry

**默认值：**你docker私仓所在的服务器，如：http://192.168.200.130:5000

![image-20220801120921332](assets\image-20220801120921332.png)



#### 4.7.6 构建执行Execute shell

![image-20210802010720937](项目部署_持续集成.assets\image-20210802010720937.png)

maven命令

```java
clean install -Dmaven.test.skip=true dockerfile:build -f heima-leadnews-service/heima-leadnews-article/pom.xml
```

shell脚本

```shell
image_tag=$docker_registry/docker_storage/$JOB_NAME
echo '================docker镜像清理================'
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]
 then
 #删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 

# 创建TAG
docker tag docker_storage/$JOB_NAME $image_tag
echo '================docker镜像推送================'
# 推送镜像
docker push $image_tag
# 删除TAG
docker rmi $image_tag
echo '================docker tag 清理 ================'
```

#### 4.7.7 在远程服务器上执行脚本

构建 > 新增构建步骤 > Execute shell script on remote host using ssh

如果看不到这个选项，证明的没有安装 **SSH插件**

![image-20220801111815666](assets\image-20220801111815666.png)

输入以下命令

![image-20210802010750809](项目部署_持续集成.assets\image-20210802010750809.png)

**SSH site：**root@192.168.200.130:22 （你要发布代码的服务器IP）

**Command：**远程服务器执行的shell脚本

```shell
echo '================拉取最新镜像================'
docker pull $docker_registry/docker_storage/$JOB_NAME

echo '================删除清理容器镜像================'
if [ -n  "$(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )" ]
 then
 #删除之前的容器
 docker rm -f $(docker ps -a -f  name=$JOB_NAME  --format '{{.ID}}' )
fi
 # 清理镜像
docker image prune -f 

echo '===============启动容器================'
docker run -d   --net=host -e PARAMS="--spring.profiles.active=prod" --name $JOB_NAME $docker_registry/docker_storage/$JOB_NAME
```

#### 4.7.8 构建完成以后，可以登录130服务器，查看是否有相关的镜像和容器

镜像

![image-20210802010824088](项目部署_持续集成.assets\image-20210802010824088.png)

容器

![image-20210802010835702](项目部署_持续集成.assets\image-20210802010835702.png)

### 4.8 联调测试

1.参考jenkins中heima-leadnews-user微服务把app端网关部署起来
