---
title: Shell
date: 2023-04-27 10:44:27
tags:
  - Linux
  - Shell
---

[toc]

# Shell概述

> Shell是一个命令解释器，它接收应用程序或用户的命令，然后调用操作系统内核。

**Linux提供的Shell解析器：**

```bash
cat /etc/shells
```

**bash和sh的关系：**

> sh是bash的一个软链接

```bash
ll /bin | grep bash
```

**查看默认的Shell解析器：**

```bash
echo $SHELL
```

# Shell入门

## 脚本格式

脚本以`#! /bin/bash`开头，`#!`用于指定脚本的解析器。

## Hello World

>  编写一个Shell脚本，输出**Hello World**。

**创建脚本文件**

```bash
cd /tmp
vi HelloWorld.sh
```

**HelloWorld.sh**

```shell
#! /bin/bash
echo Hello World
```

**执行脚本**

执行脚本有两种方法：

- 作为解析器参数执行

  ```bash
  sh ./HelloWorld.sh
  /bin/sh ./HelloWorld.sh
  bash ./HelloWorld.sh
  /bin/bash ./HelloWorld.sh
  ```

- 作为可执行脚本执行

  ```bash
  chmod +x ./HelloWorld.sh
  ./HelloWorld.sh
  ```

# 变量

## 定义变量

**语法：**`变量名=变量值`

**规则**

- 变量名只能包含**数字**、**字母**和**下划线**，只能以**字母**和**下划线**开头；
- `=`两边不能有空格；
- 变量值如果包含空格，需要用**单引号**或**双引号**括起来；
- 变量值默认都是字符串类型，无法直接进行数值运算；

**示例**

```shell
# vi /variableTest.sh
my_name=bob
```

## 使用变量

**方式**

- $var
- ${var}

**示例**

```shell
my_name=bob
echo $my_name
echo ${my_name}
```

## 只读变量

**语法：**`readonly var`

**示例**

```shell
my_name=bob
readonly var
# 执行报错：my_name: readonly variable
my_name=tom
```

## 删除变量

**语法：**`unset var`

**示例**

```shell
my_name=bob
echo $my_name
unset my_name
echo $my_name
```

## 特殊变量

### $n

**说明**

`$n`，n为数字，$0表示脚本名称，$1-$9表示第1到第9个参数，10以上的参数需要用大括号括起来，如${10}。

**示例**

脚本内容

```shell
echo '---------$n------------'
echo $0
echo $1
echo $2
```

执行脚本
```bash
./test.sh p1 p2
```

输出
```
---------$n------------
test.sh
p1
p2
```

### $#

**说明**

`$#`用于获取参数的个数。

**示例**

脚本内容

```shell
echo `---------$#---------`
echo $#
```

执行脚本

```bash
./test.sh p1 p2 p3
```

输出内容

```bash
---------$#---------
3
```

### $*、#@

**说明**

- `$*`获取命令行中所有参数，并将所有参数看做一个整体；
- `$@`获取命令行中所有参数，并区分对待每个参数；

**示例**

脚本内容

```shell
#!/bin/bash

echo "-- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done
```

执行脚本

```bash
$ chmod +x test.sh 
$ ./test.sh 1 2 3
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3
```

### $?

**说明**

`$?`用于获取最后一次命令的执行状态，返回0表示执行正确。

**示例**

```bash
./test.sh
echo $?
```

### $$

**说明**

`$$`用于获取运行该脚本的进程id；

**示例**

```shell
#!/bin/bash

echo "current pid: $$"
```



## Shell字符串

> 定义字符串时，可以使用**单引号**、**双引号**或者不使用引号。

**单引号**

- 单引号中的任何字符都会原样输出，所以在单引号内引用变量是无效的。

```shell
my_name='bob'
str='hello ${my_name}'
echo $str
```

输出结果为

```
hello ${my_name}
```

**双引号**

- 双引号里可以包含变量；
- 双引号可以使用转义字符；

```shell
my_name='bob'
str="hello \"${my_name}\""
echo -e $str
```

输出结果为

```
hello "bob"
```

**拼接字符串**

```shell
my_name='bob'
str="hello "${my_name}", i am tom."
echo $str
```

**获取字符串长度**

**语法：**`${#var}`

**示例**

```shell
my_name="bob"
echo ${#my_name} # 输出3
```

**截取字符串**

**语法：**`${var:startIndex:length}`，从`startIndex`开始，截取长度为`length`的字符串

**示例**

```shell
str="hello, my name is bob!"
echo ${str:2:3} # 输出 llo
```

## Shell数组

> 仅支持一维数组，下标从0开始。

**定义数组**

```shell
# 数组arr1，包含四个值
arr1=(v0 v1 v2 v3)
# 数组arr2，包含三个值
arr2[0]=v0
arr2[1]=v1
arr2[2]=v2
```

**读取数组**

**语法：**`${arr[index]}`，当`index`为`@`时，表示读取所有元素。

**示例**

```shell
echo ${arr1[1]}
echo ${arr2[@]}
```

**获取数组长度**

```shell
# 获取数组长度
length=${#arr1[@]}
# 获取数组长度
length1=${#arr1[*]}
# 获取数组中下标为1的元素的长度
length2=${#arr1[1]}
```

## Shell注释

**单行注释**

在行前加上`#`符号即可。

**多行注释**

**语法：**`:<<var`，`var`可随意指定。

**示例**

```shell
:<<EOF
注释内容
注释内容
注释内容
EOF

:<<CCCC
注释内容
注释内容
注释内容
CCCC
```

# 运算符

**语法**

- `$((运算式))`
- `$[运算式]`

**示例**

```bash
echo $((2 * 3))
6
```

**关系运算符**

| 运算符 | 说明                                     | 示例          |
| :----- | :--------------------------------------- | :------------ |
| -eq    | equals，两个数相等时为ture               | [ $a -eq $b ] |
| -ne    | not equals，两个数不相等是为true         | [ $a -ne $b ] |
| -gt    | greater than，左边大于右边时为true       | [ $a -gt $b ] |
| -lt    | less than，左边小于右边时为true          | [ $a -lt $b ] |
| -ge    | greater equals，左边大于等于右边时为true | [ $a -ge $b ] |
| -le    | less equals，左边小于等于右边时为true    | [ $a -le $b ] |

**布尔运算符**

| 运算符 | 说明   | 示例                       |
| :----- | :----- | :------------------------- |
| !      | 非运算 | [ !false ]                 |
| -o     | 或运算 | [ $a -gl 20 -o $b -gt 30 ] |
| -a     | 与运算 | [ $a -gl 20 -a $b -gt 30 ] |

**逻辑运算符**

| 运算符 | 说明 | 示例                           |
| :----- | :--- | :----------------------------- |
| &&     | AND  | [ $a -lt 100 && $b -gt 100 ]   |
| \|\|   | OR   | [ $a -lt 100 \|\| $b -gt 100 ] |

**字符串运算符**

| 运算符 | 说明                       | 示例         |
| :----- | :------------------------- | :----------- |
| =      | 两个字符串相等时返回true   | [ $a = $b ]  |
| !=     | 两个字符串不相等时返回true | [ $a != $b ] |
| -z     | 字符串长度为0时返回true    | [ -z $a ]    |
| -n     | 字符串长度不为0时返回true  | [ -n $a ]    |
| $      | 字符串不为空时返回true     | [ $a ]       |

**文件测试运算符**

| 运算符 | 说明                     | 示例         |
| :----- | :----------------------- | :----------- |
| -e     | 文件存在时返回true       | [ -e $file ] |
| -f     | 文件为普通文件时返回true | [ -f $file ] |
| -d     | 文件为目录时返回true     | [ -d $file ] |
| -r     | 文件可读时返回true       | [ -r $file ] |
| -w     | 文件可写时返回true       | [ -w $file ] |
| -x     | 文件可执行时返回true     | [ -x $file ] |

# 条件判断

**语法**

- `test condition`;
- `[ condition ]`，condition前后需要有空格；

**示例**

```shell
#! /bin/bash
a=10
b=20
if [ $a -gt $b ]
then
  echo 'a greater than b'
else
  echo 'a less than b'
fi
```

执行后输出

```bash
./test.sh 
a less than b
```

# 流程控制

> 流程控制语句写到一行时，需要加分号；如
>
> `if condition; then command; else command; fi;`

## if

**单分支**

```shell
if condition
then 
	command
fi
```
**多分支**

```shell
if condition
then 
	command
elif condition
	command
then 
	command
else
	command
fi
```

## for

**语法一**

```shell
for var in v1 v2 ... vn
do
	command
done
```

**语法二**

```shell
for (( 初始值; 条件; 变量变化 ))
do
	command
done
```

**示例**

```shell
#! /bin/bash
for a in v0 v1 v2 v3
do
  echo "hello, ${a}"
done

for (( i=0; i<4; i++  ))
do
  echo "i=${i}"
done
```

```bash
./test.sh
hello, v0
hello, v1
hello, v2
hello, v3
i=0
i=1
i=2
i=3
```

## while

**语法**

```shell
while condition
do
	command
done
```

**示例**

```shell
#!/bin/bash
sum=0
i=1
while [ $i -le 100 ]
do
	sum=$[$sum+$i]
	i=$[$i+1]
done
```

## case

**语法**

```shell
case 变量 in
值1)
	command
;;
值2)
	command
;;
*)
	command
esac
```

- `case`开头，`esac`结尾；
- case 行尾必须为单词`in`，每一个模式匹配必须以右括号`)`结束;
- 双分号`;;`表示命令序列结束，相当于 java 中的 break;
- 最后的`*）`表示默认模式，相当于 java 中的 default。

**示例**

```shell
#! /bin/bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
    1)  echo '你选择了 1'
    ;;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    *)  echo '你没有输入 1 到 4 之间的数字'
esac
```

# read读取控制台输入

**语法：**`read [options] var`

**选项**

- -p：指定读取值时的提示语；
- -t：指定读取值时的超时时间，单位为秒，不加-t表示一直等待；

**示例**

```shell
#! /bin/bash

read -p 'enter your age in 7 seconds:' -t 7 age
```

# 函数

## 系统函数

### basename

**语法：**`basename [path] [suffix]`

**说明：**获取给定path中的文件名，若指定了suffix，则会删除文件名中的suffix。

**示例**

```bash
basename /tmp/1.txt # 输出1.txt
basename /tmp/1.txt .txt # 输出1
```

### dirname

**语法：**`dirname [path]`

**说明：**获取给定path中的目录。

**示例**

```bash
dirname /tmp/1.txt # 输出/tmp
```

## 自定义函数

**语法**

```shell
[function] funname[()]
{
	command
	[return int]
}
```

**说明**

- Shell是由上而下顺序执行，所以在使用函数前必须先声明函数；
- `function`关键字可省略，直接写声明函数名；
- `return`后跟数值n（0-255），不声明return时，函数会将最后一条命令的返回值作为运行结果，可通过`$?`获取函数返回值；

**示例**

```shell
#! /bin/bash

sum()
{
	s=0
	s=$[ $1+$2 ]
	echo "$1+$2=$s"
	return $s
}

read -p "enter fister num: " n1
read -p "enter second num: " n2
sum $n1 $n2

echo "function exec result: $?"
```

# 正则表达式

> 正则表达式使用单个字符串来描述、匹配一系列符合某个语法规则的字符串。在很多文本编辑器里，正则表达式通常被用来检索、替换哪些符合某个模式的文本。在Linux中，grep、sed、awk等文本处理工具都支持通过正则表达式进行模式匹配。

## 常规匹配

一串不包含特殊字符的正则表达式只匹配它自身，如`cat /etc/passwd | grep root`就只会匹配包含`root`字符串的行。

## 常用特殊字符

| 字符 | 描述                                                    | 案例                                                         |
| ---- | ------------------------------------------------------- | ------------------------------------------------------------ |
| ^    | 匹配一行的开头                                          | `cat /etc/passwd | grep ^r`<br />匹配所有以`r`开头的行       |
| $    | 匹配一行的结尾                                          | `cat /etc/passwd |grep t$`<br />匹配所有以`t`结尾的行        |
| .    | 匹配任意一个字符                                        | `cat /etc/passwd |grep r..t`<br />匹配root、rabt、rxxt等行   |
| *    | 与上一个字符配合使用，<br />表示匹配上一个字符0次或多次 | `cat /etc/passwd | grep ro*t`<br />匹配rt、rot、root、rooot等行 |
| []   | 匹配某个范围内的一个字符                                | `[6, 8]`匹配6和8<br />`[3-8]`匹配3-8之间的任一数字<br />`[0-9]*`匹配任意长度的数字字符串<br />`[a-z]`匹配一个a-z之间的字符<br />`[a-z]*`匹配任意长度的小写字母字符串<br />`[a-g, x-z]`匹配a-g或x-z之间的任意一个字符 |
| \    | `\`表示转义，不会单独使用                               | `cat /etc/passwd | grep a\$`<br />表示匹配包含`a$`的所有行   |

# 文本处理工具

## cut

> cut命令从文件的每一行剪切字节、字符和字段，并将这些字节、字符和字段输出。

**命令：**`cut [options] file`

**选项**

- -d：指定分隔符，默认的分隔符是制表符`\t`；
- -f：指定分割后需要提取的列号；
- -c：按字符进行分割，后面加n表示提取第几列；

**说明**

分割后需要提取多列时，可用一下几种方法来表示多列：

- 多列用**逗号**隔开，如`cut -f1,2,4`表示提取第1、2、4列；
- `N-`，表示提取从第N列开始后的所有列（包括N列），如`cut -c2-`表示提取第2列后的所有列；
- `N-M`，表示提取第N列到第M列（包括N和M列），如`cut -f1-3`表示提取第1-3列；
- `-M`，表示提取第M列前的所有列（包括M列），如`cut -f-3`表示提取第3列前的所有列；

**示例**

```bash
# 截取/etc/passwd的第1列和第3列
cat /etc/passwd | cut -d: -f1,3
# 截取/etc/passwd的前3列
cat /etc/passwd | cut -d: -f-3
# 截取/etc/passwd的第3列到第5列
cat /etc/passwd | cut -d: -f3-5
# 截取/etc/passwd的第3列后的所有列
cat /etc/passwd | cut -d: -f3-
# 截取/etc/passwd的前4个字符
cat /etc/passwd | cut -c-4
```

## awk

> awk是一个强大的文本分析工具，它把文件逐行读入，再以分隔符将每行切开，切开的部分再进行分析处理。

**命令：**`awk [options] '/pattern1/{action1} /pattern2/{action2}...' file`

`pattern`表示匹配模式，`action`表示匹配到内容时执行的命令。

**选项**

- -F：指定分隔符，默认的分隔符是空格；
- -v：赋值一个用户定义的变量；

**说明**

- `BEGIN{action}`表示在所有数据行执行前执行的脚本，如`awk 'BEGIN{print "begin!!!"}' file`;
- `END{action}`表示在所有数据行执行后执行的脚本，如`awk 'END{print "end!!!"}' file`;

**内置变量**

- `FILENAME`：文件名；
- `NR`：已读的记录数（行号）；
- `NF`：浏览记录的域的个数（分割后列的个数）；

**示例**

```bash
# 输出/etc/passwd中以root开头的行，以：分割后的第七列
awk -F : '/^root/{print $7}' /etc/passwd
# 输出/etc/passwd中以root开头的行，以：分割后的第一列和第七列，两列之间用逗号分隔
awk -F : '/^root/{print $1","$7}' /etc/passwd
# 输出/etc/passwd中所有行的第1列和第7列，并在所有行前打印hello，在所有行前打印end
awk -F : 'BEGIN{print "hello"} {print $1","$7} END{print "end"}' /etc/passwd
# 将/etc/passwd中的root用户的id+1后输出
awk -F: -v i=1 '/^root/{print $3+i}' /etc/passwd

# 统计/etc/passwd文件名，每行行号，每行列数
awk -F : '{print "filename="FILENAME", rownum="NR", colnum="NF}' /etc/passwd
```

## sed

> sed可依靠脚本的指令来处理、编辑文件。

**语法：**`sec [-ni] [-e<script>] [-f<script ile>] file`

**参数说明**

- -n：仅显示script处理后的结果；
- -i：修改源文件；
- -e<script>：以指定的script来处理输入的文件；
- -f<script file>：以指定的script文件来处理输入的文件；

**动作说明**

- a：新增，a后面跟的字符串，会在新的一行出现（当前行的下一行）；
- c：取代，c后面跟的字符串，会取代n1、n2之间的行；
- d：删除；
- i：插入，i后面跟的字符串，会在新的一行出现（当前行的上一行）；
- p：打印，打印某个选项的数据，通常与-n参数一起使用；
- s：取代，可搭配正则表达式，如`1,20s/old/new/g`;

**经验**

- 当只有一个动作时，可以不用写`-e`选项；
- 新增、取代多行内容时，每行内容需要用反斜杠`\n`隔开；
- 脚本需要用单引号`''`包起来，第一列表示行号，第二列表示动作；
- 使用`-e`选项指定多个动作时，多个动作用分号`;`隔开；
- 表示多行时，多行用逗号`,`隔开，如`2,5`表示第2行到第5行，`2,$`表示第2行到最后一行；
- `^`表示第一行，`$`表示最后一行；
- 使用动作`p`时，通常与`-n`一起使用；
- 使用动作`s`时，查找与替换的方法与`vi`类似；

**示例**

```bash
# 创建测试文件
$ cat testfile.txt 
HELLO LINUX!  
Linux is a free unix-type opterating system.  
This is a linux testfile!  
Linux test 
Google
Taobao
Runoob
Tesetfile
Wiki
```

```bash
# 在testfile.txt文件的第4行后添加一行，内容为newLine
sed '4 a newLine' ./testfile.txt

# 删除第2到第5行
nl ./testfile.txt | sed '2,5 d'
# 删除第1行到第3行
nl ./testfile.txt | sed '^,3 d'
# 在第二行前加入两行，内容为drink tea... drink coffee 
nl ./testfile.txt | sed '2 i drink tea \n drink cokkee'
# 替换第二行到第五行为 no content
nl ./testfile.txt | sed '2,5 c no content'
# 显示第三行到第五行
nl ./testfile.txt | sed -n '3,5 p'

# 搜索包含'oo'的行并显示
nl ./testfile.txt | sed -n '/oo/p'
# 删除包含‘oo'的行
nl ./testfile.txt | sed -n '/oo/d'

# 将'oo'替换为'kk',g表示全局查找替换
sed 's/oo/kk/g' ./testfile.txt
# 将文件中的oo全部替换为kk
sed -i 's/oo/kk/g' ./testfile.txt

```

