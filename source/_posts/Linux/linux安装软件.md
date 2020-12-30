---
title: Linux安装软件
date: 2019-11-22 15:04:20
tags:	
categories: Linux
---

## 安装JDK

由于是Centos这里我们使用yum来安装

### 检查是否已安装`JDK`及卸载

- 以下命令二选一，中括号选一即可

```shell
yum list installed | grep [java][jdk]
rpm -qa | grep [java][jdk][gcj]
```

- 卸载`JAVA`环境

```shell
yum -y remove java-1.6.0-openjdk*  //表时卸载所有openjdk相关文件输入
yum -y remove tzdata-java.noarch   //卸载tzdata-java
```

### 安装`JDK`

- 查看`JDK`软件包列表

```shell
yum search java | grep -i --color jdk
```

- 选择版本安装

```shell
yum install -y java-1.8.0-openjdk java-1.8.0-openjdk-devel
#或者如下命令，安装jdk1.8.0的所有文件
yum install -y java-1.8.0-openjdk*
```

- 查看`JDK`是否安装成功

```shell
java -version
```

###  配置环境变量

+ `JDK`默认安装路径`/usr/lib/jvm`  查看自己的安装版本 供后面的配置环境提供参数

+ 在`/etc/profile`文件添加如下命令 

 ```shell
# set java environment  
JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.232.b09-0.el7_7.x86_64
PATH=$PATH:$JAVA_HOME/bin  
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar  
export JAVA_HOME  CLASSPATH  PATH 
 ```

__注意这里的JAVA_HOME__参数就是我们上一步所看到的版本

- 保存关闭`profile`文件，执行如下命令生效

```shell
source  /etc/profile
```

#### 检查

```shell
javac
```

## Maven安装

Maven的下载地址是：http://maven.apache.org/download.cgi

直接在centos上下载

```shell
cd /usr/local/src/
wget http://mirrors.hust.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz
tar zxf apache-maven-3.6.3-bin.tar.gz
mv apache-maven-3.6.3 /usr/local/maven3  #移动到文件夹maven3中
```

修改配置环境

```shell
vi /etc/profile
#在适当的位置添加
export M2_HOME=/usr/local/maven3
export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
```


我的环境是

```shell
#set java environment

JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el7_8.x86_64
M2_HOME=/usr/local/maven3
PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

重启

```shell
source /etc/profile
```

验证版本

```shell
mvn -v
```

## git 安装

```shell
# 安装
yum install -y git

# 查看版本
git version

# git version 1.8.3.1
```

## 安装nacos

此方法为源码安装，需要JDK，Maven。获取源码需要git

[官网地址](https://nacos.io/zh-cn/docs/quick-start.html)

下载源码

下载太慢了我直接从windows拉上去了

在lib包下启动即可

阿里云服务需要打开8848端口

http://123.123.123.123:8848/nacos/#/login

## 安装unzip

```shell
yum install -y unzip zip
```



