---
title: v2ray
date: 2020-4-15 15:04:20
tags:
	
categories: 乱炖
---

 ## **V2Ray**

**V2Ray**是近几年兴起的科学上网技术，采用新的协议，因功能强大、能有效抵抗墙的干扰而广受好评。**V2Ray官网**是 [https://v2ray.com](https://v2ray.com/)，目前已无法直接打开。V2Ray安装部署及流量伪装请参考：[V2Ray教程](https://tlanyan.me/v2ray-tutorial) 和 [V2Ray高级技巧：流量伪装](https://tlanyan.me/v2ray-traffic-mask/)。

## 安装

### 安装docker

内容自己百度

### 利用docker安装v2ray docker容器

#### 1. 拉取v2ray docker镜像

```
sudo docker pull v2ray/official
```

#### 2. 在 /etc 目录下新建一个文件夹 v2ray, 并把你的配置写好后命名为 config.json 放入 v2ray 文件夹内**（ 这一步至关重要 ）**

> /etc/v2ray/config.json ，如下：

```
{
  "log": {
    "access": "etc/v2ray/access.log",
    "error": "etc/v2ray/error.log",
    "loglevel": "warning"
  },
  "dns": {},
  "stats": {},
  "inbounds": [
    {
      "port": 80,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "",
            "alterId": 32
          }
        ]
      },
      "tag": "in-0",
      "streamSettings": {
        "network": "tcp",
        "security": "none",
        "tcpSettings": {}
      }
    }
  ],
  "outbounds": [
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked"
      }
    ]
  },
  "policy": {},
  "reverse": {},
  "transport": {}
}
```

#### 3. 部署v2ray docker容器

> 将vps的 /etc/v2ray 文件夹映射到 v2ray docker 容器的 /etc/v2ray 文件夹下

> 将vps的 8888 端口映射到 v2ray docker 容器的 80 端口

```
sudo docker run -d --name v2ray -v /etc/v2ray:/etc/v2ray -p 9999:80 v2ray/official v2ray -config=/etc/v2ray/config.json
```

键入以上命令后,命令行会出现一串字符,代表容器部署成功。可以立即通过客户端连接并开始使用了，如果不行，则查看 docker log。

通过以下命令来启动 V2Ray:

```
sudo docker container start v2ray
```

停止 V2Ray:

```
sudo docker container stop v2ray
```

重启 V2Ray:

```
sudo docker container restart v2ray
```

查看日志:

```
sudo docker container logs v2ray
```

更新配置后，需要重新部署容器，命令如下：

> 1. 先停止运行的容器。2. 再移除容器 3. 最后重新部署

```shell
sudo docker container stop v2ray
sudo docker container rm v2ray
sudo docker run -d --name v2ray -v /etc/v2ray:/etc/v2ray -p 8888:80 v2ray/official v
```

## 官网安装的方法

**更新系统并安装常用软件**
`yum update -y && yum install wget curl unzip -y`
**安装v2ray服务端**
这里使用是官方的安装脚本，一键即可安装完成
`bash <(curl -L -s https://install.direct/go.sh)`
安装完成会显示连接端口和UUID，这是我们客户端连接要用到的信息，记录起来。

安装完成后v2ray并不会自动启动，使用如下命令启动它。
`service v2ray start`

