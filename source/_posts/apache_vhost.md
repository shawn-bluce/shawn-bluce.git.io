---
title: Apache配置虚拟主机 VirtualHost 多站点
date: 2016-05-08 15:30
tags:
  - Apache
  - Web服务
  - 虚拟主机
  - PHP
---


如果我们只有一台服务器，应该怎么实现让这台服务器同时处理PHP和JSP的请求？
这里的解决方案是通过Apache的虚拟主机（vhost）来进行端口转发。
Apache会通过访问服务器的域名将请求转发至不同的端口或者不同的服务器。

# 0X00 前提&目的
前提：
　　拥有一个域名，并有两个A解析，同时解析到这台服务器的IP
　　分别拥有一个JSP和PHP的页面（网站）
目的：
　　使用php.test.com访问的时候解析到PHP的网站上
　　使用jsp.test.com访问的时候解析到JSP的网站上
操作系统：
    Centos 7.x 如果是之前的版本或是其他系统可能出现不同的情况

# 0X01 安装httpd  (Apache)
安装并启动服务
```
yum install httpd
systemctl start httpd.service
```

# 0X02 安装PHP
```
yum install php
```

# 0X03 安装JDK用来配合JSP
```
yum install java-1.8.0-openjdk
```

# 0X04 安装tomcat用于解析JSP页面
```
yum install tomcat tomcat-webapps tomcat-admin-webapps
systemctl start tomcat.service
```
# 0X05 配置httpd用于同时支持PHP和JSP
打开配置文件
```
vim /etc/httpd/conf/httpd.conf
```
在配置文件的最前端添加如下内容
```xml
NameVirtualHost *:80

<VirtualHost *:80>
        ServerName php.test.com #指定一个域名
        DocumentRoot /var/www/html #PHP网站的位置
        ErrorLog logs/php.test.com-error.log #日志位置
        CustomLog logs/php.test.com-access.log common #日志位置
</VirtualHost>

<VirtualHost *:80>
        ServerName jsp.test.com #指定另一个域名
        DocumentRoot /var/lib/tomcat/webapps/ROOT #JSP网站的位置
        ErrorLog logs/jsp.test.com-error.log  #日志位置
        CustomLog logs/jsp.test.com-access.log common #日志位置
        ProxyPass / http://127.0.0.1:8080/ #转发位置
        ProxyPassReverse / http://127.0.0.1:8080/ #转发位置
</VirtualHost>
```

# 0X06 最后
```
systemctl restart httpd.service
systemctl restart tomcat.service
```
现在就可以使用php.test.com 和 jsp.test.com分别访问到PHP和JSP的页面了
