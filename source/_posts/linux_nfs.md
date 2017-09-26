---
title: NFS 网络文件系统 安装 配置 挂载 卸载
date: 2016-06-12 12:13
tags:
  - Linux
  - NFS
  - 文件系统
---

# 0X00 NFS简介
NFS的全称是Net-File-System也就是网络文件系统。这和Samba与FTP不同，FTP的主要用途是用来上传和下载文件，Samba的主要功能是共享文件，而NFS的主要功能是用作文件系统。也就是说和NTFS、FAT32、EXT4等是类似的性质。我们可以将这个NFS当做一个磁盘分区挂载到自己的操作系统上，像操作自己的分区一样，甚至可以从NFS启动操作系统。

>实验环境：两台虚拟机CentOS7.x
>同处在一个内网环境下

# 0X01 安装NFS软件和服务
```bash
# 安装软件
yum install rpcbind
yum install nfs-utils
```

# 0X02 创建测试目录并修改权限
```bash
# 创建测试用的目录
mkdir /home/share

# 创建测试用的文件(让文件里有内容，方便后来判断是否搭建成功)
ls / > /home/share/test1
ls /etc/ > /home/share/test2

# 创建挂载点、以后就把NFS挂载到这里
mkdir /home/test

# 将这个测试目录设置为777的权限
chmod 777 /home/share
```

# 0X03 修改配置文件
配置文件是`/etc/exports`  使用文本编辑器打开配置文件并进行修改
```bash
# 添加如下配置  192.168.123.132是客户端IP
/home/share/ 192.168.123.132(rw, sync) *(ro)
```
`/home/share/`表示NFS的路径
`192.168.123.132(rw, sync)`表示192.168.123.132访问此NFS时使用后面的配置、具有rw权限（读写）、sync同步模式，表示内存中的数据实时写入磁盘
`*(ro)`表示所有IP访问时使用后面的配置、ro表示read only只读
>每个路径下面可以接好多个访问项，就是`192.168.123.132(rw, sync)`或者`*(ro)`，使用空格分开

# 0X04 启动服务并检查NFS配置
```bash
# 启动服务
systemctl start portmap
systemctl start nfs
```

```bash
# 在客户端检查 192.168.123.123是服务端
showmount -e 192.168.123.123

# 如果输出成如下这样就是正确了
Export List for 192.168.123.123:
/home/share *
```

# 0X05 挂载和卸载
```bash
# 挂载
mount -t nfs 192.168.123.123:/home /home/test

# 卸载
umount /home/test
```
