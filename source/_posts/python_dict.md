---
title: Python之字典
date: 2016-09-13 20:03
---


# 0X00 什么是字典
字典，顾名思义就是通过一个条件可以找到相应的值，字典由Key-Value组成。像是下面这样创建一个字典
字典中的数据是没有顺序的，不像列表一样有顺序，在字典中是没有固定顺序的
```python
>>> a = {'name':'xiaoming', 'sex':'F', 'age':22}  #直接创建一个字典
>>> print a
    {'age': 22, 'name': 'xiaoming', 'sex': 'F'}
>>> print a['name']
    xiaoming

>>> b = dict(name='xiaogang', sex='M', age=23)	 #通过dict函数创建一个字典
>>> print b['name']
    xiaogang
```
> 下文说的Key就是键， Value就是值
> Key-Value 就是键值对，一个键对应着一个值
> Key的值是可以随意改变的，但是Key的类型是固定的不能改变
> 如果为一个不存在的键赋值，那么会自动添加这个K-V

# 0X01 字典操作
## len 测量长度
测量这个字典中有多少
```python
>>> d = {'username':'admin', 'password':'123456')
>>> print len(d)
	2
```

## d[k] 调用字典
根据已知的Key来查找Key所对应的Value
```python
>>> d = {'username':'admin', 'password':'123456'}
>>> print d['username']
	admin
```

## d[k] = v	字典赋值
为某个特定的Key赋值，如果这个Key在字典中不存在则创建这个Key
```python
>>> d = {'username':'admin', 'password':'123456'}
>>> d['password'] = '2336666'
>>> print d['password']
	2336666
```

## del d[k] 删除Key-Value
删除相应的Key和Value
```python
>>> d = {'username':'admin', 'password':'123456'}
>>> del d['username']
>>> print d
	{'password': '123456'}
>>> del d['username']	#删除一个不存在的K-V会抛出异常
    Traceback (most recent call last):
      File "<stdin>", line 1, in <module>
    KeyError: 'username'
```

## k in d 成员检查
检查某字典中是否存在某Key，成员检查时用的是Key而不是Value
```python
>>> d = {'username':'admin', 'password':'123456'}
>>> 'username' in d
	True
>>> 'phonenumber' in d
	False
```

# 0X02 字典的递归
字典中是Key-Value，然而字典的Value是可以是字典的，也就是Key-Value的Value是字典，也就是递归。
这样就可以建立一个递归的字典，字典里的字典可以是一层一层递归包括下去
```python
>>> phones = {
        'xiaoming':{
        'num':'123',
        'addr':'hebei'
        },
        'xiaohua':{
        'num':'456',
        'addr':'sichuan'
        }
    }
>>> print phones
    {'xiaoming': {'num': '123', 'addr': 'hebei'}, 'xiaohua': {'num': '456', 'addr': 'sichuan'}}
>>> print phones['xiaoming']
    {'num': '123', 'addr': 'hebei'}
>>> print phones['xiaoming']['addr']
    hebei
```

# 0X03 字典方法
字典有好多方法可以调用，对字典进行操作
## clear 清除
clear可以清除字典所有项，这项操作是直接操作原来的字典而不是修改字典然后返回新的字典
下面展示一下这个方法的用处
A
```python
>>> x = {}
>>> y = x
>>> x['key'] = 'value'
>>> y
	{'key': 'value'}
>>> x = {}
>>> y
	{'key': 'value'}
```
B
```python
>>> x = {}
>>> y = x
>>> x['key'] = 'value'
>>> y
	{'key': 'value'}
>>> x.clear()
>>> y
	{}
```
对比A和B这两种情况，就大概知道什么时候clear方法可以发挥用处了

## copy 复制
可以复制一个全新的字典出来，并返回这个新的字典。字典的复制分为浅度复制和深度复制
浅度复制
```python
>>> x = {'username':'admin', 'machines':['foo', 'bar', 'baz']}
>>> y = x.copy()
>>> y['username'] = 'mlh'
>>> y['machines'].remove('bar')
>>> y
    {'username': 'mlh', 'machines': ['foo', 'baz']}
>>> x
    {'username': 'admin', 'machines': ['foo', 'baz']}
```

深度复制
```python
>>> from copy import deepcopy
>>> d = {}
>>> d['names'] = ['Alfred', 'Bertrand']
>>> c = d.copy()
>>> dc = deepcopy(d)
>>> d['names'].append('Clive')
>>> c
    {'names': ['Alfred', 'Bertrand', 'Clive']}
>>> dc
    {'names': ['Alfred', 'Bertrand']}
```

## fromkeys 空字典
使用给定的键来建立一个没有值的字典，也可以给一个默认的值让所有键的值都是这个默认值
```python
>>> dict.fromkeys(['name', 'age'])	#一个纯空的字典
    {'name': None, 'age':None}
>>> dict.fromkeys(['name', 'age'], 'unknow')   #给键创建一个默认值
    {'name':'unknow', 'age':'unknow'}
```

## get 获取
比较宽松的获取数据，以前用d[k]的方式调用一个值的话，如果这个键不存在就会抛出异常，用get获取就不会这样
```python
>>> d = {}
>>> print d.get('name')  #获取一个不存在的数据，不会抛出异常，而显示None
    None
```

## has_key 判断键
判断字典中是否有这个键，返回True和False
```python
>>> d = {'name':'admin', 'password':'123456'}
>>> d.has_key('name')
    True
>>> d.has_key('hehe')
    False
```

## items iteritems 返回字典
items 可以将整个字典转化成列表并返回
iteritems 可以将整个字典转化成迭代器返回
```python
>>> d = {'username':'admin', 'password':'123'}
>>> d.items()
    [('username', 'admin'), ('password', '123')]
>>> d.iteritems()
    <built-in method iteritems of dict object at 0x7f62ac08e6e0>
>>> list(d.iteritems())
    [('username', 'admin'), ('password', '123')]
```

## keys iterkeys 返回键
keys 以列表的方式返回整个字典中所有的key
iterkeys 以迭代器的方式返回整个字典中所有的key
```python
>>> d = {'username':'admin', 'password':'123'}
>>> d.keys()
    ['username', 'password']
>>> d.iterkeys()
    <dictionary-keyiterator object at 0x7f62ac0972b8>
```

## pop 出栈
因为字典中是没有顺序的，所以出栈的时候必须自己指定一个Key才能弹出这个K-V，如果了解“栈”这个数据结构的话就能非常清晰这个方法。使用pop弹出一个数据的时候回在原字典中删除这个数据并返回这个数据。
```python
>>> d = {'username':'admin', 'password':'123'}
>>> d.pop('password')
    '123'
>>> d
    {'username': 'admin'}
```

## popitem 随机出栈
随机从字典中弹出一组K-V
```python
>>> d = {'one':'1', 'two':'2', 'three':'3', 'four':'4', 'five':'5', 'six':'6'}
>>> d.popitem()
    ('six', '6')
>>> d.popitem()
    ('three', '3')
>>> d.popitem()
    ('two', '2')
>>> d.popitem()
    ('four', '4')
>>> d.popitem()
    ('five', '5')
>>> d.popitem()
    ('one', '1')
```

## setdefault 设置默认
给字典中某个Key设定一个默认的Value，当这个Key没有Value的时候就默认为那个默认的Value，如果有数据则默认Value不会生效
```python
>>> d = {}
>>> d.setdefault('name', 'N/A')
    'N/A'
>>> d
    {'name': 'N/A'}
>>> d['name'] = 'admin'
>>> d.setdefault('name', 'N/A')
    'admin'
>>> d
    {'name': 'admin'}
```

## update 更新字典
可以用一个新的字典去更新旧的字典，新旧字典中Key重合的部分以新字典为准，旧字典中有的Key且新字典中没有的话则不变，旧字典中没有的Key且新字典中有的话则添加这个K-V
```python
>>> a = {'username':'admin', 'password':'123456'}
>>> b = {'password':'2336666', 'sex':'F'}
>>> a.update(b)
>>> a
    {'username': 'admin', 'password': '2336666', 'sex': 'F'}
```

## values intervalues
values 返回字典中所有的值组成的列表
intervalues 返回字典中所有的值相应的迭代器
