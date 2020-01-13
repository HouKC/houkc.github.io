---
layout:     post
title:      设计模式（四）代理模式
subtitle:   根据《Python设计模式（第2版）》一书的学习，记录下来的笔记。本篇博客是代理模式，是一种结构型模式。
date:       2020-01-13
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - 设计模式
    - Python
    - 笔记
---

## 前言
本章介绍一下代理模式，代理模式也是一种结构型设计模式。

先从代理开始理解吧。代理通常是介于寻求方和提供方之间的中介系统。寻求方是发出请求的，提供方是根据请求提供资源的。

在web世界中，代理相当于代理服务器。当客户端访问网站时，首先连接到代理服务器，向它请求资源，
代理服务器在内部评估此请求，将其发送到适当的服务器，当它收到响应后，就会将响应传递给客户端。

## 1. 代理模式定义
在设计模式中，单例是充当实际对象接口的类。对象类型可以多样化，如网络连接、内容、文件等。

代理模式的主要目的是为其他对象提供一个代理者或占位符，从而控制对实际对象的访问。

## 2. 代理模式场景
- 以简单的方式表示一个系统，如提供一个简单的接口充当客户端代理。
- 提高安全性。
- 为不同服务器上的远程对象提供本地接口，如客户端想在远程系统上运行某些命令。
- 为消耗大量内存的对象提供了一个轻量级的句柄，如个人简介头像，很大，可以先用代理预显示一个缩略图。

举个例子：制作公司想找演员拍戏，通常是跟经纪人交流，而不是跟演员交流。经纪人根据演员的日程
安排和其他合约情况，来答复制作公司该演员是否有空，以及是否对该戏感兴趣。制作公司就是客户端，
经纪人是代理，演员是资源。
```python
class Actor(object):
    def __init__(self):
        self.isBusy = False

    def occupied(self):
        self.isBusy = True
        print(type(self).__name__, "is occupied with current movie")

    def available(self):
        self.isBusy = False
        print(type(self).__name__, "is free for the movie")

    def getStatus(self):
        return self.isBusy


class Agent(object):
    def __init__(self):
        self.principal = None

    def work(self):
        self.actor = Actor()
        if self.actor.getStatus():
            self.actor.occupied()
        else:
            self.actor.available()


if __name__ == "__main__":
    r = Agent()
    r.work()
```
代理设计模式实现了对原始对象的访问控制。

它还可以用作一个层或接口，支持分布式访问。

通过增加代理，保护真正的组件不受意外的影响。


## 后记