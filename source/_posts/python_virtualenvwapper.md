---
title: python_virtualenvwapper
date: 2017-09-20 00:52:50
tags:
    - Python
    - Virtualenvwapper
---

# 0X00 virtualenv好用但有瓶颈
virtualenv固然好用，可以给你每一个Python项目创建一个独立的Python环境互不干扰。有三五个Python项目的时候用的很开心，有十几个项目的时候还凑合，如果有更多的项目virtualenv就会出现瓶颈。因为virtualenv会给每一个Python虚拟环境创建一个目录来保存相关文件，项目一多这个虚拟环境的目录也就多了起来，每次在多个环境之间`source ../../../xxx/bin/active` 和 `deactive` 也挺烦的，并且很容易把某些环境搞丢。不过开源世界最不缺的就是解决问题的方法了，既然有人遇到了这个问题，那么八成就已经有了解决这个问题的好办法。


# 0X01 virtualenvwrapper
这个东西名字确实有点长，顾名思义就是把virtualenv包装起来。首先来安装一波这个东西
```
sudo apt install virtualenvwrapper  # Debian系

sudo yum install virtualenvwrapper  # RHEL系
```
安装好后要进行简单的配置
```
vim ~/.bashrc # 添加一条环境变量，可以根据自己用的shell来修改
```
向文件中添加 `WORKON_HOME=~/Envs` 表示将未来所有的虚拟环境都放在 `~/Envs` 中。然后创建这个目录 `mkdir -p $WORKON_HOME` 。最后`source`一下安装文件，`source /usr/bin/virtualenvwrapper.sh` 会显示创建了很多文件，到这里就安装完成了。
如果`source`的时候没有这个`virtualenvwrapper.sh`文件，那就用`which virtualenvwrapper.sh`找一下，不过八成都是在`/usr/bin/virtualenvwrapper.sh`


# 0X02 把它用起来
以前用`virtualenv`的时候要每次`source xxx/bin/active`，用完了再`deactive`，这次就方便多了。下面列出几个常用命令。

| 命令 | 功能 |
| -- |
| mkvirtualenv blog | 创建一个名为blog的虚拟环境，并切换过去 |
| workon blog | 切换到名为blog的虚拟环境中 |
| workon | 列出当前所有的虚拟环境 |
| rmvirtualenv blog | 删除名为blog的虚拟环境 |
| deactive | 退出当前所处的虚拟环境 |

下面演示一下这个用法
```bash
# 创建一个新的虚拟环境，名为blog
[root@localhost ~]# mkvirtualenv blog
New python executable in blog/bin/python
Installing Setuptools..............................................................................................................................................................................................................................done.
Installing Pip.....................................................................................................................................................................................................................................................................................................................................done.
virtualenvwrapper.user_scripts creating /root/Envs/blog/bin/predeactivate
virtualenvwrapper.user_scripts creating /root/Envs/blog/bin/postdeactivate
virtualenvwrapper.user_scripts creating /root/Envs/blog/bin/preactivate
virtualenvwrapper.user_scripts creating /root/Envs/blog/bin/postactivate
virtualenvwrapper.user_scripts creating /root/Envs/blog/bin/get_env_details

# 创建一个新的虚拟环境，名为student_admin
(blog)[root@localhost ~]# mkvirtualenv student_admin
New python executable in student_admin/bin/python
Installing Setuptools..............................................................................................................................................................................................................................done.
Installing Pip.....................................................................................................................................................................................................................................................................................................................................done.
virtualenvwrapper.user_scripts creating /root/Envs/student_admin/bin/predeactivate
virtualenvwrapper.user_scripts creating /root/Envs/student_admin/bin/postdeactivate
virtualenvwrapper.user_scripts creating /root/Envs/student_admin/bin/preactivate
virtualenvwrapper.user_scripts creating /root/Envs/student_admin/bin/postactivate
virtualenvwrapper.user_scripts creating /root/Envs/student_admin/bin/get_env_details

# 查看所有的虚拟环境
(student_admin)[root@localhost ~]# workon
blog
student_admin

# 切换到blog环境
(student_admin)[root@localhost ~]# workon blog

# 删除student_admin环境
(blog)[root@localhost ~]# rmvirtualenv student_admin
Removing student_admin...

# 再看一下所有环境，student_admin已经不在了
(blog)[root@localhost ~]# workon
blog

# 退出当前环境
(blog)[root@localhost ~]# deactivate 
[root@localhost ~]# 
```
`` ````
