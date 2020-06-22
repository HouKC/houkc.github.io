---
layout:     post
title:      新BugKu平台web题writeup
subtitle:   新Bugku平台，又名newbugku，打靶CTF，CTF_论剑场。包含题目web26、web1、web9、流量分析、web2、web5、web6、web11、web13、日志审计、web18、web20、web3、web4、web15、web14、web21、web23、web7。
date:       2020-06-18
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - CTF
    - web
    - writeup
    - 网络安全
---

## web26
```php
<?php
$num=$_GET['num'];
$str=$_GET['str'];
show_source(__FILE__);
if (isset($num)&&isset($str)) {
    if (preg_match('/\d+/sD',$str)) {
        echo "vagetable hhhh";
        exit();
    }
    $result=is_numeric($num) and is_numeric($str);
    if ($result) {
        include "flag.php";
        echo "$flag";
    }
    else{
        echo "vagetablessssss";
    }
} 
```
可以看出需要get请求带两个参数num和str，根据preg_match('/\d+/sD‘,$str)可知是匹配，当str为数值(\d+)时就会匹配到。

s是.匹配符可以匹配换行，以及D是如果有$结尾，则不允许结尾换行。这两个与本题没什么关系，反正就是匹配数值，str不是数值就行。

然后我们又在这行$result=is_numeric($num) and is_numeric($str);发现了php逻辑运算符and，由于php中**and**的优先级比 **=** 低，所以会先将is_numeric($num) 的值赋值给result，这里只要保证num是数值，那么不管str是不是数值，也就是不管is_numeric($str)是真还是假，result都是真。

因此构造url为：
```
http://123.206.31.85:10026/?num=123&str=abc
```
即可返回flag
```
?> No No No Don't want to go back the door!!!flag{f0058a1d652f13d6}
```


## web1
审计图片里的代码，可以分析到需要get请求带两个参数，一个是a，一个是b，b要传一个文件，但这里我们利用php://input可以从http请求中读取。

我们随便给a一个值，比如1，然后给b的值是php://input，接着我们用burpsuite抓包，补充上准备给php://input读取的值，写在http请求中。

get请求链接如下：
```
http://123.206.31.85:10001/?a=1&b=php://input
```
bp抓包后修改如下：
![web1-bp抓包](https://github.com/HouKC/houkc.github.io/blob/master/img/web1-bp.png)
即可收到返回的flag

```
flag{c3fd1661da5efb989c72b91f3c378759}
```


## web9



## 流量分析
## web2
## web5
## web6
## web11
## web13
## 日志审计
## web18
## web20
## web3
## web4
## web15
## web14
## web21
## web23
## web7