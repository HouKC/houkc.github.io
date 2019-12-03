---
layout:     post
title:      git安装与配置（下）
subtitle:   windows下安装git，并且配置其提交到GitHub项目
date:       2019-12-03
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - git
    - windows
    - github
---

## 前言

紧接上回git配置，完成配置之后我们需要连接到GitHub，确保你已经申请了GitHub账号。

如果你在GitHub上已有仓库，那么直接克隆下来；如果没有，可以直接在本地新建完上传上去。

#### 1. 本地新建仓库
打开一个空的文件夹（比如新建一个文件夹workspace），你将在这个文件夹下创建git配置和GitHub工程。

右键打开Git Bash， 输入以下命令初始化git环境和创建工程：
```
git init project_name       # project_name就是你要创建的工程名，或者说仓库名
```

#### 2. 克隆远程仓库
打开一个空的文件夹（比如新建一个文件夹workspace），