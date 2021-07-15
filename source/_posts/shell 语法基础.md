---
title: shell 语法基础
tags: shell
date: 2021-7-15
---

## shell 语法基础

简单来说一下shell是什么：shell是基于C写的一个程序，因为确实是用C语言实现的东西。同时shell是命令语言，众所周知，在linux命令行下执行的命令一般都是shell命令。同时shell是一种编程语言，这更好理解，通过与其他编程语言对比就可以知道，shell也可以通过编写程序实现所需的功能。

> Shell 和 Shell脚本并不相同，Shell脚本是通过Shell实现的程序文件

接下来就不啰嗦，直接尽可能的精简去认识shell的语法。

* hello world

  * echo 打印内容

* 变量

  * 如何定义变量
  * 如何使用变量
  * 只读变量
  * 删除变量

* 字符串：单引号和双引号的区别

  * 单引号变量
  * 双引号变量
  * 获取字符串长度
  * 截取子字符串
  * 查询字符在字符串中的位置

  

### hello world

关键字`echo`：打印内容   

输出`hello world`

```shell
echo "hello world"
```

### 变量

shell中提到的三种变量：

* 局部变量：在脚本或命令中定义，生命周期只在其定义的实例中有效。
* 环境变量：系统的环境变量
* shell变量：就。。。扯淡，不就是变量的统称？！

与其他编程语言对变量合法性的要求相同，不能包含特殊字符，只能由下划线、数字、字母，首字母不能以数字开头。  

**定义变量：**

```shell
my_blog="shell language"
```

等号与左边的变量以及右边的值之间都不能有空格！

**使用变量**：

```shell
echo ${my_blog}	
```

输出的结果是`shell language`

> 使用花括号引着变量更规范，不必要时可以不加{}

只读变量：

```shell
my_name="soro"
readonly my_name
```

在定义变量之后，使用`readonly`标记变量为只读变量，该变量在之后如果尝试被修改则会报错，避免了重要的变量内容被覆盖！

删除变量：

```shell
unset my_name
```

> 注意：对变量的**非读写**操作unset和readonly后，只跟变量名，没有$符号。

### shell字符串

**单引号字符串**

* 字符串内容原样输出
* 单引号中可以包含多对单引号，不能只出现一个单引号

```shell
my_name='bai'shao'jie'
echo $my_name
```

输出结果是`baishaojie`

**双引号字符串**

* 在双引号中的变量以及特殊字符可以被转义输出
* 在单引号中如果要转义，需要用单引号拼接！

```shell
# 单引号中转义
name='shaojie'
my_name='bai'$name''

# 双引号中
name='shaojie'
my_name='bai${name}'
```

**获取字符串长度 & 提取子字符串**

```shell
# 获取字符串长度
string='abcd'
str_len=${#string}
echo $str_len

# 提取子字符串
string='this is an apple'
substr=${string:1:4}
echo $substr
substr_len=${#substr}
echo $substr_len

# $str_len: 4
# $substr: 'his '   
# $substr_len: 4
```

**查找字符位置**

```shell
# 查找 i/o 字符在$string中的位置，输出的位置下标值是最先被找到的字符的位置！
string='this is an apple'
echo `expr index "$string" io`
```

