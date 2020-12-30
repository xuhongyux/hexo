---
title: Docker Compose
date: 2019-09-19 15:04:20
tags: 
	- Docker
	- DockerCompos
categories:
	- Docker
---



Docker 容器编排工具

## 概述

Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 OpenStack 中的 Heat 十分类似。

其代码目前在 [https://github.com/docker/compose](http://www.qfdmy.com/wp-content/themes/quanbaike/go.php?url=aHR0cHM6Ly9naXRodWIuY29tL2RvY2tlci9jb21wb3Nl) 上开源。

Compose 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig。

我们知道使用一个 Dockerfile 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 **docker-compose.yml** 模板文件（YAML 格式）来定义一组相关联的应用容器为一个项目（Project）。Compose 中有两个重要的概念：

- 服务 (Service)：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 (Project)：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义。

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

## 安装 Docker Compose

在[github](https://github.com/docker/compose/releases)上发布的版本自己去寻找最新的。或者

直接下载编译好的二进制文件即可。

```shell
# 下载
curl -L https://github.com/docker/compose/releases/download/1.24.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 赋予权限
chmod +x /usr/local/bin/docker-compose
```

验证安装是否成功

```shell
docker-compose version
# 输出如下
docker-compose version 1.24.0, build 0aa59064
docker-py version: 3.7.2
CPython version: 3.6.8
OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
```

## 修改IP和DNS

课程演示会采用多虚拟机模拟分布式场景，为防止 IP 冲突，无法联网等问题，需要预先设置好主机名、IP、DNS 配置

### 修改主机名(UBTO)

- 修改 cloud.cfg 防止重启后主机名还原

```shell
vi /etc/cloud/cloud.cfg
# 该配置默认为 false，修改为 true 即可preserve_hostname: true
```

- 修改主机名

```yml
# 修改主机名
hostnamectl set-hostname deployment
# 配置 hosts
cat >> /etc/hosts << EOF
192.168.141.130 deployment
EOF
```

### 修改 IP

编辑 `vi /etc/netplan/50-cloud-init.yaml` 配置文件，修改内容如下

```xml
network: 
	ethernets:     
		ens33:     
			addresses: [192.168.141.130/24]     
			gateway4: 192.168.141.2    
			nameservers:       
				addresses: [192.168.141.2]    
	version: 2
```

使用 `netplan apply` 命令让配置生效

### 修改 DNS

```shell
# 取消 DNS 行注释，并增加 DNS 配置如：114.114.114.114，修改后重启下计算机
vi /etc/systemd/resolved.conf
```

### centosn的IP配置

__修改主机名称__

[参考文章]( https://blog.csdn.net/xuheng8600/article/details/79983927 )

```shell
永久生效

//永久性的修改主机名称，重启后能保持修改后的。
hostnamectl set-hostname xxx	
 
//删除hostname
hostnamectl set-hostname ""
hostnamectl set-hostname "" --static
hostnamectl set-hostname "" --pretty
```

重启CentOS 7 

```shell
reboot -f 
```

__设置\修改IP地址__

如果要让IP地址永久生效，需要编辑网卡配置文件

使用VI编辑器设置，如 

```shell
vi /etc/sysconfig/network-scripts/ifcfg-eth0
```

**说明一下这个文**

第一项：修改BOOTPROTO="static"也就是将dhcp改为static；

第二项：修改ONBOOT="yes" 意思是将网卡设置 为开机启用，同时在文字下方添加：

```xml
IPADDR=192.168.0.100 #静态IP
GATEWAY=192.168.0.1 #默认网关
NETMASK=255.255.255.0 #子网掩码
DNS1=192.168.0.1 #第一个dns填写网关最好
DNS2=8.8.8.8    #谷歌dns地址
```



重启

```shell
 service network restart
```



## DockerCompose命令

1. Docker compose的使用非常类似于docker命令的使用，**但是需要注意的是大部分的compose命令都需要到docker-compose.yml文件所在的目录下才能执行。**
2. compose以守护进程模式运行加-d选项
   $ docker-compose up -d
3. 查看有哪些服务，使用docker-compose ps命令，非常类似于 docker 的ps命令。
4. 查看compose日志
   $ docker-compose logs web
   $ docker-compose logs redis
5. 停止compose服务
   $ docker-compose stop
   $ docker-compose ps
6. 重启compose服务
   $ docker-compose restart
   $ docker-compose ps
7. kill compose服务
   $ docker-compose kill
   $ docker-compose ps
8. 删除compose服务
   $ docker-compose rm
9. 更多的docker-compose命令可以使用docker-compose --help查看



## DockerCompose部署

### 进入vim的粘贴模式

```Shell
:set paste i
```

### 部署mysql

选择合适的目录

```shell
/usr/local/docker/mysql
```

创建docker-compose.yml

```shell
vim docker-compose.yml
```

docker-compose.yml内容

```yml
version: '3.1'
services:
  db:
    # 目前 latest 版本为 MySQL8.x
    image: mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 123456
    command:
      --default-authentication-plugin=mysql_native_password
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
    ports:
      - 3306:3306
    volumes:
      - ./data:/var/lib/mysql
  # MySQL 的 Web 客户端
  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

执行

```shell
docker-compose up -d
```

失败的时候处理

```shell
docker exec -it mysql_db_1 bash//进入容器的命令总会吧。
mysql -u root -p //登录MySQL
 mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%'WITH GRANT OPTION; //任何远程主机都可以访问数据库
 mysql> FLUSH PRIVILEGES; //需要输入次命令使修改生效
 
 
 docker restart mysql//重启容器
```







### 部署Tomcat

```shell
version: '3.1'
services:
  tomcat:
    restart: always
    image: tomcat
    container_name: tomcat
    ports:
      - 8080:8080
    volumes:
      - ./webapps:/usr/local/tomcat/webapps
    environment:
      TZ: Asia/Shanghai
```

### 部署GItLab（docker-compose.yml详解）

`docker-compose.yml` 配置如下：

```yml
version: '3'
services:
    web:
      # 镜像名称
      image: 'twang2218/gitlab-ce-zh' 
      # 开机镜像
      restart: always
      #域名
      hostname: '192.168.75.145' 
      #环境
      environment:
      	#时区
        TZ: 'Asia/Shanghai'
        #专属的配置
        GITLAB_OMNIBUS_CONFIG: 
          external_url 'http://192.168.75.145'
          gitlab_rails['gitlab_shell_ssh_port'] = 2222
          unicorn['port'] = 8888
          nginx['listen_port'] = 80
      #暴露的端口
      ports:
        - '80:80'
        - '443:443'
        - '2222:22'
      #数据卷
      volumes:
        - ./config:/etc/gitlab
        - ./data:/var/opt/gitlab
        - ./logs:/var/log/gitlab
```

### 部署redis

```yaml
version: '3'
services:
  redis:
    image: redis:4.0.13
    container_name: redis
    restart: always
    command: --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - ./redis_data:/data
```

docker-compose up -d

或者

手写redis的docker文件，通过docker-compose配置redis

1、首先我们创建下docker文件的目录，并新建Dockerfile、redis-entrypoint.sh、redis.conf

Dockerfile文件内容如下：

```
FROM redis:latest        #指定启动容器的镜像
 
MAINTAINER cc <cc@qq.com>  #署名
 
RUN mkdir -p /redis/log;   #在容器里运行创建目录/redis/log
 
WORKDIR /redis     #设置工作目录为/redis
 
COPY redis.conf .  #拷贝redis.conf配置文件到工作目录(这里其实就是在Dockerfile同级下的redis.conf文件拷贝到容器内当前工作目录，也就是/redis目录)
COPY redis-entrypoint.sh /usr/local/bin/ #拷贝redis.entrypoint.sh 到容器内/usr/local/bin/目录下
 
RUN chown redis:redis /redis/* && \   #给容器内的/redis/*设置归属用户，并设置redis.entrypoint.sh文件的可执行权限
    chmod +x /usr/local/bin/redis-entrypoint.sh
 
EXPOSE 6379       #暴露端口6379
 
CMD ["redis-entrypoint.sh"]  #执行redis-entrypoint.sh文件
```

redis.conf内容如下：

```
#修改daemonize为yes，即默认以后台程序方式运行（还记得前面手动使用&号强制后台运行吗）。
daemonize no
#可修改默认监听端口
port $REDIS_PORT
#修改生成默认日志文件位置
logfile "/redis/log/redis.log"
#配置持久化文件存放位置
dir "/tmp"
requirepass $REDIS_PASSWORD
```

redis-entrypoint.sh内容如下：

```sh
#!/usr/bin/env sh
 
sed -i "s/\$REDIS_PORT/$REDIS_PORT/g" /redis/redis.conf   #声明参数，为了docker-compose里面可以动态配置
sed -i "s/\$REDIS_PASSWORD/$REDIS_PASSWORD/g" /redis/redis.conf  #声明参数
 
redis-server /redis/redis.conf
```

到这步，我们就以及把redis的所有配置都准备好了，接下来在Dockerfile的目录，执行脚本，生成redis镜像文件

2、生成docker 镜像文件

```
docker build -t iqeq/redis:1.0 .   # 生成了一个iqeq/redis:1.0的镜像文件
```

3、编排docker-compose文件

```
version: '3' #版本号

services:
redis:
container_name: redis_container #容器名，自定义
image: iqeq/redis:1.0  #刚才生成的镜像名
environment:      #环境参数：配置刚才shell启动脚本里面声明的2个参数
\- REDIS_PORT=6379
\- REDIS_PASSWORD=密码
ports:         #暴露容器内部端口6379并映射到外部也为6379
\- "6379:6379"
restart: unless-stopped  #启动方式
volumes:
\- $PWD/redis/data:/data   #文件绑定挂载：$PWD表示当前目录，然后这里就是当前目录下的/redis/data子目录，挂载为容器内的/data目录
\- $PWD/redis/log:/redis/log
```

然后保存文件为docker-compose.yml

最后：

我们在docker-compose.yml的同级目录下，执行以下脚本：

```
docker-compose up -d
```

### 部署Nexus

我们使用 Docker 来安装和运行 Nexus，`docker-compose.yml` 配置如下：

```yml
version: '3.1'
services:
  nexus:
    restart: always
    image: sonatype/nexus3
    container_name: nexus
    ports:
      - 80:8081
    volumes:
      - data:/nexus-data
volumes:
  data:
```

验证安装是否成功

- **地址：** [http://ip:port/](http://www.qfdmy.com/wp-content/themes/quanbaike/go.php?url=aHR0cDovL2lwOnBvcnQv)
- **用户名：** admin
- **密码：** admin123

**注意：** 新版本密码在 `cat /var/lib/docker/volumes/nexus_data/_data/admin.password`

#### Maven 仓库介绍

#### 代理仓库(Proxy Repository)

- 第三方仓库
  - **maven-central**
  - **nuget.org-proxy**
- 版本策略(Version Policy)
  - **Release：** 正式版本
  - **Snapshot：** 快照版本
  - **Mixed：** 混合模式
- 布局策略(Layout Policy)
  - **Strict：** 严格
  - **Permissive：** 宽松

#### 宿主仓库(Hosted Repository)

- 存储本地上传的组件和资源的
  - **maven-releases**
  - **maven-snapshots**
  - **nuget-hosted**
- 部署策略(Deployment Policy)
  - **Allow Redeploy：** 允许重新部署
  - **Disable Redeploy：** 禁止重新部署
  - **Read-Only：** 只读

#### 仓库组(Repository Group)

通常包含了多个代理仓库和宿主仓库，在项目中只要引入仓库组就可以下载到代理仓库和宿主仓库中的包

- **maven-public**
- **nuget-group**

#### 在项目中使用 Nexus

##### 配置认证信息

在 Maven `settings.xml` 中添加 Nexus 认证信息 (**servers** 节点下)

```xml
<server>
  <id>nexus-releases</id>
  <username>admin</username>
  <password>admin123</password>
</server>
<server>
  <id>nexus-snapshots</id>
  <username>admin</username>
  <password>admin123</password>
</server>
```

##### Snapshots 与 Releases 的区别

- **nexus-releases：** 用于发布 Release 版本
- **nexus-snapshots：** 用于发布 Snapshot 版本（快照版）

Release 版本与 Snapshot 定义

```yml
Release: 1.0.0/1.0.0-RELEASE
Snapshot: 1.0.0-SNAPSHOT
```

- 在项目 `pom.xml` 中设置的版本号添加 `SNAPSHOT` 标识的都会发布为 `SNAPSHOT` 版本，没有 `SNAPSHOT` 标识的都会发布为 `RELEASE` 版本。
- `SNAPSHOT` 版本会自动加一个时间作为标识，如：`1.0.0-SNAPSHOT` 发布后为变成 `1.0.0-SNAPSHOT-20180522.123456-1.jar`

##### 配置自动化部署

在 `pom.xml` 中添加如下代码

```xml
<distributionManagement>  
  <repository>  
    <id>nexus-releases</id>  
    <name>Nexus Release Repository</name>  
    <url>http://127.0.0.1:8081/repository/maven-releases/</url>  
  </repository>  
  <snapshotRepository>  
    <id>nexus-snapshots</id>  
    <name>Nexus Snapshot Repository</name>  
    <url>http://127.0.0.1:8081/repository/maven-snapshots/</url>  
  </snapshotRepository>  
</distributionManagement> 
```

**注意事项**

- ID 名称必须要与 `settings.xml` 中 Servers 配置的 ID 名称保持一致
- 项目版本号中有 `SNAPSHOT` 标识的，会发布到 Nexus Snapshots Repository， 否则发布到 Nexus Release Repository，并根据 ID 去匹配授权账号

##### 部署到仓库

```
mvn deploy
```

##### 配置代理仓库

```xml
<repositories>
    <repository>
        <id>nexus</id>
        <name>Nexus Repository</name>
        <url>http://127.0.0.1:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </repository>
</repositories>
<pluginRepositories>
    <pluginRepository>
        <id>nexus</id>
        <name>Nexus Plugin Repository</name>
        <url>http://127.0.0.1:8081/repository/maven-public/</url>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
        <releases>
            <enabled>true</enabled>
        </releases>
    </pluginRepository>
</pluginRepositories>
```

#### 扩展阅读

##### 手动上传第三方依赖

Nexus 3.1.x 开始支持页面上传第三方依赖功能，以下为手动上传命令

```shell
# 如第三方JAR包：aliyun-sdk-oss-2.2.3.jar
mvn deploy:deploy-file 
  -DgroupId=com.aliyun.oss 
  -DartifactId=aliyun-sdk-oss 
  -Dversion=2.2.3 
  -Dpackaging=jar 
  -Dfile=D:\aliyun-sdk-oss-2.2.3.jar 
  -Durl=http://127.0.0.1:8081/repository/maven-3rd/ 
  -DrepositoryId=nexus-releases
```

**注意事项**

- 建议在上传第三方 JAR 包时，创建单独的第三方 JAR 包管理仓库，便于管理有维护。（maven-3rd）
- `-DrepositoryId=nexus-releases` 对应的是 `settings.xml` 中 **Servers** 配置的 ID 名称。（授权）

