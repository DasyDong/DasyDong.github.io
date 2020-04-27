---
title: "Python中的abc模块"
layout: post
date: 2017-10-13 08:18
image: /img/images/markdown.jpg
headerImage: false
tag:
- python
category: blog
author: dong
description: Markdown summary with different options
# jemoji: '<img class="emoji" title=":ramen:" alt=":ramen:" src="https://assets.github.com/images/icons/emoji/unicode/1f35c.png" height="20" width="20" align="absmiddle">'
---

- [介绍](#介绍)
- [应用场景](#应用场景)
    - [我们去检查某个类是否有某种方法](#我们去检查某个类是否有某种方法)
    - [实现ABC类](#实现abc类)
    - [我们在某些情况之下希望判定某个对象的类型](#我们在某些情况之下希望判定某个对象的类型)
- [Python2&3的兼容问题](#python23的兼容问题)

## 介绍
 ABC，Abstract Base Class（抽象基类），主要定义了基本类和最基本的抽象方法，可以为子类定义共有的API，不需要具体实现。相当于是Java中的接口或者是抽象类。
    抽象基类可以不实现具体的方法（当然也可以实现，只不过子类如果想调用抽象基类中定义的方法需要使用super()）而是将其留给派生类实现。

抽象基类提供了逻辑和实现解耦的能力，即在不同的模块中通过抽象基类来调用，可以用最精简的方式展示出代码之间的逻辑关系，让模块之间的依赖清晰简单。同时，一个抽象类可以有多个实现，让系统的运转更加灵活。而针对抽象类的编程，让每个人可以关注当前抽象类，只关注其方法和描述，而不需要考虑过多的其他逻辑，这对协同开发有很大意义。极简版的抽象类实现，也让代码可读性更高。

抽象基类的使用：

1：直接继承
    直接继承抽象基类的子类就没有这么灵活，抽象基类中可以声明”抽象方法“和“抽象属性”，只有完全覆写（实现）了抽象基类中的“抽象”内容后，才能被实例化，而虚拟子类则不受此影响。

2：虚拟子类
将其他的类”注册“到抽象基类下当虚拟子类（调用register方法），虚拟子类的好处是你实现的第三方子类不需要直接继承自基类，可以实现抽象基类中的部分API接口，也可以根本不实现，但是issubclass(), issubinstance()进行判断时仍然返回真值。

Python 对于ABC的支持模块是abc模块，定义了一个特殊的metaclass：ABCMeta 还有一些装饰器：@abstractmethod 和 @abstarctproperty 。abc.ABCMeta 用于在Python程序中创建抽象基类。而抽象基类如果想要声明“抽象方法”，可以使用 @abstractmethod ，如果想声明“抽象属性”，可以使用 @abstractproperty 。

## 应用场景
### 我们去检查某个类是否有某种方法
```
from collections.abc import Sized

class A(object):
    def __len__(self):
        pass

if __name__ == "__main__":
    a = A()
    print("存在__len__方法" if isinstance(a, Sized) else "没有__len__方法")

# 输出：
存在__len__方法
```

### 实现ABC类
```
import abc

class A(metaclass=abc.ABCMeta):
	# 利用装饰器修饰greet()
    @abc.abstractmethod
    def greet(self):
        print("hell world")

if __name__ == "__main__":
    a = A()
```
解释器如期抛错：
```
TypeError: Can't instantiate abstract class A with abstract methods greet
```
这是因为A类现在就是一个抽象基类了，不可以被实例化，同时，它的子类还必须实现greet()方法，否则实例化子类时解释器也要报错

### 我们在某些情况之下希望判定某个对象的类型
```
from collections.abc import Sized
isinstance(com, Sized)

b = B()
print(isinstance(b, A))
```


## Python2&3的兼容问题
为了解决Python2&3的兼容问题，需要引入six模块，该模块中有一个针对类的装饰器 @six.add_metaclass(MetaClass) 可以为两个版本的Python类方便地添加metaclass

 Des | Code |
:--- | :---  |
通用做法。<br> @six.add_metaclass(MetaClass) 的作用是在不同版本的Python之间提供一个优雅的声明类的metaclass的手段 <br> 事实上不用它也可以，只是使用了它代码更为整洁明了 | import six <br> @six.add_metaclass(Meta) <br> class MyClass(object): <br>       pass <br> |
在Python 3 等价于 | import six  <br>class MyClass(object, metaclass = Meta): <br>  pass |
在Python 2.x (x >= 6)中等价于 | import six <br> class MyClass(object):  <br> __metaclass__ = Meta <br> pass |

实例
```
import abc
import six


@six.add_metaclass(abc.ABCMeta)
class BaseClass(object):
    @abc.abstractmethod
    def func_a(self, data):
        """
        an abstract method need to be implemented
        """

    @abc.abstractmethod
    def func_b(self, data):
        """
        another abstract method need to be implemented
        """

class SubclassImpl(BaseClass):
    def func_a(self, data):
        print("Overriding func_a, " + str(data))

    @staticmethod
    def func_d(self, data):
        print(type(self) + str(data))

class RegisteredImpl(object):
    @staticmethod
    def func_c(data):
        print("Method in third-party class, " + str(data))
BaseClass.register(RegisteredImpl)


if __name__ == '__main__':
    for subclass in BaseClass.__subclasses__():
        print("subclass of BaseClass: " + subclass.__name__)
    print("subclass do not contains RegisteredImpl")
    print("-----------------------------------------------")

    print("RegisteredImpl is subclass: " + str(issubclass(RegisteredImpl, BaseClass)))
    print("RegisteredImpl object  is instance: " + str(isinstance(RegisteredImpl(), BaseClass)))
    print("SubclassImpl is subclass: " + str(issubclass(SubclassImpl, BaseClass)))

    print("-----------------------------------------------")
    obj1 = RegisteredImpl()
    obj1.func_c("RegisteredImpl new object OK!")
    print("-----------------------------------------------")
    obj2 = SubclassImpl()  #由于没有实例化所有的方法，所以这里会报错 Can't instantiate abstract class SubclassImpl with abstract methods func_b
    obj2.func_a("It's right!")
```

```
结果如下：
subclass of BaseClass: SubclassImpl
subclass do not contains RegisteredImpl
-----------------------------------------------
RegisteredImpl is subclass: True
RegisteredImpl object  is instance: True
SubclassImpl is subclass: True
-----------------------------------------------
Method in third-party class, RegisteredImpl new object OK!
-----------------------------------------------
Traceback (most recent call last):
  File "/Users/wangqi/Git/Python/scrapy_crawler_learn/test/ABCTest.py", line 51, in <module>
    obj2 = SubclassImpl()  #由于没有实例化所有的方法，所以这里会报错 Can't instantiate abstract class SubclassImpl with abstract methods func_b
TypeError: Can't instantiate abstract class SubclassImpl with abstract methods func_b
```
