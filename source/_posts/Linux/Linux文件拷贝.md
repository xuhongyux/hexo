---
title: Linux文件拷贝及操作
date: 2019-11-21 15:04:20
tags:	
categories: Linux
---

### 文件拷贝

一、复制文件：
 （1）将本地文件拷贝到远程
 scp 文件名 用户名@计算机IP或者计算机名称:远程路径
 本地192.168.1.8客户端

```shell
scp /root/install.* root@192.168.1.12:/usr/local/src
```

 （2）从远程将文件拷回本地
 scp 用户名@计算机IP或者计算机名称:文件名 本地路径

 

 本地192.168.1.8客户端取远程服务器12、11上的文件

```shell
scp root@192.168.1.12:/usr/local/src/*.log /root/
```

 二、复制文件夹（目录）：

 （1）将本地文件夹拷贝到远程
 scp -r 目录名 用户名@计算机IP或者计算机名称:远程路径

```shell
scp -r /home/test1 zhidao@192.168.0.1:/home/test2 
#test1为源目录，test2为目标目录，zhidao@192.168.0.1为远程服务器的用户名和ip地址。
```

 （2）从远程将文件夹拷回本地
 scp -r 用户名@计算机IP或者计算机名称:目录名 本地路径

```shell
scp  -r zhidao@192.168.0.1:/home/test2 /home/test1
#zhidao@192.168.0.1为远程服务器的用户名和ip地址，test1为源目录，test2为目标目录。
```

### 查看文件或者文件夹大小

 du -sh. 系统只显示当前文件夹所占用的总空间 

```shell
du -sh
```

 du -a. 显示的是所有的文件.包括子文件夹下所有文件也显示 

```shell
du -a
```

### 搜索文件

Centos之文件搜索命令find

find [搜索范围] [搜索条件]

\#搜索文件

find / -name install.log