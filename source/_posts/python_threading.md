---
title: Python使用threading实现多线程
date: 2016-12-12 15:39
---

# 0X00 多线程
多线程是个提高程序运行效率的好办法，本来要顺序执行的程序现在可以并行执行，可想而知效率要提高很多。但是多线程也不是能提高所有程序的效率。程序的两个极端是‘CPU密集型’和‘I/O密集型’两种，多线程技术比较适用于后者，因为在串行结构中当你去读写磁盘或者网络通信的时候CPU是闲着的，毕竟网络比磁盘要慢几个数量级，磁盘比内存慢几个数量级，内存又比CPU慢几个数量级。多线程技术就可以同时执行，比如你的程序需要发送N个http数据包（10秒），还需要将文件从一个位置复制到另一个位置（20秒），然后还需要统计另一个文件中'hello,world'字符串的出现次数（4秒），现在一共是要用34秒。但是因为这些操作之间没有关联，所以可以写成多线程程序，几乎只需要20秒就完成了。这是针对I/O密集型的，如果是CPU密集型的就不行了。比如我的程序要计算1000的阶乘（10秒），还要计算100000的累加（5秒），那么即使程序是并行的，还是会要用15秒，甚至更多。因为当程序使用CPU的时候CPU是通过轮转来执行的，IO密集型的程序可以在IO的同时用CPU计算，但是这里的CPU密集型就只能先执行一会儿线程1再执行一会儿线程2。所以就需要15秒，甚至会更多，因为CPU在切换的时候需要耗时。解决CPU密集型程序的多线程问题就是CPU的事情了，比如Intel的超线程技术，可以在同一个核心上真正的并行两个线程，所以称之为‘双核四线程’或者‘四核八线程’，我们这里具体的先不谈，谈我也不知道。

# 0X01 Python骗人
说了这么多多线程的好处，但是其实Python不支持真正意义上的多线程编程。在Python中有一个叫做GIL的东西，中文是 *全局解释器锁* ，这东西控制了Python，让Python只能同时运行一个线程。相当于说真正意义上的多线程是由CPU来控制的，Python中的多线程由GIL控制。如果有一个CPU密集型程序，用C语言写的，运行在一个四核处理器上，采用多线程技术的话最多可以获得4倍的效率提升，但是如果用Python写的话并不会有提高，甚至会变慢，因为线程切换的问题。所以Python多线程相对更加适合写I/O密集型程序，再说了真正的对效率要求很高的CPU密集型程序都用C/C++去了。

# 0X02 第一个多线程
Python中多线程的库一般用`thread`和`threading`这两个，`thread`不推荐新手和一般人使用，`threading`模块就相当够用了。
有一个程序，如下。两个循环，分别休眠3秒和5秒，串行执行的话需要8秒。
```Python
#!/usr/bin/env python
# coding=utf-8

import time


def sleep_3():
    time.sleep(3)


def sleep_5():
    time.sleep(5)

if __name__ == '__main__':
    start_time = time.time()
    print 'start sleep 3'
    sleep_3()
    print 'start sleep 5'
    sleep_5()
    end_time = time.time()
    print str(end_time - start_time) + ' s'
```
输出是这样的
```bash
start sleep 3
start sleep 5
8.00100016594 s
```
然后我们对它进行修改，使其变成多线程程序，虽然改动没有几行。首先引入了threading的库，然后实例化一个threading.Thread对象，将一个函数传进构造方法就行了。然后调用Thread的start方法开始一个线程。join()方法可以等待该线程结束，就像我下面用的，如果我不加那两个等待线程结束的代码，那么就会直接执行输出时间的语句，这样一来统计的时间就不对了。
```python
#!/usr/bin/env python
# coding=utf-8

import time
import threading    # 引入threading


def sleep_3():
    time.sleep(3)


def sleep_5():
    time.sleep(5)

if __name__ == '__main__':
    start_time = time.time()
    print 'start sleep 3'
    thread_1 = threading.Thread(target=sleep_3)     # 实例化一个线程对象，使线程执行这个函数
    thread_1.start()        # 启动这个线程
    print 'start sleep 5'
    thread_2 = threading.Thread(target=sleep_5)     # 实例化一个线程对象，使线程执行这个函数
    thread_2.start()        # 启动这个线程
    thread_1.join()     # 等待thread_1结束
    thread_2.join()     # 等待thread_2结束
    end_time = time.time()
    print str(end_time - start_time) + ' s'
```
执行结果是这样的
```bash
start sleep 3
start sleep 5
5.00099992752 s
```

# 0X03 daemon 守护线程
在我们理解中守护线程应该是很重要的，类比于Linux中的守护进程。但是在`threading.Thread`中偏偏不是。
> 如果把一个线程设置为守护线程，就表示这个线程是不重要的，进程退出的时候不需要等待这个线程执行完成。 ---------《Python核心编程 第三版》

在Thread对象中默认所有线程都是非守护线程，这里有两个例子说明区别。这段代码执行的时候就没指定`my_thread`的`daemon`属性，所以默认为非守护，所以进程等待他结束。最后就可以看到100个hello,world
```python
#!/usr/bin/env python
# coding=utf-8

import threading


def hello_world():
    for i in range(100):
        print 'hello,world'


if __name__ == '__main__':
    my_thread = threading.Thread(target=hello_world)
    my_thread.start()
```
这里设置了`my_thread`为守护线程，所以进程直接就退出了，并没有等待他的结束，所以我们看不到100个hello,world只有几个而已。甚至还会抛出一个异常告诉我们有线程没结束。
```python
#!/usr/bin/env python
# coding=utf-8

import threading


def hello_world():
    for i in range(100):
        print 'hello,world'


if __name__ == '__main__':
    my_thread = threading.Thread(target=hello_world)
    my_thread.daemon = True   # 设置了标志位True
    my_thread.start()
```

# 0X04 传个参数
之前的代码都是直接执行一段代码，没有过参数的传递，那么怎么传递参数呢？其实还是很简单的。`threading.Thread(target=hello_world, args=('hello,', 'world'))`就可以了。args后面跟的是一个元组，如果没有参数可以不写，如果有参数就直接在元组里按顺序添加就行了。
```python
#!/usr/bin/env python
# coding=utf-8

import threading


def hello_world(str_1, str_2):
    for i in range(10):
        print str_1 + str_2

if __name__ == '__main__':
    my_thread = threading.Thread(target=hello_world, args=('hello,', 'world'))    # 这里传递参数
    my_thread.start()
```


# 0X05 再来个多线程
threading有三种创建Thread对象的方式，但是一般只会用到两种，一种是上面`0X02`说的传个函数进去，另一种就是这里说的继承`threading.Thread`。在这儿我们自己定义了两个类，类里重写了run()方法，也就是调用start()之后执行的代码，开启线程就和之前开启是一样的。之前的方式更面向过程，这个更面向对象。
```python
#!/usr/bin/env python
# coding=utf-8

import threading


class MyThreadHello(threading.Thread):

    def run(self):
        for i in range(100):
            print 'hello'


class MyThreadWorld(threading.Thread):

    def run(self):
        for i in range(100):
            print 'world'

if __name__ == '__main__':
    thread_hello = MyThreadHello()
    thread_world = MyThreadWorld()
    thread_hello.start()
    thread_world.start()
```
