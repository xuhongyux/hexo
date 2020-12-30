---
title: shell 编写
date: 2019-12-23 14:04:20
categories: 部署及调试 
---

## 一个进程死亡时重启它

```sh
while :
do
  echo "Current DIR is "
  pid=`ps aux | grep xmarking-report-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}'`
  echo $pid
  if [ -n "$pid" ] ;
   then
    echo "正常运行" 
   else
    nohup java -jar -Dspring.profiles.active=test  /App/xuhongyu/xmarking-report-0.0.1-SNAPSHOT.jar > /App/yunhenedu/marking-report_8091/logs/app.log &
echo "service started"
  fi
  sleep 10
done

```

给  .sh 文件赋予权限

```shell
chmod u+x *.sh
```

程序后台运行的推荐做法

首先创建nohup.out文件

```shell
$ nohup command &
上述命令执行后出现：
nohup: ignoring input and appending output to 'nohup.out'
可以重定向到一个文本文件
$ jobs
Ctrl + D 或关闭窗口，之后进程会在后台继续运行。
```

后台运行shell脚本

```sh
nohup ./youshell.sh &
```

查询线程号

```shell
ps aux | grep xmarking-report-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}'
```



## if 语句

1、基本语法:

```sh
if [ command ]; then
     符合该条件执行的语句
fi
```

2、扩展语法：

```sh
if [ command ];then
     符合该条件执行的语句
elif [ command ];then
     符合该条件执行的语句
else
     符合该条件执行的语句
fi
```

3、语法说明：
   bash shell会按顺序执行if语句，如果command执行后且它的返回状态是0，则会执行符合该条件执行的语句，否则后面的命令不执行，跳到下一条命令。
当有多个嵌套时，只有第一个返回0退出状态的命令会导致符合该条件执行的语句部分被执行,如果所有的语句的执行状态都不为0，则执行else中语句。
返回状态：最后一个命令的退出状态，或者当没有条件是真的话为0。

注意：

```
1、[ ]表示条件测试。注意这里的空格很重要。要注意在'['后面和']'前面都必须要有空格
2、在shell中，then和fi是分开的语句。如果要在同一行里面输入，则需要用分号将他们隔开。
3、注意if判断中对于变量的处理，需要加引号，以免一些不必要的错误。没有加双引号会在一些含空格等的字符串变量判断的时候产生错误。比如[ -n "$var" ]如果var为空会出错
4、判断是不支持浮点值的
5、如果只单独使用>或者<号，系统会认为是输出或者输入重定向，虽然结果显示正确，但是其实是错误的，因此要对这些符号进行转意
6、在默认中，运行if语句中的命令所产生的错误信息仍然出现在脚本的输出结果中
7、使用-z或者-n来检查长度的时候，没有定义的变量也为0
8、空变量和没有初始化的变量可能会对shell脚本测试产生灾难性的影响，因此在不确定变量的内容的时候，在测试号前使用-n或者-z测试一下
9、? 变量包含了之前执行命令的退出状态（最近完成的前台进程）（可以用于检测退出状态）
```

常用参数：
文件/目录判断：

常用的：
[ -a FILE ] 如果 FILE 存在则为真。
[ -d FILE ] 如果 FILE 存在且是一个目录则返回为真。
[ -e FILE ] 如果 指定的文件或目录存在时返回为真。
[ -f FILE ] 如果 FILE 存在且是一个普通文件则返回为真。
[ -r FILE ] 如果 FILE 存在且是可读的则返回为真。
[ -w FILE ] 如果 FILE 存在且是可写的则返回为真。（一个目录为了它的内容被访问必然是可执行的）
[ -x FILE ] 如果 FILE 存在且是可执行的则返回为真。

不常用的：
[ -b FILE ] 如果 FILE 存在且是一个块文件则返回为真。
[ -c FILE ] 如果 FILE 存在且是一个字符文件则返回为真。
[ -g FILE ] 如果 FILE 存在且设置了SGID则返回为真。
[ -h FILE ] 如果 FILE 存在且是一个符号符号链接文件则返回为真。（该选项在一些老系统上无效）
[ -k FILE ] 如果 FILE 存在且已经设置了冒险位则返回为真。
[ -p FILE ] 如果 FILE 存并且是命令管道时返回为真。
[ -s FILE ] 如果 FILE 存在且大小非0时为真则返回为真。
[ -u FILE ] 如果 FILE 存在且设置了SUID位时返回为真。
[ -O FILE ] 如果 FILE 存在且属有效用户ID则返回为真。
[ -G FILE ] 如果 FILE 存在且默认组为当前组则返回为真。（只检查系统默认组）
[ -L FILE ] 如果 FILE 存在且是一个符号连接则返回为真。
[ -N FILE ] 如果 FILE 存在 and has been mod如果ied since it was last read则返回为真。
[ -S FILE ] 如果 FILE 存在且是一个套接字则返回为真。
[ FILE1 -nt FILE2 ] 如果 FILE1 比 FILE2 新, 或者 FILE1 存在但是 FILE2 不存在则返回为真。
[ FILE1 -ot FILE2 ] 如果 FILE1 比 FILE2 老, 或者 FILE2 存在但是 FILE1 不存在则返回为真。
[ FILE1 -ef FILE2 ] 如果 FILE1 和 FILE2 指向相同的设备和节点号则返回为真。字符串判断

```
[ -z STRING ] 如果STRING的长度为零则返回为真，即空是真
[ -n STRING ] 如果STRING的长度非零则返回为真，即非空是真
[ STRING1 ]　 如果字符串不为空则返回为真,与-n类似
[ STRING1 == STRING2 ] 如果两个字符串相同则返回为真
[ STRING1 != STRING2 ] 如果字符串不相同则返回为真
[ STRING1 < STRING2 ] 如果 “STRING1”字典排序在“STRING2”前面则返回为真。
[ STRING1 > STRING2 ] 如果 “STRING1”字典排序在“STRING2”后面则返回为真。
```

数值判断

```
[ INT1 -eq INT2 ] INT1和INT2两数相等返回为真 ,=
[ INT1 -ne INT2 ] INT1和INT2两数不等返回为真 ,<>
[ INT1 -gt INT2 ] INT1大于INT2返回为真 ,>
[ INT1 -ge INT2 ] INT1大于等于INT2返回为真,>=
[ INT1 -lt INT2 ] INT1小于INT2返回为真 ,<
[ INT1 -le INT2 ] INT1小于等于INT2返回为真,<=
```

逻辑判断

```
[ ! EXPR ] 逻辑非，如果 EXPR 是false则返回为真。
[ EXPR1 -a EXPR2 ] 逻辑与，如果 EXPR1 and EXPR2 全真则返回为真。
[ EXPR1 -o EXPR2 ] 逻辑或，如果 EXPR1 或者 EXPR2 为真则返回为真。
[ ] || [ ] 用OR来合并两个条件
[ ] && [ ] 用AND来合并两个条件
```

其他判断

```
[ -t FD ] 如果文件描述符 FD （默认值为1）打开且指向一个终端则返回为真
[ -o optionname ] 如果shell选项optionname开启则返回为真
```

## 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                  | 举例                       |
| :----- | :---------------------------------------------------- | :------------------------- |
| -eq    | 检测两个数是否相等，相等返回 true。                   | [ $a -eq $b ] 返回 false。 |
| -ne    | 检测两个数是否不相等，不相等返回 true。               | [ $a -ne $b ] 返回 true。  |
| -gt    | 检测左边的数是否大于右边的，如果是，则返回 true。     | [ $a -gt $b ] 返回 false。 |
| -lt    | 检测左边的数是否小于右边的，如果是，则返回 true。     | [ $a -lt $b ] 返回 true。  |
| -ge    | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le    | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。  |

### Shell 基本运算符

```sh
#!/bin/bash

val=`expr 2 + 2`
echo "两数之和为 : $val"

两数之和为 : 4
```

两点注意：

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
- 完整的表达式要被 **` `** 包含，注意这个字符不是常用的单引号，在 Esc 键下边。

算术运算符

下表列出了常用的算术运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                          | 举例                          |
| :----- | :-------------------------------------------- | :---------------------------- |
| +      | 加法                                          | `expr $a + $b` 结果为 30。    |
| -      | 减法                                          | `expr $a - $b` 结果为 -10。   |
| *      | 乘法                                          | `expr $a \* $b` 结果为  200。 |
| /      | 除法                                          | `expr $b / $a` 结果为 2。     |
| %      | 取余                                          | `expr $b % $a` 结果为 0。     |
| =      | 赋值                                          | a=$b 将把变量 b 的值赋给 a。  |
| ==     | 相等。用于比较两个数字，相同则返回 true。     | [ $a == $b ] 返回 false。     |
| !=     | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。      |

**注意：**条件表达式要放在方括号之间，并且要有空格，例如: **[$a==$b]** 是错误的，必须写成 **[ $a == $b ]**。

### 布尔运算符

下表列出了常用的布尔运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明                                                | 举例                                     |
| :----- | :-------------------------------------------------- | :--------------------------------------- |
| !      | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。                  |
| -o     | 或运算，有一个表达式为 true 则返回 true。           | [ $a -lt 20 -o $b -gt 100 ] 返回 true。  |
| -a     | 与运算，两个表达式都为 true 才返回 true。           | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |

### 逻辑运算符

以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:

| 运算符 | 说明       | 举例                                       |
| :----- | :--------- | :----------------------------------------- |
| &&     | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false  |
| \|\|   | 逻辑的 OR  | [[ $a -lt 100 \|\| $b -gt 100 ]] 返回 true |

### 字符串运算符

下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明                                      | 举例                     |
| :----- | :---------------------------------------- | :----------------------- |
| =      | 检测两个字符串是否相等，相等返回 true。   | [ $a = $b ] 返回 false。 |
| !=     | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z     | 检测字符串长度是否为0，为0返回 true。     | [ -z $a ] 返回 false。   |
| -n     | 检测字符串长度是否为0，不为0返回 true。   | [ -n "$a" ] 返回 true。  |
| $      | 检测字符串是否为空，不为空返回 true。     | [ $a ] 返回 true。       |

### 文件测试运算符

文件测试运算符用于检测 Unix 文件的各种属性。

属性检测描述如下：

| 操作符  | 说明                                                         | 举例                      |
| :------ | :----------------------------------------------------------- | :------------------------ |
| -b file | 检测文件是否是块设备文件，如果是，则返回 true。              | [ -b $file ] 返回 false。 |
| -c file | 检测文件是否是字符设备文件，如果是，则返回 true。            | [ -c $file ] 返回 false。 |
| -d file | 检测文件是否是目录，如果是，则返回 true。                    | [ -d $file ] 返回 false。 |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。 | [ -f $file ] 返回 true。  |
| -g file | 检测文件是否设置了 SGID 位，如果是，则返回 true。            | [ -g $file ] 返回 false。 |
| -k file | 检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。  | [ -k $file ] 返回 false。 |
| -p file | 检测文件是否是有名管道，如果是，则返回 true。                | [ -p $file ] 返回 false。 |
| -u file | 检测文件是否设置了 SUID 位，如果是，则返回 true。            | [ -u $file ] 返回 false。 |
| -r file | 检测文件是否可读，如果是，则返回 true。                      | [ -r $file ] 返回 true。  |
| -w file | 检测文件是否可写，如果是，则返回 true。                      | [ -w $file ] 返回 true。  |
| -x file | 检测文件是否可执行，如果是，则返回 true。                    | [ -x $file ] 返回 true。  |
| -s file | 检测文件是否为空（文件大小是否大于0），不为空返回 true。     | [ -s $file ] 返回 true。  |
| -e file | 检测文件（包括目录）是否存在，如果是，则返回 true。          | [ -e $file ] 返回 true。  |

其他检查符：

- **-S**: 判断某文件是否 socket。
- **-L**: 检测文件是否存在并且是一个符号链接。

## Shell echo命令

Shell 的 echo 指令与 PHP 的 echo 指令类似，都是用于字符串的输出。命令格式：

```
echo string
```

您可以使用echo实现更复杂的输出格式控制。

### 显示普通字符串:

```
echo "It is a test"
```

这里的双引号完全可以省略，以下命令与上面实例效果一致：

```
echo It is a test
```

### 显示转义字符

```
echo "\"It is a test\""
```

结果将是:

```
"It is a test"
```

同样，双引号也可以省略

### 显示变量

read 命令从标准输入中读取一行,并把输入行的每个字段的值指定给 shell 变量

```
#!/bin/sh
read name 
echo "$name It is a test"
```

以上代码保存为 test.sh，name 接收标准输入的变量，结果将是:

```
[root@www ~]# sh test.sh
OK                     #标准输入
OK It is a test        #输出
```

### 显示换行

```
echo -e "OK! \n" # -e 开启转义
echo "It is a test"
```

输出结果：

```
OK!

It is a test
```

### 显示不换行

```
#!/bin/sh
echo -e "OK! \c" # -e 开启转义 \c 不换行
echo "It is a test"
```

输出结果：

```
OK! It is a test
```

### 显示结果定向至文件

```
echo "It is a test" > myfile
```

### 原样输出字符串，不进行转义或取变量(用单引号)

```
echo '$name\"'
```

输出结果：

```
$name\"
```

### 显示命令执行结果

```
echo `date`
```

**注意：** 这里使用的是反引号 **`**, 而不是单引号 **'**。

结果将显示当前日期

```
Thu Jul 24 10:08:46 CST 2014
```

## 注意

### 后台运行shell文件

```sh
nohup ./youshell.sh &
```

### 查询线程号

```shell
ps aux | grep xmarking-report-0.0.1-SNAPSHOT.jar | grep -v grep | awk '{print $2}'
```

### linux系统应用启动、重启、停止shell脚本

启动命令：

```shell
sh *.sh start
```

重启命令：

```shell
sh *.sh restart
```

停止命令：

```shell
sh *.sh stop
```

### 在Shell脚本中调用另一个脚本

**先来说一下主要以下有几种方式：**

- **fork:** 如果脚本有执行权限的话，path/to/foo.sh。如果没有，sh path/to/foo.sh。
- **exec:** exec path/to/foo.sh
- **source:** source path/to/foo.sh

fork例子

```sh
 fork: /App/xuhongyu/start.sh
```



## 安装定时器插件

```shell
yum -y install vixie-cron
yum -y install crontabs
```

操作

```shell
service crond start     //启动服务
service crond stop      //关闭服务
service crond restart   //重启服务
service crond reload    //重新载入配置
service crond status    //查看crontab服务状态
```

