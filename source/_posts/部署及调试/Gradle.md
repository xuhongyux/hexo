---
title: Gradle
date: 2020-11-25 15:04:20

categories: 部署及调试
---

## Gradle 介绍

Gradle是一个基于Apache Ant和Apache Maven概念的项目自动化构建开源工具。它使用一种基于Groovy的特定领域语言(DSL)来声明项目设置，目前也增加了基于Kotlin语言的kotlin-based DSL，抛弃了基于XML的各种繁琐配置。

面向Java应用为主。当前其支持的语言限于Java、[Groovy](https://baike.baidu.com/item/Groovy/180590)、[Kotlin](https://baike.baidu.com/item/Kotlin/1133714)和[Scala](https://baike.baidu.com/item/Scala/2462287)，计划未来将支持更多的语言。

## 安装

[下载地址](https://services.gradle.org/distributions/)

配置环境变量

测试

```shell
gradle -v
```

仓库地址

环境变量配置

```shell
GRADLE_USER_HOME
```

