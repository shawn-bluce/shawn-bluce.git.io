---
title: Python 奇技淫巧 (二) 字符串、文本
date: 2017-01-19 14:51
tags:
  - Python
  - String
---

> ##### 文章中的代码仅在Python3中测试成功，没有在Python2中测试。

# 0X00 split升级
字符串有一个`split`方法，可以用某个字符或字符串把源字符串切开。但是存在一个弊端，切割位置是固定的，不能灵活切割。有这样一个需求，将这个字符串`hello 1 wrld 2 python 3 linux`切割开，以每个数字为分隔符。这样标准的`str.split`就不能完成任务了。但是在`re`模块中有一个`re.split`可以完成这任务。这个方法的分隔符不是使用准确不变的字符/串而是使用正则表达式。
```Python
#!/usr/bin/python
# coding=utf-8

import re

if __name__ == '__main__':
    my_str = 'hello 1 wrld 2 python 3 linux'
    res = re.split('[0-9]', my_str)
    print(res)
```
这里使用的正则表达式就是普通的字符串形式，而不需要`re.compile`进行编译。有了这个方法就可以更加灵活地切割字符串了。

# 0X01 字符串开头结尾的匹配
当我们有一堆的url，想在url中找到http开头且.jpg结尾的图片文件，以前我总是直接`str.strip('http://') == str`来判断开头是不是'http://'但是这样太蠢了，而且也不是很靠谱、因为万一开头不是而结尾是的话就会误判。这里有两个方法可以非常简单地做出这种判断：`str.startswith()`和`str.endswitch()`两个。一个是用来判断字符串是否以xxx开头、另一个是用来判断字符串是否以xxx结尾。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    url = 'http://blog.just666.cn/img/01.jpg'
    print(url.startswith('http://'))
    print(url.endswith('.jpg'))
```
这种方式可以有一个简单的改变，使用列表推导式来批量判断。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    url_list = ['http://hey.sdf.we/sdfw.jpg',
                'http://asdf.ser.x/zxvw.jpg',
                'http://sdf.re.xcv/ind.html',
                'http://zx.er.cxv/held.html',
                'http://zx.sdf.vs/hell.jpg']
	# 这里是列表推导式
    jpg_list = [jpg for jpg in url_list if jpg.endswith('.jpg')]
    print(jpg_list)
```
也可以将后两行换成`print(all(jpg.startswith('.jpg') for jpg in url_list))`就会输出`False`因为并不是所有都以'.jpg'结尾。
还可以使用匹配的方式，比如你需要在N多url中找到'http/ftp'这两个协议的url，可以这么写
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    choices = ['http://', 'ftp://']
    choices = tuple(choices)	# 这里必须要使用元组类型

    url = 'http://www.baidu.com'
    print(url.startswith(choices))

    url = 'ftp://192.168.1.2'
    print(url.startswith(choices))

    url = 'https://www.taoba.com'
    print(url.startswith(choices))
```
`startswith`和`endswitch`两个函数完全可以被正则表达式替代，但是对于简单匹配来说没必要用正则表达式，这两个函数比正则要快且可读性搞书写快。

# 0X02 Shell通配符
在匹配字符串的时候不仅可以使用比较复杂的正则表达式，还可以用比较简单的通配符。使用通配符需要注意的一个问题就是大小写。在Linux/Unix/Mac上要区分大小写，在Windows上不区分大小写。`fnmatch`下有两个方法，`fnmatch`按操作系统来判断到底区不区分大小写，而`fnmatchcase`则强制区分大小写。使用方法如下：
```Python
>>> from fnmatch import fnmatch, fnmatchcase
>>> filename = 'hello.c'
>>> fnmatch(filename, '*.c')
True
>>> fnmatch(filename, 'hell?.c')
True
>>> fnmatch(filename, 'hellO.c')
False
>>> fnmatch(filename, 'hello.c')
True
```

# 0X03 查找替换
将字符串A中所有的某个子字符串B替换为另外的字符串C，可以简单的使用字符串的`replace`函数
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    text = 'hello world hello python'
    print(text.replace('hello', 'hey'))
```
还有一种使用`re`模块的方案，可以使用正则匹配来查找并替换。`re.sub()`方法可以做到这一点。这里sub的第一个参数是匹配的正则表达式，第二个参数是替换的字符串（其中\1 \2 \3表示匹配的编号），第三个参数就是待匹配替换的字符串了。这个例子将`1/19/2017`转变为`2017/1/19`
```python
#!/usr/bin/python
# coding=utf-8

import re


if __name__ == '__main__':
    text = 'hello world 1/19/2017'
    print(re.sub(r'(\d+)/(\d+)/(\d+)', r'\3-\1-\2', text))
```


# 0X04 Unicode大法好
我们有的时候会遇到一些奇怪的字符串问题，比如看起来明明完全一样的两个字符串在对比的时候居然不相等。得益于Python3使用的Unicode我们可以简单的对字符串统一规范。
```python
#!/usr/bin/python
# coding=utf-8

import unicodedata


if __name__ == '__main__':
    s1 = 'char: \u00f1'
    s2 = 'char: n\u0303'
    print(s1)
    print(s2)
    print(s1 == s2, '   ', len(s1), '   ', len(s2))
    
    # 改一下编码
    s1 = unicodedata.normalize('NFC', s1)
    s2 = unicodedata.normalize('NFC', s2)
    print(s1)
    print(s2)
    print(s1 == s2, '   ', len(s1), '   ', len(s2))
```
这里面用到的那个奇怪的字符我也不知道是什么，是在《Python Cookbok》这本书上找的例子。就是说看起来`\u00f1`这个字符和`n\u0303`是一样的，但是很明显前者是一个字符而后者是两个字符，所以我们在对比的时候才会出现字符串不相同甚至长度不同的问题。然后引入了`unicodedata`模块之后用里面的`normalize`方法可以将字符串规范化，`s1 = unicodedata.normalize('NFC', s1)`就是将s1采用NFC方式规范。所谓NFC方式就是 **全组成** 也就是说“如果可能的话就是用单个代码点，也就是s1那种方式”（这里和近场通讯的NFC很明显没半点关系）。可选的除了NFC还有NFD（尽量使用组合字符，也就是s2那种方式），还支持NFKC和NFKD这里就自行Google一下吧。


# 0X05 字符串对齐
有的时候我们需要对字符串进行对齐操作，比如在终端中模拟界面之类的。可以使用C语言风格的%10S这种去替代，但是有更好用简单的方法，就是使用字符串内置的`ljust/rjust/center`方法。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    text = 'Main Menu'
    print('左对齐：', text.ljust(30))
    print('右对齐：', text.rjust(30))
    print('中对齐：', text.center(30))

    print('左对齐填充：', text.ljust(30, '+'))
    print('右对齐填充：', text.rjust(30, '='))
    print('中对齐填充：', text.center(30, '*'))
```
还有一个炫酷的融合函数叫`format`。这个函数接收两个参数，第一个参数是待处理字符串，第二个参数是选项。具体选项如下：其中'^'是居中，'>'是右对齐，'<'是左对齐，后面跟着的数字是宽度，对齐字符前面是填充字符。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    text = 'Main Menu'
    print(format(text, '>20'))
    print(format(text, '<20'))
    print(format(text, '^20'))

    print(format(text, '->20'))
    print(format(text, '=<20'))
    print(format(text, '*^20'))
```

