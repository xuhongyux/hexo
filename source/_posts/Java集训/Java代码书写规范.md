---
title: Java书写规范
date:  2019-11-10 8:53:40
categories: Java 集训
---

## 简单汇总详解

**一般原则**

1. 尽量使用完整的英文描述符
2. 采用适用于相关领域的术语
3. 采用大小写混合增强可读性
4. 尽量少用缩写，但如果用了，要明智地使用，且在整个工程中统一
5. 避免使用长的名字
6. 避免使用类似的名字，或者仅仅是大小写不同的名字
7. 避免使用下划线（除静态常量等）

**命名的字母大小写问题**

1. 包名： 字母全小写 例如: cn.coderstory.Activity.Main
2. 类，接口 ：首字母大写，其他全小写 例如: class Container
3. 方法，变量 ：第二个单词开始首字母大写 例如: seedMessage
4. 常量： 大写，单词用“_”分割 例如: final static MIN_WIDTH = 4
5. 接口 ：首字母大写 ，后缀Impl 例如: class ContainerImpl
6. 异常类： 首字母大写， 后缀Exception 例如: DataNotFoundException
7. 抽象类 ：首字母大写， 前缀Abstract 例如: AbstractBeanDefinition
8. Test类： 首字母大写， 后缀Test 例如: public Location newLocation()

**方法的命名**

1. 类中获取值方法，一般要求被方法名使用被访问字段名，前面加上前缀get，如getLastUser(), getUserCount()
2. 返回布尔型的判断方法一般要求方法名使用单词 is 做前缀，如isPersistent(),isString()。或者使用具有逻辑意义的单词，例如equal 或equals
3. 用于修改某些设置的方法（一般返回类型为void）：被访问字段名的前面加上前缀 set，如setFirstName(),setLastName()，setWarpSpeed()。
4. 已办的方法一般采用完整的英文描述说明成员方法功能，第一个单词尽可能采用一个生动的动词，第一个字母小写，如 openFile(), addAccount()。
5. 接口 ：首字母大写 ，后缀Impl 例如: class ContainerImpl
6. 异常类： 首字母大写， 后缀Exception 例如: DataNotFoundException
7. 抽象类 ：首字母大写， 前缀Abstract 例如: AbstractBeanDefinition
8. Test类： 首字母大写， 后缀Test 例如: public Location newLocation()

**Java注释约定**

- 类的整体注释：遵循JavaDoc的规范，在每一个源文件的开头注明该CLASS的作用, 作简要说明, 并写上源文件的作者, 编写日期。如果是修改别人编写的源文件，要在修改信息上注明修改者和修改日期。

例如：

```java
`/**``* @（#）:CLASSNAME.java``* @description: Description of this java``* @author: PROGRAMMER'S NAME YYYY/MM/DD``* @version: Version No.``* @modify:``* @Copyright: 版权由拥有``*/`
```

- 类中方法的注释：遵循JavaDoc的规范，在每个方法的前部用块注释的方法描述此方法的作用，以及传入，传出参数的类型和作用，以及需要捕获的错误。

例如：

```java
`/**``* 方法的描述``*``*``*@param 参数的描述``*@return 返回类型的描述``*@exception 出错信息的描述``*/`
```

- 行注释：使用//…的注释方法来注释需要表明的内容。并且把注释的内容放在需要注释的代码的前面一行或同一行。
- 块注释：使用/**和*/注释的方法来注释需要表明的内容。并且把注释的内容放在需要注释的代码的前面。
- 注释哪些部分：类的目的（即类所完成的功能）、设置接口的目的以及应如何被使用、成员方法注释（对于设置与获取成员方法，在成员变量已有说明的情况下，可以不加注释；普通成员方法要求说明完成什么功能，参数含义是什么？返回什么？）、普通成员方法内部注释（控制结构、代码做了些什么以及为什么这样做，处理顺序等）、实参和形参的含义以及其他任何约束或前提条件、字段或属性描述。而对于局部变量，如无特别意义的情况下不加注释。

**JAVA文件声明顺序**

类或接口应该按以下顺序声明（其实是加载顺序的问题）：

1. 包的定义
2. impot类（输入包的顺序、避免使用*）输入包应该按照java.*.*，javax.*.*，org.*.* ，com.*.*的顺序import在import的时候不应该使用* (例如: java.util.*)
3. 类或接口的定义
4. 静态变量定义，按public，protected，private顺序
5. 实例变量定义，按public，protected，private顺序
6. 构造方法
7. 方法定义顺序按照public方法(类自己的方法)，实现接口的方法，重载的public法，受保护方法，包作用域方法和私有方法。建议：类中每个方法的代码行数不要超过100行。
8. 内部类的定义

## P3C插件（阿里规范）

idea插件

 【p3c-idea-plugin】：【https://plugins.jetbrains.com/plugin/10046-alibaba-java-coding-guidelines】 

下载最新的版本启用即可

在用的时候绿色的方框代表代码规范检查

蓝色的圈圈代表是否开启的实时监控



## Java关键字表

| 关键字名称   | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| abstract     | 表明类或者成员方法具有抽象属性                               |
| assert       | 用来进行程序调试                                             |
| boolean      | 基本数据类型之一，布尔类型                                   |
| break        | 提前跳出一个块                                               |
| byte         | 基本数据类型之一，字节类型                                   |
| case         | 用在switch语句之中，表示其中的一个分支                       |
| catch        | 用在异常处理中，用来捕捉异常                                 |
| char         | 基本数据类型之一，字符类型                                   |
| class        | 类                                                           |
| const        | 保留关键字，没有具体含义                                     |
| continue     | 回到一个块的开始处                                           |
| default      | 默认，例如，用在switch语句中，表明一个默认的分支             |
| do           | 用在do-while循环结构中                                       |
| double       | 基本数据类型之一，双精度浮点数类型                           |
| else         | 用在条件语句中，表明当条件不成立时的分支                     |
| enum         | 枚举                                                         |
| extends      | 表明一个类型是另一个类型的子类型，这里常见的类型有类和接口   |
| final        | 用来说明最终属性，表明一个类不能派生出子类，或者成员方法不能被覆盖，或者成员域的值不能被改变 |
| finally      | 用于处理异常情况，用来声明一个基本肯定会被执行到的语句块     |
| float        | 基本数据类型之一，单精度浮点数类型                           |
| for          | 一种循环结构的引导词                                         |
| goto         | 保留关键字，没有具体含义                                     |
| if           | 条件语句的引导词                                             |
| implements   | 表明一个类实现了给定的接口                                   |
| import       | 表明要访问指定的类或包                                       |
| instanceof   | 用来测试一个对象是否是指定类型的实例对象                     |
| int          | 基本数据类型之一，整数类型                                   |
| interface    | 接口                                                         |
| long         | 基本数据类型之一，长整数类型                                 |
| native       | 用来声明一个方法是由与计算机相关的语言（如C/C++/FORTRAN语言）实现的 |
| new          | 用来创建新实例对象                                           |
| package      | 包                                                           |
| private      | 一种访问控制方式：私用模式                                   |
| protected    | 一种访问控制方式：保护模式                                   |
| public       | 一种访问控制方式：共用模式                                   |
| return       | 从成员方法中返回数据                                         |
| short        | 基本数据类型之一,短整数类型                                  |
| static       | 表明具有静态属性                                             |
| strictfp     | 用来声明FP_strict（单精度或双精度浮点数）表达式遵循IEEE 754算术规范 |
| super        | 表明当前对象的父类型的引用或者父类型的构造方法               |
| switch       | 分支语句结构的引导词                                         |
| synchronized | 表明一段代码需要同步执行                                     |
| this         | 指向当前实例对象的引用                                       |
| throw        | 抛出一个异常                                                 |
| throws       | 声明在当前定义的成员方法中所有需要抛出的异常                 |
| transient    | 声明不用序列化的成员域                                       |
| try          | 尝试一个可能抛出异常的程序块                                 |
| void         | 声明当前成员方法没有返回值                                   |
| volatile     | 表明两个或者多个变量必须同步地发生变化                       |
| while        | 用在循环结构中                                               |