---
title: 使用nmcli和ip命令配置CentOS/RHEL的网络
date: 2016-10-28 17:42
tags:
  - Linux
  - nmcli
  - ip
  - 网络
---

# 0X00 查看网络配置文件
在CentOS中网络是以配置文件的形式存在系统里的，在`/etc/sysconfig/network-scripts/`目录下，一般情况下网卡的配置文件都在这里了，以`ifcfg-`就是配置文件了，打开配置文件看一下。下面注释一下关键的配置项
```config
TYPE=Ethernet	# 网络类型
BOOTPROTO=static	# 协议取值，常见的是static和dhcp
IPADDR=10.13.7.33	# 给网卡ip赋值
NETMASK=255.255.255.0	# 给网卡子网掩码赋值
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s8
USERCTL=no	# 是否允许非root用户控制
UUID=4c967913-c4c9-4961-ae03-de7865f144d0	# 网卡的唯一标识码
DEVICE=enp0s8	# 设备名
ONBOOT=no	# 是否在开机时激活
```
但是一般不建议直接使用编辑器修改网络配置文件，因为这样容易出现一些语法错误和逻辑错误，所以建议使用命令行来管理配置网络，虽然本质上都是去修改配置文件。但是使用命令行去管理网络，命令都是确保配置没有问题才会写入到文件，所以会更加安全。包括下面介绍的`ip`和`nmcli`命令，都是通过修改配置文件来完成功能的。

# 0X01 ifconfig 命令
这个命令在CentOS7中已经不建议使用了，不过由于之前的版本都是在用这个命令，还是说一下。`ifconfig`是`interface configuration`的缩写，也就是接口配置。
## 查看网络
直接输入这个命令就可以看到现在启动着的所有网络。也可以接上某个特定的网卡来查看单独的信息`ifconfig enp0s3`
```bash
[root@localhost ~]# ifconfig
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
如果想查看包括已经关闭了的网络时，使用`ifconfig -a`就可以了
> 最后的那个lo是回环网络，暂时不用管

## 开关网络
`ifconfig`还可以开关网络，命令后面接`interface name`也就是网卡名，然后接上`up/down`就可以开关网络了
```bash
[root@localhost ~]# ifconfig enp0s8 down	# 关闭网络
[root@localhost ~]# ifconfig
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig enp0s8 up	# 打开网络
[root@localhost ~]# ifconfig
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 123.123.123.2  netmask 255.255.255.128  broadcast 123.123.123.1
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 0  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

## 配置网络
`ifconfig`命令可以在不重启的情况下开关网络接口，修改IP、掩码、网关等信息。
```bash
[root@localhost ~]# ifconfig enp0s8	# 查看ep0s8的网卡信息
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig enp0s8 123.233.233.123	# 修改ip为123.233.233.123
[root@localhost ~]# ifconfig enp0s8
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 123.233.233.123  netmask 255.0.0.0  broadcast 123.255.255.255
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

[root@localhost ~]# ifconfig enp0s8 netmask 255.255.255.0	# 修改子网掩码为255.255.255.0
[root@localhost ~]# ifconfig enp0s8
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 123.233.233.123  netmask 255.255.255.0  broadcast 123.233.233.255
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
[root@localhost ~]# ifconfig enp0s8 123.123.123.2 netmask 255.255.255.128	# 当然也可以把这些写成一行
[root@localhost ~]# ifconfig enp0s8
enp0s8: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 123.123.123.2  netmask 255.255.255.128  broadcast 123.123.123.
        ether 08:00:27:79:c4:b1  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

# 0X02 nmtui 简单的类图形管理工具
在终端中输入`nmtui`就可以打开一个类图形的界面，用这个界面可以更简单得管理配置网络，但是不能做比较细致的配置，而且使用比较简单，所以这里就不多做介绍了，可以自己在终端上打开看看。这个命令的一大优点是可以在ssh远程连接的时候使用，在Windows下的XShell等软件中都可以直接调出。

# 0X03 ip 命令
ip是现在推荐使用的命令，功能比较强大。
## ip命令管理设备开关
```bash
[root@localhost ~]# ip link show enp0s8	# 这个命令大致相当于 ifconfig enp0s8 查看这个网卡的信息
3: enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 08:00:27:79:c4:b1 brd ff:ff:ff:ff:ff:ff
[root@localhost ~]# ip link set dev enp0s8 up	# 设置一个device，enp0s8，打开
[root@localhost ~]# ip link show enp0s8
3: enp0s8: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 08:00:27:79:c4:b1 brd ff:ff:ff:ff:ff:ff
[root@localhost ~]# ip link set dev enp0s8 down	# 设置 设备 网卡名 打开/关闭
[root@localhost ~]# ip link show enp0s8
3: enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 08:00:27:79:c4:b1 brd ff:ff:ff:ff:ff:ff
```

## ip命令修改网卡MAC地址
```bash
[root@localhost ~]# ip link set dev enp0s8 address 00:00:ff:bb:aa:22	# 修改网卡的物理地址，也就是MAC地址
[root@localhost ~]# ip link show enp0s8
3: enp0s8: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 00:00:ff:bb:aa:22 brd ff:ff:ff:ff:ff:ff
```

# 0X04 nmcli 管理网络
`nmcli`是`network manager command line interface`的简写，这个命令可以用来管理配置网络。
## 查看网络接口状态
```bash
[root@localhost ~]# nmcli -p g
=============================================================
                    NetworkManager status
=============================================================
STATE      CONNECTIVITY  WIFI-HW  WIFI     WWAN-HW  WWAN    
-------------------------------------------------------------
connected  full          enabled  enabled  enabled  enabled 
```

## 查看修改主机名
```bash
[root@localhost ~]# nmcli general hostname
localhost.localdomain
[root@localhost ~]# nmcli general hostname test
[root@localhost ~]# nmcli general hostname
test
```

## 查看网络设备
以前可以用ifconfig来查看网络设备，ip命令也可以查看。可以直接查看所有的，也可以指定某一个设备查看。
```bash
[root@localhost ~]# nmcli device show	# 莎看所有设备
GENERAL.DEVICE:                         enp0s3
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         08:00:27:35:C7:CE
GENERAL.MTU:                            1500
.......................	# 输出太多，就不全放在这里了
IP6.ADDRESS[1]:                         ::1/128
IP6.GATEWAY:                 
[root@localhost ~]# nmcli device show enp0s8	# 查看指定设备
GENERAL.DEVICE:                         enp0s8
GENERAL.TYPE:                           ethernet
GENERAL.HWADDR:                         08:00:27:79:C4:B1
GENERAL.MTU:                            1500
GENERAL.STATE:                          100 (connected)
GENERAL.CONNECTION:                     enp0s8
GENERAL.CON-PATH:                       /org/freedesktop/NetworkManager/ActiveConnection/7
WIRED-PROPERTIES.CARRIER:               off
IP4.ADDRESS[1]:                         123.123.123.2/25
IP4.GATEWAY:                            
IP6.GATEWAY:          
```

## 修改网卡配置
一个设备可以有多个连接，在CentOS7中网络是以连接管理的。虽然每个设备可以有多个连接，但是同时生效的只能有一个。我们可以使用`nmcli connection`查看连接
```bash
[root@localhost ~]# nmcli connection show
NAME    UUID                                  TYPE            DEVICE 
enp0s8  ae99f48d-5f20-4a9c-a487-c4ebafa3f92e  802-3-ethernet  enp0s8 
enp0s3  2edc4731-888c-4102-8ff5-236ea47eeedb  802-3-ethernet  enp0s3 
```
我们可以进行如下操作`nmcli connection add/delete/edit`也就是增删改三个操作。
每一个连接都有一个名字，我们可以根据名字索引来操作对应到的连接。我们先来删除掉之前配置的网络。
```bash
[root@localhost ~]# nmcli connection delete enp0s8 	# 这样可以删掉之前的连接
Connection 'enp0s8' (ae99f48d-5f20-4a9c-a487-c4ebafa3f92e) successfully deleted.
```
然后添加一个新的连接，名字叫'test_conn'，接口是'enp0s8'，类型是'ethernet'也就是以太网，ip使用v4版本123.123.123.123，子网掩码是24位，ipv4的网关是123.123.123.1
```bash
[root@localhost ~]# nmcli connection add con-name 'test_conn' ifname enp0s8 type ethernet ip4 123.123.123.123/24 gw4 123.123.123.1
Connection 'test_conn' (05c7cd70-a48e-4a12-a0de-9d57724cf0d0) successfully added.
[root@localhost ~]# nmcli connection show test_conn
# 这行命令的输出太多了就不展示了。但是我们可以通过这行命令看到自己创建的连接，信息和自己填写的命令相对应。
```
修改一个连接可以使用`nmcli connection modify`，下面我们来测试一下修改一个连接
```bash
[root@localhost ~]# nmcli connection show test_conn | grep ipv4.dns	# 使用grep搜索查看dns设置
ipv4.dns:                               
ipv4.dns-search:                        
[root@localhost ~]# nmcli connection modify test_conn ipv4.dns 8.8.8.8	# 修改ipv4的dns地址
[root@localhost ~]# nmcli connection show test_conn | grep ipv4.dns
ipv4.dns:                               8.8.8.8
ipv4.dns-search:     
```
