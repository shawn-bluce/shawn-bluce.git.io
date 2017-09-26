---
title: OpenSSH 配置 免密码登陆 公钥和私钥 私钥签名
date: 2016-10-17 14:41
tags:
  - Linux
  - OpenSSH
  - 公钥
  - 私钥
  - 签名
  - 加密
  - 非对称加密
---


# 0X00 安装OpenSSH
一般情况下我们的系统中都是自带SSH服务端和客户端的，万一没有的话就需要我们手动安装这个服务。

`yum install -y openssh`

然后重启OpenSSH服务

`systemctl restart sshd`

# 0X01 两行简单的配置
OpenSSH的配置文件在`/etc/ssh/`目录下，有两个配置文件，一个是针对服务端的一个是针对客户端的，我们只需要修改针对服务端的`sshd_config`即可。

配置文件里比较重要的两行是`PermitRootLogin`和`PasswordAuthentication`。

* `PermitRootLogin` 当这个值为yes时，才允许root用户使用ssh登陆
* `PasswordAuthentication` 当这个值为yes时，允许使用密码登陆，反之则拒绝密码登陆(只能使用密钥)。

```config
PermitRootLogin yes
PasswordAuthentication yes
```
> 这里的配置就允许使用root用户登陆，也允许输入密码登陆

# 0X02 私钥和公钥————非对称加密
在ssh中可以使用用户名密码的形式登陆，也可以使用密钥的形式登陆。

**非对称加密** 就是说加密和解密用的密码不同。非对称加密里有**公钥**和**私钥**，使用公钥加密的数据只有使用私钥才能解开，虽然是使用公钥加密的，但是并不能通过公钥反向解密。这点和传统的对称加密区别比较大。

下面假设有这么一个场景：有一台服务器S和三个管理员A1、A2、A3。  S生成了自己的一对公钥和私钥，将公钥公开出去，这时候A1就能能看到这个公钥，所以都可以用这个公钥将发给S的数据加密。虽然A2和A3也看到了这个公钥，但是不能通过这个公钥将这个加密的数据解开。数据只有在S上通过对应的私钥才能解开。
* 公钥：一般是公开出去，并用于加密
* 私钥：保存在自己这里，用于解密
> 公钥和私钥是一对的，一个公钥和一个私钥两两对应

# 0X03 在SSH生成公钥和私钥
在Linux里SSH可以使用公钥和私钥来登陆系统，也就是前面我们说的那个`PasswordAuthentication`选项，如果禁止密码登陆的话就只能使用公钥和私钥登陆了。

`ssh-keygen`可以生成一对公钥和私钥。我们一般在自己用户的主目录里的`.ssh`目录里执行这个命令、执行完了之后会提示输入加密，这里是给公钥私钥加密，可以暂时不用管，一路回车就行了，直到看到一堆乱七八糟的图像，类似于这样就算好了。
```
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
09:7c:28:a5:75:sr:ab:5c:82:43:17:81:f8:78:zs:1e root@buyongkan.zhelishi.gaiguode
The key's randomart image is:
+--[ RSA 2048]----+
|   . .+o.        |
|  . .=8o.        |
|   oo+-+.        |
|  . =.oo..       |
|   X +  SB.      |
|    o . o        |
|     o +         |
|      )          |
|                 |
+-----------------+
```

`.ssh`目录如果不存在的话，执行一下`ssh localhost`然后输入密码登陆以下本地，就会有了。生成完之后目录里会多出两个文件，`id_rsa` 和 `id_rsa.pub` 后面pub结尾的是public也就是公钥，我们可以打开看看是一堆看似乱码的东西。

# 0X04 使用公钥和私钥免密码登陆
如果我们有两台机器，一个叫Server一个叫Desktop，我想让Desktop可以免密码登陆到Server上，就可以用这个方法。

原理大概是这样的：在Desktop上生成一对公钥和私钥，然后将Desktop上的公钥追加到Server的`.ssh`目录下的`authorize_keys`里，这个文件就是用来保存可以免密码登录到自己机器上的那些用户的公钥的。

```bash
[root@iZ28jaak5nnZ .ssh]# ssh-keygen  #生成一对公钥私钥
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
30:05:7d:63:sf:10:aa:cb:e1:b7:84:48:54:5f:42:4d root@iZ28jaak5nnZ
The key's randomart image is:
+--[ RSA 2048]----+
|..o o EO+o...    |
| o = * n. +.     |
|  + * N  o .     |
|   S + a         |
|    . . S        |
|m            s   |
|                 |
|    x            |
|                 |
+-----------------+
[root@iZ28jaak5nnZ .ssh]# ssh-copy-id -i id_rsa.pub root@182.234.214.243 #使用ssh-copy-id来将自己的公钥发送到Server上去，会自动找到那个文件
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@182.254.214.250's password:   # 还没配置好所以要输密码

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@182.254.214.250'"
and check to make sure that only the key(s) you wanted were added.
[root@iZ28jaak5nnZ .ssh]# ssh root@182.254.214.250  # 登陆
Last login: Mon Oct 17 13:47:23 2016 from 43.13.56.7
☁  ~  hostname  # 成功登陆
blog.just666.cn
☁  ~  
```

# 0X05 使用私钥签名
公钥私钥对可以对数据加密，是用公钥加密私钥解密。也可以使用公钥私钥对进行数字签名。

当Server公开自己的公钥之后，大家都可以用这个公钥进行加密，然后传给Server，Server用私钥解密就能看到内容。

Server如果想加密一段数据给其他人的话，可以用自己的私钥加密，将密文发送给其他人，其他人就能用Server的**公钥去解密**。因为除了Server意外，任何人都不知道Server的私钥，所以其他人可以确信这条消息是Server发出来的。这种行为称之为**签名**。

注意一个问题：**公钥加密的数据可以用私钥解开**且**私钥加密的数据也可以用公钥解开**
