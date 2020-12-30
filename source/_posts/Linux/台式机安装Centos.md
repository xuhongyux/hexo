---
title: 台式机安装Centos
date: 2019-11-21 15:04:20
tags:	
categories: Linux
---

## 安装Centos

### 下载镜像

__[下载地址]( http://isoredirect.centos.org/centos/8/isos/x86_64/CentOS-8-x86_64-1905-dvd1.iso )__

推荐使用阿里云的下载地址速度回快点

这里用最笨的方法使用装机工具，老毛桃，等等等等，生成启动盘

### 安装过程

插上U盘，U盘启动，这里千万要注意，千万要注意，千万要注意，我们需要设置安装位置

 按下键盘键进入第二个Test this media & install Centos 7，将最下面的
`vmlinuz initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 rd.live.check quiet`改为`inst.stage2=hd:/dev/sdb4(你自己u盘的盘符) quiet` Ctrl + x 开始安装，如果你的硬盘只有一块，那么sda就是本机硬盘，sdb就是移动硬盘，大部分都是sdb4分区，如果不对的话重启先修改为`vmlinuz initrd=initrd.img linux dd quiet`，进入后可以看到磁盘是哪个再修改sdb4为正确的磁盘进入安装。 

之后和虚拟机的就一样了。

还要一个需要注意的点是，在网络配置的时候设置为自动匹配不然连接不上网络。

### 注意

+ 在安装的时候首先确定安装位置
+ 网络设置成自动匹配

