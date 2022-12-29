# Mysql高级

## mysql的b+树形结构

矮胖型的，有利于旋转目的是 两边平衡。

## 索引

### 创建与删除

```
create index idx_字段名字段名 on 表（使用的字段）
drop index idx_字段名字段名 on 表
show index from 表;
```



### 聚簇索引

主键id row_id,主键id尽量用自增id

### 唯一索引

unique

唯一索引，在插入数据的时候，会检查是否已经存在当前键值，会有额外开销

### 单值索引

以单个键值为索引创建数据

### 主键索引

### 复合索引

多个键值创建索引

## explain 查询优化

### type

一眼看到就明白啥意思

system：通过系统缓存查询数据

const：常亮 比如 where id=100

ref: 其他字段索引生效 带有where条件

range：范围查找

index:扫描全部索引树

all：没有用到索引，全盘扫描数据表



最差也要做到range级别

### key_len

可以感知到复合索引使用是否充分

### 索引创建与优化

#### 全值匹配我最爱

#### 最佳左前缀 记住

中间不能断

### 存储过程使用

plsql

redis缓存

### join多表关联查询

```
求各个门派对应的掌门人名称

#1 获取部门信息
select * from t_dept

#2 把ceoid 换成名字

select d.id ,d.deptName, e.name as ceo from t_dept d inner join t_emp e on d.ceo = e.id

```





所有掌门人的平均年龄

```
select avg(e.age) from t_dept d inner join t_emp e on d.ceo = e.id
```





求所有人，对应的掌门是谁

第一种 以CEO 入手

```


#1. 以CEO 入手，

select e.name ceoname, d.id ceodeptid  from t_emp e INNER JOIN t_dept d on e.id = d.ceo


#2. 所有人 关联到 ceodeptid上


select * from t_emp e join (


select e.name ceoname, d.id ceodeptid  from t_emp e INNER JOIN t_dept d on e.id = d.ceo

)  tmp

on

e.deptId = tmp.ceodeptid

```



第二种 从部门入手

```
#1 获取 所有人 所对应部门的 信息

select * from t_emp e LEFT JOIN t_dept d on e.deptid = d.id

#2 把ceo id 换成名称


select tmp.myname, e.name as ceoname from t_emp e inner JOIN 

(
select e.id empid,e.name myname, d.ceo as ceoid  from t_emp e LEFT JOIN t_dept d on e.deptid = d.id
) tmp

on

e.id = tmp.ceoid
```





第三种写法 从用户入手 直接join

```
# 第三种写法 从用户入手

select e.name,d.deptName,e2.name as ceoname
from t_emp e 
# 拿到部门相关信息
LEFT JOIN t_dept d on e.deptId = d.id

# 获取ceo的名字 

LEFT JOIN t_emp e2 on d.ceo = e2.id
# 20分钟 消化 

```



第四种 子查询

```
# 第四种子查询


select e.name,(select e2.name from t_emp e2 where e2.id = d.ceo  ) ceoname from t_emp e left join t_dept d on
 e.deptid = d.id
```



#1、列出自己的掌门比自己年龄小的人员

```

#1 拿到所有人员对应的部门信息，需要ceoid

select * from t_emp e 

LEFT JOIN t_dept d on e.deptid = d.id

LEFT JOIN t_emp e2 on d.ceo = e2.id


# 保留需要的字段

select e.name ,e.age, e2.name as ceoname ,e2.age as ceoage  from t_emp e 

LEFT JOIN t_dept d on e.deptid = d.id

LEFT JOIN t_emp e2 on d.ceo = e2.id
#增加筛选条件
where e2.age < e.age
```

\#2、列出所有年龄低于 自己门派平均年龄的人员





```
#2、列出所有年龄低于 自己门派平均年龄的人员

#1 求平均年龄
select avg(age) from t_emp


#2 每个门派的平均年龄 

select e.deptid ,avg(e.age)  from t_emp e 

inner JOIN t_dept d on e.deptid = d.id


GROUP BY e.deptid

#3 关联所有人，比对自己年龄 是否比本部门的年龄小


select * from t_emp e LEFT JOIN 

(

select e.deptid as deptid ,avg(e.age) as avgage  from t_emp e 

inner JOIN t_dept d on e.deptid = d.id
GROUP BY e.deptid

) tmp 
on
e.deptId = tmp.deptid

#增加筛选条件
where e.age < tmp.avgage
```







#4、至少有2位非掌门人成员的门派

```

select d.id ,d.deptName,count(*) num from t_emp e 

LEFT JOIN t_dept d on e.deptid = d.id

left join t_dept d2 on e.id = d.ceo

where d2.id is null

GROUP BY d.id ,d.deptName
HAVING num >1

```

\#5、列出全部人员，并增加一列备注“是否为掌门”，如果是掌门人显示是，不是掌门人显示否

```sql

#5、列出全部人员，并增加一列备注“是否为掌门”，如果是掌门人显示是，不是掌门人显示否

select * from t_emp e 

LEFT JOIN t_dept d on e.id = d.ceo


#case when (布尔值) 做逻辑判断
# then 执行 true
# else 执行 false
# 最后 使用end 结束逻辑判断

select e.*,case WHEN d.id is null then '不是掌门'  else '是掌门' end '是否是掌门' from t_emp e 

LEFT JOIN t_dept d on e.id = d.ceo
```



#6、列出全部门派，并增加一列备注“老鸟or菜鸟”，若门派的平均值年龄>50显示“老鸟”，否则显示“菜鸟”

```

#1 门派和所有人关联
select * from t_dept d left join t_emp e on d.id = e.deptId

#1 取每个门派年龄平均值
select d.id, d.deptName,avg(e.age) from t_dept d left join t_emp e on d.id = e.deptId GROUP BY d.id,d.deptName
#2 加逻辑判断

select d.id, 

d.deptName,

case when avg(e.age)>50 then '老鸟' else '菜鸟' end '什么鸟'
,
avg(e.age) 平均年龄


from t_dept d inner join t_emp e on d.id = e.deptId GROUP BY d.id,d.deptName
```





#8 每个门派年龄第二大的

```
#8、显示每个门派年龄第二大的人


#设置临时变量
select e.*,@num from t_emp e,(select @num:=0) a ORDER BY age desc


#设置临时变量
select e.*,@num:=@num+1 num  from t_emp e,(select @num:=0) a ORDER BY age desc



# 外部变量

#设置临时变量
set @num=0;
select e.*,@num:=@num+1 num  from t_emp e ORDER BY age desc;



#收集数据
set @deptid=0;
select e.*,@deptid:=e.deptId deptId  from t_emp e ORDER BY age desc;



set @deptid=0;
select e.*,@deptid:=e.deptId deptId  from t_emp e ORDER BY e.deptId,age desc;

# 根据部门id 倒序排序，deptid 岁数最大的 最先显示，第二个显示的就是岁数第二大的
set @deptid=0;
set @num =0;

select emp.* from 

(
select e.*,

if(@deptid = e.deptId,@num:=@num+1  ,@num:=0  ) cou
,
@deptid:=e.deptId deptIdx


from t_emp e ORDER BY e.deptId,age desc
) emp 

where emp.cou = 1;
```





### 关联查询优化



- 保证**被驱动表**的join字段被索引

-  join 时，选择**小表**作为驱动表，大表作为被驱动表
- inner join 时，mysql会自动将小结果集的表选为驱动表
- 能够直接多表关联的尽量直接关联，不用子查询





### 排序和分组

文件的排序 filesort

全盘数据扫描 all

- 方向反 必排序 desc
- 不过滤 不索引
- 顺序错 必排序





### Group by优化

l group by 先排序再分组，遵照索引建的最佳左前缀法则

l 当无法使用索引列，增大max_length_for_sort_data和sort_buffer_size参数的设置

l where高于having,能写在where限定的条件就不要写在having中了

l group by没有过滤条件，也可以用上索引。Order By 必须有过滤条件才能使用上索引。





### filesort 两种排序算法

- 单路排序
  - 会把所有字段一次性load上来进行排序
  - 单行数据 小于 max_length_for_sort_data
  - 受限于sort_buffer_size的限制，放不下会产生磁盘临时文件
  - 优化
    - select 的内容 很关键
    - 至少要增加范围查询，range级别，
    - 分页，按需二次查询
    - ES（考试）
- 双路排序
  - blob、text 两种字段排序时，一定使用双路排序
  - 回表：先从索引拿主键，再拿主键去取数据







### 覆盖索引

覆盖索引就是创建的索引字段，完全覆盖住查询内容的字段时，这个索引就可以叫覆盖索引。



Ø 禁止使用select *

Ø 禁止查询与业务无关字段

Ø 尽量使用覆盖索引



## 三个 sql优化工具

熟练使用mysql sql调优、索引调优工具，

- show profiles（sql执行过程，打开表，关闭表，读取数据，争抢锁，事物所消耗的时间）

-  explain （索引执行结果使用情况）

- OPTIMIZER_TRACE（一条sql在mysql内部分析优化sql的过程）

binlog

redis 里的aof日志



blackhole





## 第四天



logbin格式：

- binlog_format=STATEMENT（默认）：数据操作的时间，同步时不一致
  - 文件比较小
  - mysql函数相关的sql 在恢复的时候 数据不一致
  - 稳定，健壮
- binlog_format=ROW：批量数据操作时，效率低
  - 复制数据的一致性比较高
- binlog_format=MIXED：是以上两种level的混合使用，有函数用ROW，没函数用STATEMENT，但是无法识别系统变量

集群的架构方式

- M-S
  - 一主多从
  - 整体吞吐量提升
- M-M-M
  - 多主同时在线
  - 互相同步数据
  - 提供高可用
- M-S-S







## 一主一从 集群配置

### 1复制虚拟机

把复制出来的虚拟机，mac地址重新生成一下

### 2修改IP地址

建议mysql版本一致

### 3修改配置文件

/etc/my.cnf

在[mysqld]的标签里

#### master上

```
server-id=1
log-bin=mysql-bin
binlog-ignore-db=mysql
binlog-ignore-db=infomation_schema
binlog-do-db=mytestdb
binlog_format=STATEMENT

```

先不要创建mytestdb数据库。主从搭建好了再创建。



slave上

```
server-id=2
relay-log=mysql-relay

```

### 4  主从机重启配置生效

systemctl restart mysqld

### 5 在master上创建用户

```
create user slave identified by 'Yiming@Sgg666';
grant all privileges on *.* to 'slave';
```

### 6 配置主从

查询master的状态: `show master status;`

记下File和Position的值

执行完此步骤后**不要再操作主服务器**MYSQL，防止主服务器状态值变化





在slave配置

```
CHANGE MASTER TO MASTER_HOST='192.168.44.201',
MASTER_USER='slave',MASTER_PASSWORD='Yiming@Sgg666',
MASTER_LOG_FILE='mysql-bin.000001',MASTER_LOG_POS=682; 

```

`start slave; ` 开启同步

`show slave status\G;`查看运行状态

```
Slave_IO_Running: No
Slave_SQL_Running: Yes
这两个属性都是yes才表示配置成功
```



报错

```
Fatal error: The slave I/O thread stops because master and slave have equal MySQL server UUIDs; these UUIDs must be different for replication to work.
说明mysql是复制出来的
去修改UUID
vi /var/lib/mysql/auto.cnf
随便欢迎数就行
然后重启mysql
```



如果出现问题

- 检查防火墙

- 查看报错信息

  -                Last_IO_Errno: 0
                   Last_IO_Error: 
                   Last_SQL_Errno: 0
                   Last_SQL_Error: 

- 账号密码
- UUID

## Mycat

shardingsphere

## 安装配置

1. #### 下载，最新版本才支持8.0的mysql

http://www.mycat.org.cn/mycat1.html



#### 2.上传解压

挪到`/usr/local`下

#### 3 配置文件

`schema.xml` 数据源相关配

`server.xml`系统相关配置

`rule.xml` 分库分表相关规则

#### 4 创建账号

```
mysql> create user mycat identified by 'Yiming@sGG666';
Query OK, 0 rows affected (0.02 sec)

mysql> grant all privileges on *.* to mycat;

```

##### 5 修改配置文件

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">

<!--   定义虚拟数据库，用来让客户端连接的  -->
	<schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" dataNode="dn1">
	
	</schema>

<!--  真实的物理数据库 数据源-->
<dataNode name="dn1" dataHost="localhost1" database="mytestdb" />


<!-- 数据源的详细配置-->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="2"
			  writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- 可写的服务器 -->
		<writeHost host="hostM1" url="jdbc:mysql://192.168.44.201:3306" user="mycat"
				   password="Yiming@sGG666">

			<!--  只读服务器-->
			 <readHost host="hostS1" url="jdbc:mysql://192.168.44.202:3306" user="mycat" password="Yiming@sGG666" />
		</writeHost>

	</dataHost>
</mycat:schema>
```



6修改Mycat 账号密码

server.xml中

```
	<user name="mycat" defaultAccount="true">
		<property name="password">111111</property>
		<property name="schemas">TESTDB</property>
		<property name="defaultSchema">TESTDB</property>
	</user>
```

## 分库





业务场景：

1. 单库里 表过多，把一些没有关联关系的而且消耗资源多的 挪到另外的库里
2. 历史数据，备份
3. 在mycat 不支持join查询



分表

1. 100w条订单数据，拆出来一部分 放到另外的数据库里的 表中
2. 支持join查询



#### 1 停止mycat



### 2 停止主从

mysql命令行执行

```
stop slave
```



### 3 修改配置文件

schema.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">




<!--   定义虚拟数据库，用来让客户端连接的  -->
	<schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" dataNode="dn1">
	
<!-- 在虚拟库下 定义 不同表的规则 -->
 		<table name="customer" dataNode="dn2" ></table>

	</schema>

<!--  真实的物理数据库 数据源-->
<dataNode name="dn1" dataHost="localhost1" database="mytestdb" />
<dataNode name="dn2" dataHost="localhost2" database="mytestdb" />




<!-- 数据源的详细配置-->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- 可写的服务器 -->
		<writeHost host="hostM1" url="jdbc:mysql://192.168.44.201:3306" user="mycat"
				   password="Yiming@sGG666">
		</writeHost>
	</dataHost>


        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM2" url="jdbc:mysql://192.168.44.202:3306" user="mycat"
				   password="Yiming@sGG666">
                </writeHost>
        </dataHost>

</mycat:schema>
```





4 创建两个数据库

```
在数据节点dn1、dn2上分别创建数据库orders
CREATE DATABASE orders;

```

## 分表



### 1 修改配置文件

schema.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE mycat:schema SYSTEM "schema.dtd">
<mycat:schema xmlns:mycat="http://io.mycat/">




<!--   定义虚拟数据库，用来让客户端连接的  -->
	<schema name="TESTDB" checkSQLschema="true" sqlMaxLimit="100" dataNode="dn1">
	
<!-- 在虚拟库下 定义 不同表的规则 -->
 		<table name="customer" dataNode="dn2" ></table>
        <!--  把orders表拆分开 mod_rule 里定义了拆分规则-->
		<table name="orders" dataNode="dn1,dn2"  rule="mod_rule" ></table>
	</schema>

<!--  真实的物理数据库 数据源-->
<dataNode name="dn1" dataHost="localhost1" database="mytestdb" />
<dataNode name="dn2" dataHost="localhost2" database="mytestdb" />




<!-- 数据源的详细配置-->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="0"
			  writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
		<heartbeat>select user()</heartbeat>
		<!-- 可写的服务器 -->
		<writeHost host="hostM1" url="jdbc:mysql://192.168.44.201:3306" user="mycat"
				   password="Yiming@sGG666">
		</writeHost>
	</dataHost>


        <dataHost name="localhost2" maxCon="1000" minCon="10" balance="0"
                          writeType="0" dbType="mysql" dbDriver="jdbc" switchType="1"  slaveThreshold="100">
                <heartbeat>select user()</heartbeat>
                <!-- can have multi write hosts -->
                <writeHost host="hostM2" url="jdbc:mysql://192.168.44.202:3306" user="mycat"
				   password="Yiming@sGG666">
                </writeHost>
        </dataHost>

</mycat:schema>
```

rule.xml文件



```xml
<tableRule name="mod_rule">
             <rule>
                        <columns>customer_id</columns>
                        <algorithm>mod-long</algorithm>
               </rule>
       </tableRule>
…
在原有的配置中将 3--->2即可！
<function name="mod-long" class="io.mycat.route.function.PartitionByMod">
                <!-- how many data nodes -->
                <property name="count">2</property>
     </function>

```

### 2 在数据节点dn2上建orders表

```
CREATE TABLE orders(
    id INT AUTO_INCREMENT,
    order_type INT,
    customer_id INT,
    amount DECIMAL(10,2),
    PRIMARY KEY(id)  
); 

```

### 3 重启Mycat配置生效，将配置文件重新导入



### 4 导入数据测试

```
#在mycat里向orders表插入数据，INSERT时字段不能省略
INSERT INTO orders(id,order_type,customer_id,amount) VALUES (1,101,100,100100);
INSERT INTO orders(id,order_type,customer_id,amount) VALUES(2,101,100,100300);
INSERT INTO orders(id,order_type,customer_id,amount) VALUES(3,101,101,120000);
INSERT INTO orders(id,order_type,customer_id,amount) VALUES(4,101,101,103000);
INSERT INTO orders(id,order_type,customer_id,amount) VALUES(5,102,101,100400);
INSERT INTO orders(id,order_type,customer_id,amount) VALUES(6,102,100,100020);

```



# 重点

1. mysql 8.0 使用
2. innodb的b+树
3. sql书写 多表关联
4. 索引优化
5. 动手把例子走一遍
