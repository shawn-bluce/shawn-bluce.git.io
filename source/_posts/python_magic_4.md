---
title: Python 奇技淫巧 (四) 文件&I/O
date: 2017-01-24 21:43
tags:
  - Python
  - I/O
---

> ##### 文章中的代码仅在Python3中测试成功，没有在Python2中测试。

# 0X00 指定编码
每个文本文件都是以某一编码格式保存的，如果解码格式和文本格式不同就会出现乱码，在Python中可以简单的控制用什么编码来打开文件以读写文件。使用`open`打开文件的时候指定一个`encoding`参数就可以使用其他而非默认编码打开文件了。这里用到了一个打开文件的方式是`with open() as f:`这样，这样做的话在这个with下面的代码块中可以直接调用f这个文件对象，并且执行到with代码块之外的时候会自动关闭文件，不需要再手动关闭文件。
```Python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    
    # 使用utf-8编码，写模式，打开文件D:/test.txt
    with open('D:/test.txt', 'w', encoding='utf-8') as f:
        f.write('你好，世界')    # 写一行汉子
    
    # 使用utf-8编码，读模式，打开文件D:/test.txt
    with open('D:/test.txt', 'r', encoding='utf-8') as f:
        print(f.read())       # 因为是编码相同所以可以正常读出文字
    
    # 使用latin-1编码，读模式，打开文件D:/test.txt
    with open('D:/test.txt', 'r', encoding='latin-1') as f:
        print(f.read())       # 因为使用的编码格式不同，所以会出现乱码
```

# 0X01 输出重定向
在Linux中可以对命令的输出进行重定向，将本应该输出到屏幕的东西输出到指定的文件里，在Python中也是可以简单做到这一点的。假设一个已经用写入模式打开的文件对象f，在输出文字的时候就可以直接这样调用`print('hello,world', file=f)`就可以直接将输出的内容重定向到文件中。这里需要注意的就是文件必须已经用可写模式打开，且是文本模式。

# 0X02 指定分隔符和结尾
我们可以使用这样一条语句打印多个字符串`print('hello', 'world', 'hello', 'python')`，会直接将字符串连接到一起，默认没有分隔符且使用系统默认作为结尾符号。可以给`print()`指定两个参数来设置分隔符和结尾符。`print('hello,', world', 'hello', 'python', sep=',', end='\n')`这里指定了使用逗号分隔开这些字符串，并且使用`\n`作为结尾符号。如果使用空字符串作为结尾符号，输出的时候最后就不自动换行。

# 0X03 读写二进制文件
有一个最常见的二进制文件读写实例：从网上下载东西到本地。比如有一个url是`http://blog.just666.cn/usr/themes/Themia/img/sj/134.jpg`，那怎么把这个图片下载到本地呢？可以使用下面这段代码。先找到url，然后使用urlopen打开这个网络文件并获取到文件内容，最后用二进制模式写入到新的本地文件就可以了。
```python
#!/usr/bin/python
# coding=utf-8

from urllib.request import urlopen


if __name__ == '__main__':
    web_img = urlopen('http://blog.just666.cn/usr/themes/Themia/img/sj/134.jpg')	# 使用urlopen打开一个url
    web_img = web_img.read()	# 获得文件内容，当然这里是二进制的所以没有可读性
	# 新打开一个文件，使用二进制写入模式
    with open('D:/hey.jpg', 'wb') as f:	# 在w后面指定一个b也就是二进制写入模式
        f.write(web_img)	# 将新文件写入进去
```

# 0X04 路径名
在Python中可以使用`os.path`处理路径名的问题，比较常用的三个方法`os.path.basename()`、`os.path.dirname()`、`os.path.join()`，分别用来显示一个完整地址的最后文件名、显示某完整地址文件的目录地址、将目录和文件拼接起来。因为Python比较强大，所以可以做到容错效果，比如说在Windows中地址是这样的`D:\game\steam\steamapps\csgo`，但是如果我写成了Linux下的格式`D:/game/steam/steamapps/csgo`也是没有问题的，照样可以用这些方法处理，没有影响。
```Python
#!/usr/bin/python
# coding=utf-8

import os


if __name__ == '__main__':
    path = '/var/www/html/index.html'	# 这里是一个完整地址的文件
    print(os.path.basename(path))		# 可以显示文件名  index.html
    print(os.path.dirname(path))		# 可以显示当前文件的地址位置 /var/www/html/
    print(os.path.join('D:/', 'hehe.exe'))	# 将连接拼起来编程 D:/hehe.exe
```

# 0X05 小技巧
检验文件是否存在：`os.path.exists('D:/test.txt')`如果文件存在则返回True否则就是False
获取文件元数据：`os.path.getatime('D:/test.txt')`查看修改时间   `getsize`获取文件大小


