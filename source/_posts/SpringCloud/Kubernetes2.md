---
title: Kubernetes  (二)
deta: 2019-11-20 12:00:20
tags:
	- Nginx
	- Kubernetes
categories: Springcloud 
---

# Nginx 虚拟主机

## 什么是 Nginx

Nginx 是一款高性能的 HTTP 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。由俄罗斯的程序设计师 Igor Sysoev 所开发，官方测试 Nginx 能够支支撑 5 万并发链接，并且 CPU、内存等资源消耗却非常低，运行非常稳定。

## Nginx 的应用场景

- HTTP 服务器：Nginx 是一个 HTTP 服务可以独立提供 HTTP 服务。可以做网页静态服务器。
- 虚拟主机：可以实现在一台服务器虚拟出多个网站。例如个人网站使用的虚拟主机。
- 反向代理，负载均衡：当网站的访问量达到一定程度后，单台服务器不能满足用户的请求时，需要用多台服务器集群可以使用 Nginx 做反向代理。并且多台服务器可以平均分担负载，不会因为某台服务器负载高宕机而某台服务器闲置的情况。

## Nginx 虚拟主机

### 什么是虚拟主机

虚拟主机是一种特殊的软硬件技术，它可以将网络上的每一台计算机分成多个虚拟主机，每个虚拟主机可以独立对外提供 www 服务，这样就可以实现一台主机对外提供多个 web 服务，每个虚拟主机之间是独立的，互不影响的。

通过 Nginx 可以实现虚拟主机的配置，Nginx 支持三种类型的虚拟主机配置

- 基于 IP 的虚拟主机
- 基于域名的虚拟主机
- 基于端口的虚拟主机

### Nginx 配置文件的结构

```json
# ...
events {
    # ...
}
http {
    # ...
    server{
        # ...
    }
    # ...
    server{
        # ...
    }
}
```

> **注意：** 每个 server 就是一个虚拟主机

## 基于 Docker 部署 Nginx

```yaml
version: '3.1'
services:
  nginx:
    restart: always
    image: nginx
    container_name: nginx
    ports:
      - 80:80
    volumes:
      - ./conf/nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
```

### 基于端口的虚拟主机配置

#### 需求

- Nginx 对外提供 80 和 8080 两个端口监听服务
- 请求 80 端口则请求 html80 目录下的 html
- 请求 8080 端口则请求 html8080 目录下的 html

#### 操作流程

- 创建目录及文件，在 `/usr/local/docker/nginx/html` 目录下创建 `html80` 和 `html8080` 两个目录，并分别创建两个 `index.html` 文件
- 配置虚拟主机，创建并修改 `/usr/local/docker/nginx/conf` 目录下的 `nginx.conf`

```conf
# 启动进程,通常设置成和 CPU 的数量相等
worker_processes  1;
events {
    # epoll 是多路复用 IO(I/O Multiplexing) 中的一种方式
    # 但是仅用于 linux2.6 以上内核,可以大大提高 nginx 的性能
    use epoll;
    # 单个后台 worker process 进程的最大并发链接数
    worker_connections  1024;
}
http {
    # 设定 mime 类型,类型由 mime.type 文件定义
    include       mime.types;
    default_type  application/octet-stream;
    # sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
    # 必须设为 on，如果用来进行下载等应用磁盘 IO 重负载应用，可设置为 off，以平衡磁盘与网络 I/O 处理速度，降低系统的 uptime.
    sendfile        on;
    # 连接超时时间
    keepalive_timeout  65;
    # 设定请求缓冲
    client_header_buffer_size 2k;
    # 配置虚拟主机 192.168.141.121
    server {
    # 监听的 IP 和端口，配置 192.168.141.121:80
        listen       80;
    # 虚拟主机名称这里配置 IP 地址
        server_name  192.168.141.121;
    # 所有的请求都以 / 开始，所有的请求都可以匹配此 location
        location / {
        # 使用 root 指令指定虚拟主机目录即网页存放目录
        # 比如访问 http://ip/index.html 将找到 /usr/local/docker/nginx/html/html80/index.html
        # 比如访问 http://ip/item/index.html 将找到 /usr/local/docker/nginx/html/html80/item/index.html
            root   /usr/share/nginx/html/html80;
        # 指定欢迎页面，按从左到右顺序查找
            index  index.html index.htm;
        }
    }
    # 配置虚拟主机 192.168.141.121
    server {
        listen       8080;
        server_name  192.168.141.121;
        location / {
            root   /usr/share/nginx/html/html8080;
            index  index.html index.htm;
        }
    }
}
```

### 基于域名的虚拟主机配置

#### 需求

- 两个域名指向同一台 Nginx 服务器，用户访问不同的域名显示不同的网页内容
- 两个域名是 ingress.funtl.com 和 service.funtl.com
- Nginx 服务器使用虚拟机 192.168.141.121

#### 操作流程

- 配置 Hosts，通过 host 文件指定 service1.funtl.com 和 service2.funtl.com 对应 192.168.141.121 虚拟机，修改 window 的 hosts 文件：（C:\Windows\System32\drivers\etc）
- 创建目录及文件，在 `/usr/local/docker/nginx/html` 目录下创建 `service1` 和 `service1` 两个目录，并分别创建两个 index.html 文件
- 配置虚拟主机，修改 `/usr/local/docker/nginx/conf` 目录下的 `nginx.conf`

```json
user  nginx;
worker_processes  1;
events {
    use epoll;
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  service1.funtl.com;
        location / {
            root   /usr/share/nginx/html/service1;
            index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  service2.funtl.com;
        location / {
            root   /usr/share/nginx/html/service2;
            index  index.html index.htm;
        }
    }
}
```











# Nginx 反向代理

## 什么是代理服务器

代理服务器，客户机在发送请求时，不会直接发送给目的主机，而是先发送给代理服务器，代理服务接受客户机请求之后，再向主机发出，并接收目的主机返回的数据，存放在代理服务器的硬盘中，再发送给客户机。

![img](Kubernetes2/edaa206ef7713d0.png)

## 为什么要使用代理服务器

- **提高访问速度：** 由于目标主机返回的数据会存放在代理服务器的硬盘中，因此下一次客户再访问相同的站点数据时，会直接从代理服务器的硬盘中读取，起到了缓存的作用，尤其对于热门站点能明显提高请求速度。
- **防火墙作用：** 由于所有的客户机请求都必须通过代理服务器访问远程站点，因此可在代理服务器上设限，过滤某些不安全信息。
- **通过代理服务器访问不能访问的目标站点：** 互联网上有许多开放的代理服务器，客户机在访问受限时，可通过不受限的代理服务器访问目标站点，通俗说，我们使用的翻墙浏览器就是利用了代理服务器，虽然不能出国，但也可直接访问外网。

## 什么是正向代理

正向代理，架设在客户机与目标主机之间，只用于代理内部网络对 Internet 的连接请求，客户机必须指定代理服务器,并将本来要直接发送到 Web 服务器上的 HTTP 请求发送到代理服务器中。

![img](Kubernetes2/150135f1004fc35.png)

## 什么是反向代理

反向代理服务器架设在服务器端，通过缓冲经常被请求的页面来缓解服务器的工作量，将客户机请求转发给内部网络上的目标服务器；并将从服务器上得到的结果返回给 Internet 上请求连接的客户端，此时代理服务器与目标主机一起对外表现为一个服务器。

![img](Kubernetes2/230002f000af055.png)

## 反向代理有哪些主要应用

现在许多大型 web 网站都用到反向代理。除了可以防止外网对内网服务器的恶性攻击、缓存以减少服务器的压力和访问安全控制之外，还可以进行负载均衡，将用户请求分配给多个服务器。

## 使用 Nginx 反向代理 Tomcat

### 需求

- 两个 Tomcat 服务通过 Nginx 反向代理
- Nginx 服务器：192.168.141.121:80
- Tomcat1 服务器：192.168.141.121:8081
- Tomcat2 服务器：192.168.141.121:8082

### 启动 Tomcat 容器

启动两个 Tomcat 容器，映射端口为 8081 和 8082，`docker-compose.yml` 如下：

```yaml
version: '3.1'
services:
  tomcat1:
    image: tomcat
    container_name: tomcat1
    ports:
      - 8081:8080
  tomcat2:
    image: tomcat
    container_name: tomcat2
    ports:
      - 8082:8080
```

### 配置 Nginx 反向代理

修改 `/usr/local/docker/nginx/conf` 目录下的 `nginx.conf` 配置文件：

```yaml
user  nginx;
worker_processes  1;
events {
    use epoll;
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    # 配置一个代理即 tomcat1 服务器
    upstream tomcatServer1 {
        server 192.168.141.121:8081;
    }
    # 配置一个代理即 tomcat2 服务器
    upstream tomcatServer2 {
        server 192.168.141.121:8082;
    }
    # 配置一个虚拟主机
    server {
        listen 80;
        server_name service1.funtl.com;
        location / {
                # 域名 service1.funtl.com 的请求全部转发到 tomcatServer1 即 tomcat1 服务上
                proxy_pass http://tomcatServer1;
                # 欢迎页面，按照从左到右的顺序查找页面
                index index.jsp index.html index.htm;
        }
    }
    server {
        listen 80;
        server_name service2.funtl.com;
        location / {
            # 域名 service2.funtl.com 的请求全部转发到 tomcatServer2 即 tomcat2 服务上
            proxy_pass http://tomcatServer2;
            index index.jsp index.html index.htm;
        }
    }
}
```

> **注意：** 新版 Nginx 的 `MARKDOWN_HASHbc3b0556316b0ba241ae6bb86b76e8a2MARKDOWN*HASH*` *配置中的名称不可以有下划线("*")，否则会报 `400` 错误

# Nginx 负载均衡

## 什么是负载均衡

负载均衡建立在现有网络结构之上，它提供了一种廉价有效透明的方法扩展网络设备和服务器的带宽、增加吞吐量、加强网络数据处理能力、提高网络的灵活性和可用性。

负载均衡，英文名称为 Load Balance，其意思就是分摊到多个操作单元上进行执行，例如 Web 服务器、FTP 服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。

## Nginx 实现负载均衡

- Nginx 作为负载均衡服务器，用户请求先到达 Nginx，再由 Nginx 根据负载配置将请求转发至 Tomcat 服务器
- Nginx 负载均衡服务器：192.168.141.121:80
- Tomcat1 服务器：192.168.141.121:8081
- Tomcat2 服务器：192.168.141.121:8082
- 修改 `/usr/local/docker/nginx/conf` 目录下的 `nginx.conf` 配置文件：

```yaml
user  nginx;
worker_processes  1;
events {
    use epoll;
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream myapp1 {
        server 192.168.141.121:8081 weight=10;
        server 192.168.141.121:8082 weight=10;
    }
    server {
        listen 80;
        server_name nginx.funtl.com;
        location / {
            proxy_pass http://myapp1;
            index index.jsp index.html index.htm;
        }
    }
}
```

## 相关配置说明

```yaml
# 定义负载均衡设备的 Ip及设备状态 
upstream myServer {
    server 127.0.0.1:9090 down;
    server 127.0.0.1:8080 weight=2;
    server 127.0.0.1:6060;
    server 127.0.0.1:7070 backup;
}
```

在需要使用负载的 Server 节点下添加

```
proxy_pass http://myServer;
```

- `upstream`：每个设备的状态:
- `down`：表示当前的 `server` 暂时不参与负载
- `weight`：默认为 1 `weight` 越大，负载的权重就越大。
- `max_fails`：允许请求失败的次数默认为 1 当超过最大次数时，返回 `proxy_next_upstream` 模块定义的错误
- `fail_timeout`:`max_fails` 次失败后，暂停的时间。
- `backup`：其它所有的非 `backup` 机器 `down` 或者忙的时候，请求 `backup` 机器。所以这台机器压力会最轻

# Nginx Ingress Controller

## 概述

本次实践的主要目的就是将入口统一，不再通过 LoadBalancer 等方式将端口暴露出来，而是使用 Ingress 提供的反向代理负载均衡功能作为我们的唯一入口。通过以下步骤操作仔细体会。

> **注意：** 下面包含资源配置的步骤都是自行创建 YAML 配置文件通过 `kubectl create -f ` 或 `kubectl apply -f ` 部署，`kubectl delete -f ` 删除

## 安装 Ingress

Ingress Controller 有许多种，我们选择最熟悉的 Nginx 来处理请求，其它可以参考 [官方文档]( https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/ )

- 下载 Nginx Ingress Controller 配置文件

```shell
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
```

- 修改配置文件，找到配置如下位置 (搜索 `serviceAccountName`) 在下面增加一句 `hostNetwork: true`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  # 可以部署多个实例
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      # 增加 hostNetwork: true，意思是开启主机网络模式，暴露 Nginx 服务端口 80
      hostNetwork: true
      containers:
        - name: nginx-ingress-controller
          # 使用 Azure 中国镜像
          image: quay.azk8s.cn/kubernetes-ingress-controller/nginx-ingress-controller:0.24.1
          args:
            - /nginx-ingress-controller
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
// 以下代码省略...
```

- 通过命令 `kubectl apply -f mandatory.yaml` 部署
- 通过命令 `kubectl get pods -n ingress-nginx -o wide` 查看

## 部署 Ingress

- 创建一个名为 `ingress.yml` 的资源配置文件

```yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: nginx-web
  annotations:
    # 指定 Ingress Controller 的类型
    kubernetes.io/ingress.class: "nginx"
    # 指定我们的 rules 的 path 可以使用正则表达式
    nginx.ingress.kubernetes.io/use-regex: "true"
    # 连接超时时间，默认为 5s
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "600"
    # 后端服务器回转数据超时时间，默认为 60s
    nginx.ingress.kubernetes.io/proxy-send-timeout: "600"
    # 后端服务器响应超时时间，默认为 60s
    nginx.ingress.kubernetes.io/proxy-read-timeout: "600"
    # 客户端上传文件，最大大小，默认为 20m
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    # URL 重写
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  # 路由规则
  rules:
  # 主机名，只能是域名，修改为你自己的
  - host: k8s.funtl.com
    http:
      paths:
      - path:
        backend:
          # 后台部署的 Service Name
          serviceName: tomcat-http
          # 后台部署的 Service Port
          servicePort: 8080
```

- 通过命令 `kubectl apply -f ingress.yml` 部署
- 通过命令 `kubectl get ingress` 查看

## 部署 Tomcat

部署 Tomcat 但仅允许在内网访问，我们要通过 Ingress 提供的反向代理功能路由到 Tomcat 之上，创建一个名为 `tomcat.yml` 资源配置文件

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: tomcat-app
spec:
  replicas: 2
  template:
    metadata:
      labels:
        name: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5.43
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat-http
spec:
  ports:
    - port: 8080
      targetPort: 8080
  # ClusterIP, NodePort, LoadBalancer
  type: ClusterIP
  selector:
    name: tomcat
```

- 通过命令 `kubectl apply -f tomcat.yml` 部署

## 验证是否成功

### 查看 Tomcat

```shell
kubectl get deployment
# 输出如下
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
tomcat-app   2/2     2            2           88m
```

```shell
kubectl get service
# 输出如下
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP    2d5h
tomcat-http   ClusterIP   10.97.222.179   <none>        8080/TCP   89m
```

### 查看 Ingress

```shell
kubectl get pods -n ingress-nginx -o wide
# 输出如下，注意下面的 IP 地址，就是我们实际访问地址
NAME                                        READY   STATUS    RESTARTS   AGE   IP                NODE                 NOMINATED NODE   READINESS GATES
nginx-ingress-controller-76f9fddcf8-vzkm5   1/1     Running   0          61m   192.168.141.121   kubernetes-node-02   <none>       
```

```shell
kubectl get ingress
# 输出如下
NAME        HOSTS          ADDRESS   PORTS   AGE
nginx-web   k8s.funtl.com             80      61m
```

### 测试访问

成功代理到 Tomcat 即表示成功

```yaml
# 不设置 Hosts 的方式请求地址，下面的 IP 和 HOST 均在上面有配置
curl -v http://192.168.141.121 -H 'host: k8s.funtl.com'
```

# Kubernetes 准备数据卷

## 概述

在 Docker 中就有数据卷的概念，当容器删除时，数据也一起会被删除，想要持久化使用数据，需要把主机上的目录挂载到 Docker 中去，在 K8S 中，数据卷是通过 Pod 实现持久化的，如果 Pod 删除，数据卷也会一起删除，k8s 的数据卷是 docker 数据卷的扩展，K8S 适配各种存储系统，包括本地存储 EmptyDir，HostPath， 网络存储（NFS，GlusterFS，PV/PVC）等。

我们以部署 MySQL8 为例，采用 **NFS + PV/PVC** 网络存储方案实现我们的 Kubernetes 数据持久化。

## 什么是 NFS

NFS 是 Network File System 的简写，即网络文件系统，NFS 是 FreeBSD 支持的文件系统中的一种。NFS 基于 RPC (Remote Procedure Call) 远程过程调用实现，其允许一个系统在网络上与它人共享目录和文件。通过使用 NFS，用户和程序就可以像访问本地文件一样访问远端系统上的文件。NFS 是一个非常稳定的，可移植的网络文件系统。具备可扩展和高性能等特性，达到了企业级应用质量标准。由于网络速度的增加和延迟的降低，NFS 系统一直是通过网络提供文件系统服务的有竞争力的选择 。

### NFS 原理

NFS 使用 RPC (Remote Procedure Call) 的机制进行实现，RPC 使得客户端可以调用服务端的函数。同时，由于有 VFS 的存在，客户端可以像使用其它普通文件系统一样使用 NFS 文件系统。经由操作系统的内核，将 NFS 文件系统的调用请求通过 TCP/IP 发送至服务端的 NFS 服务。NFS 服务器执行相关的操作，并将操作结果返回给客户端。

![img](Kubernetes2/4c530fd29b79c58.jpg)

### NFS 服务主要进程

- rpc.nfsd：最主要的 NFS 进程，管理客户端是否可登录
- rpc.mountd：挂载和卸载 NFS 文件系统，包括权限管理
- rpc.lockd：非必要，管理文件锁，避免同时写出错
- rpc.statd：非必要，检查文件一致性，可修复文件

### NFS 的关键工具

- 主要配置文件：`/etc/exports`
- NFS 文件系统维护命令：`/usr/bin/exportfs`
- 共享资源的日志文件：` /var/lib/nfs/*tab`
- 客户端查询共享资源命令：` /usr/sbin/showmount`
- 端口配置：` /etc/sysconfig/nfs`

### NFS 服务端配置

在 NFS 服务器端的主要配置文件为 `/etc/exports` 时，通过此配置文件可以设置共享文件目录。每条配置记录由 NFS 共享目录、NFS 客户端地址和参数这 3 部分组成，格式如下：

```json
[NFS 共享目录] [NFS 客户端地址 1 (参数 1, 参数 2, 参数 3……)] [客户端地址 2 (参数 1, 参数 2, 参数 3……)]
```

- NFS 共享目录：服务器上共享出去的文件目录
- NFS 客户端地址：允许其访问的 NFS 服务器的客户端地址，可以是客户端 IP 地址，也可以是一个网段 (192.168.141.0/24)
- 访问参数：括号中逗号分隔项，主要是一些权限选项

#### 访问权限参数

| 序号 | 选项 | 描述                                   |
| :--- | :--- | :------------------------------------- |
| 1    | ro   | 客户端对于共享文件目录为只读权限。默认 |
| 2    | rw   | 客户端对于共享文件目录具有读写权限     |

#### 用户映射参数

| 序号 | 选项           | 描述                                                         |
| :--- | :------------- | :----------------------------------------------------------- |
| 1    | root_squash    | 使客户端使用 root 账户访冋时，服务器映射为服务器本地的匿名账号 |
| 2    | no_root_squash | 客户端连接服务端时如果使用的是 root，那么也拥有对服务端分享的目录的 root 权限 |
| 3    | all_squash     | 将所有客户端用户请求映射到匿名用户或用户组（nfsnobody)       |
| 4    | no_all_squash  | 与上相反。默认                                               |
| 5    | anonuid=xxx    | 将远程访问的所有用户都映射为匿名用户，并指定该用户为本地用户(UID=xxx) |
| 6    | anongid=xxx    | 将远程访问的所有用户组都映射为匿名用户组账户，并指定该匿名用户组账户为本地用户组账户(GUI=xxx) |

#### 其它配置参数

| 序号 | 选项       | 描述                                                         |
| :--- | :--------- | :----------------------------------------------------------- |
| 1    | sync       | 同步写操作，数据写入存储设备后返回成功信息。默认             |
| 2    | async      | 异步写提作，数据在未完全写入存储设备前就返回成功信息，实际还在内存， |
| 3    | wdelay     | 延迟写入选项，将多个写提请求合并后写入硬盘，减少 I/O 次数， NFS 非正常关闭数据可能丢失。默认 |
| 4    | no_wdelay  | 与上相反，不与 async 同时生效，如果 NFS 服务器主要收到小且不相关的请求，该选项实际会降低性能 |
| 5    | subtree    | 若输出目录是一个子目录，则 NFS 服务器将检查其父目录的权限。默认 |
| 6    | no_subtree | 即使输出目录是一个子目录， NFS 服务器也不检查其父目录的权限，这样可以提高效率 |
| 7    | secure     | 限制客户端只能从小于 1024 的 TCP/IP 端口连接 NFS 服务器。默认 |
| 8    | insecure   | 允许客户端从大于 1024 的 TCP/IP 端口连接服务器               |

## 安装 NFS 服务端

由于 NFS 是一套分布式文件系统，我们再创建一台独立的虚拟机作为我们 NFS 服务端，配置如下

| 主机名             | IP              | 系统                | CPU/内存 | 磁盘 |
| :----------------- | :-------------- | :------------------ | :------- | :--- |
| kubernetes-volumes | 192.168.141.130 | Ubuntu Server 18.04 | 2 核 2G  | 20G  |

- 创建一个目录作为共享文件目录

```shell
mkdir -p /usr/local/kubernetes/volumes
```

- 给目录增加读写权限

```shell
chmod a+rw /usr/local/kubernetes/volumes
```

- 安装 NFS 服务端

```shell
apt-get update
apt-get install -y nfs-kernel-server
```

- 配置 NFS 服务目录，打开文件vi /etc/exports，在尾部新增一行，内容如下
  - **`/usr/local/kubernetes/volumes`：** 作为服务目录向客户端开放
  - ***：**表示任何 IP 都可以访问
  - **rw：** 读写权限
  - **sync：** 同步权限
  - **no_subtree_check：** 表示如果输出目录是一个子目录，NFS 服务器不检查其父目录的权限
  - **no_root_squash：** 客户端连接服务端时如果使用的是 root，那么也拥有对服务端分享的目录的 root 权限

```shell
/usr/local/kubernetes/volumes *(rw,sync,no_subtree_check,no_root_squash)
```

- 重启服务，使配置生效

```shell
/etc/init.d/nfs-kernel-server restart
```

## 安装 NFS 客户端

安装客户端的目的是验证是否可以上传文件到服务端，安装命令如下

```powershell
apt-get install -y nfs-common
```

- 创建 NFS 客户端挂载目录

```shell
mkdir -p /usr/local/kubernetes/volumes-mount
```

- 将 NFS 服务器的 `/usr/local/kubernetes/volumes` 目录挂载到 NFS 客户端的 `/usr/local/kubernetes/volumes-mount` 目录

```shell
mount 192.168.141.130:/usr/local/kubernetes/volumes /usr/local/kubernetes/volumes-mount
```

- 使用 `df` 命令查看挂载信息

```shell
df
# 输出如下
Filesystem                                    1K-blocks    Used Available Use% Mounted on
udev                                             977544       0    977544   0% /dev
tmpfs                                            201732    1232    200500   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv              19475088 4780912  13681852  26% /
tmpfs                                           1008648       0   1008648   0% /dev/shm
tmpfs                                              5120       0      5120   0% /run/lock
tmpfs                                           1008648       0   1008648   0% /sys/fs/cgroup
/dev/loop0                                        90624   90624         0 100% /snap/core/7270
/dev/loop1                                        93184   93184         0 100% /snap/core/6350
/dev/sda2                                        999320   77944    852564   9% /boot
tmpfs                                            201728       0    201728   0% /run/user/0
# 有此输出表示挂载成功
192.168.141.130:/usr/local/kubernetes/volumes  19475200 4780800  13681920  26% /usr/local/kubernetes/volumes-mount
```

## 验证 NFS 服务

- 测试文件上传

```shell
ip addr > /usr/local/kubernetes/volumes-mount/test.txt
```

- 查看 `/usr/local/kubernetes/volumes` 目录下是否有 `test.txt` 文件，有则表示成功

## 取消 NFS 客户端挂载

> **注意：** 不要直接在挂载目录下执行，否则会报错

```shell
umount /usr/local/kubernetes/volumes-mount
```

# Kubernetes 使用数据卷

## 概述

存储管理与计算管理是两个不同的问题。Persistent Volume 子系统，对存储的供应和使用做了抽象，以 API 形式提供给管理员和用户使用。要完成这一任务，我们引入了两个新的 API 资源：**Persistent Volume（持久卷）** 和 **Persistent Volume Claim（持久卷消费者）**。

Persistent Volume（PV）是集群之中的一块网络存储。跟 Node 一样，也是集群的资源。PV 跟 Volume (卷) 类似，不过会有独立于 Pod 的生命周期。这一 API 对象包含了存储的实现细节，例如 NFS、iSCSI 或者其他的云提供商的存储系统。Persistent Volume Claim (PVC) 是用户的一个请求。跟 Pod 类似，Pod 消费 Node 的资源，PVC 消费 PV 的资源。Pod 能够申请特定的资源（CPU 和内存）；Claim 能够请求特定的尺寸和访问模式（例如可以加载一个读写，以及多个只读实例）

## PV 与 PVC

PV 是集群的资源。PVC 是对这一资源的请求，也是对资源的所有权的检验。PV 和 PVC 之间的互动遵循如下的生命周期。

- **供应：** 集群管理员会创建一系列的 PV。这些 PV 包含了为集群用户提供的真实存储资源，它们可利用 Kubernetes API 来消费。
- **绑定：** 用户创建一个包含了容量和访问模式的持久卷申请。Master 会监听 PVC 的产生，并尝试根据请求内容查找匹配的 PV，并把 PV 和 PVC 进行绑定。用户能够获取满足需要的资源，并且在使用过程中可能超出请求数量。如果找不到合适的卷，这一申请就会持续处于非绑定状态，一直到出现合适的 PV。例如一个集群准备了很多的 50G 大小的持久卷，（虽然总量足够）也是无法响应 100G 的申请的，除非把 100G 的 PV 加入集群。
- **使用：** Pod 把申请作为卷来使用。集群会通过 PVC 查找绑定的 PV，并 Mount 给 Pod。对于支持多种访问方式的卷，用户在使用 PVC 作为卷的时候，可以指定需要的访问方式。一旦用户拥有了一个已经绑定的 PVC，被绑定的 PV 就归该用户所有了。用户的 Pods 能够通过在 Pod 的卷中包含的 PVC 来访问他们占有的 PV。
- **释放：** 当用户完成对卷的使用时，就可以利用 API 删除 PVC 对象了，而且他还可以重新申请。删除 PVC 后，对应的卷被视为 “被释放”，但是这时还不能给其他的 PVC 使用。之前的 PVC 数据还保存在卷中，要根据策略来进行后续处理。
- **回收：** PV 的回收策略向集群阐述了在 PVC 释放卷的时候，应如何进行后续工作。目前可以采用三种策略：保留，回收或者删除。保留策略允许重新申请这一资源。在持久卷能够支持的情况下，删除策略会同时删除持久卷以及 AWS EBS/GCE PD 或者 Cinder 卷中的存储内容。如果插件能够支持，回收策略会执行基础的擦除操作（`rm -rf /thevolume/*`），这一卷就能被重新申请了。

## 定义 PV

### 持久卷插件

持久卷是以插件方式实现的，目前支持的插件如下：

- GCEPersistentDisk
- AWSElasticBlockStore
- **NFS（我们采用的是该方案）**
- iSCSI
- RBD (Ceph Block Device)
- Glusterfs
- HostPath (单节点测试使用)
- 本地持久卷

### YAML 配置

创建一个名为 `nfs-pv-mysql.yml` 的配置文件

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv-mysql
spec:
  # 设置容量
  capacity:
    storage: 5Gi
  # 访问模式
  accessModes:
    # 该卷能够以读写模式被多个节点同时加载
    - ReadWriteMany
  # 回收策略，这里是基础擦除 `rm-rf/thevolume/*`
  persistentVolumeReclaimPolicy: Recycle
  nfs:
    # NFS 服务端配置的路径
    path: "/usr/local/kubernetes/volumes"
    # NFS 服务端地址
    server: 192.168.141.130
    readOnly: false
```

```shell
# 部署
kubectl create -f nfs-pv-mysql.yml
# 删除
kubectl delete -f nfs-pv-mysql.yml
# 查看
kubectl get pv
NAME           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
nfs-pv-mysql   5Gi        RWX            Recycle          Available                                   29m
```

### 配置说明

#### Capacity（容量）

一般来说，PV 会指定存储容量。这里需要使用 PV 的 capcity 属性。目前存储大小是唯一一个能够被申请的指标，今后会加入更多属性，例如 IOPS，吞吐能力等。

#### AccessModes（访问模式）

只要资源提供者支持，持久卷能够被用任何方式加载到主机上。每种存储都会有不同的能力，每个 PV 的访问模式也会被设置成为该卷所支持的特定模式。例如 NFS 能够支持多个读写客户端，但是某个 NFS PV 可能会在服务器上以只读方式使用。每个 PV 都有自己的一系列的访问模式，这些访问模式取决于 PV 的能力。访问模式的可选范围如下：

- **ReadWriteOnce：** 该卷能够以读写模式被加载到一个节点上
- **ReadOnlyMany：** 该卷能够以只读模式加载到多个节点上
- **ReadWriteMany：** 该卷能够以读写模式被多个节点同时加载

在 CLI 下，访问模式缩写为：

- **RWO：** ReadWriteOnce
- **ROX：** ReadOnlyMany
- **RWX：** ReadWriteMany

另外，一个卷不论支持多少种访问模式，同时只能以一种访问模式加载。例如一个 GCE Persistent Disk 既能支持 ReadWriteOnce，也能支持 ReadOnlyMany。

#### RecyclingPolicy（回收策略）

当前的回收策略可选值包括：

- **Retain：** 人工重新申请
- **Recycle：** 基础擦除（`rm-rf/thevolume/*`）
- **Delete：** 相关的存储资产例如 AWS EBS，GCE PD 或者 OpenStack Cinder 卷一并删除

目前，只有 NFS 和 HostPath 支持 Recycle 策略，AWS EBS、GCE PD 以及 Cinder 卷支持 Delete 策略。

#### 阶段（Phase）

一个卷会处于如下阶段之一：

- **Available：** 可用资源，尚未被绑定到 PVC 上
- **Bound：** 该卷已经被绑定
- **Released：** PVC 已经被删除，但该资源尚未被集群回收
- **Failed：** 该卷的自动回收过程失败

## 定义 PVC

创建一个名为 `nfs-pvc-mysql-myshop.yml` 的配置文件

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc-mysql-myshop
spec:
  accessModes:
  # 需要使用和 PV 一致的访问模式
  - ReadWriteMany
  # 按需分配资源
  resources:
     requests:
       storage: 1Gi
```

```shell
# 部署
kubectl create -f nfs-pvc-mysql-myshop.yml
# 删除
kubectl delete -f nfs-pvc-mysql-myshop.yml
# 查看
kubectl get pvc
```

## 部署 MySQL8

> **注意：** 要确保每台 Node 都安装了 NFS 客户端，`apt-get install -y nfs-common`

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: mysql-myshop
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: mysql-myshop
    spec:
      containers:
        - name: mysql-myshop
          image: mysql:8.0.16
          # 只有镜像不存在时，才会进行镜像拉取
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3306
          # 同 Docker 配置中的 environment
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "123456"
          # 容器中的挂载目录
          volumeMounts:
            - name: nfs-vol-myshop
              mountPath: /var/lib/mysql
      volumes:
        # 挂载到数据卷
        - name: nfs-vol-myshop
          persistentVolumeClaim:
            claimName: nfs-pvc-mysql-myshop
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-myshop
spec:
  ports:
    - port: 3306
      targetPort: 3306
  type: LoadBalancer
  selector:
    name: mysql-myshop
```

### 测试运行

部署成功后可以使用 `kubectl get service` 查看我们 MySQL 的运行端口，再使用连接工具连接会报如下错误

![img](Kubernetes2/85bc9545ce9ec0c.png)

意思为无法使用密码的方式登录，在 Docker 部署时我们可以在 YAML 中配置相关参数解决这个问题；下一节我们讲解在 Kubernetes 中采用 **ConfigMap** 的方式配置 MySQL

# Kubernetes ConfigMap

## 概述

ConfigMap 是用来存储配置文件的 Kubernetes 资源对象，所有的配置内容都存储在 etcd 中。它可以被用来保存单个属性，也可以用来保存整个配置文件或者 JSON 二进制对象。ConfigMap API 资源提供了将配置数据注入容器的方式，同时保证该机制对容器来说是透明的。配置应该从 Image 内容中解耦，以此来保持容器化应用程序的可移植性。

## 使用 ConfigMap 配置 MySQL

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-myshop-config
data:
  # 这里是键值对数据
  mysqld.cnf: |
    [client]
    port=3306
    [mysql]
    no-auto-rehash
    [mysqld]
    skip-host-cache
    skip-name-resolve
    default-authentication-plugin=mysql_native_password
    character-set-server=utf8mb4
    collation-server=utf8mb4_general_ci
    explicit_defaults_for_timestamp=true
    lower_case_table_names=1
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: mysql-myshop
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: mysql-myshop
    spec:
      containers:
        - name: mysql-myshop
          image: mysql:8.0.16
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 3306
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "123456"
          volumeMounts:
            # 以数据卷的形式挂载 MySQL 配置文件目录
            - name: cm-vol-myshop
              mountPath: /etc/mysql/conf.d
            - name: nfs-vol-myshop
              mountPath: /var/lib/mysql
      volumes:
        # 将 ConfigMap 中的内容以文件形式挂载进数据卷
        - name: cm-vol-myshop
          configMap:
            name: mysql-myshop-config
            items:
                # ConfigMap 中的 Key
              - key: mysqld.cnf
                # ConfigMap Key 匹配的 Value 写入名为 mysqld.cnf 的文件中
                path: mysqld.cnf
        - name: nfs-vol-myshop
          persistentVolumeClaim:
            claimName: nfs-pvc-mysql-myshop
---
apiVersion: v1
kind: Service
metadata:
  name: mysql-myshop
spec:
  ports:
    - port: 3306
      targetPort: 3306
  type: LoadBalancer
  selector:
    name: mysql-myshop
```

```shell
# 查看 ConfigMap
kubectl get cm
kubectl describe cm <ConfigMap Name>
```

# Kubernetes Dashboard

## 概述

Kubernetes Dashboard 是 Kubernetes 集群的 Web UI，用于管理集群。

## 安装

GitHub 地址：[Kubernetes Dashboard]( https://github.com/kubernetes/dashboard )

下载配置文件

```shell
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0-beta2/aio/deploy/recommended.yaml
```

修改配置如下

```yaml
# 省略部分代码...
# ------------------- Dashboard Service ------------------- #
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  # 修改类型为 NodePort 访问
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
      # 设置端口号为 30001
      nodePort: 30001
  selector:
    k8s-app: kubernetes-dashboard
```

部署到集群

```shell
# 部署
kubectl create -f recommended.yaml
# 查看
kubectl -n kubernetes-dashboard get pods
kubectl -n kubernetes-dashboard get service kubernetes-dashboard
kubectl -n kubernetes-dashboard describe service kubernetes-dashboard
```

## 访问

需要使用 NodeIP:30001 访问 Dashboard，因为证书原因除火狐浏览器外其它浏览器无法直接打开页面

Chrome 浏览器显示如下

![img](Kubernetes2/fcacca76f64339a.png)

Firefox 浏览器显示如下

![img](Kubernetes2/3a719c594e6fdeb.png)

点击 **接受风险并继续** 即可显示欢迎界面

![img](Kubernetes2/af3fa90ff0439c4.png)

## 登录

我们采用 Token 方式登录

- 创建登录账号，创建一个名为 `dashboard-adminuser.yaml` 的配置文件

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
```

```shell
kubectl create -f dashboard-adminuser.yaml
```

- 打印 Token 信息

```shell
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
# 输出如下
Name:         admin-user-token-2v8vd
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: f6f23bc9-3f34-4452-9977-8915e77fbef3
Type:  kubernetes.io/service-account-token
Data
====
namespace:  20 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLTJ2OHZkIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmNmYyM2JjOS0zZjM0LTQ0NTItOTk3Ny04OTE1ZTc3ZmJlZjMiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.d7TVh5L9OKBLY62FXF8ZDjp7dZMH6hmOgd0QKCwPaNEGL_pzJrz1j5JC0TlSaQgFXJKVhUq3WzBsKC2yAsgOc53AKNfJbv0UWKVk5sLXwi79rZiaUdB6cGHOVxiMxUHsluhDAs7DoALyOT3svY5JSh_f5f5h92ZUxBTx5CeFHr47B_MnjG0kgeRR7p8i2vV1OLnW0JQDg-eI0usfFYIzjBy3z1NiVjJJ8ON_ygUNapSXQLuLanjv0Vei_XVTaxfqL9-8AabSm_VDo-mTkqDJWK-utBvQ1AajOWs9fbz8ey6GiC33KevaxxPjNvKfIJI2h0FvDUffy6qysT-IuN2ZEA
ca.crt:     1025 bytes
```

- 将 Token 输入浏览器，成功登陆后效果如下

![img](Kubernetes2/9f4c1fb1a6afb0a.png)

# 【补充】使用 Kuboard 替代 Kubernetes Dashboard

## 概述

Kubernetes 已然是当下容器编排领域事实上的标准，各大云服务商都急于推出 Kubernetes 服务，互联网公司也纷纷跟进，将自己的应用容器化，并使用 Kubernetes 编排，在 Kubernetes 图形化工具方面，我们已经获得了极大的可选择空间：

- 各云服务商自己推出的 Kubernetes 服务所搭载的管理控制台，例如 阿里云的 Kubernetes 服务，青云推出的 KubeSphere，其他云服务商的 CaaS 类服务
- Kubernetes 官方的图形管理界面 Kubernetes Dashboard
- 面向企业私有化部署的 Rancher

## 什么是 Kuboard

Kuboard 类似于 Kubernetes Dashboard，是一款轻量级产品，按照 Kuboard 的文档，您可以很快地搭建一套学习 Kubernetes 用的集群环境，您也可以在自己已经有的集群上轻易地完成 Kuboard 的安装

## 安装 Kuboard

如果您已经有了 Kubernetes 集群，只需要一行命令即可安装 Kuboard：

```shell
kubectl apply -f https://kuboard.cn/install-script/kuboard.yaml
```

然后访问您集群中任意节点的 32567 端口（[http://any-of-your-node-ip:32567]()） ，即可打开 Kuboard 界面，比如我的 Node 节点 IP 为：[http://10.3.50.121:32567]()

### 获取 Token

此 Token 拥有 ClusterAdmin 的权限，可以执行所有操作

```shell
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep kuboard-user | awk '{print $1}')
# 输出如下
Name:         kuboard-user-token-dsccx
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: kuboard-user
              kubernetes.io/service-account.uid: 08aad1b1-3d1b-4b6f-a379-3e320620a09f
Type:  kubernetes.io/service-account-token
Data
====
ca.crt:     1025 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJrdWJvYXJkLXVzZXItdG9rZW4tZHNjY3giLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoia3Vib2FyZC11c2VyIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMDhhYWQxYjEtM2QxYi00YjZmLWEzNzktM2UzMjA2MjBhMDlmIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmt1Ym9hcmQtdXNlciJ9.rndy-u-CxDF_yfaubq2HmIb7o04uLI3z7soqTf581VPLFo31dLmQvDPBJXWmFXHAfy0f-msCZjXWU35Cvwvlg3A5LZXgmfsJJi-kwYgpjnsOpFqquezAR6mDDnyXC1NLb8HD4PXgb4wbqnz6-jXlj7T0EXaDK0U5JvNtnkVy8y66mucz6tGnANqLYsHIYPT86P2VTOAWPrnz3pJHtoo_2AMI_y5Pc9RiVH1cro5I5D8Qf8_XDBE4oTsNXaqvWItnjg1O-nhxwsRpe-Szbe9wtFLbqjsO0_ZF5jPT1SdNrkuzoRJ4FpJsjBpbSpLB_DXtc0Y15oJVNZwpmJXKywCXRw
```

### 登录 Kuboard

使用刚才生成的 Token 直接登录即可

![img](Kubernetes2/2d5d15920d2ff56.png)

![img](Kubernetes2/9a6a1abff8f2ee5.png)

## 为什么使用 Kuboard

### 非侵入式

Rancher 为了支持多云管理（MeSos，Docker Swarm，Kubernetes 等），引入了一系列复杂的概念，又需要新增安装 Rancher Server、Rancher Agent 等，本来学习 Kubernetes 已然不堪重负，Rancher 再把多云整合到一起来，无疑又增加了学习和使用的复杂性。如果您只是打算使用 Kubernetes 的话，也许并不需要选择 Rancher 这样重量级的产品。Kuboard 仅仅依赖于原生 Kubernetes，可以运行在各种公有云、私有云上，您也可以自己基于物理机、vsphere、vmware 等已经有的基础设施搭建 Kubernetes，轻松实现现有 Infrastructure 的容器化改造

### 名称空间

在 Kuboard 中，名称空间的展示形式以微服务参考分层架构的形式，将所有的微服务分为如下几层：

- 展现层
  - 终端用户访问的 Web 应用
- API 网关层
  - Spring Cloud Gateway / Zuul / Kong 等接口网关
- 微服务层
  - Spring Boot 微服务，或 PHP / Python / Go 实现的微服务
- 持久层
  - MySQL 数据库等（开发及测试环境里，将 MySQL 部署于 Kubernetes 可以极大地降低环境维护的任务量）
- 中间件层
  - 消息队列
  - 服务注册 Eureka / Zookeeper / Consul 等
- 监控层
  - Prometheus + Grafana
  - Pinpooint 等

![img](Kubernetes2/9843af61439f784.jpg)

更为贴心的，当某一个工作负载存在部署方面的问题时， Kuboard 将以红色高亮出该工作负载，并配有合适的方式辅助用户定位错误信息

### 无需手写 Yaml

学习 Kubernetes 时，需要花费许多的时间，在理解一个概念之后，模仿着写一个 `yaml` 文件，再使用 `kubectl` 应用该文件，使用 Kuboard 提供的工作负载编辑器，可以直观的完成应用的部署

![img](Kubernetes2/c6c02c114acc74a.png)

### 应用升级

Kuboard 提供批量修改容器镜像版本的功能，轻松升级应用程序的版本

![img](Kubernetes2/d08f0d4acdff85d.jpg)

完成

# 亲测体验

### 修改主机名称和IP地址

#### 修改hostname

1、查看当前hostname

```shell
[root@localhost~]# hostname
localhost
```

2、配置新主机名

```shell
[root@localhost~]# hostnamectl set-hostname kubernetes-master
```

3、修改/etc/hosts文件并保存

```shell
[root@localhost~]# vi /etc/hosts
127.0.0.1  localhost localhost.localdomain localhost4 localhost4.localdomain4 test88（主机名称）
::1     localhost localhost.localdomain localhost6 localhost6.localdomain6

：wq
```

#### 修改ip地址

1、查看本机网卡信息

```shell
ip addr
```

2、编辑对应的网卡文件

注意网卡名称

 [root@test00 ~]#vi /etc/sysconfig/network-scripts/ifcfg-ens33

```test
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static #静态Ip
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=556a7ab3-7f57-4153-84e8-e0b605a69e54
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.10.140  # ip地址
NETMASK=255.255.255.0  #子网掩码
GATEWAY=192.168.10.2	#网关  连不上网络网关出错
DNS1=8.8.4.4
DNS2=8.8.8.8
```

修改对应的ip地址，保存退出

3、重启网络服务即可生效

[root@test00 ~]# service network restart

####  关闭防火墙

1、直接关闭防火墙

```shell
systemctl stop firewalld.service #停止firewall
systemctl disable firewalld.service #禁止firewall开机启动
```



### 安装  **kubeadm**，**kubelet**，**kubectl** 

```shell
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
# 安装
yum install -y kubectl kubelet kubeadm
# 开机启动
systemctl enable kubelet
# 启动
systemctl start kubelet
```

### centos7设置时间和时区

1、安装ntp服务软件包：yum install ntp

2、将ntp设置为缺省启动：systemctl enable ntpd

3、修改启动参数，增加-g -x参数，允许ntp服务在系统时间误差较大时也能正常工作：vi /etc/sysconfig/ntpd

```shell
# Command line options for ntpd
OPTIONS ="-g -x"
```

4、启动ntp服务：service ntpd restart

5、将系统时区改为上海时间（即CST时区）：ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

6、输入date命令查看时间是否正确

### 坑一

__错误__

```shell
Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

__解决办法 __

```shell
echo "1" >/proc/sys/net/bridge/bridge-nf-call-iptables
modprobe br_netfilter
```

### 坑二

__错误__

```shell
[kubelet-check] Initial timeout of 40s passed.

Unfortunately, an error has occurred:
	timed out waiting for the condition

This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
	- 'systemctl status kubelet'
	- 'journalctl -xeu kubelet'

Additionally, a control plane component may have crashed or exited when started by the container runtime.
To troubleshoot, list all containers using your preferred container runtimes CLI, e.g. docker.
Here is one example how you may list all Kubernetes containers running in docker:
	- 'docker ps -a | grep kube | grep -v pause'
	Once you have found the failing container, you can inspect its logs with:
	- 'docker logs CONTAINERID'
error execution phase wait-control-plane: couldn't initialize a Kubernetes cluster
To see the stack trace of this error execute with --v=5 or higher
```

__Docker 容器错误日志__

```shell
2019-11-14 06:52:44.573523 I | etcdmain: etcd Version: 3.3.15
2019-11-14 06:52:44.573563 I | etcdmain: Git SHA: 94745a4ee
2019-11-14 06:52:44.573566 I | etcdmain: Go Version: go1.12.9
2019-11-14 06:52:44.573568 I | etcdmain: Go OS/Arch: linux/amd64
2019-11-14 06:52:44.573571 I | etcdmain: setting maximum number of CPUs to 2, total number of available CPUs is 2
2019-11-14 06:52:44.573865 I | embed: peerTLS: cert = /etc/kubernetes/pki/etcd/peer.crt, key = /etc/kubernetes/pki/etcd/peer.key, ca = , trusted-ca = /etc/kubernetes/pki/etcd/ca.crt, client-cert-auth = true, crl-file = 
2019-11-14 06:52:44.574088 C | etcdmain: open /etc/kubernetes/pki/etcd/peer.crt: permission denied
```

__解决办法__

```shell
[root@localhost ~]# setenforce 0 //临时关闭
[root@localhost ~]# getenforce
Permissive
[root@localhost ~]# vi /etc/sysconfig/selinux //永久关闭
将SELINUX=enforcing 改为 SELINUX=disabled 。
```

### 坑三

Kubernetes的网络配置calico安装启动不起来，分支没连接上去

```shell
NAMESPACE     NAME                                        READY   STATUS                  RESTARTS   AGE
kube-system   calico-kube-controllers-6b64bcd855-6s8fb    1/1     Running                 0          7m54s
kube-system   calico-node-bcgjn                           1/1     Running                 0          7m54s
kube-system   calico-node-hkjq2                           0/1     Init:CrashLoopBackOff   6          7m54s
kube-system   calico-node-vmftt                           1/1     Running                 0          7m54s
kube-system   coredns-58cc8c89f4-427vg                    1/1     Running                 0          24m
kube-system   coredns-58cc8c89f4-dqpbp                    1/1     Running                 0          24m
kube-system   etcd-kubernetes-master                      1/1     Running                 1          23m
kube-system   kube-apiserver-kubernetes-master            1/1     Running                 1          23m
kube-system   kube-controller-manager-kubernetes-master   1/1     Running                 1          23m
kube-system   kube-proxy-5sfmm                            1/1     Running                 1          24m
kube-system   kube-proxy-gh8sd                            1/1     Running                 1          21m
kube-system   kube-proxy-mxgbp                            1/1     Running                 1          20m
kube-system   kube-scheduler-kubernetes-master            1/1     Running                 1          23m
```

__解决办法__ 对分支

```shell
[root@localhost ~]# setenforce 0 //临时关闭
[root@localhost ~]# getenforce
Permissive
[root@localhost ~]# vi /etc/sysconfig/selinux //永久关闭
将SELINUX=enforcing 改为 SELINUX=disabled 。
```



