# 1、MySQL数据库的卸载

## 步骤一：软件的卸载准备

学习网络编程时，TCP/IP协议程序有服务器端和客户端。mysql这个数据库管理软件是使用TCP/IP协议。我们现在要卸载的是mysql的服务器端，它没有界面。

 【计算】-->右键-->【管理】-->【服务】-->【mysql的服务】-->【停止】

![image-20211127131123822](MySQL8.0_安装和使用文档.assets/image-20211127131123822.png)

## 步骤二：软件的卸载

### **方式一：通过控制面板卸载**

![image-20210727180032098](MySQL8.0_安装和使用文档.assets/image-20210727180032098.png)

![image-20211127105018580](MySQL8.0_安装和使用文档.assets/image-20211127105018580.png)

### 方式二：通过mysql8的安装向导卸载

#### 1、双击mysql8的安装向导

![image-20211127111201381](MySQL8.0_安装和使用文档.assets/image-20211127111201381.png)

#### 2、取消更新

![image-20211127111141171](MySQL8.0_安装和使用文档.assets/image-20211127111141171.png)

![image-20211127111248477](MySQL8.0_安装和使用文档.assets/image-20211127111248477.png)

#### 3、选择要卸载的mysql服务器软件的具体版本

![image-20211127111636535](MySQL8.0_安装和使用文档.assets/image-20211127111636535.png)

![image-20211127111756888](MySQL8.0_安装和使用文档.assets/image-20211127111756888.png)

#### 4、确认删除数据目录

![image-20211127111904711](MySQL8.0_安装和使用文档.assets/image-20211127111904711.png)

#### 5、执行删除

![image-20211127112049612](MySQL8.0_安装和使用文档.assets/image-20211127112049612.png)

![image-20211127112146641](MySQL8.0_安装和使用文档.assets/image-20211127112146641.png)

#### 6、完成删除

![image-20211127112224983](MySQL8.0_安装和使用文档.assets/image-20211127112224983.png)

![image-20211127112303454](MySQL8.0_安装和使用文档.assets/image-20211127112303454.png)

## 步骤三：清理残余文件（部分同学需要）

如果卸载后还有残余文件，先对残余文件进行清理后再安装。

（1）服务目录：mysql服务的安装目录

（2）数据目录：如果没有指定过默认在C:\ProgramData\MySQL

如果自己单独指定过，就找到自己的数据目录，例如安装时指定过如下目录：

![img](MySQL8.0_安装和使用文档.assets/clip_image002.jpg)

## 步骤四：清理服务列表中的服务名

如果在windows操作系统，卸载后mysql后，服务没有卸载干净，可以通过系统管理员在cmd命令行删除服务。

![image-20211127131325742](MySQL8.0_安装和使用文档.assets/image-20211127131325742.png)

```
sc  delete  服务名
```

![image-20211127131515955](MySQL8.0_安装和使用文档.assets/image-20211127131515955.png)

## 步骤五：清理原来的环境变量

找到path环境变量，将其中关于mysql的环境变量删除即，**切记<font color='red'>不要</font>把整个path删除。**

例如：删除  D:\ProgramFiles\MySQL\MySQLServer8.0_Server\bin;  这个部分

![image-20211127113140430](MySQL8.0_安装和使用文档.assets/image-20211127113140430.png)

![image-20211127113205093](MySQL8.0_安装和使用文档.assets/image-20211127113205093.png)

![image-20211127113258108](MySQL8.0_安装和使用文档.assets/image-20211127113258108.png)

![image-20211127113327805](MySQL8.0_安装和使用文档.assets/image-20211127113327805.png)

## 步骤六：清理注册表（选做，反复安装不成功的，可以尝试）

如何打开注册表编辑器：在系统的搜索框中输入regedit

* HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\Eventlog\Application\MySQL服务 目录删除

* HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\MySQL服务 目录删除

* HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\Eventlog\Application\MySQL服务 目录删除

* HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\MySQL服务 目录删除

* HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Eventlog\Application\MySQL服务目录删除

* HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\MySQL服务删除

> 注册表中的ControlSet001,ControlSet002,不一定是001和002,可能是ControlSet005、006之类



# 2、MySQL数据库的安装（如果安装失败请看7）

<font color='red'>**注意：**</font>

<font color='red'>**必须用系统管理员身份运行mysql安装程序。**</font>

<font color='red'>**安装目录切记不要用中文。**</font>



## 步骤一：双击mysql8的安装向导

![image-20211127111201381](MySQL8.0_安装和使用文档.assets/image-20211127111201381.png)

## 步骤二：分为首次安装和再安装

### 1、首次安装

#### （1）如果是首次安装mysql系列的产品，需要先安装mysql产品的安装向导

![](MySQL8.0_安装和使用文档.assets/微信图片_20211127130718.jpg)

#### （2）选择安装模式

![image-20211128175722806](MySQL8.0_安装和使用文档.assets/image-20211128175722806.png)



### 2、不是首次安装

#### （1）取消更新（如果电脑上有mysql相关软件才有）

![image-20211127113631758](MySQL8.0_安装和使用文档.assets/image-20211127113631758.png)



![image-20211127111248477](MySQL8.0_安装和使用文档.assets/image-20211127111248477.png)

#### （2）选择Add安装

![image-20211127113738546](MySQL8.0_安装和使用文档.assets/image-20211127113738546.png)

## 步骤三：选择要安装的产品

![image-20211127114653481](MySQL8.0_安装和使用文档.assets/image-20211127114653481.png)

![image-20211127114719245](MySQL8.0_安装和使用文档.assets/image-20211127114719245.png)

![image-20211127114744905](MySQL8.0_安装和使用文档.assets/image-20211127114744905.png)

## 步骤四：设置软件安装目录<font color='red'>（切记服务安装目录不要有中文字符，否则有问题）</font>

![image-20211127115035455](MySQL8.0_安装和使用文档.assets/image-20211127115035455.png)

![image-20211127115150647](MySQL8.0_安装和使用文档.assets/image-20211127115150647.png)

![image-20211127115242110](MySQL8.0_安装和使用文档.assets/image-20211127115242110.png)

![image-20211127115529359](MySQL8.0_安装和使用文档.assets/image-20211127115529359.png)

![image-20211127115719270](MySQL8.0_安装和使用文档.assets/image-20211127115719270.png)

## 步骤五：执行安装

![image-20211127115748337](MySQL8.0_安装和使用文档.assets/image-20211127115748337.png)

![image-20211127115812289](MySQL8.0_安装和使用文档.assets/image-20211127115812289.png)

## 步骤六：完成安装

![image-20211127115844966](MySQL8.0_安装和使用文档.assets/image-20211127115844966.png)

## 步骤七：准备设置

![image-20211127120041368](MySQL8.0_安装和使用文档.assets/image-20211127120041368.png)

# 3、MySQL实例初始化和设置

## 步骤一：选择安装的电脑类型、设置端口号

![image-20211127120247934](MySQL8.0_安装和使用文档.assets/image-20211127120247934.png)

![image-20211127120515458](MySQL8.0_安装和使用文档.assets/image-20211127120515458.png)

## 步骤二：选择mysql账号密码加密规则

在MySQL 5.x中默认的身份认证插件为“mysql_native_password”。

在MySQL 8.x中，默认的身份认证插件是“caching_sha2_password”，替代了之前的“mysql_native_password”。

![image-20211127120743104](MySQL8.0_安装和使用文档.assets/image-20211127120743104.png)

## 步骤三：设置root账户密码

![image-20211127121133127](MySQL8.0_安装和使用文档.assets/image-20211127121133127.png)

## 步骤四：设置mysql服务名和服务启动策略

如果电脑上可能安装多个版本mysql，请在服务名后面保留版本标识，例如：MySQL80，这样可以区别用哪个版本的mysql

![image-20211127121615732](MySQL8.0_安装和使用文档.assets/image-20211127121615732.png)

## 步骤五：执行设置（初始化mysql实例）

![image-20211127121929986](MySQL8.0_安装和使用文档.assets/image-20211127121929986.png)

![image-20211127122012517](MySQL8.0_安装和使用文档.assets/image-20211127122012517.png)

## 步骤六：完成设置

![image-20211127122037556](MySQL8.0_安装和使用文档.assets/image-20211127122037556.png)

![image-20211127122105747](MySQL8.0_安装和使用文档.assets/image-20211127122105747.png)

![image-20211127122124855](MySQL8.0_安装和使用文档.assets/image-20211127122124855.png)

![image-20211127130231815](MySQL8.0_安装和使用文档.assets/image-20211127130231815.png)

# 4、MySQL数据库服务的启动和停止

MySQL软件的服务器端必须先启动，客户端才可以连接和使用使用数据库。

如果接下来天天用，可以设置自动启动。

## 方式一：图形化方式

* 计算机（点击鼠标右键）》管理（点击）》服务和应用程序（点击）》服务（点击）》MySQL80（点击鼠标右键）==》启动或停止（点击）
* 控制面板（点击）》系统和安全（点击）》管理工具（点击）》服务（点击）》MySQL80（点击鼠标右键）==》启动或停止（点击）
* 任务栏（点击鼠标右键）》启动任务管理器（点击）》服务（点击）》MySQL80（点击鼠标右键）》启动或停止（点击）

## 方式二：命令行方式

必须是系统管理员才能运行下面的命令。

```cmd
启动 MySQL 服务命令：
net start MySQL80

停止 MySQL 服务命令：
net stop MySQL80
```

# 5、MySQL数据库环境变量的配置

如果运行mysql命令，报错如下错误，说明需要配置环境变量

![image-20211128172817265](MySQL8.0_安装和使用文档.assets/image-20211128172817265.png)

![image-20211127133531030](MySQL8.0_安装和使用文档.assets/image-20211127133531030.png)



| 环境变量名 | 操作 |                 环境变量值                  |
| :--------: | :--: | :-----------------------------------------: |
| MYSQL_HOME | 新建 | D:\ProgramFiles\MySQL\MySQLServer8.0_Server |
|    path    | 编辑 |              %MYSQL_HOME%\bin               |

或者直接

| 环境变量名 | 操作 |                   环境变量值                    |
| :--------: | :--: | :---------------------------------------------: |
|    path    | 编辑 | D:\ProgramFiles\MySQL\MySQLServer8.0_Server\bin |

![image-20211127165256909](MySQL8.0_安装和使用文档.assets/image-20211127165256909.png)

# 6、MySQL数据库客户端的登录

```java
MySQL服务器昨天已经装好了，默认在3306端口。

MySQL的客户端有哪些？
（1）cmd命令行
（2）mysql数据库管理系统的服务器本地有一个自带客户端，
只能以'root'@'localhost'用户从本地登录，只需要输入密码即可。
（3）可视化图形界面工具
SQLyog、Navicate、MySQL Front、DBeaver、MySQLWorkbench等
```





## 方式一：MySQL自带客户端

开始菜单==》所有程序==》MySQL==》MySQL Server 8.0==》MySQL 8.0 Command Line Client

![image-20211127163824213](MySQL8.0_安装和使用文档.assets/image-20211127163824213.png)

> 说明：仅限于root用户

## 方式二：cmd命令行客户端

**mysql -h 主机名 -P 端口号 -u 用户名 -p密码**

```sql
例如：mysql -h localhost -P 3306 -u root -proot   

-h：host 主机名/IP地址
-P：port端口号
-u：user 用户名
-p：password密码
```

注意：

（1）-p与密码之间不能有空格，其他参数名与参数值之间可以有空格也可以没有空格

```sql
mysql -hlocalhost -P3306 -uroot -proot
```

（2）密码建议在下一行输入

```sql
mysql -h localhost -P 3306 -u root -p
Enter password:****
```

（3）如果是连本机：-hlocalhost就可以省略，如果端口号没有修改：-P3306也可以省略

  简写成：

```sql
mysql -u root -p
Enter password:******
```

（4）如果输入mysql命令报“不是内部或外部命令”，把mysql安装目录的bin目录配置到环境变量path中

![image-20211127165424591](MySQL8.0_安装和使用文档.assets/image-20211127165424591.png)

## 方式三：可视化工具SQLyog

SQLyog是一款简介高效且功能强大的图形化数据库管理工具。这款工具是使用C++语言开发的。用户可以使用这款软件来有效地管理MySQL数据库。该工具可以方便地创建数据库、表、视图和索引等，还可以方便地进行插入、更新和删除等操作，同时可以方便地进行数据库、数据表的备份和还原。该工具不仅可以通过SQL文件进行大量文件的导入和导出，还可以导入和导出XML、HTML和CSV等多种格式的数据。使用SQLyog中文社区版进行演示，下载地址为https://github.com/webyog/sqlyog-community/wiki/Downloads。

使用SQLyog图形化界面工具连接MySQL数据库的操作步骤如下。

步骤1：数据库菜单→点击“创建新连接”选项→打开连接管理窗口。在连接管理窗口可以选择“新建”按钮创建新的连接，也可以直接连接已保存的连接，然后进行参数设置，需要输入MySQL服务器IP地址、端口号、用户名、密码以及要连接的数据库名称等，其中数据库名称如果不写表示显示该用户有权限查看和操作的全部数据库。设置完成后，可以单击右侧的“测试连接”按钮，测试是否成功，如果没有问题，单击“连接”按钮连接数据库。

![image-20211127165755722](MySQL8.0_安装和使用文档.assets/image-20211127165755722.png)

 步骤2：连接成功后，就可以对数据库进行管理和操作了。

![image-20211127165825204](MySQL8.0_安装和使用文档.assets/image-20211127165825204.png)

## 方式四：可视化工具DBeaver

DBeaver是一个通用的数据库管理工具和 SQL 客户端，支持所有流行的数据库：MySQL、PostgreSQL、SQLite、Oracle、DB2、SQL Server、 Sybase、MS Access、Teradata、 Firebird、Apache Hive、Phoenix、Presto等。DBeaver比大多数的SQL管理工具要轻量，而且支持中文界面。DBeaver社区版作为一个免费开源的产品，和其他类似的软件相比，在功能和易用性上都毫不逊色。下载地址：https://dbeaver.io/download/。DBeaver的下载安装都非常简单，唯一需要注意是DBeaver 是用Java编程语言开发的，所以需要拥有 JDK（Java Development ToolKit）环境。JDK是 Java 语言开发工具包，也是整个Java 的核心，包括运行环境、工具以及基础类库。如果电脑上没有JDK，在选择安装DBeaver组件时，勾选“Include Java”即可。

![image-20211127170034226](MySQL8.0_安装和使用文档.assets/image-20211127170034226.png)

使用DBeaver图形化界面工具连接MySQL数据库也很简单，操作步骤如下。

步骤1：数据库菜单→单击“新建连接”选项→打开连接管理窗口。选择要连接的数据库类型，单击“下一步”按钮。注意，如果提示缺少相应的数据库驱动，则直接根据提示下载即可。

![image-20211127170101530](MySQL8.0_安装和使用文档.assets/image-20211127170101530.png)

步骤2：填写连接参数，需要指定要连接的MySQL服务器的IP地址，端口号，用户名密码、MySQL服务器版本等，如图2-43所示。填写完成之后，可以单击“测试链接”按钮，查看是否连接成功。如果没问题，单击“完成”按钮即可。

![image-20211127170119285](MySQL8.0_安装和使用文档.assets/image-20211127170119285.png)

步骤3：连接成功后，就可以对数据库进行管理和操作了。

![image-20211127170133578](MySQL8.0_安装和使用文档.assets/image-20211127170133578.png)

## 方式四：可视化工具MySQL Workbench

MySQL Workbench是MySQL官方提供的图形化界面管理工具，完全支持MySQL5.0以上的版本。它是著名的数据库设计工具DBDesigner4的继任者。MySQL Workbench 为数据库管理员、程序开发者和系统规划师提供可视化设计、模型建立、以及数据库管理功能。它包含了用于创建复杂的数据建模ER模型，正向和逆向数据库工程，也可以用于执行通常需要花费大量时间的、难以变更和管理的文档任务。MySQL工作台可在Windows、Linux和Mac上使用。随MySQL8一起发布的MySQL Workbench 8，可以直接连接MySQL8，不需要修改加密方式。当你创建、修改数据库及其表等数据库对象时，或针对表中的数据的添加、修改、删除操作时，可以提供生成SQL功能，对已经存在的表、函数等也可以提供生成SQL功能，这对于开发人员，或者初学者SQL的读者来说是个福音。下载地址：https://dev.mysql.com/downloads/workbench/。

​    使用MySQL Workbench图形化界面工具连接MySQL数据库的操作步骤如下。

步骤1：Database菜单→单击“Manage Server Connections”选项→打开连接管理窗口，如图2-37所示。在连接管理窗口中可以选择“New”按钮创建新的连接，也可以在左边“已有连接列表”中选择某个连接进行参数设置。需要指定要连接的MySQL服务器的IP地址，端口号，用户名和密码等。参数设置完成之后，可以单击“Test Connection”按钮测试某个连接是否可以连接成功。如果测试成功，可以看到“Successfully made the MySQL connection”的提示对话框。

![image-20211127170215686](MySQL8.0_安装和使用文档.assets/image-20211127170215686.png)

![image-20211127170227692](MySQL8.0_安装和使用文档.assets/image-20211127170227692.png)

步骤2：Database菜单→单击“Connect to Database”选项→打开数据库连接窗口。选择之前创建并设置的某个连接后，单击“OK”按钮进行连接登录MySQL数据库。

![image-20211127170246731](MySQL8.0_安装和使用文档.assets/image-20211127170246731.png)

步骤3：连接成功后，就可以对MySQL数据库进行管理了。

![image-20211127170303177](MySQL8.0_安装和使用文档.assets/image-20211127170303177.png)

# 7、安装失败问题

## 安装问题1：无法打开MySQL8.0软件安装包？

​    在运行MySQL8.0软件安装包之前，用户需要确保系统中已经安装了.Net Framework相关软件，如果缺少此软件，将不能正常地安装MySQL8.0软件

![image-20211127170411358](MySQL8.0_安装和使用文档.assets/image-20211127170411358.png)

解决方案：到这个地址https://www.microsoft.com/en-us/download/details.aspx?id=42642下载Microsoft .NET Framework 4.5并安装后，再去安装MySQL。

## 安装问题2：需要C++库

另外，还要确保Windows Installer正常安装。Windows上安装MySQL8.0需要操作系统提前已安装好Microsoft Visual C++ 2015-2019。

![image-20211127170434387](MySQL8.0_安装和使用文档.assets/image-20211127170434387.png)

## 安装问题3：丢失MSVCP140.dll

![image-20211127170442613](MySQL8.0_安装和使用文档.assets/image-20211127170442613.png)

解决方案同样是，提前到微软官网https://support.microsoft.com/en-us/topic/the-latest-supported-visual-c-downloads-2647da03-1eea-4433-9aff-95f26a218cc0下载相应的环境。

如果电脑提示需要更新操作系统，请做好更新后再安装。

# 8、可视化工具连接MySQL8问题

有些图形界面工具，特别是旧版本的图形界面工具，在连接MySQL8时出现“Authentication plugin 'caching_sha2_password' cannot be loaded”错误。

![image-20211127170917407](MySQL8.0_安装和使用文档.assets/image-20211127170917407.png)

出现这个原因是MySQL8之前的版本中加密规则是mysql_native_password，而在MySQL8之后，加密规则是caching_sha2_password。

解决问题方法有两种：

第一种是升级图形界面工具版本。

第二种是把MySQL8用户登录密码加密规则还原成mysql_native_password。

第二种解决方案如下，用命令行登录MySQL数据库之后，执行如下命令修改用户密码加密规则并更新用户密码，这里修改用户名为“root@localhost”的用户密码规则为“mysql_native_password”，密码值为“123456”。

```sql
#修改'root'@'localhost'用户的密码规则和密码
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '密码'; 
#刷新权限
FLUSH PRIVILEGES;
```

![image-20211127171046998](MySQL8.0_安装和使用文档.assets/image-20211127171046998.png)

# 9、mysql8忘记root用户密码

当出现忘记root用户密码的情况时，如果此时有其他用户拥有系统库mysql的user表的UPDATE权限，可以由其他用户通过SET语句修改root用户密码。但是如果遇到一种特殊情况，此时没有其他用户，或者其他用户没有系统库mysql的user表的UPDATE权限，也没有GRANT（给用户授权）的权限，那么怎么处理呢？操作步骤如下：

1.首先停止mysql的服务
2.新建一个文本文件，文本文件中就写一条修改密码的语句

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

例如在D盘根目录下新建一个文本文件“root_newpass.txt”，文件内容就上面一条语句。

![image-20211128193356182](MySQL8.0_安装和使用文档.assets/image-20211128193356182.png)

![image-20211128193420899](MySQL8.0_安装和使用文档.assets/image-20211128193420899.png)

3.使用管理员权限运行cmd命令行，运行以下命令：

```mysql
mysqld --defaults-file="D:\ProgramFiles\MySQL\MySQLServer8.0_Data\my.ini" --init-file="d:\root_newpass.txt"
```

注意：my.ini文件的路径看你自己的安装路径，找数据目录

![image-20211128193734236](MySQL8.0_安装和使用文档.assets/image-20211128193734236.png)

上面命令意思就是初始化启动一次数据库，并运行这个修改密码的文件。效果演示如下：

![image-20211128193623962](MySQL8.0_安装和使用文档.assets/image-20211128193623962.png)

上面的命令执行后，就像卡住了一样，这就是启动MySQL服务了。

4.然后按CTRL+C结束上面的运行命令

![image-20211129084057307](MySQL8.0_安装和使用文档.assets/image-20211129084057307.png)

5.最后重新启动MySQL服务，用新密码登录即可

# 10、修改用户密码（记得原密码）

在命令行可以使用mysqladmin命令修改用户密码，命令格式如下：

```mysql
mysqladmin -u 用户名 -h 主机名  -p password "新密码"
Enter password:输入旧密码
```

![image-20211128194355818](MySQL8.0_安装和使用文档.assets/image-20211128194355818.png)

# 11、修改用户密码（不记得原密码）

例如：“root”用户登录后，修改用户名为“shangguigu1”，主机名为“localhost”的用户的密码为“atguigu”。

```mysql
SET PASSWORD FOR 'shangguigu1'@'localhost' = '新密码';
```

![image-20211128194603636](MySQL8.0_安装和使用文档.assets/image-20211128194603636.png)

