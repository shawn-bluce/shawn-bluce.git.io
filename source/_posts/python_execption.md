---
title: Python 异常处理 捕获异常
date: 2016-10-27 19:36
tags:
  - Python
  - Exception
---


# 0X00 什么是异常
程序在运行出错的时候就会抛出异常，异常时在正确的代码里发生的，不是代码出现了错误。下面就是一个异常
```python
#!/usr/bin/python
#coding=utf-8

num_1 = 10
num_2 = 0
# 很明显这里是用一个数字去除以0
# 小学老师就说过0不能作为被除数
# 那么我们来看Python是如何处理这个问题的
num_3 = num_1 / num_2
print num_3
```
运行这个程序就会报出下面的错误，错误提示说在`hello.py`这个文件的第6行，出现了一个错误`integer division or modulo by zero`也就是说Python解释器发现你试图除以0或者试图用0取模。
```bash
Traceback (most recent call last):
  File "./hello.py", line 6, in <module>
    num_3 = num_1 / num_2
ZeroDivisionError: integer division or modulo by zero
```
这里提示的`ZeroDivisionError`就是一个异常，我们可以在后面捕获这个异常，然后进行一些处理。如果不捕获这个异常的话，程序运行到这里，异常就会直接抛出到用户界面，中断程序的运行。

# 0X01 自己放出一个异常
我们可以用`raise`抛出一个自己的异常，这样我们可以在调试程序的时候判断到底出了什么错误，通过抛出的异常信息就可以判断。
```python
#!/usr/bin/python
#coding=utf-8

name = raw_input('name: ')
if name == '':  # 姓名不允许为空
    raise Exception('name is null') # 抛出一个自定义的Exception内容是name is null

age  = input('age : ')
if age <= 0:    # 不允许年龄小于等于0
    raise Exception('age too little') # 爆出一个自顶一个Excep内容是age too little

print 'name is ' + name
print 'age is ' + str(age)
```
上面这段代码只是简单地输入name和age两个变量，合法的话就输出出来。我们这里运行一下试试
```bash
[root@iZ28jaak5nnZ ~]# ./hello.py 
name: shawn     #合法输入的话，就可以顺利输出
age : 20
name is shawn
age is 20
[root@iZ28jaak5nnZ ~]# ./hello.py 
name:   # 这里变量内容为空
Traceback (most recent call last):
  File "./hello.py", line 6, in <module>
    raise Exception('name is null')     # 就是在我设置的地方抛出了异常
Exception: name is null # 异常内容和类型都是我所规定的
[root@iZ28jaak5nnZ ~]# ./hello.py 
name: shawn
age : -1
Traceback (most recent call last):
  File "./hello.py", line 10, in <module>
    raise Exception('age too little')
Exception: age too little
```
我们可以用这种方式在自己的代码中抛出异常，用来做中间值检测，防止中间的数据出现意外导致一些不可思议的后果。

# 0X02 捕获异常
 我们在代码中不管是解释器自己抛出的异常还是你手动抛出的异常，都可以手动的捕获到这个异常，并做出相应的处理。这样就可以提高代码的健壮性。在Python使用`try...except...else`来捕获处理异常。
 ```python
 try:
     # 这里执行一些可能会抛出异常的代码
 except (ExceptionA, ExceptionB, ExceptionC):   # 一个except可以捕获好多个异常
     # 当抛出ABC三种异常的时候，执行这里的代码，执行完之后跳出try...except并继续执行代码
 except ExceptD:
     # 当抛出D异常的时候就会执行这里的代码，执行完后也跳出
 except ExceptE, e:
     # 当抛出E异常的时候在这里处理，e就是这个异常对象，我们可以看e中的信息
     print e # 输出e
 except:
     # 当抛出了一个上面两个except捕获不到的异常的时候，执行这里的操作
 else:
     # 当没有异常抛出的时候执行这里的代码
 finally:
     # 不管代码有没有抛出异常，都会执行这里的代码
 ```
 下面有一个样例，还是除0异常的样例，当除数是0的时候就抛出异常并捕获，然后处理这个异常（提示并重新输入），直到没有除0异常才计算成功并退出程序
 ```python
 #!/usr/bin/python
 #coding=utf-8
 
 while True:
 	num_1 = input('num1: ')
 	num_2 = input('num2: ')
 
 	try:
 		num_3 = num_1 / num_2
 	except ZeroDivisionError:
 		print 'num_1 is 0 !!!'
 		continue
 	else:
 		print num_3
 		exit()
 ```
 下面有一个运行样例
 ```bash
 [root@iZ28jaak5nnZ ~]# ./hello.py 
 num1: 9
 num2: 0
 num_1 is 0 !!!
 num1: 2
 num2: 0
 num_1 is 0 !!!
 num1: 0
 num2: 0
 num_1 is 0 !!!
 num1: 4
 num2: 2
 2
 ```
 当然我们也可以将异常处理完了之后继续抛出，只要你需要。下面的代码和上面的是完全一样的，就只有12行的地方从continue换成了raise，意思就是抛出异常
 ```python
 #!/usr/bin/python
 #coding=utf-8
 
 while True:
 	num_1 = input('num1: ')
 	num_2 = input('num2: ')
 
 	try:
 		num_3 = num_1 / num_2
 	except ZeroDivisionError:
 		print 'num_1 is 0 !!!'
 		raise    # 只有这里是不同的，从continue换成了没有参数的raise，就是把异常继续抛出
 	else:
 		print num_3
 		exit()
 ```
 运行的样例就是下面这样的，执行下去之后会执行except中的处理代码，但是由于raise的存在还是会抛出这个异常
 ```bash
 [root@iZ28jaak5nnZ ~]# ./hello.py 
 num1: 123
 num2: 0
 num_1 is 0 !!!     # 这里就是except处的处理代码
 Traceback (most recent call last):
   File "./hello.py", line 9, in <module>
     num_3 = num_1 / num_2
 ZeroDivisionError: integer division or modulo by zero
 ```
