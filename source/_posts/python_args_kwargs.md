---
title: Python中的*args和**kwargs
date: 2017-09-05 22:54:55
tags:
    - Python
---

# 0X00 \*args是什么
我们知道Python3中的print从一个关键字变成了一个函数，那么调用的时候我们可以这样调用这个函数，可以随便接受几个参数。
```python
>>> print(1)
1
>>> print(1, 2, 3)
1 2 3
>>> print(1, "hello", 6.66)
1 hello 6.66
```
那么如果我们想自己实现类似这样‘变态’的函数该怎么实现呢？这就需要用到\*args了，可以将一个非键值对的可变数量的参数列表传给一个函数（换个书佛啊：可以传n个参数给函数，而且n不是固定的），举个例子就容易理解多了。
```python
def say_something(*args):
    for i in args:
        print i
    print '--------'

say_something(1)
say_something(1, 2, 3)
say_something('hello')
say_something('hello', 'world')
```
运行这个例子的输出就是这样的
```bash
1
--------
1
2
3
--------
hello
--------
hello
world
--------
```

还有一个更棒的例子[来自Gitbook](https://eastlakeside.gitbooks.io/interpy-zh/content/args_kwargs/Usage_args.html)
```python
def test_var_args(f_arg, *args):
    print("first normal arg:", f_arg)
    for arg in args:
        print("another arg through *args:", arg)

test_var_args('yasoob', 'python', 'eggs', 'test')
```
输出是这样的
```bash
('first normal arg:', 'yasoob')
('another arg through *args:', 'python')
('another arg through *args:', 'eggs')
('another arg through *args:', 'test')
```
这个例子完整的说明了`\*args`的用法，我们传入的第一个参数被函数指定的`f_arg`接收到了，其余的都被`*args`接收到了。


# 0X01 \*\*kwargs是什么
写代码的时候还会有一种函数调用，大概是这个样子`json.dumps(dict_data)`和`json.dumps(dict_data, indent=4)`。当然，实现这种的方式有一个最简单的方案就是`def dumps(input_data, indent=0)`。在可选参数只有一两个的时候这种方式固然是好用的，但是如果像是requests这种库中的常用方法，有很多很多个可选参数那就该用上这个\*\*kwargs了。顾名思义这个就是`keyworkargs`的意思，也就是说是带有key的可变参数。可以这样定义一个函数
```python
def foo(**kwargs):
    for key in kwargs:
        print key
        print kwargs[key]
        print '-----'

foo(a=1, b=2, c=3, d=4, e=5)
```
运行出来的结果可想而知：
```bash
a
1
-----
c
3
-----
b
2
-----
e
5
-----
d
4
-----
```

# 0X02 合在一起怎么用
值得一提的是如何把这两个放在一起用，这里列举个例子来演示一下
```python
#!/usr/bin/env python
# coding=utf-8


def foo(name, sex, *args, **kwargs):
    print 'name is ', name
    print 'sex is ', sex
    print 'other is ', args
    for key in kwargs:
        print key, ' is ', kwargs[key]


def bar(*args, **kwargs):
    print 'args is ', args
    print 'kwargs is ', kwargs


foo('shawn', '???', 'hello', 'world', hobby='computer', number=666)
print '--------------------------'
bar('shawn', '???', 'hello', 'world', hobby='computer', number=666)
```
输出结果是这样的
```bash
name is  shawn
sex is  ???
other is  ('hello', 'world')
hobby  is  computer
number  is  666
--------------------------
args is  ('shawn', '???', 'hello', 'world')
kwargs is  {'hobby': 'computer', 'number': 666}
```

> 这里有需要注意的一点：参数的名字不一定非要是`*args`和`**kwargs`，所以我们定义函数的时候不一定是`def foo(*args, **kwargs):`，也同样可以定义成`def bar(*hehe, **haha):`，这里真正标识的是星号而不是名字。不过建议命名的时候符合大家的习惯。
