---
title: Linux权限控制列表 ACL修改
date: 2016-11-02 17:16
tags:
  - Linux
  - ACL
  - 权限
---


# 0X00 ACL是什么
ACL的全称是`Access Control List`访问控制列表。在Linux中可以给文件设置权限，`-rwx-rw-rw`这样，但是这里并不能细分，只能分到用户、组、其他用户。如果我想给某个单独的用户设置权限的话是做不到的。所以有了ACL的出现。通过ACL可以给Linux下的文件提供详细的访问控制，比如我们在设置了基本的`rwx`权限之后，可以通过ACL在细分用户对文件的权限。

# 0X01 查看文件的ACL
使用`getfacl`命令可以查看文件的ACL和详细的权限设置。
```bash
[root@iZ28jaak5nnZ ~]# ls -l
total 4
-rwxr-xr-x 1 root root 1714 Oct 28 22:24 hello.py
[root@iZ28jaak5nnZ ~]# getfacl hello.py
# file: hello.py
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
```
这里显示了文件名、所属用户、所属组、还有相对应的权限。

# 0X02 创建测试用户/组/文件
先创建测试用户、测试组、测试文件。创建了`xiaoming`和`xiaohong`两个用户，在`china`组，创建了一个`jack`用户在`usa`组。然后用root用户在`/tmp/`目录下创建了一个acltest目录，用来做测试，因为这个目录是任何人都可以访问的，但是由于是root用户创建的子目录，所以要给这个目录`777`的权限，让其他用户可以在里面测试。现在里面又创建了一些目录和文件，但是全部都是root用户的，文件权限是`644`，目录权限是`755`。
```bash
[root@iZ28jaak5nnZ ~]# groupadd china
[root@iZ28jaak5nnZ ~]# useradd xiaoming -g china
[root@iZ28jaak5nnZ ~]# useradd xiaohong -g china
[root@iZ28jaak5nnZ ~]# groupadd usa
[root@iZ28jaak5nnZ ~]# useradd jack -g usa
[root@iZ28jaak5nnZ ~]# cd /tmp
[root@iZ28jaak5nnZ tmp]# mkdir acltest
[root@iZ28jaak5nnZ tmp]# chmod 777 acltest
[root@iZ28jaak5nnZ tmp]# cd acltest/
[root@iZ28jaak5nnZ acltest]# touch file_{1,3}
[root@iZ28jaak5nnZ acltest]# mkdir dir_{1,3}
```

# 0X03 设置文件的ACL
使用`setfacl`命令可以设置文件ACL。这个命令有下面这几个常用参数

## setfacl 各个参数
* 所谓的后续ACL就是在默认ACL的基础上添加的新的规则。

### -m 设置后续ACL
对某一个文件/目录设置某一个用户的访问权限， u表示用户 冒号后面是用户名 再一个冒号后面是权限 最后接文件/目录
`[root@iZ28jaak5nnZ acltest]# setfacl -m u:user_1:rwx file_1`

对某一个文件/目录设置某一个用户组的访问权限，u表示组 冒号后面是组名 再一个冒号后面是权限 最后接文件/目录
`[root@iZ28jaak5nnZ acltest]# setfacl -m g:group_1:rwx file_1`
### -x 删除后续ACL
删除之前添加的ACL项，指定用户或者指定组都是可以的，语法和上面差不多。这里删除的是一条ACL数据，下面说的-b参数是删除所有的ACL数据
```bash
[root@iZ28jaak5nnZ acltest]# setfacl -x u:xiaoming file_1
[root@iZ28jaak5nnZ acltest]# getfacl file_1
# file: file_1
# owner: root
# group: root
user::rw-
group::r--
mask::r--
other::r--
```

### -b 删除所有后续ACL
这里是删除之前创建的所有ACL，包括下面会说的默认ACL
```bash
[root@iZ28jaak5nnZ acltest]# getfacl file_1
# file: file_1
# owner: root
# group: root
user::rw-
user:xiaoming:rwx
user:xiaohong:rw-
group::r--
mask::rwx
other::r--

[root@iZ28jaak5nnZ acltest]# setfacl -b file_1
[root@iZ28jaak5nnZ acltest]# getfacl file_1
# file: file_1
# owner: root
# group: root
user::rw-
group::r--
other::r--
```

### -d 设置默认ACL
设置默认ACL只能为目录设置，为目录设置了ACL之后里面新建的目录和文件都是使用这个默认的ACL
```bash
[root@iZ28jaak5nnZ acltest]# getfacl dir_1
# file: dir_1
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
[root@iZ28jaak5nnZ acltest]# setfacl -m d:u:jack:rwx dir_1	# 设置目录的默认ACL
[root@iZ28jaak5nnZ acltest]# getfacl dir_1	# 我们可以看到现在出现了一段默认ACL
# file: dir_1
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:user:jack:rwx
default:group::r-x
default:mask::rwx
default:other::r-x
[root@iZ28jaak5nnZ acltest]# cd dir_1
[root@iZ28jaak5nnZ dir_1]# touch hello
[root@iZ28jaak5nnZ dir_1]# getfacl hello	# 新建的文件也使用这些默认设置
# file: hello
# owner: root
# group: root
user::rw-
user:jack:rwx			#effective:rw-
group::r-x			#effective:r--
mask::rw-
other::r--
[root@iZ28jaak5nnZ acltest]# setfacl -m u::rwx -d dir_3	# 设置为每个用户，也可以修改为用户组
[root@iZ28jaak5nnZ acltest]# getfacl dir_3
# file: dir_3
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:group::r-x
default:other::r-x
```

### -k 删除默认ACL
这里可以删除之前设置的默认ACL，只限默认ACL
```bash
[root@iZ28jaak5nnZ acltest]# getfacl dir_3 # 查看ACL，这里显示有默认的ACL
# file: dir_3
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
default:user::rwx
default:group::r-x
default:other::r-x

[root@iZ28jaak5nnZ acltest]# setfacl -k dir_3	# 删除dir_3的默认ACL
[root@iZ28jaak5nnZ acltest]# getfacl dir_3
# file: dir_3
# owner: root
# group: root
user::rwx
group::r-x
other::r-x
```

### -R 递归设置ACL
给某一个目录设置递归的ACL之后这个目录和这个目录里的文件和子目录全部都会应用这个ACL，也就是说是相当于应用到了这个目录下的所有文件和目录
```bash
# 首先创建一下测试用的目录结构
[root@iZ28jaak5nnZ acltest]# mkdir -p dir1/dir2/dir3
[root@iZ28jaak5nnZ acltest]# touch dir1/hello.c
[root@iZ28jaak5nnZ acltest]# touch dir1/dir2/hey.c
[root@iZ28jaak5nnZ acltest]# setfacl -m u:jack:r -R dir1	递归设置ACL
[root@iZ28jaak5nnZ acltest]# getfacl dir1
# file: dir1
# owner: root
# group: root
user::rwx
user:jack:r--
group::r-x
mask::r-x
other::r-x
[root@iZ28jaak5nnZ acltest]# getfacl dir1/hello.c 
# file: dir1/hello.c
# owner: root
# group: root
user::rw-
user:jack:r--
group::r--
mask::r--
other::r--
```
