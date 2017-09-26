---
title: Python 奇技淫巧 (三) 函数
date: 2017-01-21 21:52
tags:
  - Python
  - function
---


> ##### 文章中的代码仅在Python3中测试成功，没有在Python2中测试。

# 0X00 任意个参数
Python中一般定义函数是这样的`def add(a, b)`，参数的个数是固定的，那么怎么才可以接收任意多个参数就像`rm 1.txt 2.jpg 3.mp3 4.cpp`这样？很简单，使用`*`和`**`就可以。下面代码里第一个参数a接收到了`hello,world`而`*b`则接收到了其余所有的参数，将其作为一个元组。
```Python
#!/usr/bin/python
# coding=utf-8


def add(a, *b):
    print(a)
    return b


if __name__ == '__main__':
    x = add('hello,world', 2, 3, 4, 5)
    print(x)
```

# 0X01 添加注解
在Python中定义函数的同时可以也给函数添加注解，注解可以帮助我们在调用函数的时候起到一个提醒的作用。虽然几十行的代码不会遇到看不懂的情况，但是在修改别人代码或者编写一个大项目的时候必然会有这种问题。我们可以直接在代码中加注释来解释说明，但是使用注解还是要比注释来得简单方便。不过通过注解注解指定的类型不像是C语言那样有实际意义，就算是你传入的参数和返回的值不是按照注解来的也不会报错。
```Python
#!/usr/bin/python
# coding=utf-8


def add(a: int, b: int) -> int:	# 这里声明了a和b都是int型，返回值也是int型
    return a + b

if __name__ == '__main__':
    print(add(3, 5))
```

# 0X02 默认参数
我们常用的一些内置函数是有好多个可选参数的，不过我们不需要每个参数都要传入，因为Python可以给参数设置默认值，如果没有传入那个参数就会选择使用默认值，比如下面这个`add`函数。
```Python
#!/usr/bin/python
# coding=utf-8


def add(a = 3, b = 5):
    return a + b


if __name__ == '__main__':
    print(add())		# 没有任何参数，默认使用３和５，最后结果则为8
    print(add(1))		# 传入了参数a为1，最后结果则为6
    print(add(4, 6))	# 传入了参数a和b分别为4和6，最后结果为10
	print(add(b = 3))	# 值传入了b参数为3，所以最后结果为6
	print(add(b = 3, a = 10))	# 指定参数的话也可以不按顺序
```

# 0X03 函数mini 匿名函数
这里称之为匿名函数感觉还是有点别扭，因为这儿定义的函数并不是真的匿名，也是有名字的，因为函数自身非常短小倒不如称之为函数mini。在Python中有一个关键字`lambda`，可以定义一个匿名函数，使用这个关键字定义函数的时候函数声明、返回值、函数体只能写成一行。这样的函数功能肯定不能很强大，不过确实能减少代码量，少写好多重复的代码。正式代码的第一行就定义了一个函数，名为add，参数是x和y，返回值是x+y。所以说标准是这样的`函数名 = lambda 参数 ： 返回值`。这里还有个例子：`my_sqrt = lambda x : math.sqrt(x)`。注意，在匿名函数里什么`if-else`、`while`、`try-except`都是不能用的，总之你的函数就只能写一行。
```Python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    add = lambda x, y : x + y
    print(add(3, 5))
    print(add(2, 7))
    print(add(1, 9))
```

