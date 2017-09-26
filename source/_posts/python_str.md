---
title: Python之字符串
date: 2016-09-05 10:12
tags:
  - Python
  - String
---

# 0X00 如何定义一个字符串
**Python不需要定义**其实就是这样的。。在Python中的字符串通常这么写
```python
>>> str1 = 'hello,world'
>>> str2 = "It's work"
>>> str3 = """ Jack:"oh shit" """
```
str1 的声明方式是最普通的声明方式
str2 的声明方式可以在字符串中存在单引号‘
str3 的声明方式可以在字符串中存在双引号 “ 还能在字符串中换行

# 0X01 拼接字符串
```python
>>> str1 = "hello"
>>> str2 = ","
>>> str3 = "world"
>>> print str1 + str2 + str3
	hello,world
```
注意：
连接的时候加号左右都要是字符串，如果是字符串加数字就不行了。除非把数字转成字符串格式

# 0X02 输入字符串
标准输入就是直接把你输入的东西写到代码里，甚至可以用变量名
原始输入就是直接输入字符串，纯字符串
具体情况可以从下面的Demo中看到效果
```python
>>> hello = "hello,world"
>>> str1 = input("what's your name:")   #获取标准输入
	hello
>>> str2 = raw_input("what's your name:")	#获取原始输入
	"hello"
>>> print str1
	hello,world
>>> print str2
	hello
```

# 0X03 字符串格式化
学过C的能迅速的理解Python里的字符串格式化，没学过C的可以快速的理解Python里的字符串格式化  +_+
```python
>>> from string import Template
>>> text = Template("1---$a  2---$b   3---$c   4---$$")	# $a $b $c 都是字符串占位符，先写好后赋值
>>> text.substitute(a="hello,", b="world", c="wow")
	'1---hello,  2---world   3---wow   4---$'

#  %s  是字符串占位符，将后面的字符串加到前面
>>> print "hello, %s" % ("world")
	hello, world

# %15s  是将字符串向前扩充打15位
>>> print "hello, %15s" % ("world")
	hello,           world
>>> print "%15s, world" % ("hello")
    hello, world

# %-15s 是将字符串向后扩充到15位
>>> print "%-15s, world" % ("hello")
	hello          , world
```

# 0X04 字符串处理函数
## find 字符串查找
从一个字符串中查找另一个字符串，返回最左端索引，找不到就返回 -1
```python
>>> str = "hello,world"
>>> str.find(",")
	5
>>> str = "hello,hello,world,world"
>>> str.find(",", 8, 12)
	11
```
> 如果find函数返回了0，并不是没找到，而是在0的位置找到了。毕竟程序员的世界从来都是从0开始数数的

## join 连接
连接序列中的元素
```python
>>> str1 = ['hello', 'world']
>>> str2 = "---".join(str1)
>>> print str2
	hello---world
```
> 注意这里是  str.join(list)  而不是  list.join(str)

## lower 我要小写
返回字符串的全小写版
```python
>>> str1 = "HELLO,WORLD"
>>> str2 = str1.lower()
>>> print str2
	hello,world
```

## replace 查找并替换
查找并替换全部
```python
>>> str1 = "my world, my house, my phone"
>>> str2 = str1.replace("my", "your")	#我就这么把所有东西过户给了你+_+
>>> print str2
	your world, your house, your phone
```

## split 分割
分割字符串
```python
>>> str1 = "hello,world"
>>> str2 = str1.split(",")
>>> print str2
	['hello', 'world']	#返回了一个列表
```

## strip 清理字符串
去除两侧的东西
strip默认去除两侧的空格，当然也可以加参数
```python
>>>	str1 = "  hello,world  "
>>>	str2 = str1.strip()	#默认去除空格
>>>	print str2
	hello,world


>>> str1 = "aaahello,worldaaa"
>>> str2 = str1.srtip("a")  #加了参数就删除两侧的参数里的内容
>>> print str2
	hello,world
```
