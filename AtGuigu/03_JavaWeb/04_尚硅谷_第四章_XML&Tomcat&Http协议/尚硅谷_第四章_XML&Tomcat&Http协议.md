# 第四章 XML&Tomcat&Http协议

## 学习目标

* 了解配置文件的作用
* 了解常见的配置文件类型
* 掌握properties文件的编写规范
* 掌握xml文件的编写
* 了解xml文件的约束  
* 掌握xml文件的解析
* 掌握Tomcat的安装
* 掌握Tomcat的使用
* 掌握Tomcat在IDEA中的使用
* 了解HTTP协议的发展历程
* 了解HTTP1.0和HTTP1.1的区别
* 掌握请求报文和响应报文的格式和内容

## 1. xml解析(了解)

### 1.1 配置文件

#### 1.1.1 配置文件的作用

配置文件是用于给应用程序提供配置参数以及初始化设置的一些有特殊格式的文件

#### 1.1.2 常见的配置文件类型

1. properties文件,例如druid连接池就是使用properties文件作为配置文件
2. XML文件,例如Tomcat就是使用XML文件作为配置文件
3. YAML文件,例如SpringBoot就是使用YAML作为配置文件
4. json文件,通常用来做文件传输，也可以用来做前端或者移动端的配置文件

### 1.2 properties文件

#### 1.2.1 文件示例

```properties
atguigu.jdbc.url=jdbc:mysql://192.168.198.100:3306/bj1026
atguigu.jdbc.driver=com.mysql.cj.jdbc.Driver
atguigu.jdbc.username=root
atguigu.jdbc.password=root
```

#### 1.2.2 语法规范

- 由键值对组成
- 键和值之间的符号是等号
- 每一行都必须顶格写，前面不能有空格之类的其他符号

### 1.3 XML文件

#### 1.3.1 文件示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">

    <!-- 配置SpringMVC前端控制器 -->
    <servlet>
        <servlet-name>dispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <!-- 在初始化参数中指定SpringMVC配置文件位置 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:spring-mvc.xml</param-value>
        </init-param>

        <!-- 设置当前Servlet创建对象的时机是在Web应用启动时 -->
        <load-on-startup>1</load-on-startup>

    </servlet>
    <servlet-mapping>
        <servlet-name>dispatcherServlet</servlet-name>

        <!-- url-pattern配置斜杠表示匹配所有请求 -->
        <!-- 两种可选的配置方式：
                1、斜杠开头：/
                2、包含星号：*.atguigu
             不允许的配置方式：前面有斜杠，中间有星号
                /*.app
         -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```

#### 1.3.2 概念介绍

XML是e<span style="color:blue;font-weight:bold;">X</span>tensible <span style="color:blue;font-weight:bold;">M</span>arkup <span style="color:blue;font-weight:bold;">L</span>anguage的缩写，翻译过来就是<span style="color:blue;font-weight:bold;">可扩展标记语言</span>。所以很明显，XML和HTML一样都是标记语言，也就是说它们的基本语法都是标签。

**可扩展**

<span style="color:blue;font-weight:bold;">可扩展</span>三个字<span style="color:blue;font-weight:bold;">表面上</span>的意思是XML允许<span style="color:blue;font-weight:bold;">自定义格式</span>。但是别美，这<span style="color:blue;font-weight:bold;">不代表</span>你<span style="color:blue;font-weight:bold;">可以随便写</span>。

![images](images/img002.png)

在XML基本语法规范的基础上，你使用的那些第三方应用程序、框架会通过设计<span style="color:blue;font-weight:bold;">『XML约束』</span>的方式<span style="color:blue;font-weight:bold;">『强制规定』</span>配置文件中可以写什么和怎么写，规定之外的都不可以写。

XML基本语法这个知识点的定位是：我们不需要从零开始，从头到尾的一行一行编写XML文档，而是在第三方应用程序、框架<span style="color:blue;font-weight:bold;">已提供的配置文件</span>的基础上<span style="color:blue;font-weight:bold;">修改</span>。要改成什么样取决于你的需求，而怎么改取决于<span style="color:blue;font-weight:bold;">XML基本语法</span>和<span style="color:blue;font-weight:bold;">具体的XML约束</span>。

#### 1.3.3 XML的基本语法

- XML文档声明

这部分基本上就是固定格式，要注意的是文档声明一定要从第一行第一列开始写

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

- 根标签

根标签有且只能有一个。

- 标签关闭
  - 双标签：开始标签和结束标签必须成对出现。
  - 单标签：单标签在标签内关闭。
- 标签嵌套
  - 可以嵌套，但是不能交叉嵌套。
- 注释不能嵌套
- 标签名、属性名建议使用小写字母
- 属性
  - 属性必须有值
  - 属性值必须加引号，单双都行

看到这里大家一定会发现XML的基本语法和HTML的基本语法简直如出一辙。其实这不是偶然的，XML基本语法+HTML约束=HTML语法。在逻辑上HTML确实是XML的子集。

```html
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN"
"http://www.w3.org/TR/html4/loose.dtd">
```

从HTML4.01版本的文档类型声明中可以看出，这里使用的DTD类型的XML约束。也就是说http://www.w3.org/TR/html4/loose.dtd这个文件定义了HTML文档中可以写哪些标签，标签内可以写哪些属性，某个标签可以有什么样的子标签。

#### 1.3.4 XML的约束(稍微了解)

将来我们主要就是根据XML约束中的规定来编写XML配置文件，而且会在我们编写XML的时候根据约束来提示我们编写, 而XML约束主要包括DTD和Schema两种。

- DTD

- Schema

Schema约束要求我们一个XML文档中，所有标签，所有属性都必须在约束中有明确的定义。

下面我们以web.xml的约束声明为例来做个说明：

```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
```

#### 1.3.5 XML解析

##### ① XML解析的作用

用Java代码读取xml中的数据

##### ② DOM4J的使用步骤

1. 导入jar包 dom4j.jar
2. 创建解析器对象(SAXReader)
3. 解析xml 获得Document对象
4. 获取根节点RootElement
5. 获取根节点下的子节点

##### ③ DOM4J的API介绍

1. 创建SAXReader对象

```java
SAXReader saxReader = new SAXReader();
```

2. 解析XML获取Document对象: 需要传入要解析的XML文件的字节输入流

```java
Document document = reader.read(inputStream);
```

3. 获取文档的根标签

```java
Element rootElement = documen.getRootElement()
```

4. 获取标签的子标签

```java
//获取所有子标签
List<Element> sonElementList = rootElement.elements();
//获取指定标签名的子标签
List<Element> sonElementList = rootElement.elements("标签名");
```

5. 获取标签体内的文本

```java
String text = element.getText();
```

6. 获取标签的某个属性的值

```java
String value = element.AttributeValue("属性名");
```

## 2. Tomcat(最重要)

### 2.1 Web服务器

- Web服务器通常由硬件和软件共同构成。

  - 硬件：电脑，提供服务供其它客户电脑访问

    ![1561995738943](images/1561995738943.png)

  - 软件：电脑上安装的服务器软件，安装后能提供服务给网络中的其他计算机，**将本地文件映射成一个虚拟的url地址供网络中的其他人访问。**

- Web服务器主要用来接收客户端发送的请求和响应客户端请求。

- 常见的JavaWeb服务器：

  - **Tomcat（Apache）**：当前应用最广的JavaWeb服务器
  - JBoss（Redhat红帽）：支持JavaEE，应用比较广EJB容器 –> SSH轻量级的框架代替
  - GlassFish（Orcale）：Oracle开发JavaWeb服务器，应用不是很广
  - Resin（Caucho）：支持JavaEE，应用越来越广
  - Weblogic（Orcale）：要钱的！支持JavaEE，适合大型项目
  - Websphere（IBM）：要钱的！支持JavaEE，适合大型项目

### 2.2 Tomcat服务器

#### 2.2.1 Tomcat简介

Tomcat是Apache 软件基金会（Apache Software Foundation）的Jakarta 项目中的一个核心项目，由Apache、Sun 和其他一些公司及个人共同开发而成。由于有了Sun 的参与和支持，最新的Servlet 和JSP 规范总是能在Tomcat 中得到体现，因为Tomcat 技术先进、性能稳定，而且免费，因而深受Java 爱好者的喜爱并得到了部分软件开发商的认可，成为目前比较流行的Web 应用服务器。

#### 2.2.2 Tomcat下载

- Tomcat官方网站：<http://tomcat.apache.org/>
- 安装版：需要安装，一般不考虑使用。
- 解压版: 直接解压缩使用，我们使用的版本。
- **因为tomcat服务器软件需要使用java环境，所以需要正确配置JAVA_HOME**。

#### 2.2.3 Tomcat的版本

- 版本：企业用的比较广泛的是7.0和8.0的。授课我们使用8.0。
- tomcat6以下都不用了，所以我们从tomcat6开始比较：
  - tomcat6 	支持servlet2.5、jsp2.1、el
  - tomcat7 	支持servlet3.0、jsp2.2、el2.2、websocket1.1
  - tomcat8	 支持servlet3.1、jsp2.3、el3.0、websocket1.1
  - tomcat9	 支持servlet4.0、jsp2.3、el3.0、websocket1.1

#### 2.2.4 安装

解压apache-tomcat-8.5.27-windows-x64.zip到**非中文无空格**目录中

![1557499687108](images/1557499687108.png)

D:\developer_tools\apache-tomcat-8.5.27，这个目录下直接包含Tomcat的bin目录，conf目录等，我们称之为**Tomcat的安装目录或根目录**。

- bin：该目录下存放的是二进制可执行文件，如果是安装版，那么这个目录下会有两个exe文件：tomcat6.exe、tomcat6w.exe，前者是在控制台下启动Tomcat，后者是弹出GUI窗口启动Tomcat；如果是解压版，那么会有startup.bat和shutdown.bat文件，startup.bat用来启动Tomcat，但需要先配置JAVA_HOME环境变量才能启动，shutdawn.bat用来停止Tomcat；
- conf：这是一个非常非常重要的目录，这个目录下有四个最为重要的文件：
  - **server.xml：配置整个服务器信息。例如修改端口号。默认HTTP请求的端口号是：8080**
  - tomcat-users.xml：存储tomcat用户的文件，这里保存的是tomcat的用户名及密码，以及用户的角色信息。可以按着该文件中的注释信息添加tomcat用户，然后就可以在Tomcat主页中进入Tomcat Manager页面了；
  - web.xml：部署描述符文件，这个文件中注册了很多MIME类型，即文档类型。这些MIME类型是客户端与服务器之间说明文档类型的，如用户请求一个html网页，那么服务器还会告诉客户端浏览器响应的文档是text/html类型的，这就是一个MIME类型。客户端浏览器通过这个MIME类型就知道如何处理它了。当然是在浏览器中显示这个html文件了。但如果服务器响应的是一个exe文件，那么浏览器就不可能显示它，而是应该弹出下载窗口才对。MIME就是用来说明文档的内容是什么类型的！
  - context.xml：对所有应用的统一配置，通常我们不会去配置它。
- lib：Tomcat的类库，里面是一大堆jar文件。如果需要添加Tomcat依赖的jar文件，可以把它放到这个目录中，当然也可以把应用依赖的jar文件放到这个目录中，这个目录中的jar所有项目都可以共享之，但这样你的应用放到其他Tomcat下时就不能再共享这个目录下的jar包了，所以建议只把Tomcat需要的jar包放到这个目录下；
- logs：这个目录中都是日志文件，记录了Tomcat启动和关闭的信息，如果启动Tomcat时有错误，那么异常也会记录在日志文件中。
- temp：存放Tomcat的临时文件，这个目录下的东西可以在停止Tomcat后删除！
- **webapps：存放web项目的目录，其中每个文件夹都是一个项目**；如果这个目录下已经存在了目录，那么都是tomcat自带的项目。其中ROOT是一个特殊的项目，在地址栏中访问：http://127.0.0.1:8080，没有给出项目目录时，对应的就是ROOT项目。<http://localhost:8080/examples，进入示例项目。其中examples>就是项目名，即文件夹的名字。
- work：运行时生成的文件，最终运行的文件都在这里。通过webapps中的项目生成的！可以把这个目录下的内容删除，再次运行时会生再次生成work目录。当客户端用户访问一个JSP文件时，Tomcat会通过JSP生成Java文件，然后再编译Java文件生成class文件，生成的java和class文件都会存放到这个目录下。
- LICENSE：许可证。
- NOTICE：说明文件。



#### 2.2.5 配置

启动Tomcat前，需要配置如下的环境变量

① 配置JAVA_HOME环境变量

![1557501069603](images/1557501069603.png)

② 在Path环境变量中加入JAVA_HOME\bin目录

![1561975042021](images/1561975042021.png)

#### 2.2.6 启动 

在命令行中运行**catalina run**或者 Tomcat解压目录下**双击startup.bat** 启动Tomcat服务器，在浏览器地址栏访问如下地址进行测试

**http://localhost:8080**

![1557501275330](images/1557501275330.png)



如果启动失败，查看如下的情况：

情况一：如果双击startup.bat后窗口一闪而过，请查看JAVA_HOME是否配置正确。

> startup.bat会调用catalina.bat，而catalina.bat会调用setclasspath.bat，setclasspath.bat会使用JAVA_HOME环境变量，所以我们必须在启动Tomcat之前把JAVA_HOME配置正确。

情况二：如果启动失败，提示端口号被占用，则将默认的8080端口修改为其他未使用的值，例如8989等。

【方法】 打开：解压目录\conf\server.xml，找到第一个Connector标签，修改port属性

> web服务器在启动时，实际上是监听了本机上的一个端口，当有客户端向该端口发送请求时，web服务器就会处理请求。但是如果不是向其所监听的端口发送请求，web服务器不会做任何响应。例如：Tomcat启动监听了8989端口，而访问的地址是<http://localhost:8080>，将不能正常访问。

#### 2.2.7 在IDEA中创建Tomcat

在IDEA中配置好Tomcat后，可以直接通过IDEA控制Tomcat的启动和停止，而不用再去操作startup.bat和shutdown.bat。

① 点击File-->Settings  或者直接点击图标

![1641350780468](images\1641350780468.png)

下一步

![1557501489531](images/1592226868323.png)

下一步：

![1557501625107](images/1592226961401.png)



### 2.3 动态Web工程部署与测试

#### 2.3.1 创建**动态**Web工程

![1557506289562](images/1592227051056.png)

接着：

![1592227253541](images/1592227253541.png)

![1592227294518](images/1592227294518.png)



#### 2.3.2 开发项目目录结构说明

- **src：存放Java源代码的目录。**

- web：存放的是需要部署到服务器的文件

  - WEB-INF：**这个目录下的文件，是不能被客户端直接访问的。**
    - **lib：用于存放该工程用到的库。粘贴过来以后**
    - **web.xml：web工程的配置文件，完成用户请求的逻辑名称到真正的servlet类的映射。**

  > 凡是客户端能访问的资源(*.html或 *.jpg)必须跟WEB-INF在同一目录，即放在Web根目录下的资源，从客户端是可以通过URL地址直接访问的。

![1592227512557](images/1592227512557.png)

#### 2.3.3 tomcat实例的基础设置

由于每次创建项目随之创建的tomcat实例名字都类似，所以建议修改一下tomcat实例的名称

![1592228000124](images/1592228000124.png)

在如下界面进行基础设置

![1592228151885](images/1592228151885.png)



## 3. HTTP协议简介

![img](images/timg.jpg)

- **HTTP 超文本传输协议** (HTTP-Hypertext transfer protocol)，是一个属于应用层的面向对象的协议，由于其简捷、快速的方式，适用于分布式超媒体信息系统。它于1990年提出，经过十几年的使用与发展，得到不断地完善和扩展。**它是一种详细规定了浏览器和万维网服务器之间互相通信的规则**，通过因特网传送万维网文档的数据传送协议。

- 客户端与服务端通信时传输的内容我们称之为**报文**。**HTTP协议就是规定报文的格式。**

- HTTP就是一个通信规则，这个规则规定了客户端发送给服务器的报文格式，也规定了服务器发送给客户端的报文格式。实际我们要学习的就是这两种报文。客户端发送给服务器的称为”**请求报文**“，服务器发送给客户端的称为”**响应报文**“。

- 类比于生活中案例：① 毕业租房，签署租房协议，规范多方需遵守的规则。②与远方的朋友的写信。信封的规范。

  实际互联网：

  - 客户端 与 服务端进行通信。比如：用户 ---> 访问京东（就是一个数据传输的过程），数据传输需要按照一种协议去传输。就如，用户给服务器写信；服务器给用户回信。有格式：协议。HTTP协议规定通信规则。规定互联网之间如何传输数据。
    - 信：报文。
    - 写信：用户给服务器写信，用户给服务器发请求。把发的请求所有数据，请求报文
    - 回信：服务器回信给用户，回给浏览器。把服务器响应浏览器的所有数据，响应报文



### 3.1 HTTP协议的发展历程

- 超文本传输协议的前身是世外桃源(Xanadu)项目，超文本的概念是泰德·纳尔森(Ted Nelson)在1960年代提出的。进入哈佛大学后，纳尔森一直致力于超文本协议和该项目的研究，但他从未公开发表过资料。1989年，**蒂姆·伯纳斯·李**(Tim Berners Lee)在CERN(欧洲原子核研究委员会 = European Organization for Nuclear Research)担任软件咨询师的时候，开发了一套程序，**奠定了万维网(WWW = World Wide Web)**的基础。1990年12月，超文本在CERN首次上线。1991年夏天，继Telnet等协议之后，超文本转移协议成为互联网诸多协议的一分子。
- 当时，**Telnet协议**解决了一台计算机和另外一台计算机之间一对一的控制型通信的要求。邮件协议解决了一个发件人向少量人员发送信息的通信要求。**文件传输协议**解决一台计算机从另外一台计算机批量获取文件的通信要求，但是它不具备一边获取文件一边显示文件或对文件进行某种处理的功能。**新闻传输协议**解决了一对多新闻广播的通信要求。而**超文本要解决的通信要求是**：在一台计算机上获取并显示存放在多台计算机里的文本、数据、图片和其他类型的文件；它包含两大部分：超文本转移协议和超文本标记语言(HTML)。HTTP、HTML以及浏览器的诞生给互联网的普及带来了飞跃。

### 3.2 HTTP协议的会话方式

- 浏览器与服务器之间的通信过程要经历四个步骤

![1557672342250](images/1557672342250.png)

- 浏览器与WEB服务器的连接过程是短暂的，每次连接只处理一个请求和响应。对每一个页面的访问，浏览器与WEB服务器都要建立一次单独的连接。
- 浏览器到WEB服务器之间的所有通讯都是完全独立分开的请求和响应对。



### 3.3 HTTP1.0和HTTP1.1的区别

在HTTP1.0版本中，浏览器请求一个带有图片的网页，会由于下载图片而与服务器之间开启一个新的连接；但在HTTP1.1版本中，允许浏览器在拿到当前请求对应的全部资源后再断开连接，提高了效率。

![1557672415271](images/1557672415271.png)

### 3.4 不同浏览器监听HTTP操作

#### 3.4.1 IE浏览器中F12可查看

![1561904433377](images/1561904433377.png)

#### 3.4.2 Chrome中F12可查看

![1561904121561](images/1561904121561.png)

#### 3.4.3 FireFox中F12可查看

![1561904179928](images/1561904179928.png)

### 3.5 报文

#### 3.5.1 报文格式

![1557672592385](images/1557672592385.png)

- 报文：
  - 请求报文：浏览器发给服务器
  - 响应报文：服务器发回给浏览器

#### 3.5.2 请求报文

##### ① 报文格式 (4部分)

- 请求首行（**请求行**）；
- 请求头信息（**请求头**）；
- 空行；
- 请求体；

##### ② GET请求

1、由于请求参数在请求首行中已经携带了，所以没有请求体，也没有请求空行
2、请求参数拼接在url地址中，地址栏可见[url?name1=value1&name2=value2]，不安全
3、由于参数在地址栏中携带，所以由大小限制[地址栏数据大小一般限制为4k]，只能携带纯文本
4、get请求参数只能上传文本数据
5、没有请求体。所以封装和解析都快，效率高， 浏览器默认提交的请求都是get请求[比如：① 地址栏输入url地址回车，②点击超链接a ， ③ form表单默认方式...]

- **请求首行**

```http
GET /05_web_tomcat/login_success.html?username=admin&password=123213 HTTP/1.1

请求方式 	访问的服务器中的资源路径？get请求参数	协议版本
```

- **请求头**

```http
Host: localhost:8080   主机虚拟地址
Connection: keep-alive 长连接
Upgrade-Insecure-Requests: 1  请求协议的自动升级[http的请求，服务器却是https的，浏览器自动会将请求协议升级为https的]
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.75 Safari/537.36
- 用户系统信息
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
- 浏览器支持的文件类型
Referer: http://localhost:8080/05_web_tomcat/login.html
- 当前页面的上一个页面的路径[当前页面通过哪个页面跳转过来的]：   可以通过此路径跳转回上一个页面， 广告计费，防止盗链
Accept-Encoding: gzip, deflate, br
- 浏览器支持的压缩格式
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
- 浏览器支持的语言
```



##### ③ POST请求

- POST请求要求将form标签的method的属性设置为post

![1557672877007](images/1557672877007.png)

1、POST请求有请求体，而GET请求没有请求体。
2、post请求数据在请求体中携带，请求体数据大小没有限制，可以用来上传所有内容[文件、文本]
3、只能使用post请求上传文件
4、post请求报文多了和请求体相关的配置[请求头]
5、地址栏参数不可见，相对安全
6、post效率比get低

- 请求首行

```http
POST /05_web_tomcat/login_success.html HTTP/1.1
```

- 请求头

```http
Host: localhost:8080
Connection: keep-alive
Content-Length: 31 		-请求体内容的长度
Cache-Control: max-age=0  -无缓存
Origin: http://localhost:8080
Upgrade-Insecure-Requests: 1  -协议的自动升级
Content-Type: application/x-www-form-urlencoded   -请求体内容类型[服务器根据类型解析请求体参数]
User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.75 Safari/537.36
Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Referer: http://localhost:8080/05_web_tomcat/login.html
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en-US;q=0.8,en;q=0.7
Cookie:JSESSIONID-
```

- 请求空行
- 请求体：浏览器提交给服务器的内容

```http
username=admin&password=1232131
```



#### 3.5.3 响应报文

##### ① 报文格式(4部分)

- 响应首行（**响应行**）；
- 响应头信息（**响应头**）；
- 空行；
- **响应体；**

##### ② 具体情况

- **响应首行：**

  ```http
  HTTP/1.1 200 OK
  
  说明：响应协议为HTTP1.1，响应状态码为200，表示请求成功； 
  ```

- **响应头：**

  ```http
  Server: Apache-Coyote/1.1   服务器的版本信息
  Accept-Ranges: bytes
  ETag: W/"157-1534126125811"
  Last-Modified: Mon, 13 Aug 2018 02:08:45 GMT
  Content-Type: text/html    响应体数据的类型[浏览器根据类型解析响应体数据]
  Content-Length: 157   响应体内容的字节数
  Date: Mon, 13 Aug 2018 02:47:57 GMT  响应的时间，这可能会有8小时的时区差
  ```

- **响应空行**

- **响应体**

  ```html
  <!--需要浏览器解析使用的内容[如果响应的是html页面，最终响应体内容会被浏览器显示到页面中]-->
  
  <!DOCTYPE html>
  <html>
  	<head>
  		<meta charset="UTF-8">
  		<title>Insert title here</title>
  	</head>
  	<body>
  		恭喜你，登录成功了...
  	</body>
  </html>
  ```

##### ③ 响应码

响应码对浏览器来说很重要，它告诉浏览器响应的结果。比较有代表性的响应码如下：

- **200：**请求成功，浏览器会把响应体内容（通常是html）显示在浏览器中；

- **404：**请求的资源没有找到，说明客户端错误的请求了不存在的资源；

  ![1561905338054](images/1561905338054.png)

- **500：**请求资源找到了，但服务器内部出现了错误；

  ![1561905801690](images/1561905801690.png)

  ![1561905834066](images/1561905834066.png)

- **302：**重定向，当响应码为302时，表示服务器要求浏览器重新再发一个请求，服务器会发送一个响应头Location，它指定了新请求的URL地址；

除此之外，其它一些响应码如下：

```
200 - 服务器成功返回网页 
404 - 请求的网页不存在 
503 - 服务不可用 
详细分解：

1xx（临时响应） 
表示临时响应并需要请求者继续执行操作的状态代码。

代码 说明 
100 （继续） 请求者应当继续提出请求。服务器返回此代码表示已收到请求的第一部分，正在等待其余部分。 
101 （切换协议） 请求者已要求服务器切换协议，服务器已确认并准备切换。

2xx （成功） 
表示成功处理了请求的状态代码。

代码 说明 
200 （成功） 服务器已成功处理了请求。通常，这表示服务器提供了请求的网页。 
201 （已创建） 请求成功并且服务器创建了新的资源。 
202 （已接受） 服务器已接受请求，但尚未处理。 
203 （非授权信息） 服务器已成功处理了请求，但返回的信息可能来自另一来源。 
204 （无内容） 服务器成功处理了请求，但没有返回任何内容。 
205 （重置内容） 服务器成功处理了请求，但没有返回任何内容。 
206 （部分内容） 服务器成功处理了部分 GET 请求。

3xx （重定向） 
表示要完成请求，需要进一步操作。 通常，这些状态代码用来重定向。

代码 说明 
300 （多种选择） 针对请求，服务器可执行多种操作。服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。 
301 （永久移动） 请求的网页已永久移动到新位置。服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置。 
302 （临时移动） 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。 
303 （查看其他位置） 请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。 
304 （未修改） 自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。 
305 （使用代理） 请求者只能使用代理访问请求的网页。如果服务器返回此响应，还表示请求者应使用代理。 
307 （临时重定向） 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。

4xx（请求错误） 
这些状态代码表示请求可能出错，妨碍了服务器的处理。

代码 说明 
400 （错误请求） 服务器不理解请求的语法。 
401 （未授权） 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。 
403 （禁止） 服务器拒绝请求。 
404 （未找到） 服务器找不到请求的网页。 
405 （方法禁用） 禁用请求中指定的方法。 
406 （不接受） 无法使用请求的内容特性响应请求的网页。 
407 （需要代理授权） 此状态代码与 401（未授权）类似，但指定请求者应当授权使用代理。 
408 （请求超时） 服务器等候请求时发生超时。 
409 （冲突） 服务器在完成请求时发生冲突。服务器必须在响应中包含有关冲突的信息。 
410 （已删除） 如果请求的资源已永久删除，服务器就会返回此响应。 
411 （需要有效长度） 服务器不接受不含有效内容长度标头字段的请求。 
412 （未满足前提条件） 服务器未满足请求者在请求中设置的其中一个前提条件。 
413 （请求实体过大） 服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。 
414 （请求的 URI 过长） 请求的 URI（通常为网址）过长，服务器无法处理。 
415 （不支持的媒体类型） 请求的格式不受请求页面的支持。 
416 （请求范围不符合要求） 如果页面无法提供请求的范围，则服务器会返回此状态代码。 
417 （未满足期望值） 服务器未满足”期望”请求标头字段的要求。

5xx（服务器错误） 
这些状态代码表示服务器在尝试处理请求时发生内部错误。 这些错误可能是服务器本身的错误，而不是请求出错。

代码 说明 
500 （服务器内部错误） 服务器遇到错误，无法完成请求。 
501 （尚未实施） 服务器不具备完成请求的功能。例如，服务器无法识别请求方法时可能会返回此代码。 
502 （错误网关） 服务器作为网关或代理，从上游服务器收到无效响应。 
503 （服务不可用） 服务器目前无法使用（由于超载或停机维护）。通常，这只是暂时状态。 
504 （网关超时） 服务器作为网关或代理，但是没有及时从上游服务器收到请求。 
505 （HTTP 版本不受支持） 服务器不支持请求中所用的 HTTP 协议版本。

HttpWatch状态码Result is

200 - 服务器成功返回网页，客户端请求已成功。 
302 - 对象临时移动。服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。 
304 - 属于重定向。自上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。 
401 - 未授权。请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。 
404 - 未找到。服务器找不到请求的网页。 
2xx - 成功。表示服务器成功地接受了客户端请求。 
3xx - 重定向。表示要完成请求，需要进一步操作。客户端浏览器必须采取更多操作来实现请求。例如，浏览器可能不得不请求服务器上的不同的页面，或通过代理服务器重复该请求。 
4xx - 请求错误。这些状态代码表示请求可能出错，妨碍了服务器的处理。 
5xx - 服务器错误。表示服务器在尝试处理请求时发生内部错误。 这些错误可能是服务器本身的错误，而不是请求出错。
```



