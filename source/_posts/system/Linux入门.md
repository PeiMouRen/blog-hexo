---
title: Linux入门
date: 2022-08-16 15:15:21
categories: 
 - 系统
tags:
 - system
 - Linux
---



[toc]

# 概述

linux即常用于服务端的一个类unix操作系统，国内常用发行版有centos、ubuntu、rocky（centos的替代品）等

# 安装

> 以安装centos为例

## iso镜像安装

1. 下载centos镜像和vmware虚拟机

   ```http
   # 阿里云centos7.9镜像
   http://mirrors.aliyun.com/centos/7.9.2009/isos/x86_64/
   # vmware虚拟机免费版下载
   https://www.vmware.com/cn/products/workstation-player.html
   ```

2. 通过镜像创建虚拟机

   百度搜索

## docker镜像安装

``` bash
docker pull centos
docker run centos
```

# Linux文件及目录结构

## Linux文件

Linux中一切皆文件

## 目录结构

> 树结构

- /：根目录
- /bin：binary，存放经常使用的命令，软链接到`/usr/bin`
- /sbin：super user binary，存放系统管理员使用的系统管理程序，软链接到`/usr/sbin`
- /lib：存放系统开机所需要的最基本的动态连接共享库，类似于windows的DLL文件，软链接到`/usr/lib`
- /lib64：64位的库文件，软链接到`/usr/lib64`
- /boot：存放linux启动时使用的一些核心文件
- /dev：device，设备文件保存目录
- /etc：配置文件保存目录
- /usr：Unix Software Resource，存放系统软件资源
- /home：普通用户的主目录
- /root：超级用户的主目录
- /opt：optional，可选目录，一般软件装到这里
- /media：挂载媒体设备，如u盘、光盘等
- /mnt：mount，挂载目录，如u盘，移动硬盘等
- /proc：process，虚拟目录，是系统内存的映射，通过访问该目录可获取系统信息
- /run：虚拟文件系统，记录系统运行信息
- /srv：service，存放系统服务启动后需要提取的数据
- /sys：system，存放系统硬件相关信息
- /tmp：临时目录
- /var：variable，保存动态数据，主要保存日志、缓存等

# VI/VIM编辑器

> vi是unix和类unix系统中的文本编辑器；
>
> vim是从vi发展出来的一个更强大的文本编辑器，与vi完全兼容；
>
> vi编辑器有三种模式：一般模式、编辑模式、指令模式

## 一般模式

以vi打开一个文档就进入一般模式了，在该模式下可进行删除、复制、粘贴等动作，常用命令如下：

|       命令       |              说明              |
| :--------------: | :----------------------------: |
|        yy        |         复制光标所在行         |
| y数字y（数字yy） |     复制光标（包含）后几行     |
|      y$/y^       |   复制从光标开始后/前的内容    |
|        p         |        在光标所在行粘贴        |
|      数字p       |     在光标所在行粘贴多少次     |
|        u         |        撤销上一步的操作        |
|        dd        |         删除光标所在行         |
| d数字d（数字dd） |    删除光标（包含）后多少行    |
|        x         |   剪切一个字母，相当于delete   |
|        X         | 剪切一个字母，相当于backspace  |
|        yw        | 从光标所在位置，向后复制一个词 |
|        dw        | 从光标所在位置，向后删除一个词 |
|   shift+6（^)    |         移动光标到行头         |
|   shift+4（$）   |         移动光标到行尾         |
|      gg/1+G      |         移动光标到开头         |
|        G         |         移动光标到结尾         |
|        H         |       移动光标到当前页头       |
|        L         |       移动光标到当前页尾       |
|      数字+G      |        移动光标到目标行        |
|       r/R        |       替换一个/多个字符        |
|        w         |   移动光标到下一个单词的开头   |
|        e         |   移动光标到下一个单词的结尾   |
|        b         |   移动光标到上一个单词的开头   |



## 编辑模式

在一般模式下，按`i/I/o/O/a/A`中的任一字母后进入编辑模式，在编辑模式下按`ESC`重新进入一般模式。

## 指令模式

在一般模式中，输入`:`、`/`或`?`中的任一字符，即可将光标移动到最下方，进入指令模式，常用语法如下：

|     命令      |                             说明                             |
| :-----------: | :----------------------------------------------------------: |
|      :w       |                             保存                             |
|      :q       |                             退出                             |
|      :!       |                           强制执行                           |
|  /要查找的词  |                   n查找下一个，N查找上一个                   |
|     :noh      |                         取消高亮显示                         |
|    :set nu    |                           显示行号                           |
|   :set nonu   |                          不显示行号                          |
| :%s/old/new/g | 替换内容，%s表示所有行，s表示当前行，不加/g表示只替换每一行的第一个 |

## 模式间互相转换

![vi模式转换](vi模式转换.png)

# 网络配置

## 网络路由设置

```bash
netstat -nr # 查看网络路由器信息
route add -net 172.17.3.0/24 gw 192.168.5.1 # 添加路由
```

## 设置静态ip地址

```bash
vim /etc/sysconfig/network-scripts/ifcfg-ens33
```

```
# 下面的配置项，存在的修改，不存在的添加

TYPE="Ethernet" #网络类型（通常是 Ethemet）
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static" #IP 的配置方法[none|static|bootp|dhcp]（不使用协议|静态分配 IP|BOOTP 协议|DHCP 协议）
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="e83804c1-3257-4584-81bb-660665ac22f6" #随机id
DEVICE="ens33" #接口名（设备,网卡）
ONBOOT="yes" #系统启动的时候网络接口是否有效（yes/no）#IP 地址
IPADDR=192.168.1.100 # 静态ip地址，bootproto需设置为static
NETMASK=255.255.255.0 # 子网掩码
GATEWAY=192.168.1.2 #网关
DNS1=192.168.1.2 #域名解析器
```

```bash
service network restart # 重启网络
```

## 设置主机名

```bash
# 查看主机名
hostname
hostnamectl
# 修改主机名
# eg：node1
vim /etc/hostname # 修改完需要重启
hostnamectl set-hostname node1 # 修改完不需要重启
# ip和主机名映射，配置后可直接通过主机名访问其他主机
# eg：192.168.5.112 node2
#	  192.168.5.113 node3
vim /etc/hosts 

reboot # 重启
```

## 远程登录

```bash
ssh -p 22 root@192.168.5.110
```

## 查看网络配置

```bash
ifconfig
```

## 检查网络连接

```bash
ping
```

# 系统管理

## 修改系统编码

```bash
## 查看环境
env
## 修改编码 LANG="zh_CN.UTF-8"
vi /etc/locale.conf
source /etc/locale.conf
```

## 服务管理

**语法**

```bash
systemctl start | stop | status | restart 服务名
```

**查看系统服务**

```bash
ll /usr/lib/systemd/system
```

**示例**

```bash
# 查看防火墙服务状态
systemctl status firewalld
```

---

**设置开机自启**

```bash
# 查看服务开机自启状态
systemctl list-unit-files
# 开启/关闭服务自启
systemctl disable | enable serviceName
```

**示例**

```bash
# 开启防火墙开启自启
systemctl enable firewalld.service
```

## 系统运行级别

centos7以后2两种运行级别：

- **multi-user.target**:等价于原运行级别3（多用户有网，无图形界面）；
- **graphical.target**:等价于原运行级别5（多用户有网，有图形界面）；

---

**获取当前运行级别**

```bash
systemctl get-default
```

**修改当前运行级别**

```bash
# TARGET.targe可为multi-user.target或graphical.target
systemctl set-default TARGET.targe
```

## 防火墙

**查看防火墙状态**

```bash
systemctl status firewalld
```

**临时关闭防火墙**

```bash
systemctl stop firewalld
```

**永久关闭防火墙（开机不自启）**

```bash
systemctl disable firewalld.service
```

## 关机重启

**命令**

- **sync**:将缓存同步到磁盘；
- **halt**:停机不断电；
- **poweroff**:关机并断电；
- **reboot**：重启，相当于`shutdown -r now`；
- **shutdown [OPTIONS...] [TIME] [WALL...]**：关机；
  - 参数：H、h、P、r、k、c；
  - 时间：单位是分，还可以用**now**、**mm:ss**表示；

**示例**

```bash
# 立即重启，同reboot
shutdown -r now;
# 两分钟后关机，并打印提示信息；
shutdown -h 2 'the server will shutdown after two minutes';
# 定时18:05关机
shutdown -h 18:05;
# 只打印警告信息，不关机
shutdown -k 'the server will shutdown after two minutes';
# 取消关机操作
shutdown -c
```

# 常用命令

## 文件目录相关

### 显示当前目录

**命令：**`pwd`

### 列出目录下的内容

**命令：**`ls`

**参数：**

| 选项 |                             说明                             |
| :--: | :----------------------------------------------------------: |
|  -a  |            列出全部文件，包括以点开头的隐藏文件。            |
|  -l  | 每行列出一个文件信息。同ll。<br />列出的信息分别为：文件类型与权限、链接数、文件属主、文件<br />属组、文件大小（byte）、最近修改的日期、文件名 |

### 切换目录

**命令：**`cd`

**选项：**

|    选项     |            说明            |
| :---------: | :------------------------: |
| cd ~或者 cd |      切换到用户家目录      |
|    cd -     |     切换到上次所在目录     |
|    cd ..    | 切换到当前目录的上一级目录 |

### 创建目录

**命令：**`mkdir dirName`

**选项：**

| 选项 |     说明     |
| :--: | :----------: |
|  -p  | 创建多级目录 |

### 删除目录

**命令：**`rmdir dirName`

**选项：**

| 选项 |     说明     |
| :--: | :----------: |
|  -p  | 删除多级目录 |

### 创建文件

**命令：**`touch fileName`

**案例：**

```bash
touch 1.txt
```

### 复制文件或目录

**命令：**`cp  [option] source dest`

**选项：**

| 选项 |        说明        |
| :--: | :----------------: |
|  -r  | 递归复制整个文件夹 |

### 删除文件或目录

**命令：**`rm  [option] file`

**选项：**

| 选项 |             说明             |
| :--: | :--------------------------: |
|  -r  |      递归删除整个文件夹      |
|  -f  | 强制执行删除操作，不提示确认 |

### 移动文件或重命名

**命令：**`mv oldFile newFile`

### 查看文件全部内容

**命令：**`cat  [option] file`

**选项：**

| 选项 |   说明   |
| :--: | :------: |
|  -n  | 显示行号 |

### more分屏查看文件内容

**命令：**`more file`

**说明：**more指令是一个基于VI编辑器的文本过滤器，它以全屏的方式按页显示文件内容。more中内置了若干快捷键。

**操作：**

|     操作      |         说明         |
| :-----------: | :------------------: |
| Space（空格） |      向下翻一页      |
| Enter（回车） |      向下翻一行      |
|       q       |       退出more       |
|   Ctrl + F    |     向下滚动一屏     |
|   Ctrl + B    |      返回上一屏      |
|       =       |     输出当前行号     |
|      :f       | 输出文件名和当前行号 |

### less分屏查看文件内容

**命令：**`less file`

**说明：**less比more更加强大，支持各种终端，less在显示文件内容时，并不是一次将整个文件加载后才显示，而是根据显示需要加载内容，对于显示大文件具有较高的效率。

**操作：**

|     操作      |                        说明                        |
| :-----------: | :------------------------------------------------: |
| Space（空格） |                     向下翻一页                     |
| Enter（回车） |                     向下翻一行                     |
|       q       |                      退出less                      |
|     /字串     | 向下搜寻『字串』的功能；n：向下查找；N：向上查找； |
|     ?字串     | 向上搜寻『字串』的功能；n：向上查找；N：向下查找； |

### 输出内容到控制台

**命令：**`echo [option] content`

**选项：**

| 选项 |     说明     |
| :--: | :----------: |
|  -e  | 支持转义字符 |

**案例：**

```bash
echo 'hello\tworld'
echo -e 'hello\tworld'
echo -e 'hello\nworld'
```

### 显示文件头部内容

**命令：**`head [option] file`

**选项：**

| 选项 |      说明      |
| :--: | :------------: |
|  -n  | 指定显示的行数 |

### 显示文件尾部内容

**命令：**`tail [option] file`

**选项：**

| 选项 |                 说明                 |
| :--: | :----------------------------------: |
|  -n  |            指定显示的行数            |
|  -f  | 显示文件最新追加的内容，监视文件变化 |

### >输出重定向和>>追加

**案例：**

```bash
# 将当前目录下的文件列表覆盖到1.txt
ls -al > 1.txt
# 将hehe追加到1.txt末尾
echo 'hehe' >> 1.txt
```

### 软链接

**命令：**`ln -s file linkName`

**案例：**

```bash
# 创建名为test的软链接，指向./test/1.txt
ln -s ./test/1.txt test
# 删除test软链接
rm -rf test
```

### 查看历史命令

**命令：**`history`

## 文件权限相关

### 文件属性

**说明：**`ll`命令查看文件列表后，每一列的含义为文件类型与权限、链接数、文件属主、文件属组、文件大小（byte）、最近修改的时间、文件名。

---

**文件类型：**

- d：目录
- -：文件
- l：链接文档

---

**权限：**

- r：可读
- w：可写
- x：可执行
- -：无权限

---

**rwx作用到文件上的权限：**

- r：可以读取，查看
- w：可以修改，但是不代表可以删除该文件，删除一个文件的前提条件是对该文件所在的目录有写权限，才能删除该文件
- x：可以被系统执行

**rwx作用到目录上的权限：**

- r：可以读取，ls查看目录内容
- w：可以修改，目录内创建+删除+重命名目录
- x：可以进入该目录

---

如`drwxr-x---`分表表示文件类型为**目录**、属主权限为**rwx**、属组权限为**r-x**、其他用户权限为**---**。

---

`ll`查看的文件列表中的链接数，文件代表硬链接的个数，目录代表子文件的个数。

### 修改文件权限

**命令：**

- `chmod [{ugoa}{+-=}{rwx}] fileName`
- `chmod [777] fileName`

**说明：**

- u：user，表示用户权限
- g：group，表示用户组权限
- o：other，表示其他用户权限
- a：all，表示所有用户权限
- +：添加权限
- -：减小权限
- =：设置权限
- 777：即rwxrwxrwx=(4 + 2 + 1)(4 + 2 + 1)(4 + 2 + 1)=777

**案例：**

```bash
# 设置1.txt的权限为用户可读、可写、可执行，所在组的用户可读、可执行，其他用户只读
chmod 754 1.txt
# 给其他用户添加1.txt的执行权限
chmod o+x 1.txt
```

### 修改文件所有者

**命令：**`chown [options] [owner][:[group]] fileName`

**选项：**

| 选项 |      说明      |
| :--: | :------------: |
|  -r  | 递归修改子目录 |

**案例：**

```bash
# 修改hehe目录及其所有子目录的所属用户为test，所属组为test
chown -r test:test hehe
```

### 修改文件所属组

**命令：**`chogrp [options] groupName fileName`

**选项：**

| 选项 |      说明      |
| :--: | :------------: |
|  -r  | 递归修改子目录 |

## 文件查找相关

### find查找文件或目录

**命令：**`find [path] [options]`

**说明：**参数中的+n表示大于n，-n表示小于n，n表示精确为n。

**选项：**

|  选项  |                             说明                             |
| :----: | :----------------------------------------------------------: |
| -name  |                        按照文件名查找                        |
| -user  |                     按照文件所属用户查找                     |
| -size  | 按照文件大小查找，单位：<br />b-块（512字节）<br />c-字节<br />w-字（2字节）<br />k-千字节<br />M-兆字节<br />G-吉字节 |
| -type  |          按文件类型查找<br />f-文件 d-目录 l-软链接          |
| -perm  |                       按照文件权限查找                       |
| -mmin  |                  查找在n分钟前修改过的文件                   |
| -mtime |                   查找在n天前修改过的文件                    |

**案例：**

```bash
# 按照名称在当前目录下查找txt文件
find -name *.txt
# 按照所属用户在/usr目录下查找
find /usr -user pxz
# 按照文件大小，在/home目录下查找大于200M的文件
find /home -size +200M
# 查找当前目录下的目录
find -type d
# 查找当前目录下权限为777的文件
find -perm 777
# 假设今天是2023-04-24，查找在2023-04-22这一天修改过的文件
find -mtime 2
# 假设今天是2023-04-24，查找在2023-04-22这一天前修改过的文件
find -mtime +2
# 假设今天是2023-04-24，查找在2023-04-22这一天至今天修改过的文件
find -mtime -2
```

### locate定位文件

> locate指令利用事先建立的系统中所有文件名称及路径的locate数据库，来快速定位给定的文件。为保证查询结果的正确定，管理员需要定期更新locate数据库。

**命令：**

```bash
# 安装locate
yum -y install mlocate
# 更新locate
updatedb
# 查找文件
locate aa.txt
```

### grep过滤查找及“|”管道符

> 管道符号`|`，表示将前一个命令的处理结果输出给后面的命令处理。

**命令：**`grep [options] PARTTERN [FILE]`

**选项：**

| 选项 | 说明                |
| :--: | ------------------- |
|  -B  | 显示匹配行的前n行   |
|  -A  | 显示匹配行的后n行   |
|  -n  | 显示行号            |
|  -m  | 匹配到n行后停止搜索 |

**案例：**

```bash
# 显示v0.log文件中包含‘sub'行的前后10行，10次匹配后停止搜索
grep -A 10 -B 10 -m 10 -n 'sub' v0.log 
# 查找a.log中包含'test'的行
cat a.log | grep 'test'
```

## 打包压缩相关

### gzip/gunzip

> 压缩、解压.gz文件

**命令：**`gzip file`	

​			`gunzip file.gz`

**说明：**

- 执行压缩文件，不能压缩目录。
- 压缩后不会保留源文件。
- gzip -r directory时会在目录下产生多个压缩文件。

**案例：**

```bash
# 压缩1.txt
gzip 1.txt
# 解压1.txt.gz
gunzip 1.txt.gz
```

### zip/unzip

**命令：**`zip [options] xxx.zip file`	

​			`unzip [options] file.zip`

**选项：**

- -r:递归操作
- zip -d:相当于unzip
- unzip -d:指定解压目录

**案例：**

```bash
# 将test目录下的文件压缩为1.zip
zip -r 1.zip ./test
# 解压1.zip到aa目录
unzip 1.zip -d ./aa
```

### tar打包

**命令：**`tar [options]`

**选项：**

| 选项 | 说明                                               |
| :--: | -------------------------------------------------- |
|  -z  | 打包或取消打包的同时使用gzip压缩，或使用gunzip解压 |
|  -c  | 创建.tar归档文件                                   |
|  -x  | 从.tar归档文件中提取文件                           |
|  -v  | 显示详细信息                                       |
|  -f  | 指定打包压缩后的文件名                             |
|  -C  | 切换（解压）到指定目录                             |

**案例：**

```bash
# 将1.txt 2.txt打包压缩至1.tar.gz文件
tar -zcvf 1.tar.gz 1.txt 2.txt
# 将1.tar.gz文件解压到当前目录
tar -zxvf 1.tar.gz
# 将1.tar.gz文件解压到/tmp目录
tar -zxvf 1.tar.gz -C /tmp
```

## 时间日期相关

### 显示当前时间

**命令：**`date [+format]`

**案例：**

```bash
date
date +%Y-%m-%d %H:%M:%S
```

### 显示非当前时间

**命令：**`date -d dateStr`

**案例：**

```bash
# 显示昨天的时间
date -d '1 days ago'
# 显示明天的时间
dae -d '-1 days ago'
```

### 设置系统时间

**命令：**`date -s dateStr`

**案例：**

```bash
date -s '2023-06-19 20:52:18'
```

### 显示日历

**命令：**`cal [options] [[[day] month] year]`

**选项：**

| 选项 |           说明           |
| :--: | :----------------------: |
|  -1  |      只显示当月日历      |
|  -3  | 显示上月、当月和下月日历 |
|  -y  |       显示当年日历       |

**案例：**

```bash
# 显示当月日历
cal
# 显示当年日历
cal -y
# 显示2023-05日历
cal 05 2023
# 显示2023-05-03日历
cal 03 05 2023
```

## 用户管理相关

### 添加用户

**命令：**`useradd [options] userName`

**选项：**

| 选项 |        说明        |
| :--: | :----------------: |
|  -d  |   指定用户家目录   |
|  -g  | 将用户添加到某个组 |

**案例：**

```bash
# 创建test用户，并指定家目录为/app/test
useradd -d /app/test test
```

### 设置密码

**命令：**`passwd userName`

### 查看用户是否存在

**命令：**`id userName`

**案例：**

```bash
# 查看当前用户信息
id
# 查看test用户信息
id test
```

### 查看用户列表

**命令：**`cat /etc/passwd`

**说明：**每一列分表表示用户名、密码、UID、GID、注释、家目录、命令解释器

### 切换用户

**命令：**`su [-] userName`

**说明：**不指定userName则默认为切换为root用户，不加`-`不能获取切换用户的环境变量。

### 删除用户

**命令：**`userdel [options] userName`

**选项：**

| 选项 |           说明           |
| :--: | :----------------------: |
|  -r  | 删除用户主目录和邮件目录 |

### 查看当前登录用户

**命令：**`whoami`

### 修改用户

**命令：**`usermod [options] userName`

**选项：**

| 选项 |      说明      |
| :--: | :------------: |
|  -g  | 修改用户所在组 |
|  -d  | 修改用户家目录 |

## 用户组管理相关

### 添加组

**命令：**`groupadd groupName`

### 删除组

**命令：**`groupdel groupName`

### 设置组密码/管理组

**命令：**`gpasswd [options] groupName`

**选项：**

| 选项 |      说明      |
| :--: | :------------: |
|  -a  | 向组中添加用户 |
|  -d  | 删除组中的用户 |

### 修改组

**命令：**`groupmod [options] groupName`

**选项：**

| 选项 |   说明   |
| :--: | :------: |
|  -n  | 修改组名 |

### 查看组列表

**命令：**`cat /etc/group`

**说明：**每一列分别表示组名、组密码、GID、组中的用户（不包含初始用户）

## 磁盘和分区相关

### du

> disk usage，磁盘占用情况

**命令：**`du [options] path`

**选项：**

| 选项 |                   说明                    |
| :--: | :---------------------------------------: |
|  -h  |  以人方便阅读的格式展示大小，如K、M、G等  |
|  -a  | 显示所有文件（文件+目录），默认只显示目录 |
|  -c  |        显示完所有文件后，限制总和         |
|  -s  |                只显示总和                 |

**案例：**

```bash
# 显示/tmp目录磁盘占用情况
du -sh /tmp
```

### df

> disk free，磁盘空余情况

**命令：**`df [options]`

**选项：**

| 选项 |                  说明                   |
| :--: | :-------------------------------------: |
|  -h  | 以人方便阅读的格式展示大小，如K、M、G等 |

**案例：**

```bash
# 查看磁盘空闲情况
df -h
```

### lsblk

> 查看设备挂载情况

**命令：**`lsblk [options]`

**选项：**

| 选项 |       说明       |
| :--: | :--------------: |
|  -f  | 显示文件系统信息 |

**案例：**

```bash
# 查看设备挂载信息
lsblk -f
```

### mount/umount

**命令：**

- `mount [-t vfstype] [-o options] device dir`
- `unmount`

**参数：**

|    参数    |                             功能                             |
| :--------: | :----------------------------------------------------------: |
| -t vfstype | 指定文件系统的类型，通常不必指定。mount 会自动选择正确的类型。常用类型有：<br />光盘或光盘镜像：iso9660<br />DOS fat16 文件系统：msdos<br />Windows 9x fat32 文件系统：vfat<br />Windows NT ntfs 文件系统：ntfs<br />Mount Windows 文件网络共享：smbfs<br />UNIX(LINUX) 文件网络共享：nfs |
| -o options | 主要用来描述设备或档案的挂接方式。常用的参数有：<br />loop：用来把一个文件当成硬盘分区挂接上系统<br />ro：采用只读方式挂接设备<br />rw：采用读写方式挂接设备<br />iocharset：指定访问文件系统所用字符集 |
|   device   |                     要挂接(mount)的设备                      |
|    dir     |              设备在系统上的挂接点(mount point)               |

**案例：**

```bash
# 建立挂载点
mkdir /mnt/cdrom/
# 设备/dev/cdrom挂载到挂载点中
mount -t iso9660 /dev/cdrom /mnt/cdrom/
ll /mnt/cdrom
# 卸载设备
umount /mnt/cdrom
```

**设置开机自动挂载：**

```bash
vi /etc/fstab
```

### fdisk

> 磁盘分区

**命令：**`fdisk [options]`

**选项：**

| 选项 |          说明          |
| :--: | :--------------------: |
|  -l  | 显示所有硬盘的分区列表 |

**案例：**

```bash
# 查看磁盘分区
fdisk -l
```

## 进程管理相关

### ps

> process status，进程状态

**命令：**`ps [options]`

**选项：**

| 选项 |            说明            |
| :--: | :------------------------: |
|  a   |  列出所有用户带终端的进程  |
|  x   |     列出不带终端的进程     |
|  u   |    以面向用户的风格展示    |
|  -e  |        列出所有进程        |
|  -f  | 以完成格式显示，包括命令行 |
|  -u  | 列出某个用户关联的所有进程 |

**说明：**

- ps axu显示信息项

  - USER：创建该进程的用户；

  - PID：进程id；

  - %CPU：进程占CPU资源的百分比；

  - %MEM：进程占物理内存的百分比；

  - VSZ：该进程占用虚拟内存的大小，单位 KB；

  - RSS：该进程占用实际物理内存的大小，单位 KB；

  - TTY：该进程是在哪个终端中运行的。对于 CentOS 来说，tty1 是图形化终端，

    tty2-tty6 是本地的字符界面终端。pts/0-255 代表虚拟终端；

  - STAT：进程状态。常见的状态有：R：运行状态、S：睡眠状态、T：暂停状态、Z：僵尸状态、s：包含子进程、l：多线程、+：前台显示；
  - START：该进程的启动时间；
  - TIME：该进程占用 CPU 的运算时间，注意不是系统时间；
  - COMMAND：产生此进程的命令名；

- ps -ef显示信息项

  - UID：用户 ID；
  - PID：进程 ID；
  - PPID：父进程 ID；
  - C：CPU 用于计算执行优先级的因子。数值越大，表明进程是 CPU 密集型运算，执行优先级会降低；数值越小，表明进程是 I/O 密集型运算，执行优先级会提高；
  - STIME：进程启动的时间；
  - TTY：完整的终端名称；
  - TIME：CPU 时间；
  - CMD：启动进程所用的命令和参数；

**案例：**

```bash
ps aux
ps -ef
```

### kill

> 终止进程

**命令：**

- `kill [options] pid`，通过进程号杀死进程；
- `killall pname`，通过进程名杀死进程，在系统负载过大时很有效；

**选项：**

- -9：表示立即停止进程

**案例：**

```bash
# 强制杀死进程id为5101的进程
kill -9 5101
# 强制杀死名为aaa的进程
killall -9 aaa
```

### pstree

> 显示进程树

**命令：**`pstree [options]`

**选项：**

- -p：显示进程id；
- -u：显示进程所属用户；

**案例：**

```bash
pstree -pu
```

### top

> 实时监控进程状态

**命令：**`top [options]`

**选项：**

- -d：指定每隔多少秒刷新，默认3秒；
- -i：不显示闲置或僵尸进程；
- -p：监控指定的进程id

**操作：**

- P：按CPU使用率排序，默认就是此项；
- M：按内存的使用率排序；
- N：按PID排序；
- q：退出top；

**案例：**

```bash
# 监控pid为11232的进程
top -p 11232
```

### netstat

> 显示网络状态和端口占用信息

**命令：**`netstat [options]`

**选项：**

- -a：显示所有socket；
- -n：不显示别名，能显示数字的全部显示数字；
- -p：显示进程信息；
- -t：显示使用TCP协议的socket；
- -u：显示使用UDP协议的socket；
- -l：仅显示正在监听的socket；

**案例：**

```bash
# 查看sshd进程的网络信息
netstat -anp | grep sshd
# 查看22端口的占用情况
netstat -nlp | grep 22
```

### crontab

> cron定时任务

**命令：**`crontab [options]`

**选项：**

- -u：指定操作crontab的用户；
- -e：编辑crontab任务；
- -l：显示用户的crontab任务；
- -r：删除用户的crontab任务；

**案例：**

```bash
# 编辑pxz用户的crontab任务
# */1 * * * * /bin/echo '11' >> /tmp/test.txt
crontab -e -u pxz
# 显示当前用户的crontab任务
crontab -l
```

**crontab命令解释：**

*注意：周和日期、月份不要同时指定！*

每行任务都有6列，分别表示：

- 分钟：取值为0-59；
- 小时：取值为0-23；
- 日期：取值为1-31；
- 月份：取值为1-12；
- 周：取值为0-7,0和7都表示周日；
- 命令

每列取值中可包含的特殊符号：

- *（星号）：表示无限制，任何时间都符合要求；
- ,（逗号）：表示只限制固定的几个时间，如`1,11 * * * *`，表示每小时的第1、第11分钟执行任务；
- -（减号）：表示时间范围，如`0 8-12 * * *`，表示8点至12点间的每小时0分都执行任务；
- /n（斜线）：n为一个数字，表示每次任务间隔n，如`*/5 * * * *`表示每5分钟执行一次；

# 软件包管理

## rpm

> rpm（英文全拼：redhat package manager） 原本是 Red Hat Linux 发行版专门用来管理 Linux 各项套件的程序，由于它遵循 GPL 规则且功能强大方便，因而广受欢迎。逐渐受到其他发行版的采用。

**命令：**`rpm [options]`

**选项：**

- -q：查询模式；
- -a：显示所有包；
- -f：查询包含指定文件的包；
- -l：查询包中的文件列表；
- -e：卸载选件包；
- -i：安装包；
- -U：更新包，存在更新，不存在则安装；
- -F：更新包，存在时才更新；
- -v：显示详细信息；
- -h：显示进度条；

**案例：**

```bash
# 查询软件包
rpm -qa packageName
# 安装软件包
rpm -ivh packageName
# 更新软件包
rpm -Uvh packageName
# 卸载软件包
rpm -e packageName
# 查看/sbin/ifconfig是哪个包安装的
rpm -qf /sbin/ifconfig
# 查看net-tools包的文件
rpm -ql net-tools
```

## yum

> yum（ Yellow dog Updater, Modified）是一个在 Fedora 和 RedHat 以及 SUSE 中的 Shell 前端软件包管理器。
>
> 基于 RPM 包管理，能够从指定的服务器自动下载 RPM 包并且安装，可以自动处理依赖性关系，并且一次安装所有依赖的软件包，无须繁琐地一次次下载、安装。
>
> yum 提供了查找、安装、删除某一个、一组甚至全部软件包的命令，而且命令简洁而又好记

**命令：**`yum [options] command package`

**选项：**

- -y：对所有提问都回答yes；

**命令：**

- install：安装；
- update：更新；
- check-update：检查是否有可用的更新；
- remove：删除；
- list：显示软件包列表；
- clean：清除缓存；

**案例：**

```bash
# 安装net-tools
yum -y install net-tools
```



# 命令字典

## 查询软件安装目录

```ba
# 以查询redis为例
ps -ef | grep redis # 获取redis的进程id
ll /proc/processId/cwd
```

























