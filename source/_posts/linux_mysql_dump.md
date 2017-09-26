---
title: Linux下MariaDB/MySql的安装配置、用户管理和备份
date: 2016-11-20 22:17
tags:
  - Linux
  - MySQL
  - MariaDB
  - 数据库
  - 用户管理
  - 服务安装
  - 数据备份
---


# 0x00  MariaDB的身世

自从MySQL被Oracle收购之后，社区就一直担心MySQL可能会被闭源或者一些其他的原因导致MySQL的支持出现问题。所以现在好多发行版本默认的数据库都从MySQL转移到了Mariadb。而且社区也开始大力支持Mariadb，再加上Mariadb的使用和API和MySQL完全一样，所以这里选择使用Mariadb而不是MySQL。
>MariaDB数据库管理系统是MySQL的一个分支，主要由开源社区在维护，采用GPL授权许可。开发这个分支的原因之一是：甲骨文公司收购了MySQL后，有将MySQL闭源的潜在风险，因此社区采用分支的方式来避开这个风险。
MariaDB的目的是完全兼容MySQL，包括API和命令行，使之能轻松成为MySQL的代替品。在存储引擎方面，10.0.9版起使用XtraDB（名称代号为Aria）来代替MySQL的InnoDB。
MariaDB由MySQL的创始人麦克尔·维德纽斯主导开发，他早前曾以10亿美元的价格，将自己创建的公司MySQL AB卖给了SUN，此后，随着SUN被甲骨文收购，MySQL的所有权也落入Oracle的手中。MariaDB名称来自麦克尔·维德纽斯的女儿玛丽亚（英语：Maria）的名字。——————————维基百科

# 0X01 安装Mariadb

MariaDB是一组软件，如果只安装一部分的话后期扩展可能会出现问题，所以我们可以一次安装整个软件组
```bash
[root@iZ28jaak5nnZ ~]# yum groupinstall mariadb mariadb-client -y
```
安装需要一点时间，我们只需要等待安装结束。

# 0X02 打开Mariadb服务并配置防火墙
启动Mariadb服务。在CentOS7.x中推荐使用systemctl来配置服务的启动方式
```bash
systemctl start mariadb.service
```

配置防火墙，允许从MariaDB使用的3306端口监听，由于历史遗留问题，这里还是称之为MySql。
```bash
[root@iZ28jaak5nnZ ~]# firewall-cmd --add-service=mysql
success
[root@iZ28jaak5nnZ ~]# firewall-cmd --list-services
dhcpv6-client mysql ssh
```

# 0X03 配置MariaDB的安全性
MariaDB提供了一个脚本来为新安装的MariaDB提升安全性。但是在使用这个脚本之前必须要先打开MariaDB服务。
```bash
[root@iZ28jaak5nnZ ~]# systemctl start mariadb
[root@iZ28jaak5nnZ ~]# systemctl enable mariadb
ln -s '/usr/lib/systemd/system/mariadb.service' '/etc/systemd/system/multi-user.target.wants/mariadb.service'
```
然后运行这个脚本，这个脚本会有几次提示：
1. 询问当前密码，如果没设置密码就直接回车
2. 设置root用户的密码
3. 删除匿名用户(anonymous-user)
4. 删除可以从外部登陆的root用户
5. 删除test测试数据库
6. 重载数据库

```bash
[root@iZ28jaak5nnZ ~]# mysql_secure_installation 
/usr/bin/mysql_secure_installation: line 379: find_mysql_client: command not found

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!
```

# 0X04 登陆到MariaDB
配置好密码和接入点之后就可以登录到MariaDB了。使用mysql命令来登陆MariaDB。
```bash
[root@iZ28jaak5nnZ ~]# mysql -h localhost -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 12
Server version: 5.5.50-MariaDB MariaDB Server

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
```
参数：-h 指定主机， -u 指定用户，　-p 指定密码
这三个参数都可以省略，当我们省略主机名的时候就默认登录到本地，当省略用户名的时候默认使用root，当省略密码的时候默认没有密码登陆

# 0X05 用户管理
在MariaDB中有用户的概念和权限的概念。用户名+密码+登陆地点，三个选项唯一确定一个用户，就比如同一个用户名`shawn`在10.13.1.2和在10.13.1.3的登陆密码可以是不同的，这在MariaDB里会分成两条来存储。
```sql
创建一个名为shawn的，从localhost登陆的，密码为test的用户
MariaDB [(none)]> CREATE USER 'shawn'@'localhost' IDENTIFIED BY 'test';
Query OK, 0 rows affected (0.00 sec)

创建一个名为shawn_test的，任意地点，密码为6666的用户
MariaDB [(none)]> CREATE USER 'shawn_test'@'%' IDENTIFIED BY '6666';
Query OK, 0 rows affected (0.00 sec)
```

而且在MariaDB中用户和权限是分开的，如果只添加一个用户的话，这个用户是没有任何权限的。

删除用户的话是使用`DROP`命令
```sql
MariaDB [(none)]> DROP USER heiheihei@'localhost';
Query OK, 0 rows affected (0.00 sec)
```

# 0X06 权限管理
登录到MariaDB之后可以给用户添加具体的权限，使用`GRANT`命令。
```sql
允许从localhost登陆的shawn对mysql数据库的user表执行查询
MariaDB [(none)]> GRANT SELECT ON mysql.user to shawn@'localhost';
Query OK, 0 rows affected (0.00 sec)

允许从localhost登陆的shawn对mysql数据库的user表执行查询和插入
MariaDB [(none)]> GRANT SELECT, INSERT ON mysql.user to shawn@'localhost';
Query OK, 0 rows affected (0.00 sec)

允许从localhost登陆的shawn对mysql数据库的user表执行增删查改四种操作
MariaDB [(none)]> GRANT ALL ON mysql.user to shawn@'localhost';
Query OK, 0 rows affected (0.00 sec)

允许从localhost登陆的shawn对mysql数据库的所有表执行增删查改四种操作
MariaDB [(none)]> GRANT ALL ON mysql.* to shawn@'localhost';
Query OK, 0 rows affected (0.00 sec)

允许从localhost登陆的shawn对所有库的所有表执行增删查改四种操作
MariaDB [(none)]> GRANT ALL ON *.* to shawn@'localhost';
Query OK, 0 rows affected (0.00 sec)

允许从任意地点登陆的shawn对所有库的所有表执行增删查改四种操作
MariaDB [(none)]> GRANT ALL ON *.* to shawn@'%';
Query OK, 0 rows affected (0.00 sec)
```

使用`REVOKE`可以删除给定的权限，使用方法和`GRANT`是一样的，只是开头不同而已
使用`FLUSH PRIVILEGES;`可以刷新权限信息
使用`SHOW GRANT FOR root@localhost`可以查看某用户的权限信息

# 0X07 数据库备份
MariaDB有逻辑备份和物理备份两种备份方案，逻辑备份就是可以把表结构数据等导出成sql文件，而物理备份就是直接备份文件。
逻辑备份比较慢，因为要将备份的数据全部都查询一遍，但是可以不下线备份；物理备份比较快，但是需要下线备份。这里说的是逻辑备份.
```bash
[root@iZ28jaak5nnZ ~]# mysqldump -u root -h localhost -p --all-databases > backup.sql
Enter password:
```
这里我使用了一个`--all-databases`的参数，是备份所有数据库，可选的参数有下面这几个
1. `--all-databases`备份所有数据库
2. `--add-drop-tables`生成的sql中包含drop tables语句，删除以前的table
3. `--no-data`只生成库和表结构，没有数据
4. `--lock-all-tables`在备份结束之前，锁定所有表，保证数据完整性
5. `--add-drop-databases`生成的sql中包含drop database语句，删除以前的database


# 0X08 数据库还原
当我们有了一个备份出来的sql文件之后，可以将这个sql直接导入到数据库。这了的用法和之前登录到MariaDB的方法是一样的，只是将sql文件重定向过去就可以了
```sql
[root@iZ28jaak5nnZ ~]# mysql -u root -h localhost -p < backup.sql
Enter password:
```
