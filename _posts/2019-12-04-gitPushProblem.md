---
layout:     post
title:      git push输入用户名密码问题
subtitle:   git push的时候老是要输入用户名和密码
date:       2019-12-04
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - git
    - windows
    - github
---

## 前言
今天遇到一个git push的问题，就是git push的时候每次都要我输入用户名和密码，很繁琐，因此在网上搜索后尝试了这个办法来解决它。

原因好像是http方式push不会保存密码（虽然我之前也不用输密码，只是突然就开始要了。。），要么密码保存本地，要么改用ssh方式。
## 步骤
#### 1. 方法一，账号密码保存本地
在git push之后，按照提示输入用户名和密码，完成git push操作。然后再输入以下命令：
```
git config --global credential.helper store
```
执行上述命令之后会在C:/users/user/目录中产生一个文件.git-credentials，这个文件保存着一个链接，是记录你的账号和密码的。

#### 2. 方法二，改用ssh方式
