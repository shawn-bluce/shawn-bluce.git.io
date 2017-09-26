---
title: Mariadb/MySQL 增删查改 数据库操作 建表 建数据库
date: 2016-03-15 17:30
tags:
  - MySQL
  - MariaDB
  - 数据库
---


首先需要安装好MySQL/Mariadb的服务端和客户端，并且能连接到服务端
>命令中的大写字母是SQL的关键字，小写字母是自己的相关属性和数据

# 0X00 连接到数据库
```
使用mysql连接到127.0.0.1并用root用户登陆，密码等待输入
mysql -h 127.0.0.1 -u root -p
```

# 0X01 创建数据库
```sql
创建一个名为school的数据库
CREATE DATABASE school;
```

# 0X02 建立一个表
建立一个名为student的表
索引：
10个字符长度的name   不能为空
11个字符长度的number 不能为空
int类型的age        不能为空
```sql
use school;
使用school这个数据库

CREATE TABLE student(
name VARCHAR(10) NOT NULL,
number VARCHAR(11) NOT NULL,
age INT NOT NULL,
PRIMARY KEY (number)
);
```

# 0X03 查询数据库和表
```sql
SHOW DATABASES;
查看所有数据库

SHOW BALES;
查看正在使用的数据库中的表
```

# 0X04 插入数据
```sql
INSERT INTO student VALUES('lilei','666',15);
插入新的数据，按照顺序写

INSERT INTO student (name)VALUES('hanmeimei');
自定义顺序写入
```

# 0X05 查询数据
```sql
SELECT name FROM student;
查询student表中的所有name

SELECT name FROM student WHERE number=0002;
查询student表中number为0002的name

SELECT name FROM student WHERE age BETWEEN 20 and 30;
查询student表中age在20到30之间的name
```

# 0X06 更新数据
```sql
UPDATE student SET name='xiaohei' WHERE number='0002';
把所有number为0002的name更新为xiaohei
```

# 0X07 删除数据
```sql
DELETE FROM student WHERE number='0002'
删除所有number为0002的数据
```
