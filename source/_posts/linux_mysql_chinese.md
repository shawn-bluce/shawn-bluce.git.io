---
title: 解决Linux下MySQL/MariaDB中文变问号？问题
date: 2016-12-15 19:33
tags:
  - Linux
  - MySQL
  - MariaDB
  - 数据库
  - 编码
  - 中文
---

# 0X00 修改配置文件
MySQL/MariaDB默认并没有采用utf-8编码，所以我们要修改配置文件，以让其使用utf-8。
`/etc/my.cnf`就是配置文件，打开之后在`[mysqld]`下面加入两行，使其变成
```config
[mysqld]
character_set_server=utf8 
init_connect='SET NAMES utf8'
```
修改好配置文件之后重启服务

# 0X01 修改数据库的字符集
在修改配置文件之后新建的数据库默认就是使用utf-8了，但是之前的还不是所以要修改一下。登录到数据库，在命令行界面修改数据库的字符集。
```sql
ALTER DATABASE `databases_name` COLLATE 'utf8_bin';
```
再次重启数据库服务。这样再连接到数据库就解决掉汉字变问号的问题了

# 0X02 推荐两款软件
大家好多人都在用Navicat，但绝大多数人用的都是盗版软件，这里推荐大家用一些好用的开源软件来替代。
### 1 HeidiSQL
1. 一款开源软件
2. 可以连接MySQL/MariaDB/SQL Server
3. 官方中文支持

[下载地址：HeidiSQL](http://www.heidisql.com/)

### 2 MySQL Workbench
1. 一款开源软件
2. MySQL官方开发
3. 导出表关系图非常强大

[下载地址：MySQL Workbench](http://www.mysql.com/products/workbench/)
