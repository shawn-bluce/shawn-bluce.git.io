---
title: Python 奇技淫巧 (五) 系统脚本
date: 2017-01-31 15:39
tags:
  - Python
  - Script
---

> ##### 文章中的代码仅在Python3中测试成功，没有在Python2中测试。

# 0X00 退出程序，显示错误信息
写脚本的时候经常会有执行出错，出错的时候可以用一句话把程序退出并且打印错误信息
`raise SystemExit('error message')`

# 0X01 输入密码
有的时候需要输入用户名和密码，使用`input()`输入用户名自然没有问题，但是用相同的方法输入密码的时候时使用明文的。长期用Linux的可能对Linux中密码的输入比较有印象，输入密码的时候是密文，且没有任何提示，包括星号，所以用这种方法输入密码是非常安全的。使用`petpass`库可以简单的输入用户名和密码，输入用户名最简单还是`input()`，如果要获取当前登录的用户名就可以使用`getpass.getuser()`，输入密码就可以使用`getpass.getpass()`来实现Linux中的那种密码输入。 **如果测试的时候有问题可以在命令行下测试，比如Windows的CMD或者Linux的终端**
```python
#!/usr/bin/python
# coding=utf-8

import getpass


if __name__ == "__main__":
    user = input("Username:")
    # user = getpass.getuser()

    passwd = getpass.getpass()
```

# 0X03 执行命令
在Linux中想要用Python代替Shell必然会出现在Python中调用命令的时候，那么这个时候就可以用这个方法来执行命令并获得输出内容。`subprocess.check_output([])`这个方法的参数是一个列表，列表里是一个或多个字符串，就像下面介绍的那样把命令通过空格拆分开，放到这个列表中。`check_output`返回的是一个二进制串，可以对其进行编码转变成人类可读的字符串。 **这种方法只在Linux里测试过** 毕竟没几个人会在WIndows下写脚本是吧。
```python
#!/usr/bin/python
# coding=utf-8

import subprocess


if __name__ == "__main__":
    out_bype  = subprocess.check_output(['ls', '/dev'])
    out_text = out_bype.decode('utf-8')
    print(out_text)
```

# 0X04 复制/移动 文件/目录
在Python中复制移动文件和目录非常简单，尤其是在不考虑链接的情况下。
```Python
#!/usr/bin/python
# coding=utf-8

import shutil


if __name__ == "__main__":
    shutil.copy('/etc/passwd', 'passwd')	# 将/etc/passwd复制到当前目录，等同于Linux中的  cp /etc/passwd passwd
    shutil.copytree('/etc', 'etc')			# 复制目录

    shutil.move('passwd', 'mima')			# 移动文件，也可以重命名，和Linux中的mv命令一样
```

# 0X05 获取终端大小
在Linux中一般是在终端下工作，那么有的时候需要知道当前终端大小来控制输出的字符串长度。
```python
#!/usr/bin/python
# coding=utf-8

import os

if __name__ == "__main__":
    sz = os.get_terminal_size()
    columns = sz.columns
    lines   = sz.lines
    print(sz)
    print(columns)
    print(lines)
```

# 0X06 os.walk()
经常需要遍历一个目录，来获取目录中的内容，如果只需要查看一个目录，那么使用`os.listdir()`就足够了，如果只判断一个文件是否为目录，则`os.path.isdir()`也足够了。但是有的时候我们需要逐层遍历目录，且区别对待目录和文件，那么通常会自己手写一个递归的方法来解决。这样虽然能解决问题，但是毕竟多写了代码且效率还不高，其实`os`库里有一个方法值得我们使用就是`os.walk()`。这个方法接收一个目录作为参数，返回一个迭代器，每次迭代是一个元组，元组有三个元素，第一个元素是当前路径，第二个元素是当前目录下的目录名，第三个元素是当前目录下的文件。具体的可以看代码注释
```python
#!/usr/bin/python
# coding=utf-8

import os

if __name__ == "__main__":
    files = os.walk('D:/movie')		# 这里调用了方法，传入一个路径，返回一个可迭代对象
    for i in files:		# 开始迭代
        print('path_name: ', i[0])	# 输出当前路径
        print('dir_name : ', i[1])	# 当前目录下的目录
        print('file_name: ', i[2])	# 当前目录下的文件
        print('-----------------')
```

这个输出是下面这样的
```bash
path_name:  D:/movie
dir_name :  ['加勒比海盗', '机械师', '火影忍者', '蜘蛛侠']
file_name:  ['V字仇杀队.mkv', 'wikileaks-720p.mkv', '湄公河行动.mkv', '盗梦空间.mkv', '神奇动物在哪里.mp4', '绝地逃亡.mkv']
-----------------
path_name:  D:/movie\加勒比海盗
dir_name :  []
file_name:  ['加勒比海盗1：黑珍珠号的诅咒.mkv', '加勒比海盗2：聚魂棺.avi', '加勒比海盗3：世界尽头.avi', '加勒比海盗4：惊涛怪浪.mkv']
-----------------
path_name:  D:/movie\机械师
dir_name :  []
file_name:  ['机械师1.mkv', '机械师2：复活.mp4']
-----------------
path_name:  D:/movie\火影忍者
dir_name :  []
file_name:  ['火影忍者：博人传.mp4', '火影忍者：忍者之路.mkv', '火影忍者：终章.mp4']
-----------------
path_name:  D:/movie\蜘蛛侠
dir_name :  []
file_name:  ['蜘蛛侠1-2002.mkv', '蜘蛛侠2-2004.mkv', '蜘蛛侠3-2007.mkv', '超凡蜘蛛侠1-2012.mkv', '超凡蜘蛛侠2-2014.mp4']
-----------------
```

# 0X07 修改配置文件
在Linux中有大量的配置文件，Windows中也有一些ini格式的配置文件，语法都一样的。那么用脚本来修改配置文件当然不必要全部读完整个文件后正则匹配，有一个非常简单且好用的方法。下面是我用来做测试的实例配置文件，命名为`1.ini`放在`D:/`根目录。
```config
[home]
phone = On
kindle = Off
learn = False

[school]
phone = On
kindle = On
learn = True
```
可以看到，手机无论在哪都开机，Kindle只有在学校才用，学习也只有在学校才学。那么我们可以通过下面的方式来读取和修改这个配置文件。
```python
#!/usr/bin/python
# coding=utf-8

from configparser import ConfigParser
import sys

if __name__ == "__main__":
    cfg = ConfigParser()    # 实例化一个对象
    cfg.read('D:/1.ini')    # 读取配置文件

    tables = cfg.sections() # 获取标签
    print(tables)

    phone_value = cfg.get('home', 'Phone')  # 获取home标签下Phone的值
    print(phone_value)

    kindle_value = cfg.get('school', 'Kindle')  # 获取school下Kindle的值
    print(kindle_value)

    learn_value = cfg.get('school', 'learn')    # 获取school下learn的值
    print(learn_value)

    cfg.set('home', 'learn', 'True')     # 修改home下的learn为True
    f = open('D:/1.ini', 'w')            # 用可写模式打开文件
    cfg.write(f)        # 将数据写回
    f.close()           # 关闭文件
```

# 0X08 打开浏览器
不管是要给用户展示一个页面还是要将数据用HTML形式展示出来，都需要打开浏览器，这个在Python中可以用一行代码来搞定。`webbrowser.open_new('http://blog.just666.cn')`可以打开一个新的浏览器窗口，并打开这个链接，`webbrowser.open_new_table('http://blog.just666.cn')`可以在当前浏览器窗口新开一个标签。（需要先导入`webbrowser`这个包）
