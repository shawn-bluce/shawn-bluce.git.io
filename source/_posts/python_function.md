---
title: Python的实例方法、静态方法、类方法
date: 2017-01-01 20:34
tags:
  - Python
  - 实例方法
  - 静态方法
  - 类方法
  - OOP
---

# 0X00 实例方法
Python中的实例方法是在面向对象编程中用到的最多的方法类型了。 **实例方法** 从字面理解就可以，就是说这个方法是属于实例的。每次实例化一个对象出来，这个对象都会拥有这个方法。从下面代码中就可以看得出来，这里我定义了一个实例方法'get_name()'，定义实例方法不需要任何特殊的修饰符。
```Python
#!/usr/bin/python
# coding=utf-8

class Student:
    def __init__(self):
        self.name = None

    # 一个实例方法
    def get_name(self):
        return self.name

if __name__ == '__main__':
    a = Student()
    a.name = '小明'
    print a.get_name()
    b = Student()
    b.name = '小红'
    print b.get_name()
```
从运行结果可以看出来，针对每一个实例，调用实例方法的输出是不同的，也就可以证明这个方法是属于某个实例的。
```bash
小明
小红
```

# 0X01 静态方法
静态方法用的也很多，比如我们写正则表达式的时候经常会用到表达式的编译，一般都是这么写的're.compile()'这里就是一个静态方法。可以看到我们在调用编译方法的时候是并没有实例化一个re对象的。所以可以知道 **静态方法** 就是不需要实例化对象即可调用的方法。下面有一个例子，例子中还是上面的Student类，但是定义了一个静态方法'say_hello()'，因为这是一个静态方法，所以不需要实例化对象即可调用。正因为这些特点，在定义静态方法的时候没有一个默认的参数self。
```Python
#!/usr/bin/python
# coding=utf-8


class Student:
    def __init__(self):
        self.name = None

    def get_name(self):
        return self.name

    # 这里使用装饰器定义了一个静态方法
    @staticmethod
    def say_hello():
        print 'hello,world'

if __name__ == '__main__':
    Student.say_hello()
```
运行结果如下。我个人觉得静态方法的最大作用就是实现一些工具类，比如某些固定的重复的操作之类的。
```bash
hello,world
```

# 0X02 类方法
使用类方法需要弄清楚类中属性的种类。类里有两种属性，一种是 **类属性** 一种是 **实例属性** 。顾名思义，类属性就是说这个属性是属于类的，这个类的所有实例共享着一个属性。实例属性就是属于实例的属性，每个实例的实例属性间不共享。下面这个例子里可以看到，类属性和实例属性是可以重名的，但是调用的时候要用'cls'或者'self'来制定到底调用的是哪个属性。下面的例子中有一个类属性'name'和一个实例属性'name'。
```python
#!/usr/bin/python
# coding=utf-8


class Student:
    # 声明并赋值了一个类属性
    name = '默认的类属性'

    def __init__(self):
        self.name = '默认的对象属性'

    @classmethod
    def set_class_name(cls):
        cls.name = '修改的对象属性'

    def set_object_name(self):
        self.name = '修改的对象属性'

    @classmethod
    def get_class_name(cls):
        return cls.name

    def get_object_name(self):
        return self.name

if __name__ == '__main__':
    a = Student()
    print a.get_class_name()
    print a.get_object_name()
    a.set_class_name()
    a.set_object_name()
    print a.get_class_name()
    print a.get_object_name()
```
通过运行结果可以清晰地看出类属性的使用方式。运行结果如下。首先实例化一个对象，获取了a的实例属性和Student的类属性，然后调用了实例方法和类方法对两个属性重新赋值，最后再输出一次。
```bash
默认的类属性
默认的对象属性
修改的对象属性
修改的对象属性
```
