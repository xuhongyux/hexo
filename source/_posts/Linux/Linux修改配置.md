---
title: Linux修改配置
date: 2019-11-23 15:04:20
tags:	
categories: Linux
---

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
IPADDR=192.168.0.108  # ip地址
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

### Centos7开放及查看端口

1、开放端口

firewall-cmd --zone=public --add-port=5672/tcp --permanent # 开放5672端口

firewall-cmd --zone=public --remove-port=5672/tcp --permanent #关闭5672端口

firewall-cmd --reload # 配置立即生效

### 查看端口信息

安装插件

```shell
yum install net-tools
```

 1.查看被占用端口信息 

```shell
netstat -tunlp
```

 2.查看指定端口信息 

```shell
netstat -tunlp |grep 8080
```

 3.查看占用的进程信息 

```shell
ps 29142
```

 4.强行杀掉占用端口的进程(29142是进程PID) 

```shell
kill -9 29142
```

### linux关闭tcp6

1、打开/etc/sysctl.conf

2、添加如下三条设置

```
   net.ipv6.conf.all.disable_ipv6 = 1   net.ipv6.conf.default.disable_ipv6 = 1   net.ipv6.conf.lo.disable_ipv6 = 1
3、保存修改
4、执行：
  sudo sysctl -p
5、查看状态：

 cat /proc/sys/net/ipv6/conf/all/disable_ipv6
 显示应该是1
6、结束
```

