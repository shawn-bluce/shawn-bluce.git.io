---
title: Shadowsocks 如何科学上网 搭梯子 简明教程
date: 2016-07-23 12:12
tags:
  - Shadowsocks
  - 梯子
  - 代理
  - VPS
---

# 0X00 准备工作
1.一台海外或者香港的服务器/虚拟主机（后面统称VPS），要有独立IP
2.VPS的带宽和流量不能太小
3.一个连接VPS的软件，LInux/Mac可以用终端，Windows用户可以用XShell或者putty
4.VPS要使用Linux系统，Debian/Ubuntu/CentOS都行

>开工之前最好有Linux适用基础
>没有VPS的推荐一个购买地址，便宜好用[banwagong](http://banwagong.cn/gonglue.html)
>这个网站不是官网，但是起到了类似中文官网的作用，可以按照里面的推荐和教程去购买适合自己的VPS
>VPS买回来不止可以干这个、配置高一点的话还可以搭建一个独立博客和一些其他的服务

# 0X01 简述工作原理
**不通过伟大防火墙时**我们访问某网站，流量从我们的机器一路跑到网站服务器，然后服务器响应数据再一路跑回来。
现在**有了伟大的防火墙**不让我们和某些网站交流了，我们可以搭一个**梯子**，让流量通过梯子。其实用**镜子**比喻会更好一点。
**有了镜子**之后，我们的流量一路跑到镜子那里，镜子替我们将流量一路跑到网站服务器，然后网站服务器将数据一路发送到镜子，镜子再转发给我们。
所以造成下面几个问题：
1.你终端（电脑、手机等设备）产生的数据流量（代理流量）都要从梯子那里经过，所以梯子也要走一份流量。
2.你的网速同时取决于 你的速度、VPS的速度、网站服务器的速度
3.你的延迟同时取决于 你到VPS的延迟，VPS到网站服务器的延迟

# 0X02 安装软件
Debian/Ubuntu
```bash
sudo apt-get update				#更新系统
sudo apt-get install python-pip #安装Python-pip
sudo pip install shadowsocks 	#安装shadowsocks
```

CentOS
```bash
sudo yum update					#更新系统
sudo yum install python-setuptools && easy_install pip
sudo pip install shadowsocks	#安装shadowsocks
```

# 0X03 修改配置文件
配置文件默认不存在，我们直接创建一个就行`vim /etc/shadowsocks.json`
这里配置文件使用Json解析，看起来很清晰，便于识别修改
```json
{
	    "server":"my_server_ip",
	    "server_port":8388,
	    "local_address": "127.0.0.1",
	    "local_port":1080,
	    "password":"mypassword",
	    "timeout":300,
	    "method":"aes-256-cfb",
	    "fast_open": false
}
```
>server 修改成你VPS的外网ip
>server_port 是服务端用的端口，没有特殊需要就不用改了
>local_address 本地地址，使用默认的127.0.0.1就行
>local_port 客户机端口，使用默认的1080就行
>password 设置密码
>timeout 超时时间，使用默认即可
>method 加密算法，使用默认aes-256-cfb即可，改用rc4-md5也行，不过客户端也要跟着改
>fast_open 默认即可

# 0X04 如何开启关闭服务
这样开启服务`ssserver -c /etc/shadowsocks.json -d start`
这样关闭服务`ssserver -c /etc/shadowsocks.json -d stop`
这样重启服务`关了再开就是重启+_+`

### 0X04 下载客户端
[点击下载WIndows环境下的Shadowsocks客户端](http://o7bn7vqpt.bkt.clouddn.com/%2Fdownload%2Fshadowsocks-windows.7z)
[点击下载Android环境下的Shadowsocks客户端](http://o7bn7vqpt.bkt.clouddn.com/%2Fdownload%2Fshadowsocks-android.7z)
IOS版本的客户端在AppStore里有，不过要收费。也有免费的解决方案，因为不用IOS所以不清楚，自己去找找吧。

# 0X05 优化速度
**注意：**
1.前提是你的VPS限制流量但不限制带宽且你有足够的流量
2.所谓速度优化只针对大文件下载和在线视频有明显效果
3.速度优化之后会双倍流量发送，所以只有流量充足的用户适用

Debian/Ubuntu:
```bash
wget --no-check-certificate https://raw.githubusercontent.com/tennfy/debian_netspeeder_tennfy/master/debian_netspeeder_tennfy.sh
chmod a+x debian_netspeeder_tennfy.sh
bash debian_netspeeder_tennfy.sh
```

CentOS:
```bash
wget --no-check-certificate https://gist.github.com/LazyZhu/dc3f2f84c336a08fd6a5/raw/d8aa4bcf955409e28a262ccf52921a65fe49da99/net_speeder_lazyinstall.sh
sudo sh net_speeder_lazyinstall.sh
```

启动加速`	nohup /usr/local/net_speeder/net_speeder venet0 "ip" >/dev/null 2>&1 &`
