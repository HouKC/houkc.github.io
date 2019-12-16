---
layout:     post
title:      Django简介
subtitle:   简单介绍Django的MTV架构，以及Django的基本组成，顺便说一下怎么新建一个Django项目。
date:       2019-12-16
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - Django
    - Python
    - 后端
---

## 前言
Django是基于Python的开源web框架，开发非常高效。只要很少的代码就可以构建一个完整的网站，还可以进一步完善，快速实现各种服务需求。

## 优点
功能完善、要素齐全：自带大量常用工具和框架（比如分页，auth，权限管理), 适合快速开发企业级网站。

完善的文档：经过十多年的发展和完善，Django有广泛的实践案例和完善的在线文档。开发者遇到问题时可以搜索在线文档寻求解决方案。

强大的数据库访问组件：Django的Model层自带数据库ORM组件，使得开发者无须学习SQL语言即可对数据库进行操作。

Django先进的App设计理念: App是可插拔的，是不可多得的思想。不需要了，可以直接删除，对系统整体影响不大。

自带台管理系统admin：只需要通过简单的几行配置和代码就可以实现一个完整的后台数据管理控制平台。

Django debug信息详尽: 很容易找出代码错误所在。

## 缺点
大包大揽: 对于一些轻量级应用不需要的功能模块Django也包括了，不如Flask轻便。

过度封装: 很多类和方法都封装了，直接使用比较简单，但改动起来就比较困难。

性能劣势: 与C, C++性能上相比，Django性能偏低，当然这是python的锅，其它python框架在流量上来后会有同样问题。

模板问题: django的模板实现了代码和样式完全分离，不允许模板里出现python代码，灵活度对某些程序员来说可能不够。

## MVC架构与MTV架构
#### 1.MVC架构
MVC架构是一种软件工程设计方法，分为三部分，Model数据模块、View视图模块和Controller控制器模块，通过这三个模块之间的配合，显示出用户想要的结果。

- Model：包含系统的数据内容，通常以数据库形式来存储。如果这些内容有变动，就会通知View试试更改显示内容，一些处理数据的程序逻辑也会放在这里。
- View：创建和用户之间的界面，把用户的请求传送给Controller，并按照Controller的要求把来自Model的数据显示出来。
- Controller：派发View传来的用户请求，并按照这些请求处理数据内容以及设置要显示的数据。

一张图解释MVC
 
 ![]

## 后记
参考：

[django的优缺点总结 - Python Web开发面试必备](https://blog.csdn.net/weixin_42134789/article/details/80753010)

[Django2.2教程](http://www.liujiangblog.com/course/django/)