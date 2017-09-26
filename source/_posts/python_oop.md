---
title: Python之面向对象
date: 2016-09-14 21:53
tags:
  - Python
  - OOP
---


# 0X00 创建一个类
Python也是一个和C++、Java一样的面向对象编程语言，所以Python里也有类和对象。
```python
class Person:   #这是一个类
    def sayHello(self):     #这是一个方法
        print 'hello,world'

    def setName(self, inputName):
        self.name = inputName

    def getName(self):
        return self.getName()
```
在类中创建的方法使用def关键字定义，每个方法有一个或以上的参数，selft就是实例化的对象自己。需要返回值就return一个，不需要就可以不写return
> Python的类和Java的类还是有点区别，Java的类里主要写的是属性和方法，Python里不写属性，因为Java的变量需要定义而Python的变量并不需要定义，最多也就是在前面个各个属性一个变量名并赋初值

# 0X01 实例化一个对象
类是一个很抽象的概念，可以由类实例化好多个对象出来。Java中我们习惯说成 new一个对象，而Python中并不需要new
```python
xiaoming = Person() #实例化了一个类
xiaoming.setName('xiaoming')  #调用一个方法
print xiaoming.getName()
xiaoming.sayHello()
```
> 此段代码接着上面的类声明

# 0X02 Python的私有
Java和其他好多面向对象编程语言中会有一个private关键字，将属性和方法约束为私有的。然而Python并不能直接支持private，间接地支持也不是真正的private。在Python中的private由一种诡异的方式模拟，在方法前加上连续的两个下划线
```python
class Person:
    def __hello(self):
        return 'hello,world'

    def sayHello(self):
        print self.__hello()

xiaoming = Person()
xiaoming.sayHello()
xiaoming.__hello()
```
运行起来就是这样的，前面的`xiaoming.sayHello()`因为是可以直接调用的，然后在方法里调用了私有的`__hello()`方法，所以可以正常执行；后面的`xiaoming.__hello()`因为不能直接调用，所以报错了。
```bash
hello,world
Traceback (most recent call last):
  File "./hello.py", line 12, in <module>
    xiaoming.__hello()
AttributeError: Person instance has no attribute '__hello'
```
其实在方法前面加了两个下划线并没有真的把方法改了个名字而已，改成了`_Class__name`的类型，一个下划线+类名+两个下划线+方法名，当我们知道了这个问题之后就可以这样调用‘私有’方法了，但是既然我们都将其设为了’私有’就不要这么用了。
```python
>>> xiaoming = Person()
>>> print xiaoming._Person__hello() #不要这样做，虽然可行
    hello,world
```

# 0X03 继承
面向对象编程的特性之一：继承。Python中的继承和Java的语法差异还是挺大的，像下面这样可以声明两个类，让 一个类继承自另一个
```python
#!/usr/bin/python

class Person:

    def sayHello(self):
        print "hello, I'm person"

class Jack:

    def sayHello(self):
        print "helo, I'm Jack"

person = Person()
jack   = Jack()

person.sayHello()
jack.sayHello()
```
输出是这样的
```bash
hello, I'm person
helo, I'm Jack
```
这也体现了面向对象的覆盖的思想

# 0X04 多继承
一个类可以继承自另一个类，也可以继承自其他好多个类。
```python
class Student:  #一个父类
    def learn(self):
        print "i'can learn"


class Coder:    #另一个父类
    def programming(self):
        print "I'can programming"

class Jack(Student, Coder): #继承自两个雷的子类
    pass

jack = Jack()

#可以调用两个父类中的方法
jack.learn()
jack.programming()
```
> 书写多继承的时候要注意一个问题，如果某类继承自两个类，且那两个类有相同的方法，那么就会造成覆盖（重写）
> 是这样一个情况，如果我上面的Student和Coder类都有一个名为eat的方法，但是Student类里的eat方法是输出‘student can eat’但是‘Coder’类中eat方法是输出’coder can eat’，那么在写子类的时候就要格外小心
> 如果子类这样写`class Jack(Student, Coder)`那么这个子类的ear方法就会是’Coder‘’中的方法，输出‘Coder can eat’，如果这样写`class Jack(Coder, Student)`的话，输出就是’Student can eat‘’

# 0X05 构造方法
我们实例化一个对象的时候是这样做的`xiaoming = Person()`其内部是调用了Person类的构造方法并返回给了xiaoming这个变量。在Java中在java中构造方法是一个没有返回类型的与类名同名的方法，但是在Python中所有构造方法都叫`__init__()`。自己不手动写构造方法的话Python就会自动生成一个，这点和Java相同。当然我们也可以自己手写构造方法
```python
class Person:
    def __init__(self):
        print 'new a person'

xiaoming = Person()
```
会输出一个`new a person` 这就能证明确实在实例化对象的时候回先调用这个类的构造方法。构造方法可以加参数，就像下面这样
```python
class Person:
    def __init__(self, say='new a person'):
        print say
```
如果我实例化对象的时候这么写`xiaoming = Person()`那么就会输出一个'new a person'的字符串，因为我没传入参数，所以就使用了默认值，但是如果我这样实例化对象`xiaohong = Person('xiaohong')`就会输出一个'xiaohong'的字符串，因为我给构造方法传入参数了。
在Python中还有一个叫析构方法的`__del__`但是我们最好不要去碰它，因为Python里有像Java类似的自动垃圾回收，所以几乎不会需要我们自己去析构一个对象，但是如果真的有需要，也可以像C++一样手动析构

# 0X06 方法的重写
如果有一个类A，A中有一个sayHello的方法，然后有一个B类继承了A类，那么自然就也有了sayHello的方法，但是如果我们给B类单独设定sayHello方法会怎么样呢
```python
class Person:
    def sayHello(self):
        print "i'm Person"

class Man(Person):
    def sayHello(self):
        print "i'm Man"

xiaoming = Man()
xiaoming.sayHello()
```
输出的是"i'm Man"而不是"i'm Person"。这就是方法的重写，子类中写了一个和父类中相同名的方法，就会把已经继承过来的方法重写掉


