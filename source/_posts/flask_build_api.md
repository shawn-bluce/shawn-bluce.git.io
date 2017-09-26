---
title: 使用Flask设计实现一套API【成绩管理系统】
date: 2017-06-03 15:37
tags:
  - flask
  - Python
  - API
---

# 0X00 什么是REST风格的API
众所周知http协议有`GET/PUT/POST/PATCH/DELETE`等众多方法，还能在提交请求和发送响应的时候携带数据。REST风格的API就是使用了这些HTTP特性的API。针对一个URL可以有多种动词(方法)来表示不同的操作。
更多详细的内容可以点击查看阮一峰的博客：[理解RESTful架构](http://www.ruanyifeng.com/blog/2011/09/restful.html)

# 0X01 怎么选用HTTP动词
常见的动词有这五种，可以对应自己的需求选用

| 动词 | 类似的SQL关键字 | 功能 |
| ---- |: ---- :|: ---- :|
| GET | SELECT | 获取资源 |
| POST | CREATE | 创建资源 |
| PUT | UPDATE | 更新资源（需要提供改变后的完整资源） |
| PATCH | UPDATE | 更新资源（需要提供改变的属性） |
| DELETE | DELETE | 删除资源 |

# 0X02 设计URL
REST风格的API因为可以用HTTP的动词，所以URL中是不带有动词的，如果我要获取某个学生的信息应该是`[GET] http://api.example.com/student/id=12345678900`。HTTP动词理论上是能满足各种情况下的需求的，所以URL中只应该出现名词而不应该出现动词。这里用阮一峰举的例子来说明一下

| 动词 | 路径 | 功能 |
| --|--- |
| GET | /zoos | 列出所有动物园 |
| POST | /zoos | 新建一个动物园 |
| GET | /zoos/ID | 获取某个指定动物园的信息 |
| PUT | /zoos/ID | 更新某个指定动物园的信息（提供该动物园的全部信息） |
| PATCH | /zoos/ID | 更新某个指定动物园的信息（提供该动物园的部分信息） |
| DELETE | /zoos/ID | 删除某个动物园 |
| GET | /zoos/ID/animals | 列出某个指定动物园的所有动物 |
| DELETE | /zoos/ID/animals/ID | 删除某个指定动物园的指定动物 |


# 0X03 状态码

状态码是HTTP中的一大优势，一个响应可以只靠状态码来判请求结果。这些是常见的状态码，自己设计API的时候要严格按照规范来设计状态码，可以提高代码和API的可读性和可理解性。

| 状态码 | 信息 | 请求类型 | 含义 |
| ------|------|----------|---- |
| 200 | OK | [GET] | 服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。 |
| 201 | CREATED | [POST/PUT/PATCH] | 用户新建或修改数据成功。 |
| 202 | Accepted | [*] | 表示一个请求已经进入后台排队（异步任务）。 |
| 204 | NO CONTENT | [DELETE] | 用户删除数据成功。 |
| 400 | INVALID REQUEST | [POST/PUT/PATCH] | 用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。 |
| 401 | Unauthorized | [*] | 表示用户没有权限（令牌、用户名、密码错误）。 |
| 403 | Forbidden | [*] | 表示用户得到授权（与401错误相对），但是访问是被禁止的。 |
| 404 | NOT FOUND | [*] | 用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。 |
| 406 | Not Acceptable | [GET] | 用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。 |
| 410 | Gone | [GET] | 用户请求的资源被永久删除，且不会再得到的。 |
| 422 | Unprocesable entity | [POST/PUT/PATCH] | 当创建一个对象时，发生一个验证错误。 |
| 500 | INTERNAL SERVER ERROR | [*] | 服务器发生错误，用户将无法判断发出的请求是否成功。 |

# 0X04 如何用Python实现
Python有大量第三方库可以实现REST风格的API，我这里选用的是相对轻量化的一个 [Flask](http://flask.pocoo.org/)。安装这个库最简单的方式还是用pip，根据环境变量的不同可能具体命令有所不同，在我的Linux上是用`pip3 install flask`就可以直接安装好的。
安装好后进入Python的交互式界面输入`import flask`如果没有出现`Import Error`就是安装好了。

# 0X05创建数据库和表
现在可以开始设计API了。既然是成绩管理系统，那么首先就要创建一个数据库，我这里的数据库是用的MariaDB。

| 列名 | 类型 | 含义 | 键 |
| ---|------|--|--- |
| id | int | 编号 | 主键 |
| name | varchar(10) | 学生姓名| |
| number | char(11) | 学号 |  |
| python | float | Py成绩 | |
| cpp | float | c++成绩 | |
| os | float | 操作系统成绩 | |
| network | float | 计算机网络成绩 | |
| total | float | 总分 | |
| ave | float | 平均分 | |


# 0X06 创建Py脚本
```python
from flask import Flask, request

app = Flask(__name__)

# 从这里指定路径、方法、返回数据
@app.route('/', methods=['GET'])
def index():
    return '<h1>hello,world</h1>'


with app.test_request_context():
    app.run()
```
这段代码写好之后运行起来会在本地监听5000端口(默认的)，然后当你用浏览器访问`http://localhost:5000/`的时候就像你返回`<h1>hello,world</h1>`，在浏览器页面下看到的就是一行大号的hello,world。因为在浏览器的地址栏输入URL按回车之后就是向那个URL发送了GET请求，也就正好调用了`index()`方法。

这里先将与API无关的代码填好，下面开始正式完成各项功能。其实也就是连接了数据库而已
```python
from flask import Flask, request
import json
import pymysql

app = Flask(__name__)
database = pymysql.connect("db_host", "db_username", "db_password", "db_name")
cursor = database.cursor()

@app.route('/', methods=['GET'])
def index():
    return '<h1>hello,world</h1>'


with app.test_request_context():
    app.run()
```

# 0X07 实现一个构造返回Json数据的方法
首先我们选择使用Json来作为数据传输格式，因为Json相对XML来说更轻量一点，现在也更流行。规定客户端每次请求会后服务器都会返回下面这样类型的Json数据
```json
{
	"time": "unix_time",
    “e_msg": "error_message",
    "search_list": {
    	"item0": {
        	"name": "name",
            "number": "number",
            "python": "marks",
            "os": "marks",
            "network": "marks",
            "cpp": "marks",
            "total": "marks",
            "ave": "ave"
        },"item1" : {
        	"name": "name",
            "number": "number",
            "python": "marks",
            "os": "marks",
            "network": "marks",
            "cpp": "marks",
            "total": "marks",
            "ave": "ave"
        }
    }
}
```

API提供增删查改功能，增删改只通过状态码就可以判断执行结果，只有查询的时候才会需要从响应中获取数据。


# 0X08 增加一条新的数据
添加一条新数据按照标准应该使用动词`POST`，根据URL中只有名词不用动词只有名词的标准，隧将URL设计成`http://localhost/student`，再依据标准添加版本号上去，变成`http://localhost/v1/student`。
具体功能代码实现如下，
```python
@app.route('/v1/student', methods=['POST']) # 路径为/v1/student，方法为POST
def add_student():
    data = request.get_data().decode('utf-8')   # 将客户端传来的数据解码
    json_data = json.loads(data)    # 将数据转为Json

    # 从Json中获取数据
    name = json_data['name']
    number = str(json_data['number'])
    python = json_data['python']
    cpp = json_data['cpp']
    os = json_data['os']
    network = json_data['network']

    # 计算总分平均分
    total = python + cpp + os + network
    ave = total / 4

    # 查询数据库中是否有该学生的信息
    sql = "SELECT COUNT(*) FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    count = cursor.fetchall()[0][0]
    database.commit()
    if count >= 1:  # 该学生信息已经存在，返回400错误
        return build_json(e_msg="student early exist"), 400

    # 向数据库中插入数据
    sql = "INSERT INTO student.marks (name, number, python, os, network, cpp, total, ave)" \
          "VALUES (\"%s\", \"%s\", %s, %s, %s, %s, %s, %s)" % (name, number, python, os,
                                                               network, cpp, total, ave)
    cursor.execute(sql)
    database.commit()

    # 请求成功，返回201状态码
    return build_json(), 201
```

# 0X09 删除已经存在的数据
根据标准，将API设计成`DELETE`方法，URL为`http://localhost/v1/student/number=<number>`
第一行的`number=<number>`可以将url中符合这种规范的匹配出来，配合方法定义时的参数，可以直接将url参数传入到方法体中。
```python
@app.route('/v1/student/number=<number>', methods=['DELETE'])  # 路径为/v1/student, 方法为DELETE
def delete_student(number):
    if not number.isdigit():    # 判断number是否合法
        return build_json(e_msg="number should be digit"), 403
    sql = "DELETE FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    database.commit()
    return build_json(), 204
```

# 0X0A 查询学生成绩
根据标准，将API设计成`GET`方法，URL为`http://localhost/v1/student/sort_by=<sort_by>`。提供了`python/cpp/network/os/total/ave`排序方式，（其实是数据库实现的）。
```python
@app.route('/v1/student/sort_by=<sort_by>', methods=['GET'])
def show_student(sort_by):
    # 判断排序的key是否正确
    if sort_by not in ['python', 'cpp', 'os', 'network', 'total', 'ave']:
        return build_json(e_msg="sort_by key not found"), 404

    # 构建查询SQL
    sql = "SELECT name, number, python, cpp, os, network, total, ave FROM student.marks ORDER BY %s DESC" % sort_by
    cursor.execute(sql)
    database.commit()
    res = cursor.fetchall() # 获取查询结果
    
    # 构建查询结果Json
    res_list = {}
    count = 0
    for i in res:
        res_list['item' + str(count)] = {
            'name': i[0],
            'number': i[1],
            'python': i[2],
            'cpp': i[3],
            'os': i[4],
            'network': i[5],
            'total': i[6],
            'ave': [i[7]]
        }
        count += 1
    return build_json(search_list=res_list), 200
```

# 0X0B 修改学生成绩
修改学生成绩和添加成绩几乎是一样的操作，只有这么几点是不太一样的。添加信息时如果学号已经存在了那就不能再添加了，而修改的时候是如果学号不存在才错误；添加和修改的SQL不同。就没有别的区别了。
``` python
@app.route('/v1/student', methods=['PUT'])
def modify_student():
    data = request.get_data().decode('utf-8')
    json_data = json.loads(data)

    # 从Json中获取数据
    name = json_data['name']
    number = str(json_data['number'])
    python = json_data['python']
    cpp = json_data['cpp']
    os = json_data['os']
    network = json_data['network']

    # 查询数据库中是否有该学生的信息
    sql = "SELECT COUNT(*) FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    count = cursor.fetchall()[0][0]
    database.commit()
    if count < 1:  # 该学生信息不存在，返回404错误
        return build_json(e_msg="student not found"), 404

    # 计算总分平均分
    total = python + cpp + os + network
    ave = total / 4

    # 向数据库中插入数据
    sql = "UPDATE student.marks SET name=\"%s\", python=%s, cpp=%s, os=%s, network=%s, total=%s, ave=%s WHERE number=\"%s\"" % (name, python, cpp, os, network, total, ave, number)
    cursor.execute(sql)
    database.commit()

    # 请求成功，返回201状态码
    return build_json(), 201
```
# 0X0C 搞定所有API
现在就搞定了所有的API编写。现在我把所有代码贴上来，注意这段代码是用于Python3的。如果需要测试的话可以用Python自带的`requests`模块或者[Postman软件](https://www.getpostman.com)来测试该API。

* 声明：代码最后一行的`app.run()`方法，现在是只在本地监听的。可以改成`app.run('0.0.0.0')`就对外部监听了。

```python
#!/usr/bin/env python3
# coding=utf-8
# @Time    : 2017/6/3 11:34
# @Author  : Shawn
# @Blog    : https://blog.just666.cn
# @Email   : shawnbluce@gmail.com
# @purpose : 演示Python_API

from flask import Flask, request
import json
import pymysql
import time

app = Flask(__name__)
database = pymysql.connect("115.29.52.14", "shawn", "zhangHAO8", "student")
cursor = database.cursor()

def build_json(search_list=None, e_msg=None) -> str:
    json_data = {'time': time.time(), 'search_list': search_list, 'e_msg': e_msg}
    return json.dumps(json_data)

@app.route('/', methods=['GET'])
def index():
    return '<h1>hello,world</h1>'

@app.route('/v1/student', methods=['POST']) # 路径为/v1/student，方法为POST
def add_student():
    data = request.get_data().decode('utf-8')   # 将客户端传来的数据解码
    json_data = json.loads(data)    # 将数据转为Json

    # 从Json中获取数据
    name = json_data['name']
    number = str(json_data['number'])
    python = json_data['python']
    cpp = json_data['cpp']
    os = json_data['os']
    network = json_data['network']

    # 计算总分平均分
    total = python + cpp + os + network
    ave = total / 4

    # 查询数据库中是否有该学生的信息
    sql = "SELECT COUNT(*) FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    count = cursor.fetchall()[0][0]
    database.commit()
    if count >= 1:  # 该学生信息已经存在，返回400错误
        return build_json(e_msg="student early exist"), 400

    # 向数据库中插入数据
    sql = "INSERT INTO student.marks (name, number, python, os, network, cpp, total, ave)" \
          "VALUES (\"%s\", \"%s\", %s, %s, %s, %s, %s, %s)" % (name, number, python, os,
                                                               network, cpp, total, ave)
    cursor.execute(sql)
    database.commit()

    # 请求成功，返回201状态码
    return build_json(), 201

@app.route('/v1/student/number=<number>', methods=['DELETE'])  # 路径为/v1/student, 方法为DELETE
def delete_student(number):
    if not number.isdigit():    # 判断number是否合法
        return build_json(e_msg="number should be digit"), 403
    sql = "DELETE FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    database.commit()
    return build_json(), 204

@app.route('/v1/student/sort_by=<sort_by>', methods=['GET'])
def show_student(sort_by):
    # 判断排序的key是否正确
    if sort_by not in ['python', 'cpp', 'os', 'network', 'total', 'ave']:
        return build_json(e_msg="sort_by key not found"), 404

    # 构建查询SQL
    sql = "SELECT name, number, python, cpp, os, network, total, ave FROM student.marks ORDER BY %s DESC" % sort_by
    cursor.execute(sql)
    database.commit()
    res = cursor.fetchall()  # 获取查询结果

    # 构建查询结果Json
    res_list = {}
    count = 0
    for i in res:
        res_list['item' + str(count)] = {
            'name': i[0],
            'number': i[1],
            'python': i[2],
            'cpp': i[3],
            'os': i[4],
            'network': i[5],
            'total': i[6],
            'ave': [i[7]]
        }
        count += 1
    return build_json(search_list=res_list), 200

@app.route('/v1/student', methods=['PUT'])
def modify_student():
    data = request.get_data().decode('utf-8')
    json_data = json.loads(data)

    # 从Json中获取数据
    name = json_data['name']
    number = str(json_data['number'])
    python = json_data['python']
    cpp = json_data['cpp']
    os = json_data['os']
    network = json_data['network']

    # 查询数据库中是否有该学生的信息
    sql = "SELECT COUNT(*) FROM student.marks WHERE number=\"%s\"" % number
    cursor.execute(sql)
    count = cursor.fetchall()[0][0]
    database.commit()
    if count < 1:  # 该学生信息不存在，返回404错误
        return build_json(e_msg="student not found"), 404

    # 计算总分平均分
    total = python + cpp + os + network
    ave = total / 4

    # 向数据库中插入数据
    sql = "UPDATE student.marks SET name=\"%s\", python=%s, cpp=%s, os=%s, network=%s, total=%s, ave=%s WHERE number=\"%s\"" \
          % (name, python, cpp, os, network, total, ave, number)
    cursor.execute(sql)
    database.commit()

    # 请求成功，返回201状态码
    return build_json(), 201

with app.test_request_context():
    app.run()
```



