---
title: Mysql简单操作列表
date: 2020-1-16 17:29:15

categories: 数据库
---

## 基础 

命令行
1、显示当前数据库服务器中的数据库列表：mysql> SHOW DATABASES;
2、建立数据库：mysql> CREATE DATABASE 库名;
3、建立数据表：mysql> USE 库名;mysql> CREATE TABLE 表名 (字段名 VARCHAR(20), 字段名 CHAR(1));
4、删除数据库：mysql> DROP DATABASE 库名;
5、删除数据表：mysql> DROP TABLE 表名；
6、将表中记录清空：mysql> DELETE FROM 表名;
7、往表中插入记录：mysql> INSERT INTO 表名 VALUES ("hyq","M");
8、更新表中数据：mysql-> UPDATE 表名 SET 字段名1='a',字段名2='b' WHERE 字段名3='c';
9、用文本方式将数据装入数据表中：mysql> load data local infile "d:/mysql.txt" into table 表名;
10、导入.sql文件命令：mysql> USE 数据库名;mysql> source d:/mysql.sql;
11、命令行修改root密码：mysql> update mysql.user set password=password('新密码') where user='root';mysql> flush privileges;
12.修改密码的三种方法:mysql>update user set password=password('123456') where user='joy_pen';mysql>flush privileges;mysql>set password for 'joy_oen'=password('123456');mysql>grant usage on *.* to 'joy_pen' identified by '123456';
1、创建数据库
命令：create database <数据库名> 例如：建立一个名为xhkdb的数据库mysql> create database xhkdb;
2、显示所有的数据库
命令：show databases （注意：最后有个s）mysql> show databases;
3、删除数据库
命令：drop database <数据库名> 例如：删除名为 xhkdb的数据库mysql> drop database xhkdb;
4、连接数据库
命令： use <数据库名> 例如：如果xhkdb数据库存在，尝试存取它：mysql> use xhkdb; 屏幕提示：Database changed
5、当前选择（连接）的数据库mysql> select database();
6、当前数据库包含的表信息：mysql> show tables; （注意：最后有个s）
三、表操作，操作之前应连接某个数据库
1、建表
命令：create table <表名> ( <字段名1> <类型1> [,..<字段名n> <类型n>]);
mysql> create table MyClass(
\> id int(4) not null primary key auto_increment,
\> name char(20) not null,
\> sex int(4) not null default ''0'',
\> degree double(16,2));
2、获取表结构
命令： desc 表名，或者show columns from 表名
mysql>DESCRIBE MyClass
mysql> desc MyClass;
mysql> show columns from MyClass;
3、删除表
命令：drop table <表名>
例如：删除表名为 MyClass 的表 mysql> drop table MyClass;
4、插入数据
命令：insert into <表名> [( <字段名1>[,..<字段名n > ])] values ( 值1 )[, ( 值n )]
例如，往表 MyClass中插入二条记录, 这二条记录表示：编号为1的名为Tom的成绩为96.45, 编号为2 的名为Joan 的成绩为82.99，编号为3 的名为Wang 的成绩为96.5.
mysql> insert into MyClass values(1,'Tom',96.45),(2,'Joan',82.99), (2,'Wang', 96.59);
5、查询表中的数据
1)、查询所有行
命令： select <字段1，字段2，...> from < 表名 > where < 表达式 >
例如：查看表 MyClass 中所有数据 mysql> select * from MyClass;
2）、查询前几行数据
例如：查看表 MyClass 中前2行数据
mysql> select * from MyClass order by id limit 0,2;
6、删除表中数据
命令：delete from 表名 where 表达式
例如：删除表 MyClass中编号为1 的记录
mysql> delete from MyClass where id=1;
7、修改表中数据：update 表名 set 字段=新值,… where 条件
mysql> update MyClass set name=''Mary'' where id=1;
8、在表中增加字段：
命令：alter table 表名 add 字段 类型 其他;
例如：在表MyClass中添加了一个字段passtest，类型为int(4)，默认值为0
mysql> alter table MyClass add passtest int(4) default ''0''
9、更改表名：
命令：rename table 原表名 to 新表名;
例如：在表MyClass名字更改为YouClass
mysql> rename table MyClass to YouClass;
更新字段内容
update 表名 set 字段名 = 新内容
update 表名 set 字段名 = replace(字段名,''旧内容'',''新内容'');