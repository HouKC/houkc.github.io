---
layout:     post
title:      设计模式（二）工厂模式
subtitle:   根据《Python设计模式（第2版）》一书的学习，记录下来的笔记。本篇博客是工厂模式，是一种创建型模式。
date:       2020-01-07
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - 设计模式
    - Python
    - 笔记
---

## 前言
感觉工厂模式比起单例模式要复杂一些，得花点时间理解才行，本章主要包含了三个部分：简单工厂
模式、工厂方法模式和抽象工厂模式。在最后会做一下简单的对比。

此外，本章python代码实现时，都会把提炼抽象类和抽象方法，这样逻辑会更加统一。

## 工厂模式
#### 1. 工厂模式简介
所谓“工厂”就是一个**负责创建其他类型对象的类**。

客户端输入或者设置某些参数，让这个“工厂”产生客户所需要的产品，返回给客户端。

那么问题来了，客户端是可以直接创建对象，也就是说它可以直接选择自己生产产品，为什么还需要
一个工厂呢？

原因如下：
- **松耦合，对象的创建可以独立于类的实现**（类的实现都隐藏在工厂背后的接口中，我们只需要
修改参数让工厂输出某个类的对象即可）。
- 客户端无需了解创建对象的类，但是照样可以使用它来创建对象，只需要修改参数、方法等即可，
这样就**简化了客户端的实现**。
- 可以轻松地在工厂中**添加其他类来创建其他类型的对象，无需修改客户端代码**。客户端只需要
传递另一个参数即可。
- 工厂还可以重用现有对象。但是，如果客户端直接创建对象的话，总是创建一个新对象。

举个例子：

讨论一个制造玩具车或玩偶的公司，假设公司里有一台制造玩具车的机器，领导后来想用它来造玩偶。
这时按照工厂模式就是，机器是接口，领导是客户端，领导只关心制造的对象（玩具）和创建对象的
接口（机器）。

#### 2. 三种工厂模式对比
简单工厂模式|工厂方法模式|抽象工厂模式
---|---|---
允许接口创建对象，但不会暴露对象的创建逻辑。|允许接口创建对象，但使用哪个类来创建对象，则是交由子类决定的。|一个能创建一系列相关的对象而无需指定/公开其具体类的接口，该模式能够提供其他工厂的对象，在其内部创建其他对象。

#### 3. 简单工厂模式
想象一下森林里，有动物，有猫有狗，猫和狗都有叫声，我们现在只要输入一个参数：猫，森林工厂就输出猫叫；狗，森林工厂就输出狗叫。

也就是，森林是工厂，动物的叫声是产品，我们可以把动物设置成类，动物这个类有一个输出叫声的接口，而猫和狗继承自动物这个类；
接着森林工厂可以调用这些接口，只需要我们控制好参数，比如输入参数：猫或者狗。

```python
from abc import ABCMeta, abstractmethod


class Animal(metaclass=ABCMeta):
    @abstractmethod
    def do_say(self):
        pass


class Dog(Animal):
    def do_say(self):
        print("Bhow Bhow!!")


class Cat(Animal):
    def do_say(self):
        print("Meow Meow!!")


# forest factory defined
class ForestFactory(object):
    def make_sound(self, object_type):
        return eval(object_type)().do_say()


# client code
if __name__ == '__main__':
    ff = ForestFactory()
    animal = input("Which animal should make sound Dog or Cat?")
    ff.make_sound(animal)

```

#### 4. 工厂方法模式
工厂方法模式与简单工厂模式最直接的区别就是，不直接创建对象，而是定义了一个接口，让子类来完成。

工厂方法使设计更加具有可定制性。它可以返回相同的实例或子类，而不是某种类型的对象。

举个例子：假设我们想在不同类型的社交网络（LinkedIn、Facebook）建立个人简介，在LinkedIn上有
关于个人申请的专利或作品的区，在Facebook上有显示度假地点的照片区。此外，两者都有个人信息区。

那么我们就可以先抽象一个类作为通用产品类，并提供一个产品接口，用来定义我们的区，然后写各种
子类来实现不同的接口。

接着创建一个抽象工厂类，里面提供了一个生产产品的方法，这个方法通过子类继承来实现，子类会根
据不同参数调用不同接口来生产不同产品。
```python
from abc import ABCMeta, abstractmethod


class Section(metaclass=ABCMeta):
    @abstractmethod
    def describe(self):
        pass


class PersonalSection(Section):
    def describe(self):
        print("Personal Section")


class AlbumSection(Section):
    def describe(self):
        print("Album Section")


class PatentSection(Section):
    def describe(self):
        print("Patent Section")


class PublicationSection(Section):
    def describe(self):
        print("Publication Section")


class Profile(metaclass=ABCMeta):
    def __init__(self):
        self.sections = []
        self.createProfile()

    @abstractmethod
    def createProfile(self):
        pass

    def getSections(self):
        return self.sections

    def addSections(self, section):
        self.sections.append(section)


class linkedin(Profile):
    def createProfile(self):
        self.addSections(PersonalSection())
        self.addSections(PatentSection())
        self.addSections(PublicationSection())


class facebook(Profile):
    def createProfile(self):
        self.addSections(PersonalSection())
        self.addSections(AlbumSection())


if __name__ == '__main__':
    profile_type = input("Which Profile you'd like to create?[LinkedIn or Facebook]")
    profile = eval(profile_type.lower())()
    print("Creating Profile..", type(profile).__name__)
    print("Profile has sections --", profile.getSections())
```
上述代码中，我们首先创建了Section抽象类，描述产品接口，也就是通用区，接着通过继承Section类，
写了PersonalSection等几个不同的接口类，并提供了接口方法describe，当然具体功能没有实现，只是
随便打印了点东西。

然后建立工厂抽象类Profile，并提供初始化方法和createProfile生产产品方法，以及addSection添加
接口和getSections获取接口已添加接口的方法。通过继承这个类，我们编写了linkedin和facebook两个
子类，我们输入参数linkedin或者facebook来实例化这两个子类即可生成我们需要的产品：带有不同区域
的linkedin页面和facebook页面。

而添加接口的过程我们已经写在工厂抽象类的初始化中了，所以所有子类工厂都会在实例化的时候直接初
始化相关接口来实例化。不过我们可以在子类的实现中去决定需要添加哪些接口（createProfile方法）。

这样一来，**客户端完全不需要关心内部要传递哪些参数以及需要实例化哪些类。由于添加新类更加容
易，所以降低了维护成本**。

#### 5. 抽象工厂模式


## 后记