# Mysql高级

# 第一天

阿里巴巴开发手册  有时间看一看，工作后 敲代码息息相关。

### 数据库设计三范式

用户表

| id   | name   | 地址                                                         |      |      |
| ---- | ------ | ------------------------------------------------------------ | ---- | ---- |
| 1    | yiming | 北京市  昌平区回龙观镇xxoo小区xx-xx门 \|天津市  昌平区回龙观镇xxoo小区xx-xx门 |      |      |
|      |        |                                                              |      |      |
|      |        |                                                              |      |      |



地址表



| id   | uid  | 城市   | 区   |      |
| ---- | ---- | ------ | ---- | ---- |
| 1    |      | 北京市 |      |      |
|      |      |        |      |      |
|      |      |        |      |      |





### 三范式

- 每行数据唯一性的同时，每列也不可拆分
- 拆表 做关联
- 多对多关联

#### 范式化

优点：可读性好，存储空间小

缺点：多表查询性能差

#### 反范式化

- 冗余存储，字段不再拆分
- 并不是不做多表关联，而是冗余存储常用字段
- 更新的时候，级联使用事务同时更新





## 安装

1.下载对应操作系统的版本

2.依赖检查

```
rpm -qa | grep -i mariadb 
rpm -e --nodeps mariadb-libs
chmod -R 777 /tmp
rpm -qa|grep libaio
rpm -qa|grep net-tools
yum install -y perl-Module-Install.noarch
```

```
解压缩
tar xvf mysql-8.0.29-1.el8.x86_64.rpm-bundle.tar

在mysql的安装文件目录下执行：（必须按照顺序执行）

rpm -ivh mysql-community-common-8.0.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-plugins-8.0.29-1.el7.x86_64.rpm


rpm -ivh mysql-community-libs-8.0.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-icu-data-files-8.0.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-client-8.0.29-1.el7.x86_64.rpm
rpm -ivh mysql-community-server-8.0.29-1.el7.x86_64.rpm

```

### 默认密码

```
grep 'temporary password' /var/log/mysqld.log
```

### 强制重置密码

```
在/etc/my.cnf


[mysqld]
skip_grant_tables=1
```

### 修改密码

```
ALTER USER 'root'@'localhost' IDENTIFIED BY '111111';

atguigU@1ming
```

### 密码强度

```
密码强度校验
SHOW VARIABLES LIKE 'vali%';


validate_password_length 8 # 密码的最小长度，此处为8。

validate_password_mixed_case_count 1 # 至少要包含小写或大写字母的个数，此处为1。

validate_password_number_count 1 # 至少要包含的数字的个数，此处为1。

validate_password_policy MEDIUM # 强度等级，其中其值可设置为0、1、2。分别对应：

【0/LOW】：只检查长度。

【1/MEDIUM】：在0等级的基础上多检查数字、大小写、特殊字符。

【2/STRONG】：在1等级的基础上多检查特殊字符字典文件，此处为1。

validate_password_special_char_count 1 # 至少要包含的个数字符的个数，此处为1。

set global validate_password.policy=0;

```

### 创建用户

```
create user yiming identified by '111111';
```

### 授权

全部权限

```
grant all privileges on *.*  to 'yiming';

```

### 回收权限

```
revoke all privileges on *.* from yiming;

```





### 远程登录

`update user set host='%' where user = 'root';`

#### flush privileges;

在直接操作数据库user表的时候，必须重新刷新权限`flush privileges;`

使用mysql内置命令，不需要刷新。

### show profile



是否支持

`select @@have_profiling;`

是否打开

`select @@profiling;`

打开

`set profiling=on;`

查询缓存不支持

```
show profile cpu,block io for query 6；
```



`show profile cpu,block io for query 6；`



### myisam存储引擎

5.7

.frm 元数据

.myd 数据

.myi 索引

8.0

myisam有

sdi 元数据信息

.myd 数据

.myi 索引

innodb

.ibd索引、数据、元数据



avltree

256

128

64

32

16

8

4

2

## 作业 Join查询