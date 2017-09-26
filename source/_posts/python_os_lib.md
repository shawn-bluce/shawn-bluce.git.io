---
title: Python中OS库的常用方法
date: 2017-04-01 20:56
tags:
  - Python
---

在写一些系统脚本或者自动化运维脚本的时候经常会用到os库，这里做个整理，方便查找

### os.sep
获得当前操作系统使用的目录分隔符，比如Windows就会得到`\`而Linux/Unix就会得到`/`

### os.name
获得当前使用的操作系统，Windows是NT内核，所以会得到`nt`，而Linux/Unix用户则会得到`posix`

### os.getcwd()
获得当前工作目录，即当前Python脚本工作的目录路径。

### os.getenv()
用来获得环境变量
`os.getenv('PATH')`

### os.environ
可以获取并修改环境变量
```Python
print(os.environ['PATH'])
os.environ += 'D:/testdir/bin/'
print(os.environ["PATH"])
```

### os.listdir()
列出某目录下所有的目录和文件
`print(os.listdir())`

### os.remove()
删除文件
`os.remove('D:/test.file')`

### os.system()
运行Shell或者CMD命令
`os.system('ifconfig')`

### os.linesep
获取当前平台使用的行终止符。例如，Windows使用`\r\n`，Linux使用`\n`而Mac使用`\r`。

### os.path.split()
获得一个列表，list[0]是路径的，list[1]是文件名
```Python
path = 'D:/game/gtav/bin/gtav.exe'
print(os.path.split(path)[0])
print(os.path.split(path)[1])
```

### os.path.isfile()和os.path.isdir()
判断路径是不是文件/目录
```Python
print(os.path.isfile('D:/game/gtav/bin/gtav.exe'))
print(os.path.isdir('D:/game/gtav/bin'))
```

### os.path.existe()
函数用来检验给出的路径是否真地存在
```Python
path = 'D:\\hadoop-2.6.5\\bin'
print(os.path.exists(path))
path = 'D:\\hadoop-2.6.5\\bin\\hadoop'
print(os.path.exists(path))
```

### os.chdir(dirname)
切换工作目录，相当于`cd`的命令
```Python
os.chdir('D:/game/gtav/')
print(os.getcwd())
```

### os.path.getsize(name)
获取文件大小，以字节为单位
```Python
size = os.path.getsize('D:/iso/debian-8.6.0-amd64-DVD-1.iso')
print(size/1024/1024/1024, 'GB')
```

### os.path.abspath(name)
获取绝对路径，如果在Python工作目录下有一个文件`file.txt`，那么我就可以直接`open('file.txt')`，也可以用该方法获得其绝对路径`print(os.path.abspath('file.txt'))`。也可以用来规范路径字符串`print(os.path.abspath('D:/game\gtav\bin/gtav.exe'))`

### os.path.normpath(path)
专门用来规范路径
```Python
path = 'D:/test/sdf\zfb'
print(os.path.normpath(path))
```

### os.path.splitext()
获取文件名和扩展名
```Python
path = '/home/shawn/hello.py'
print(os.path.splitext(path))
```

### os.path.join(path,name)
连接目录和文件名，可以不用自己添加分隔符，能减少bug率提升跨平台性

### os.path.basename(path)
获取路径中的文件名

### os.path.dirname(path)
获取路径中的目录名
