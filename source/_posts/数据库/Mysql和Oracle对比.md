---
title: Mysql和Oracle对比
date: 2020-11-20 9:04:20
tags:
	- Oracle
categories: 数据库
---

## 宏观区别

1. Oracle大型数据库，Mysql小型数据库
2. Oracle 收费（昂贵）， Mysql 免费
3. Oracle 非常吃内存，Mysql少量内存

## 微观区别

1. 事务： Oracle 原生支持事务，Mysql由于是插件化引擎部分支持
2. 并发： Oracle 行锁，Mysql 有些引擎表锁，有些行锁
3. 持久性：Oracle把提交的sql操作线写入了在线联机日志文件中，保存到磁盘上，Mysql默认提交sql语句，但是如果更新过程中出现db或者主机重启的问题，也可能会丢失数据。
4. 事务隔离： Mysql 默认可重复度，Oracle不可重复读
5. 提交方式， Oracle 默认不自动提交，需要手动提交，Mysql 反之
6. 逻辑备份： Mysql逻辑备份是要锁定数据，才能保证备份的数据是一致的，影响业务正常的DML(数据操纵语言Data Manipulation Language)使用；Oracle逻辑备份时不锁定数据，且备份的数据是一致的。
7. 数据复制： MySql 的分区表还不大成熟。Oracle分区表成熟

