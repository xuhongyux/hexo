---
title: Docker 软件安装
date: 2020-12-18 15:04:20
categories: Docker
---

## Docker 安装

### Mysql安装

拉取官方mysql5.7镜像 

```shell
docker pull mysql:5.7
```

在本地创建mysql的映射目录

```shell
mkdir -p /root/mysql/data /root/mysql/logs /root/mysql/conf
```

在/root/mysql/conf中创建 *.cnf 文件(叫什么都行)

```shell
touch my.cnf
```

创建容器,将数据,日志,配置文件映射到本机

```shell
docker run -p 3306:3306 --name mysql -v /root/mysql/conf:/etc/mysql/conf.d -v /root/mysql/logs:/logs -v /root/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

**-d:** 后台运行容器

**-p** 将容器的端口映射到本机的端口

**-v** 将主机目录挂载到容器的目录

**-e** 设置参数

启动mysql容器

```shell
docker start mysql
```

快速版本

```shell
docker run --name mysqlserver -v $PWD/conf:/etc/mysql/conf.d -v $PWD/logs:/logs -v $PWD/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root -d -i -p 3306:3306 mysql:latest
```

### Redis安装

获取最新版本

```shell
docker pull redis:latest
docker images
```

构建容器

```shell
 docker run -itd --name redis-xin -p 6379:6379 redis
```

## Docker-compose安装

### 部署Seata Server

- 我们使用 Docker 部署 Seata Server，Compose 配置文件如下

```yml
version: "3"
services:
  seata-server:
    image: seataio/seata-server
    hostname: seata-server
    container_name: seata-server
    ports:
      - "8091:8091"
    environment:
      - SEATA_PORT=8091
    volumes:
      - ./config:/root/seata-config
```

- 在 Compose 同级目录下创建配置文件 `vi config/registry.conf`

```json
registry {
  type = "file"

  file {
    name = "file.conf"
  }
}

config {
  type = "file"

  file {
    name = "file.conf"
  }
}
```

- 在 Compose 同级目录下创建配置文件 `vi config/file.conf`

```json
service {
  # 自定义事务组名称 tx_group，客户端配置需要和服务端一致
  vgroup_mapping.tx_group = "default"
  default.grouplist = "127.0.0.1:8091"
  disableGlobalTransaction = false
}

store {
  mode = "file"

  file {
    dir = "sessionStore"
  }
}
```

- 启动服务

```shell
docker-compose up -d
```

