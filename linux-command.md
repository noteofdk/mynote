---
title: linux_command
categories: linux
tags: [linux]
comments: false
img: http://7xq4tu.com1.z0.glb.clouddn.com/linux.jpg
date: 2017-12-28 09:43:16
---

资料：

- [Linux 命令](http://www.cnblogs.com/peida/tag/linux%E5%91%BD%E4%BB%A4/)

# 后台运行和关闭、查看后台任务

1. `&`:
加在一个命令的最后，可以把这个命令放到后台执行
2. `ctrl + z`
可以将一个正在前台执行的命令放到后台，并且处于暂停状态。
3. `jobs`
查看当前有多少在后台运行的命令;
`jobs -l`选项可显示所有任务的PID，jobs的状态可以是running, stopped, Terminated。但是如果任务被终止了（kill），shell 从当前的shell环境已知的列表中删除任务的进程标识。
4. `fg`
将后台中的命令调至前台继续运行。如果后台中有多个命令，可以用`fg %jobnumber`（是命令编号，不是进程号）将选中的命令调出。
5. `bg`
将一个在后台暂停的命令，变成在后台继续执行。如果后台中有多个命令，可以用 `bg %jobnumber` 将选中的命令调出。
6. `kill`
- 法子1：通过`jobs`命令查看job号（假设为num），然后执行`kill %num`
- 法子2：通过`ps`命令查看job的进程号（PID，假设为pid），然后执行 `kill pid`
前台进程的终止：`Ctrl+c`
7. `nohup`
如果让程序始终在后台执行，即使关闭当前的终端也执行（之前的&做不到），这时候需要nohup。该命令可以在你退出帐户/关闭终端之后继续运行相应的进程。关闭中断后，在另一个终端 `jobs` 已经无法看到后台跑得程序了，此时利用 `ps`（进程查看命令）

`
ps -aux | grep "test.sh"
`

- a:显示所有程序
- u:以用户为主的格式来显示
- x:显示所有程序，不以终端机来区分


# 查看磁盘
## `df` 以磁盘分区为单位查看文件系统

`df -hl`
显示格式为：

文件系统 容量 已用 可用 已用% 挂载点

# 命令
## `which`
> locate a command

which  returns the pathnames of the files (or links) which would be executed in the current environment, had its arguments been given as commands in a strictly POSIX-conformant shell.  It does this by searching the PATH for executable files matching the names of the arguments.  It does not follow symbolic links.



##`whereis [-bmsu]`
> locate programs

- `-l` 列出查找的目录
- `-b` 只列出二进制格式的文档
- `-m` 只找出说明文档 manual 路径下的文档
- `-s` 只找出 source 来源档案
- `-u` 搜寻不在上述三个项目当中的其他特殊文档

The whereis utility checks the **standard binary directories** for the specified programs, printing out the paths of any it finds.



# 用户管理
Linux下管理员强行踢出用户的命令使用方法
```
pkill -kill -t pts/2
```

## 用户 `/etc/passwd`
`id`:   查看id

`users`:        查看当前系统中有哪些用户；
`who`:          当前在线用户；
`w`:            当前在线用户详细信息；
`finger`:       查看用户详细信息；

`useradd`:  增加用户
`usermod`:  修改用户

`su`:       切换用户

`passwd [LOGIN]` : 修改密码


### 用户默认 shell
- 查看 `echo $SHELL`
- 设置位置： `/etc/passwd`

- 配置自动刷新 `.bashrc`: 通过 `.profile`

## 用户组： `/etc/group`
`groups`:   查看自己所属用户组
`groupadd`:     增加用户组
`groupdel`


# 任务管理

`cron`:     周期性执行；

`at`:       特定实践执行一次；
`atq`:      查询 at 任务列表；
`atrm`:     删除任务队列中的任务；

# 系统
## locale
设定locale   让Linux能够输入中文
locale    软件运行时的语言环境

### locale分类：
语言符号及其分类(LC_CTYPE)，数字 (LC_NUMERIC)，比较和排序习惯(LC_COLLATE)，时间显示格式(LC_TIME)，货币单位(LC_MONETARY)，信息主要是提示信息,错误信息, 状态信息, 标题, 标签, 按钮和菜单等(LC_MESSAGES)，姓名书写方式(LC_NAME)，地址书写方式(LC_ADDRESS)，电话号码书写方式 (LC_TELEPHONE)，度量衡表达方式(LC_MEASUREMENT)，默认纸张尺寸大小(LC_PAPER)和locale对自身包含信息的概述(LC_IDENTIFICATION)。
eg:
```
$ locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"                    #用户所使用的语言符号及其分类
LC_NUMERIC="en_US.UTF-8"                #数字
LC_TIME="en_US.UTF-8"                      #时间显示格式
LC_COLLATE="en_US.UTF-8"                #比较和排序习惯
LC_MONETARY="en_US.UTF-8"             #LC_MONETARY
LC_MESSAGES="en_US.UTF-8"             #信息主要是提示信息,错误信息, 状态信息, 标题, 标签, 按钮和菜单等
LC_PAPER="en_US.UTF-8"                    #默认纸张尺寸大小
LC_NAME="en_US.UTF-8"                     #姓名书写方式
LC_ADDRESS="en_US.UTF-8"               #地址书写方式
LC_TELEPHONE="en_US.UTF-8"             #电话号码书写方式
LC_MEASUREMENT="en_US.UTF-8"        #度量衡表达方式
LC_IDENTIFICATION="en_US.UTF-8"      #对自身包含信息的概述
LC_ALL=
```

locale定义文件放在/usr/share/i18n/locales

字符集就是字符，尤其是非英语字符在系统内的编码方式，也就是通常所说的内码，所有的字符集都放在 /usr/share/i18n/charmaps，所有的字符集也都是用Unicode编号索引的。Unicode用统一的编号来索引目前已知的全部的符号。而字符集则是这些符号的编码方式，或者说是在网络传输，计算机内部通信的时候，对于不同字符的表达方式，Unicode是一个静态的概念，字符集是一个动态的概念，是每一个字符传递或传输的具体形式.

Locale 是软件在运行时的语言环境, 它包括语言(Language), 地域 (Territory) 和字符集(Codeset)。一个locale的书写格式为: 语言[_地域[.字符集]].
生成的locale放在/usr/lib/locale/目录中

自定义locale

优先级的关系：
LC_ALL>LC_*>LANG
LC_ALL的值将覆盖所有其他的locale设定

查看zh_CN使用的编码：
/usr/lib/locale/zh_CN/LC_* 说明了使用何种编码



## 配置自启动
### 开机启动时自动运行程序

Linux加载后, 它将初始化硬件和设备驱动, 然后运行第一个进程init。init根据配置
   文件继续引导过程，启动其它进程。通常情况下，修改放置在

    /etc/rc或
    /etc/rc.d 或
    /etc/rc?.d

目录下的脚本文件，可以使init自动启动其它程序。例如：编辑

     /etc/rc.d/rc.local

文件(该文件通常是系统最后启动的脚本)，在文件最末加上一行“xinit”或“startx”，可以在开机启动后直接进入X－Window。

### 登录时自动运行程序
用户登录时，bash先自动执行系统管理员建立的全局登录script ：

```
/ect/profile
```

然后bash在用户起始目录下按顺序查找三个特殊文件中的一个：
```
/.bash_profile、
/.bash_login、
/.profile，
```
   但只执行最先找到的一个。因此，只需根据实际需要在上述文件中加入命令就可以实
   现用户登录时自动运行某些程序（类似于DOS下的Autoexec.bat）。

# 文件系统
## 文件
### touch
`touch [OPTION]... FILE...`
Update the access and modification times of each FILE to the current time.

### rm
`rm [OPTION]... FILE...`
Remove (unlink) the FILE(s).

### mv
```
Usage: mv [OPTION]... [-T] SOURCE DEST
  or:  mv [OPTION]... SOURCE... DIRECTORY
  or:  mv [OPTION]... -t DIRECTORY SOURCE...
```
Rename SOURCE to DEST, or move SOURCE(s) to DIRECTORY.

### cat
```
Usage: cat [OPTION]... [FILE]...
```
Concatenate FILE(s), or standard input, to standard output.

### head
```
NAME
     head -- display first lines of a file

SYNOPSIS
     head [-n count | -c bytes] [file ...]

DESCRIPTION
     This filter displays the first count lines or bytes of each of the specified files, or of the standard input if no files are specified.
     If count is omitted it defaults to 10.

     If more than a single file is specified, each file is preceded by a header consisting of the string ``==> XXX <=='' where ``XXX'' is the
     name of the file.
```

### tail

### dos2unix

### 链接


## 目录
### cd
```
usage: cd [-L|[-P [-e]] [-@]] [dir]
```

### mkdir

### cp

## 权限
### 查看文件权限的命令：
```
ls -l xxx.xxx （xxx.xxx是文件名）
```
那么就会出现相类似的信息，主要都是这些：

```
-rw-rw-r--
```

一共有10位数

其中： 最前面那个 - 代表的是类型
```
中间那三个 rw- 代表的是所有者（user）
然后那三个 rw- 代表的是组群（group）
最后那三个 r-- 代表的是其他人（other）
```

```
r 表示文件可以被读（read）
w 表示文件可以被写（write）
x 表示文件可以被执行（如果它是程序的话）
- 表示相应的权限还没有被授予
```

### 修改文件权限:

在终端输入：
```
chmod o+w xxx.xxx
```
表示给其他人授予写xxx.xxx这个文件的权限

```
chmod go-rw xxx.xxx
```
表示删除xxx.xxx中组群和其他人的读和写的权限



其中：
```
u 代表所有者（user）
g 代表所有者所在的组群（group）
o 代表其他人，但不是u和g （other）
a 代表全部的人，也就是包括u，g和o
```

```
r 表示文件可以被读（read）
w 表示文件可以被写（write）
x 表示文件可以被执行（如果它是程序的话）
```


其中：rwx也可以用数字来代替
```
r ------------4
w -----------2
x ------------1
- ------------0
```


行动：
```
+ 表示添加权限
- 表示删除权限
= 表示使之成为唯一的权限
```


当大家都明白了上面的东西之后，那么我们常见的以下的一些权限就很容易都明白了：
```
-rw------- (600) 只有所有者才有读和写的权限
-rw-r--r-- (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限
-rwx------ (700) 只有所有者才有读，写，执行的权限
-rwxr-xr-x (755) 只有所有者才有读，写，执行的权限，组群和其他人只有读和执行的权限
-rwx--x--x (711) 只有所有者才有读，写，执行的权限，组群和其他人只有执行的权限
-rw-rw-rw- (666) 每个人都有读写的权限
-rwxrwxrwx (777) 每个人都有读写和执行的权限
```

# 压缩
## tar
```
-A 新增压缩文件到已存在的压缩

-B 设置区块大小

-c 建立新的压缩文件

-d 记录文件的差别

-r 添加文件到已经压缩的文件

-u 添加改变了和现有的文件到已经存在的压缩文件

-x 从压缩的文件中提取文件

-t 显示压缩文件的内容

-z 支持gzip解压文件

-j 支持bzip2解压文件

-Z 支持compress解压文件

-v 显示操作过程

-l 文件系统边界设置

-k 保留原有文件不覆盖

-m 保留文件不被覆盖

-W 确认压缩文件的正确性
```

例子：

```
tar -cvf log.tar log2012.log       仅打包，不压缩！

tar -zcvf log.tar.gz log2012.log   打包后，以 gzip 压缩

tar -jcvf log.tar.bz2 log2012.log  打包后，以 bzip2 压缩

在参数 f 之后的文件档名是自己取的，我们习惯上都用 .tar 来作为辨识。 如果加 z 参数，则以 .tar.gz 或 .tgz 来代表 gzip 压缩过的 tar包； 如果加 j 参数，则以 .tar.bz2 来作为tar包名。


```

将mifan压缩包解压:
```
tar zxvf mifan.tar.gz
```
将根目录下面的mifan压缩:
```
tar czvf mifan.tar.gz   /mifan/*
```

# 文本
## 管道

## grep 搜索文本

## sort 排序

## uniq 删除相邻重复行

## cut 截取文本

## tr 文本转换

## paste 文本合并

## split 分割大文件


# 网络
## 网络接口类型
- lo：本地回环接口
- eth[0-9]：以太网接口
- pppx：点对点的连接



## ifconfig 命令：配置网络接口
ifconfig打开或关闭网络接口，设置IP地址与子网掩码，以及其他选项和参数。
通常在启动时通过命令行从配置文件中读取参数来运行，但也可以手动运行以做修改。

命令格式：
```
ifconfig [-v] [-a] [-s] [interface]
ifconfig [-v] interface [aftype] options | address ...
```

查看活动的网卡信息
```
ifconfig
```

- `ifconfig eth0 down` 类似于 `ifdown eth0`
关闭 eth0 网卡

- `ifconfig eth0 up` 类似于 `ifup eth0`

RedHat, CentOS 网络配置文件：`/etc/sysconfig/network-scripts/` 目录下， eth0 的配置文件为： `ifcfg-eth0`


```
Usage:
  ifconfig [-a] [-v] [-s] <interface> [[<AF>] <address>]
  [add <address>[/<prefixlen>]]
  [del <address>[/<prefixlen>]]
  [[-]broadcast [<address>]]  [[-]pointopoint [<address>]]
  [netmask <address>]  [dstaddr <address>]  [tunnel <address>]
  [outfill <NN>] [keepalive <NN>]
  [hw <HW> <address>]  [metric <NN>]  [mtu <NN>]
  [[-]trailers]  [[-]arp]  [[-]allmulti]
  [multicast]  [[-]promisc]
  [mem_start <NN>]  [io_addr <NN>]  [irq <NN>]  [media <type>]
  [txqueuelen <NN>]
  [[-]dynamic]
  [up|down] ...

  <HW>=Hardware Type.
  List of possible hardware types:
    loop (Local Loopback) slip (Serial Line IP) cslip (VJ Serial Line IP)
    slip6 (6-bit Serial Line IP) cslip6 (VJ 6-bit Serial Line IP) adaptive (Adaptive Serial Line IP)
    ash (Ash) ether (Ethernet) ax25 (AMPR AX.25)
    netrom (AMPR NET/ROM) rose (AMPR ROSE) tunnel (IPIP Tunnel)
    ppp (Point-to-Point Protocol) hdlc ((Cisco)-HDLC) lapb (LAPB)
    arcnet (ARCnet) dlci (Frame Relay DLCI) frad (Frame Relay Access Device)
    sit (IPv6-in-IPv4) fddi (Fiber Distributed Data Interface) hippi (HIPPI)
    irda (IrLAP) ec (Econet) x25 (generic X.25)
    eui64 (Generic EUI-64)
  <AF>=Address family. Default: inet
  List of possible address families:
    unix (UNIX Domain) inet (DARPA Internet) inet6 (IPv6)
    ax25 (AMPR AX.25) netrom (AMPR NET/ROM) rose (AMPR ROSE)
    ipx (Novell IPX) ddp (Appletalk DDP) ec (Econet)
    ash (Ash) x25 (CCITT X.25)
```

## ip命令

### ip link：配置网络接口属性

- ip link show：查看所有网络接口属性信息
- ip -s link show：查看所有统计信息
- ip link set ethX {up|down|arp {on|off}}:设置网络接口的工作属性

### ip addr：配置网络地址

- ip addr show：查看网络信息，看到的信息和ip link show差不多，都比较简要
- ip addr add IP dev ethX ：配置IP地址（此命令配置的网卡信息利用ifconfig查看不到，需要利用ip addr show查看）
- ip addr add IP dev ethx label ethX:X：配置子Ip并对其加别名


## route 命令:指定静态路由

- `route`: 直接可以查看我们系统上的路由信息
- `route -n`: 以数字的形式显示路由信息

动态配置：

- `route add default gw 192.168.159.2`: 添加默认网关

配置文件

- `/etc/sysconfig/network` 中添加 `GATEWAY=192.168.159.2`

```
Usage: route [-nNvee] [-FC] [<AF>]           List kernel routing tables
       route [-v] [-FC] {add|del|flush} ...  Modify routing table for AF.

       route {-h|--help} [<AF>]              Detailed usage syntax for specified AF.
       route {-V|--version}                  Display version/author and exit.

        -v, --verbose            be verbose
        -n, --numeric            don't resolve names
        -e, --extend             display other/more information
        -F, --fib                display Forwarding Information Base (default)
        -C, --cache              display routing cache instead of FIB

  <AF>=Use '-A <af>' or '--<af>'; default: inet
  List of possible address families (which support routing):
    inet (DARPA Internet) inet6 (IPv6) ax25 (AMPR AX.25)
    netrom (AMPR NET/ROM) ipx (Novell IPX) ddp (Appletalk DDP)
    x25 (CCITT X.25)

```

例子
```
route add -net 192.168.45.128/25 zulu-gw.atrust.net
```
该命令通过网关路由器zulu-gw.atrust.net添加一条到192.168.45.128/25网络的路由。通常，网关路由器是相邻主机或本地主机的一个接口（Linux要求在网关地址前加上gw选项名）。route命令必须能够将zulu-gw.atrust.net解析成IP地址。



### 添加路由
```
route  add -host：添加主机路由
route  add -net：添加网络路由
route  add -net  0.0.0.0：添加默认路由
```

格式：route add -net|host DEST gw NEXTHOP
例如，添加一条路由，让主机通过172.16.7.3访问192.168.0.0/24网段

```
route add –net 192.168.0.0/24 gw 172.16.7.3
```

## ping
常用：

- `-c`: 指定 ping 次数；
- `-i`: 指定 ping 包发送间隔；
- `-w`: 如果没有反应，则在指定超时时间后推出；





## 设置静态 ip
### 修改网卡配置
文件：`/etc/network/interfaces`

```
auto eth0
iface eth0 inet static
address 192.168.0.117
gateway 192.168.0.1 #这个地址你要确认下 网关是不是这个地址
netmask 255.255.255.0
network 192.168.0.0
broadcast 192.168.0.255
```

### 修改 dns 解析

因为以前是dhcp解析，所以会自动分配dns服务器地址

而一旦设置为静态ip后就没有自动获取到的dns服务器了

要自己设置一个
```
sudo vim /etc/resolv.conf
```

写上一个公网的DNS
```
nameserver 202.96.128.86
```

### 重启网卡：
```
sudo /etc/init.d/network restart
```

## 查看路由信息

```
route -n
```

## DNS
### `/etc/hosts` 文件
最初：纪录主机名与 IP 的对应关系

目前：
- 加快域名解析
- 方便小型局域网使用内部设备


### `dig`
显示 dns 查询过程

```
dig baidu.com
```

### `dig -x <ip>`
从IP地址查询域名

### host
- 返回当前请求域名的各种记录
- 也可以用于逆向查询，即从IP地址查询域名

### nslookup
用于互动式地查询域名记录


### whois
查看域名的注册情况。

```
whois github.com
```

## ss
ss是Socket Statistics的缩写。顾名思义，ss命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

它使用了 TCP协议栈中 tcp_diag（是一个用于分析统计的模块），能直接从获得第一手内核信息，这就使得 ss命令快捷高效。在没有 tcp_diag，ss也可以正常运行。



# 包

## apt-get

## dpkg



-----

![](leanote://file/getImage?fileId=58859d8bb88f442402000000)


# curl
[curl网站开发指南](http://www.ruanyifeng.com/blog/2011/09/curl.html)


# [wget](http://www.cnblogs.com/peida/archive/2013/03/18/2965369.html)

Linux系统中的wget是一个下载文件的工具。
wget支持HTTP，HTTPS和FTP协议，可以使用HTTP代理。
wget可以在用户退出系统的之后在后台执行。这意味这你可以登录系统，启动一个wget下载任务，然后退出系统，wget将在后台执行直到任务完成。

wget 可以跟踪HTML页面上的链接依次下载来创建远程服务器的本地版本，完全重建原始站点的目录结构。这又常被称作”递归下载”。在递归下载的时候，wget 遵循Robot Exclusion标准(/robots.txt). wget可以在下载的同时，将链接转换成指向本地文件，以方便离线浏览。


