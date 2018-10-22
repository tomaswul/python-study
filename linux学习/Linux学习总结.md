---
title: Linux学习总结
date: 2018-10-08 20:11:38
tags:
---
Linux学习总结
<!--more-->

### 1.linux文件结构

/bin - 基本命令的二进制文件。
/boot - 引导加载程序的静态文件。
/dev - 设备文件。
/etc - 配置文件。
/home - 普通用户主目录的父目录。
/lib - 共享库文件。
/lib64 - 共享64位库文件。
/lost+found - 存放未链接文件。
/media - 自动识别设备的挂载目录。
/mnt - 临时挂载文件系统的挂载点。
/opt - 可选插件软件包安装位置。
/proc - 内核和进程信息。
/root - 超级管理员用户主目录。
/run - 存放系统运行时需要的东西。
/sbin - 超级用户的二进制文件。
/sys - 设备的伪文件系统。
/tmp - 临时文件夹。
/usr - 用户应用目录。
/var - 变量数据目录。


### 2.文件或者目录操作

cat 文件:查看文件内容
pwd:查看当前目录
cd 目录:切换到指定目录
ls:查看所有目录
wget url:联网下载文件
mkdir 文件夹名:创建指定文件名的文件夹
rmdir 文件夹名:删除指定名字的空文件夹
cp 资源:复制的文件或者目录
rm 资源:删除文件或者目录
mv 源文件:目标文件夹 剪切到指定文件夹下
touch 文件名:创建一个空文件

### 3.用户管理

adduser 用户名:创建新用户
passwd 用户名:设置指定用户的密码
su 用户名:切换用户(switch user)
userdel 用户名:删除用户
sudo :-super user do(以超级管理员的身份执行命令)
uname:查看系统名称
hostname:查看主机名

### 4.查询命令

who:查看登录的用户
which 命令:查看命令的全路径写法
whereis 命令:找到命令的二进制、源和手动页文件
apropos 命令:搜索指定命令的手册的名称和描述
head:查看文件的开始部分

head -3 文件名:查看文件的前三行

tail:查看文件的结束部分

tail -5 文件:查看文件的最后5行

more: 分屏显示文件内容
less :分屏显示文件内容



### 5.linux运行级别

0:系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
1:单用户工作状态，root权限，用于系统维护，禁止远程登陆
2:多用户状态(没有NFS)
3:完全的多用户状态(有NFS)，登陆后进入控制台命令行模式
4:系统未使用，保留
5:X11控制台，登陆后进入图形GUI模式
6:系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动

### 6.linux常用命令

启动服务 : systemctl 状态 服务
write 用户:给指定用户发消息
mesg n:不接收消息
mesg y:接收消息
sftp:命令

​	sftp 用户名@ip
​	sftp> get 要下载的文件
​	sftp> lls 查看本地的文件及文件夹（local ls）
​	注意:在sftp下载所有命令前加上l表示在本地执行命令
​	quit - 退出
​	bye - 退出
​	help - 查看sftp下的命令

echo 语句>文件:echo将语句重定向到文件中
安全的远程连接

```
ssh 用户名@ip
```

安全拷贝

```
scp 源文件 用户名@ip:/root/
scp root@ip:/源文件路径 root@ip2::/文件路径2
```

date 查看系统时间日期

cal 查看日历
alias 别名=长命令格式:可以给长命令起短名字

	alias rmd="rm -rf" 设置rmd别名
unalias 别名:去掉这个别名
	unalias rmd 去掉rmd的别名
|表示管道
输出重定向
	> 输出重定向
	>> 追加重定向
	2> 错误输出重定向
输入重定向
	< 输入重定向
	clear 清除屏幕
history 查看所有历史命令
grep搜索字符串

```
ls -l | grep ssh
```

ps 查看进程

```
ps aux | grep sshd
```

kill pid 杀掉进程

wc统计次数

```
wc - word count
wc -l 只看行数
wc
	行数	单词数	字节数
```

uniq-unqiue-实现文件去重
sort 对文件内容进行排序

```
sort hello.txt | uniq | wc -l
```

diff 文件一 文件二 比较两个文件是否不同

```
这个命令没什么卵用
```

压缩文件

```
-gz---gzip（压缩）/gunzip(解压缩)
-xz---xz -z (压缩)/xz -d(解压缩)
winRAR归档和解归档
tar-归档文件
归档-把多个文件合并成一个文件
解归档-把一个文件分成多个文件
```

tar命令

```
参数
	-x 解归档
	-v 看归档过程
	-f	指定文件
```

链接

```
硬链接:ln 要被链接的文件(完整路径) 链接文件
	硬链接-文件的引用，只要引用数不为0，文件就会一直存在
软连接-相当于是文件的快捷方式，如果文件被删除软链接失效
ln -s 带完整路径的文件名 链接文件名
```

exit/logout 退出登录



### 7.服务相关命令

top 动态查看cpu使用情况(cpu从高到低)
如果需要把运行中的进程终止掉-Ctrl+c
如果要暂停运行中的进程并把它放到后台ctrl+z
如果执行命令时在命令后面加上&就可以把命令放到后台运行

```
./fun.py &
```

jobs:查看当前后台进程

bg %后台程序编号 让后台暂停的经常继续在后台运行

```
bg %1
```

fg %后台进程编号 后台进程哪到前台执行

```
fg %1
```

systemctl status/start/rstart/stop 服务:查看服务状态/开启服务/重启服务/停止服务
centos6.x

```
service <name> start	启动服务
service <name> stop 停止服务
service <name> status 查看服务状态
service <name> restart 重启服务
```

nginx （直接输入也能启动nginx但是不建议使用）
journalctl -xe 查看错误日志
ps 查看进程

```
参数
 -ef 查看所有进程
ps -ef | grep nginx | grep -v grep
	grep -v grep:表示搜索结果中不要用
```

### 8.网络相关命令

查看ip地址ifconfig:也可以用ip address

netstat -an 查看所有网络端口

netstat -anp | grep 80 查看80端口的使用信息(包括端口占用程序)

firewall防火墙:

```
启动防火墙systemctl start firewalld
firewall-cmd 配置防火墙命令
firewall-cmd --permanent --add-port=端口/协议 开启指定协议下的指定端口
firewall-cmd --query-port=端口/协议  搜索指定协议的指定端口是否开启
firewall-cmd --remove-port=端口/协议 关闭指定协议下的指定端口
```

### 9.补充知识

DOS - Deny of Service(拒绝服务攻击)
DDOS - Distributed Deny of Service (分布式拒绝服务攻击)
min/avg/max/mdev = 41.489/41.533/41.608/0.207 ms

计算机网络分层架构模型
Internet --- TCP/IP协议族
TCP - Transfer Control Protocol - 传输控制协议
UDP - User Datagram Protocol - 用户数据报协议
IP - Internet Protocol -网际协议

TCP/IP模型
应用层(定义应用级协议) - HTTP/SMTP/POP3/FTP/SSH/ICQ/QQ
传输层(端到端传输数据) - TCP/UDP
网络层/网际层(寻址和路由) - IP/ICMP
物理链路层(数据分帧+校验) - 冗余检验码



