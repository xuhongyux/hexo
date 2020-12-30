---
title: Docker扩展
date: 2019-11-15 15:04:20
categories: Docker
---

### 列出所有的容器 ID

```shell
docker ps -aq
```

### 停止所有的容器

```shell
docker stop $(docker ps -aq)
```

### 删除所有的容器

```shell
docker rm $(docker ps -aq)
```

### 删除所有的镜像

```shell
docker rmi $(docker images -q)
```

### 复制文件

```shell
docker cp mycontainer:/opt/file.txt /opt/local/docker cp /opt/local/file.txt my
```

