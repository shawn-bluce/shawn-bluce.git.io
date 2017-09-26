---
title: Python 奇技淫巧 (一) 列表、集合、字典
date: 2017-01-15 15:08
tags:
  - Python
  - 数据结构
  - Dict
  - List
---

> ### 文章中的代码仅在Python3中测试成功，没有在Python2中测试。

# 0X00 \*表达式
从某个可迭代对象中分解出N个元素，但是这个可迭代的对象可能会超过N，会出现too many values to unpack异常。

比如我这儿有N个统计信息，因为第一次和最后一次的信息不准确需要删除掉，而将中间的信息保留下来，那么就可以这么弄。
```python
#!/usr/bin/python
# coding=utf-8

if __name__ == '__main__':
    grade = [23, 45, 42, 45, 78, 98, 89, 97, 69, 77, 88, 50, 65, 99, 98]
    first, *new_grade, last = grade
    print(new_grade)
```
这里的赋值就是将第一个和最后一个赋给了first和last，而中间的给了new_grade

# 0X01 定长队列
有一种情况：程序在运行的时候会记录日志，比如说web程序的访问历史。如果我们需要只保留最后的1W条数据，那么很快能想到使用一个列表，每次插入数据的时候判断长度，然后对应的append和del。但是有一个更简单且更快速的方法就是使用`collections.deque()`。下面的例子中有一个1024长的列表，我们列表里只存最新的7条。
```python
#!/usr/bin/python
# coding=utf-8

# 导入一个包
import collections

if __name__ == '__main__':
	# 当做一种数据解雇来用就可以
    auto_queue = collections.deque(maxlen=7)
    my_list = range(1024)
    for i in my_list:
        auto_queue.append(i)
    print(auto_queue)
```
运行之后可以看到，列表里只保存了最后插入的七条数据。

# 0X02 最大最小的几个元素
当我们有一个列表，需要找到列表里最大的N个元素时，一般会想到先排序然后分片，这想法当然不多，但是还有一个更好用的方法：
```python
#!/usr/bin/python
# coding=utf-8

import heapq


if __name__ == '__main__':
    my_list = [34, 234, 56, 56, 345, 456, 23, 213, 456, 8, 98, 43, 2, 67]
    print('max: ', heapq.nlargest(3,  my_list))	# 找到最大的三个
    print('min: ', heapq.nsmallest(2, my_list)) # 找到最小的两个
```
我这里用列表来演示，但是这个方法支持更复杂的数据结构。比如我有一个列表，列表里包含很多个字典，字典里是学生考试信息，那么我就可以用考试分数来找到前三名：
```python
#!/usr/bin/python
# coding=utf-8

import heapq


if __name__ == '__main__':
    my_list = [{'name': '小明', 'grade': 56}, {'name': '小红', 'grade': 87}, {'name': '小刚', 'grade': 67},
               {'name': '小志', 'grade': 46}, {'name': '小逗逼', 'grade': 99}, {'name': '小华', 'grade': 85},]
    print('max: ', heapq.nlargest(3,  my_list, key=lambda s: s['grade']))
    print('min: ', heapq.nsmallest(3, my_list, key=lambda s: s['grade']))
```
key 后面的 `lambda s: s['grade']`是用了一个 **匿名函数** 。列表里唯一的值就是排序的关键字。更多关于[更多关于匿名函数](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001431843456408652233b88b424613aa8ec2fe032fd85a000)

如果N相对总数据量来说很小，可以用`heapq.heapify()`获得更好的性能。这个函数会将原来的集合转变成列表并以 **堆** 的形式排序。而堆最重要的一个特性就是最小的那个元素一定在第一位。所以我们可以利用这个性质来获取最小的前N个。
```python
#!/usr/bin/python
# coding=utf-8

import heapq


if __name__ == '__main__':
    my_list = [234, 324, 456, 567, 345, 23, 546, 567, 98, 45, 2, 576]
    heapq.heapify(my_list)
    print(my_list)	# 查看排序结果
    print(heapq.heappop(my_list))	# 取第一个元素，并重拍
    print(heapq.heappop(my_list))
    print(heapq.heappop(my_list))
```

# 0X03 优先级队列
普通队列都是按照FIFO(first in first out)来增删数据，有些特殊情况需要给每个元素设定优先级，push元素的时候设定优先级，pop的时候找到优先级最高的。比如说操作系统的任务调度就是这样的，会给每个进程设置优先级。不过当然，不会使用Python实现的了。这里的内部也是用堆来实现的，所以在15行的位置用了`-priority`来让堆反向排、
```Python
#!/usr/bin/python
# coding=utf-8

import heapq


# 这个类就是队列类
class PriorityQueue:

    def __init__(self):
        self._queue = []    # 队列元素
        self._index = 0     # 索引

    def push(self, item, priority):
        heapq.heappush(self._queue, (-priority, self._index, item))
        self._index += 1

    def pop(self):
        return heapq.heappop(self._queue)[-1]


# 队列中的数据类型
class Item:

    def __init__(self, name):
        self.name = name    # 只有一个属性、name

    def __repr__(self):
        return 'Item({!r})'.format(self.name)   # 将格式化好的字符串返回

if __name__ == '__main__':
    my_queue = PriorityQueue()  # 实例化一个优先级队列
    my_queue.push(Item('内核'), 99)   # 内核的优先级最高了
    my_queue.push(Item('文件复制'), 40)
    my_queue.push(Item('CS:GO'), 75)
    print(my_queue.pop())   # 找到优先级最高的
```

# 0X04 一键多值
我们可以轻松的写出用一个键对应多个值的字典，只需要让键对应到列表或者集合就好了，但是要啰里啰嗦写一大堆东西。其实可以用一个内建的方法来解决这个问题。通过这个方法可以快速创建这种字典，也可以像操作普通列表一样操作里面的数据。
```python
#!/usr/bin/python
# coding=utf-8

from collections import defaultdict

if __name__ == '__main__':
    my_dic = defaultdict(list)
    my_dic['name'].append('李华')
    my_dic['qq'].append('66666')
    my_dic['qq'].append('23333')
    my_dic['qq'].append('88888')
    print(my_dic)
```

# 0X05 分片命名
Python中分片非常好用，有的时候会在程序中出现很多分片，管理起来特别麻烦。可以通过这种方式给分片命名，下次再次调用的时候可以直接使用分片的名字。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    data = 'shawn 17 M'
    name = slice(0, 5)
    age  = slice(7, 8)
    sex  = slice(9, 10)
    print(data[name])
    print(data[age])
    print(data[sex])
```

# 0X06 词频统计
从一个序列中找到出现次数最多的元素。`Counter`对象还可以进行简单的加减，比如a序列里出现了10次'hello'而b序列里出现了3次'hello'，那么a+b的话'hello'的值就会变成13。
```Python
#!/usr/bin/python
# coding=utf-8

from collections import Counter


if __name__ == '__main__':
    data = ['hello', 'world', 'hey', 'hello', 'world', 'jack', 'hey', 'york', 'hey', 'hello', 'hello']
    word_count = Counter(data)
    print(word_count.most_common(1))	# 这个参数1可以更改，表示的是出现次数最多的几个元素
```

# 0X07 对字典列表排序
比如我们从数据库中查询到了部分学生的成绩，每个学生的信息存成一个字典，多个字典组成一个列表。然后需要让列表按学生成绩排序。
```python
#!/usr/bin/python
# coding=utf-8

from operator import itemgetter


if __name__ == '__main__':
    student = [
        {'name': '小明', 'mark': 98},
        {'name': '小红', 'mark': 87},
        {'name': '小刚', 'mark': 58},
        {'name': '李华', 'mark': 100}
    ]

    student = sorted(student, key=itemgetter('name'))

    for i in student:
        print(i)
```

# 0X08 筛选序列
有一个列表，里面全是某次考试的成绩，需要成绩列表中找到所有的不及格成绩。可以轻松写出：定义空列表，for遍历成绩单，判断<60的就append。但是还有一个更方便的方案，就是使用 **列表推导式** 来完成。[更多关于列表推导式](https://eastlakeside.gitbooks.io/interpy-zh/content/Comprehensions/list-comprehensions.html)
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    my_list = [34, 56, 67, 78, 95, 23, 96, 23, 86, 78, 89, 45]
    print([n for n in my_list if n <= 60])
```

也可以用同样的方式来从字典中筛选子字典。
```python
#!/usr/bin/python
# coding=utf-8


if __name__ == '__main__':
    mark_list = {
        '小明': 58,
        '小红': 94,
        '小刚': 67,
        '小智': 76,
        '小亮': 45
    }

    unpass = {key:value for key, value in mark_list.items() if value < 60}
    print(unpass)
```
