---
layout:     post
title:      设计模式（一）单例模式
subtitle:   根据《Python设计模式（第2版）》一书的学习，记录下来的笔记。本篇博客是单例模式，是一种应用开发过程中最简单的一种创建型设计模式。
date:       2019-12-31
author:     HouKC
header-img: img/post-bg-coffee.jpeg
catalog:    true
tags:
    - 设计模式
    - Python
    - 笔记
---

## 前言
设计模式这一系列是我在看书籍《Python设计模式（第2版）》的读书笔记，记录一些重要的、关键的知识
点，主要内容都是出自书中。

为了防止以后想起来的时候忘记实现细节，或者一些关键的编程，我会尽量把笔记做好，也按照书的流程进
行梳理。

接下来简单介绍一下设计模式的内容。

## 1. 设计模式简介

#### 1.1 设计模式分三类

- 创建型设计模式
    - 运行机制**基于对象的创建方式**。
    - 将对象创建的细节隔离开来。
    - 代码与所创建的对象的类型无关。
- 结构型设计模式
    - 致力于设计出能够**通过组合获得更强大功能的对象和类的结构**。
    - 简化结构并识别类和对象之间的关系。
    - 主要关注类的继承和组合。
- 行为型设计模式
    - 关注对象之间的交互以及对象的响应性。
    - **对象应该能够交互**，同时仍然保持松散耦合。

#### 1.2 面向对象编程特性

1. 封装性

    - 对象的行为对于外部世界来说是不可见的，也就是**对象的状态是私密的**。
    - **客户端不能直接操作改变对象**内部状态，但是可以发送消息请求对象改变内部状态，比如一些成员函数。
    - python没有封装的关键字private、public或者protected等，但可以**在变量或函数前加__ 来变成私有**。

2. 多态性

    - 对象**根据输入参数提供方法的不同实现**。
    - **不同类型的对象可以使用相同的接口**。
    - python中，**多态是该语言的内置功能**，比如+号，可以整数相加，也可以字符串连接。

3. 继承性

    - **一个类可以继承父类的（大部分）功能**。
    - 继承被描述为一个**重用基类中定义的功能**，并**允许**对原始软件的实现进行独立**扩展**的选项。
    - 继承可以利用不同类的对象之间的关系建立层次结构，或者说**可以继承多个基类**。

4. 抽象性

    - 提供了一个简单的客户端接口，可以**通过该接口与类的对象进行交互**，并**可以调用该接口中定义的各个方法**。
    - 内部类的复杂性**抽象为一个接口**，这样**客户端就不需要知道内部实现**了。

5. 组合性

    - **一个对象可用于调用其他模块中的成员函数**，这样一来，无需通过继承就可以实现基本功能的跨模块使用。

贴段代码理解一下这句话：
```python
class A(object):
    def a1(self):
        print("a1")
class B(object):
    def b(self):
        print("b")
        A().a1()        # 注意这一行调用了A类中的函数，就是一种组合
objectB = B()
objectB.b()
```
#### 1.3 面向对象的设计原则

1. 开放/封闭原则
    
    **开放**类或对象及其方法的**扩展**，但是**封闭**对其**修改**。
    
    也就是说，类和方法在编写的时候要考虑通用性，甚至可以写一些抽象基类，通过继承来扩展功能。

2. 控制反转原则

    高层级的模块不应该依赖于低层级的模块，**两个层级的模块应该都依赖于抽象**。
    
    **细节依赖抽象**，抽象不应该依赖细节。
    
    消除模块之间的紧耦合，**尽量让每个模块独立**，模块之间可以用抽象层来解决依赖关系（如钩子）。
    模块的细节是基于通用的抽象类扩展的，而不是把细节直接写在抽象类中。

3. 接口隔离原则

    客户端不应该依赖于它们不需要使用的接口。
    
    换言之，其实就是**不该属于该类的方法**，我们就**没必要写**，尽量精简。如Pizza接口不应
    该提供名为add_chicken()方法，因为基于Pizza接口的Veg Pizza类（蔬菜披萨）不应该强制实现该方法。

4. 单一职责原则
    
    **类的职责单一**，引起类变化的原因单一。
    
    如果一个类实现了两个功能，那么最好将它们分开。换言之，**模块尽量功能单一，减少依赖**。
    
5. 替换原则

    **派生类必须能够完全取代基类**。就是说派生类要替换基类，对基类封闭，扩展基类。

#### 1.4 设计模式的定义和优点
1. 定义

    设计模式是解决特定问题的解决方案。
    
    它与使用的语言无关，而且是动态的、可定制的，随时会有新的模式引入。
    
    本质上设计模式就是从别人的成功进行学习。

2. 优点

    可以在多个项目中重复使用。
    
    问题可以在架构级别解决。
    
    设计模式是经过了时间的验证和良好的证明，是开发人员的宝贵经验。

## 2. 单例设计模式
#### 2.1 定义
单例设计模式是一种创建型设计模式，提供了一个机制，即确保**有且仅有一个特定类型的对象**，并提供**全局访问点**。

可以用来解决日志记录或数据库操作、打印机后台处理程序等。

- 有且仅有一个对象被创建。
- 为对象提供一个访问点，使程序可以全局访问该对象。
- 控制共享资源的并行访问。

#### 2.2 经典单例模式
**使构造函数私有化**，并创建一个静态方法来完成对象的初始化。

经典的单例模式包含两件事：一是只允许生成一个实例，二是如果有实例了，就重用那个实例。
```python
class Singleton(object):
    def __new__(cls):
        if not hasattr(cls, 'instance'):
            cls.instance = super().__new__(cls)
        return cls.instance
s = Singleton()
print("Object created", s)
s1 = Singleton()
print("Object created", s1)
```
输出结果：
```sh
<__main__.Singleton object at 0x00000000021A9588>
<__main__.Singleton object at 0x00000000021A9588>
```
其中hasattr函数用于检测对象（第1个对象）是否含有某个属性（第2个参数），'instance'属性是检测该类是否生成了一个对象。

#### 2.3 单例模式中的懒汉式实例化
确保在实际需要时才创建对象。
```python
class Singleton:
    __instance = None

    def __init__(self):
        if not Singleton.__instance:
            print("__init__ method called..")
        else:
            print("Instance already created:", self.getInstance())

    @classmethod
    def getInstance(cls):
        if not cls.__instance:
            cls.__instance = Singleton()
        return cls.__instance


s = Singleton()
print("Object created", Singleton.getInstance())
s1 = Singleton()
```
cls表示类本身，@classmethod指此函数属于类而不属于instance。

上述代码在s=Singleton()时，会调用初始化init方法，但不会创建新对象，而是
在Singleton.getInstance()时才创建的。

#### 2.4 模块级别的单例模式
**python默认所有的模块都是单例模式**，判断是否已经导入，如果导入了就返回该模块的对象，
否则实例化该模块。**一个模块有且仅有一个对象**。

#### 2.5 Monostate单例模式
Monostate应该成为**单态模式**，因为它强调的是状态和行为，即所有对象共享相同的状态。
```python
class Borg:
    __shared_state = {"1": "2"}

    def __init__(self):
        self.x = 1
        self.__dict__ = self.__shared_state
        pass


b = Borg()
b1 = Borg()
b.x = 4
print("Borg Object 'b': ", b)
print("Borg Object 'b1': ", b1)
print("Object State 'b1':", b1.__dict__)
```
\_\_dict\_\_存储一个类所有对象的状态，我们故意把\_\_shared_state赋给所有已经创建的实例，
这样实例b和b1的状态是相同的，也就是说，就算b的对象变量x发生了变化，b1也会共享状态，x的值
由1变4。

此外，new方法也可以产生Monostate模式的类：
```python
class Borg:
    __shared_state = {}
    def __new__(cls, *args, **kwargs):
        obj = super().__new__(cls, *args, **kwargs)
        obj.__dict__ = cls.__shared_state
        return obj
b = Borg()
b1 = Borg()
print("Borg Object 'b': ", b)
print("Borg Object 'b1': ", b1)
print("Object State 'b1':", b1.__dict__)
```
我的理解是，在写类的时候，如果给\_\_dict\_\_属性赋值，那么不管是用new方法还是init方法
都会共享状态。

#### 2.6 单例和元类
先说一下元类。

**元类是一个类的类**，也就是一个类是它的元类的实例化。

python的自带的元类就是type，可以通过A=type(name, bases, dict)创建它。其中name是类的
名称，bases是基类，dict是属性变量。
```python
class MyInt(type):
    def __call__(cls, *args, **kwargs):
        print("*****Here's My int *****", args)
        print("Now do whatever you want with these objects...")
        return type.__call__(cls, *args, **kwargs)


class int(metaclass=MyInt):
    def __init__(self, x, y):
        self.x = x
        self.y = y


i = int(4, 5)
```
就是说先创建一个类继承type元类，然后在int类中用metaclass继承，这样MyInt这个类也是
一个元类，而int类就是它的实例。

当我们使用int(4，5)实例化int类时，MyInt元类中的call方法会被调用，也就是元类控制着
对象的实例化。

为了控制类的创建和初始化，元类将覆盖\_\_new\_\_和\_\_init\_\_方法。

基于元类实现单例模式：
```python

```


## 后记
待完成。