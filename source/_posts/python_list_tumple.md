---
title: Python之序列：列表、元组
date: 2016-09-07 21:38
tags:
  - Python
  - 数据结构
---


# 0X00 列表和元组
Python里有个东西叫做**序列**，可以想象成一堆数据。可以简单的通过序列实现数组、链表、栈和队列等数据结构。
序列有几种，常见的是列表和元组。

# 0X01 序列分片
我们可以从序列中截取一部分，这种操作被称为分片
分片的时候我们可以选择起始点和结束点，还能选择步长，甚至乃能倒序
分片使用`:`分隔开参数，一般情况下有两个参数，截取第一个参数到第二个参数，左开右闭
如果参数是负数的话，则表示倒数第几个
但是可以接受第三个参数，第三个参数表示步长。如果第二个参数是2那么就是接一跳一。
如果参数为空则表示极限。  具体可以看下面的代码
```python
>>> username = 'hello,world'
>>> print username[4:8]			#截取从4到8，左开右闭
	o,wo
>>> print username[4:-2]		#截取4到倒数第4的参数，如果想要包括最后一个是不能用-1的，要用下面的方式
	o,wor
>>> print username[2:]			#截取包括最后一个的话不能用-1，因为-1是最后一个，然后区间是左开右闭，所有右边留空就表示极限了
	llo,world
>>> print username[:]			#两头取极限，就是完整的序列
	hello,world
>>> print username[1:8:2]		#演示步长，此处步长为2
	el,o
>>> print username[8:0:-1]		#当步长为-1的时候，就是从后向前的
	row,olle
```

# 0X02 序列拼接
序列拼接就和Java里的字符串拼接差不多，可以单纯的用一个加号连在一起。当然Python比Java方便的一点就是，不只是字符串，什么东西只要是在序列里就能用序列拼接到一起。
Python中用加号的方式把序列拼接在一起是**返回一个新的序列**而不是直接修改其中的一个序列。
```python
>>> username = 'hello'
>>> password = 'world'
>>> print username + ',' + password
	hello,world
```

序列不只能做加法，还能做乘法。序列乘n之后返回一个重复了n次的序列
```python
>>> username = 'hello,world'
>>> print username * 3
	hello,worldhello,worldhello,world
```

# 0X03 空序列
空序列是空的，而不是值为0。也许现在不知道这东西干嘛用，等到时候用到了就豁然开朗了
```Python
username = [None] * 10 #这样就生成了一个长度为10的空序列
```

# 0X04 成员判断
成员判断就是判断一个元素是不是存在于一个序列里
这里返回的是布尔值，True或者False
```python
>>> username = 'hello,world'
>>> ',' in username		#判断元素是不是在序列里
	True
>>> 'hello' in username	#判断序列是不是在序列里
	True
>>> username = ['hello', 'world']
>>> 'hello' in username
	True
>>> 'hel' in username
	False
```

# 0X05 长度&统计
可以统计一个序列的长度，还能计算出序列所有元素的最大和最小
具体的排序方法可以去网上找找或者自己尝试一下，针对每种类型的排序方式是不一样的
```python
>>> username = 'hello,world'	#获取长度
>>> print len(username)
	12
>>> number = [1, 2, 3, 4, 5, 6, 7, 8, 9]
>>> print max(number)	#统计最大
	9
>>> print min(number)	#统计平均
	1
```

# 0X06 列表赋值
对列表的赋值和对其他编程语言里的数组赋值几乎是一样的
```python
>>> username = [0, 1, 2, 3, 4, 5]
>>> username[3] = 33
>>> username[5] = 55
>>> print username
	[0, 1, 2, 33, 4, 55]
```

# 0X07 列表删除数据
删除列表里的数据也非常易于理解
```python
>>> username = ['h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd']
>>> del username[5]	#删除索引为5的元素，也就是第6个
>>> print username
	['h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd'] #处在原来5的位置的逗号不在了
```

# 0X08 列表分片赋值
分片赋值相当于把以前的部分数据盖上，写上新的数据
```python
>>> username = list('hello,world')
>>> username[1:5] = list('++++++')
>>> username[2:2] = list('------')
>>> print username	#数据添加成功
	['h', '+', '-', '-', '-', '-', '-', '-', '+', '+', '+', '+', '+', ',', 'w', 'o', 'r', 'l', 'd']
>>> username[3:5] = []  #理论上可以通过这种方式去删除列表中的数据，不过非常不建议这么做，没人愿意看这种代码，包括几天之后的你自己
```

# 0X09 列表常用方法

## append 和 extend
append()方法是 **向列表中添加一个元素**，而extend()则是扩展原有列表。这两个方法都是修改之前的列表，而不是返回一个新的列表。
```python
#!/usr/bin/python
# coding=utf-8

if __name__ == '__main__':
	a = [1, 2, 3, 4, 5, 6, 7, 8, 9]
	b = ['a', 'b', 'c', 'd', 'e', 'f']
	# a.append(b)
	a.extend(b)
	print a
```
上面的代码中，留下append()方法后运行结果如下，可以看到是向原来的列表中加入了一个元素
```bash
[1, 2, 3, 4, 5, 6, 7, 8, 9, ['a', 'b', 'c', 'd', 'e', 'f']]
```

留下extend()方法后运行结果如下，可以看到是将列表b中的元素扩展到了列表a中
```bash
[1, 2, 3, 4, 5, 6, 7, 8, 9, 'a', 'b', 'c', 'd', 'e', 'f']
```

## count 统计数据
计数，统计一个列表在另一个列表里出现了多少次
```python
>>> username = 'hello,world'
>>> username.count('l')
	3
>>> username = ['hello', 'hello', 'world', 'test']
>>> username.count('hello')
	2
```

## index 索引查找
查找第一个匹配的位置，并返回索引位置。如果返回0则是在0的位置上找到了，而不是没找到。没找到的话会直接抛出异常
```python
>>> username = 'hello,world'
>>> username.index('l')	#返回位置
	2
>>> username.index('h')	#返回0的位置
	0
>>> username.index('x') #找不到，抛出异常了
	Traceback (most recent call last):
  		File "<stdin>", line 1, in <module>
	ValueError: substring not found
```
## insert 插入数据-准
向列表中插入数据，可选参数有插入位置和插入内容
```python
>>> username = [1, 2, 3, 4, 5, 6, 7]
>>> username.insert(3, 666)
>>> print username	#向3的位置上插入666
	[1, 2, 3, 666, 4, 5, 6, 7]
```
## pop 弹出数据-出栈
将列表中的最后一个数据弹出来，返回且删除它。  如果知道数据结构中的栈的话，就明白了，可以比喻成  出栈
```python
>>> username = [1, 2, 3, 4, 5, 6, 7, 8]
>>> username.pop()	#出栈
	8
>>> username.pop(2)	#选择删除
	3
```
## remove 匹配删除
移除匹配到的第一项
```python
>>> username = [1, 2, 3, 4, 5, 6]
>>> username.remove(2)
>>> print username
	[1, 3, 4, 5, 6]
```
## sort 排序方法
可以通过Python内置算法排序，甚至还可以自定义参数
```python
>>> username = [1, 5, 2, 5, 65, 23, 54675, 8, 34, 5568, 345]
>>> username.sort()
>>> print username
	[1, 2, 5, 5, 8, 23, 34, 65, 345, 5568, 54675]
```
## sorted 排序函数
类似sort，不过这个是返回一个新的列表
```python
>>> username = [1, 5, 2, 234, 3465, 234, 4657, 5, 65, 23, 54675, 8, 34, 5568, 345]
>>> sorted(username)
	[1, 2, 5, 5, 8, 23, 34, 65, 234, 234, 345, 3465, 4657, 5568, 54675]
```
## list(reversed(x)) 反向排序
逆向
```python
>>> username = [1, 5, 2, 234, 3465, 234, 4657, 5, 65, 23, 54675, 8, 34, 5568, 345]
>>> username = sorted(username)
>>> print username
	[1, 2, 5, 5, 8, 23, 34, 65, 234, 234, 345, 3465, 4657, 5568, 54675]
>>> list(reversed(username))
	[54675, 5568, 4657, 3465, 345, 234, 234, 65, 34, 23, 8, 5, 5, 2, 1]
```
# 0X0A 元组简介
* 元组一般用括号表示
* 元组和列表相比，列表可以修改而元组不能修改
```python
>>> username = ('h', 'e', 'l', 'l', 'o', ',', 'w', 'o', 'r', 'l', 'd')
>>> password = ('x', ) #创建一个只包含一个数据的元组
```
