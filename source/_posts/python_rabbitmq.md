---
title: 使用Python操作消息队列RabbitMQ
date: 2017-06-13 20:56
tags:
  - Python
  - RabbieMQ
  - 消息队列
  - demo
---

内容参考自[python - 操作RabbitMQ](http://www.cnblogs.com/pangguoping/p/5720134.html)

# 0X00 安装环境
首先是在Linux上安装rabbitmq
```bash
# 环境为CentOS 7
yum install rabbitmq-server	# 安装RabbitMQ
systemctl start rabbitmq-server	# 启动
systemctl enable rabbitmq-server	# 开机自启
systemctl stop firewall-cmd		# 临时关闭防火墙
```

然后用pip安装Python3的开发包
```bash
pip3 install pika
```

安装好软件之后可以访问`http://115.xx.xx.xx:15672/`来访问自带的web页面来查看和管理RabbitMQ。默认管理员的用户密码都是`guest`

# 0X01 简单的向队列中加入消息
```python
#!/usr/bin/env python3
# coding=utf-8
# @Time    : 2017/6/13 19:25
# @Author  : Shawn
# @Blog    : https://blog.just666.cn
# @Email   : shawnbluce@gmail.com
# @purpose : RabbitMQ_Producer

import pika

# 创建连接对象
connection = pika.BlockingConnection(pika.ConnectionParameters(host='115.xx.xx.xx'))

# 创建频道对象
channel = connection.channel()

# 指定一个队列，如果该队列不存在则创建
channel.queue_declare(queue='test_queue')

# 提交消息
for i in range(10):
    channel.basic_publish(exchange='', routing_key='test_queue', body='hello,world' + str(i))
    print("sent...")

# 关闭连接
connection.close()
```

# 0X02 简单的从队列中获取消息

```python
#!/usr/bin/env python3
# coding=utf-8
# @Time    : 2017/6/13 19:40
# @Author  : Shawn
# @Blog    : https://blog.just666.cn
# @Email   : shawnbluce@gmail.com
# @purpose : RabbitMQ_Consumer

import pika

credentials = pika.PlainCredentials('guest', 'guest')

# 连接到RabbitMQ服务器
connection = pika.BlockingConnection(pika.ConnectionParameters('115.xx.xx.xx', 5672, '/', credentials))
channel = connection.channel()

# 指定一个队列，如果该队列不存在则创建
channel.queue_declare(queue='test_queue')

# 定义一个回调函数
def callback(ch, method, properties, body):
    print(body.decode('utf-8'))


# 告诉RabbitMQ使用callback来接收信息
channel.basic_consume(callback, queue='test_queue', no_ack=False)

print('waiting...')

# 开始接收信息，并进入阻塞状态，队列里有信息才会调用callback进行处理。按ctrl+c退出。
channel.start_consuming()
```
# 0X03 万一消费者掉线了
想象这样一种情况：

消费者从消息队列中获取了n条数据，正要处理呢结果宕机了，那该怎么办？在RabbieMQ中有一个ACK可以用来确认消费者处理结束。就有点类似网络中的ACK，消费者每次从队列中获取了数据之后队列不会立刻将数据移除，而是等待对应的ACK。消费者获取到数据并处理完成之后会向队列发送一个ACK包，通知RabbitMQ这堆消息已经处理妥当了，可以删除了，这时候RabbitMQ才会将数据从队列中移除。所以这种情况下即使消费者掉线也没有什么问题，数据依旧会在队列中存在，留给其他消费者处理。

在Python中这样实现：

消费者有这样一行代码`channel.basic_consume(callback, queue='test_queue', no_ack=False)`，其中`no_ack=False`表示不发送确认包。将其修改为`no_ack=True`就会在每次处理完之后向RabbitMQ发送一个确认包，以确认消息处理完毕。

# 0X04 万一RabbitMQ宕机了呢
虽然有了ACK包，但是万一RabbitMQ挂了那数据还是会损失。所以我们可以给RabbitMQ设置一个数据持久化存储。RabbitMQ会将数据持久化存储到磁盘上，保证下次再启动的时候队列还在。

在Python中这样实现：

我们声明一个队列是这样的`channel.queue_declare(queue='test_queue')`，如果需要持久化一个队列可以这样声明`channel.queue_declare(queue='test_queue', durable=True)`。不过这行直接放在代码中是不能执行的，因为以前已经有了一个名为`test_queue`的队列，RabbitMQ不允许用不同的方式声明同一个队列，所以可以换一个队列名新建来指定数据持久化存储。不过如果只是这样声明的话，在RabbitMQ宕机重启后确实队列还在，不过队列里的数据就没有了。除非我们这样来声明队列`channel.basic_publish(exchange='', routing_key="test_queue", body=message, properties=pika.BasicProperties(delivery_mode = 2,))`。

# 0X05 最简单的发布订阅
最简单的发布订阅在RabbitMQ中称之为`Fanout模式`。也就是说订阅者订阅某个频道，然后发布者向这个频道中发布消息，所有订阅者就都能接收到这条消息。不过因为发布者需要使用订阅者创建的随机队列所以需要先启动订阅者才能启动发布者。

发布者代码：
```python
#!/usr/bin/env python3
# coding=utf-8
# @Time    : 2017/6/13 20:21
# @Author  : Shawn
# @Blog    : https://blog.just666.cn
# @Email   : shawnbluce@gmail.com
# @purpose : RabbitMQ_Publisher

import pika

# 创建连接对象
connection = pika.BlockingConnection(pika.ConnectionParameters(host='115.xx.xx.xx'))

# 创建频道对象
channel = connection.channel()

# 定义交换机，exchange表示交换机名称，type表示类型
channel.exchange_declare(exchange='my_fanout',
                         type='fanout')

message = 'Hello Python'
# 将消息发送到交换机
channel.basic_publish(exchange='my_fanout',  # 指定exchange
                      routing_key='',  # fanout下不需要配置，配置了也不会生效
                      body=message)
connection.close()
```

订阅者代码：
```python
#!/usr/bin/env python3
# coding=utf-8
# @Time    : 2017/6/13 20:20
# @Author  : Shawn
# @Blog    : https://blog.just666.cn
# @Email   : shawnbluce@gmail.com
# @purpose : RabbitMQ_Subscriber

import pika

credentials = pika.PlainCredentials('guest', 'guest')

# 连接到RabbitMQ
connection = pika.BlockingConnection(pika.ConnectionParameters('115.xx.xx.xx', 5672, '/', credentials))
channel = connection.channel()

# 定义交换机，进行exchange声明，exchange表示交换机名称，type表示类型
channel.exchange_declare(exchange='my_fanout',
                         type='fanout')

# 随机创建队列
result = channel.queue_declare(exclusive=True)  # exclusive=True表示建立临时队列，当consumer关闭后，该队列就会被删除
queue_name = result.method.queue

# 将队列与exchange进行绑定
channel.queue_bind(exchange='my_fanout',
                   queue=queue_name)

# 定义回调方法
def callback(ch, method, properties, body):
    print(body.decode('utf-8'))


# 从队列获取信息
channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```
