---
title: Linux 中计划任务和周期任务
date: 2016-11-02 22:17
tags:
  - Linux
  - Crontab
  - 计划任务
---

# 0X00 Linux中的计划任务
我们使用Linux更多的时候是在服务器上，然而我们有的时候就需要让计算机在固定的某个时间做一些事情。比如我们就可能有有如下需求：
1. 临时有事需要离开电脑，但是一个小时后需要备份某个目录里的文件
2. 写了个爬虫去抓取某网站的新闻，每隔十分钟就去爬取一次
3. 周期性的执行某脚本，但放在后台的话退出ssh就会被自动关掉
4. 其实还有好多这种可能............

在Linux中有两种常见的任务管理，一个是`at`也就是在某时做某事，另一个是`crontab`也就是周期性任务表。使用at可以方便地给Linux设置一个在什么时候做什么事的计划，用crontab可以方便地给Linux设置我要做某事，多久做一次。

# 0X01 使用at命令
## 检查atd服务是否开启
`atd`就是at命令的守护进程，系统默认是打开着的，但是也有可能被关掉，在RHEL系中可以使用`systemctl status atd`来查看服务是否已经开启，没有开启的话可以用`systemctl restart atd`来打开服务

## 创建一个计划任务
先创建一个在今天的`21:09`的任务，任务内容是输出hello,world重定向到/hello文件。然后到时间之后再检查这个文件是否出现了。当我们只指定时分的时候，默认是当天，如果已经过了的时间的话，会默认为次日。
```bash
# 一个即日的计划任务
[root@iZ28jaak5nnZ ~]# date
Wed Nov  2 21:07:07 CST 2016
[root@iZ28jaak5nnZ ~]# at 21:09
at> echo "hello,world" > /hello
at> <EOT>
job 5 at Wed Nov  2 21:09:00 2016
[root@iZ28jaak5nnZ ~]# date
Wed Nov  2 21:09:10 CST 2016
[root@iZ28jaak5nnZ ~]# cat /hello
hello,world
```
当我们输入`at 21:09`之后，就进入了at模式，我们在这里输入的命令就是之后将要执行的命令。当输入完命令之后按`Ctrl + D`就可以退出at模式，此时计划任务创建完毕，系统会提示你计划任务的执行时间。

下面还有几个例子
```bash
# 一个准确定时的计划任务
[root@iZ28jaak5nnZ ~]# at 00:00 2016-11-11	# 在2016光棍节零点输出一个'hey 单身狗'
at> echo "hey single dog"
at> <EOT>
job 7 at Fri Nov 11 00:00:00 2016

# 在十分钟后执行
[root@iZ28jaak5nnZ ~]# at now+10min
at> echo 'hello single dog'
at> <EOT>
job 9 at Wed Nov  2 21:26:00 2016


# 在一小时后执行
[root@iZ28jaak5nnZ ~]# at now+1hour
at> echo 'hey single dog'
at> <EOT>
job 10 at Wed Nov  2 22:16:00 2016
```

## 查看已有的at
可以使用`atq`命令来查看存在的at计划任务，注意这里并不一定全都是用户自己创建的，也有的是系统创建的。通过atq查看到之后可以使用`at -c `来查看某个计划任务的具体信息。
```bash
[root@iZ28jaak5nnZ ~]# at now+1hour
at> echo "hello"
at> <EOT>
job 11 at Wed Nov  2 22:23:00 2016
[root@iZ28jaak5nnZ ~]# atq
# 这里输出的第一列就是at的编号，下面查看详细信息就是根据编号查看的
7	Fri Nov 11 00:00:00 2016 a root
6	Thu Nov  3 03:00:00 2016 a root
10	Wed Nov  2 22:16:00 2016 a root
9	Wed Nov  2 21:26:00 2016 a root
11	Wed Nov  2 22:23:00 2016 a root
1	Wed Nov  2 21:52:00 2016 a root
[root@iZ28jaak5nnZ ~]# at -c 11
#!/bin/sh
# atrun uid=0 gid=0
# mail root 0
umask 22
XDG_SESSION_ID=669; export XDG_SESSION_ID
............................... # 这里省略了好多环境变量，重点在下面
XDG_RUNTIME_DIR=/run/user/0; export XDG_RUNTIME_DIR
cd /root || {
	 echo 'Execution directory inaccessible' >&2
	 exit 1
}
${SHELL:-/bin/sh} << 'marcinDELIMITER0e9efce8'	# 这里是执行的命令
echo "hello"

marcinDELIMITER0e9efce8
```

## 删除一个at
使用一个`atrm`命令可以指定at号删除特定的at计划任务。
```bash
[root@iZ28jaak5nnZ ~]# at now+1hour
at> echo 'hello'
at> <EOT>
job 12 at Wed Nov  2 22:27:00 2016
[root@iZ28jaak5nnZ ~]# atq
7	Fri Nov 11 00:00:00 2016 a root
6	Thu Nov  3 03:00:00 2016 a root
10	Wed Nov  2 22:16:00 2016 a root
11	Wed Nov  2 22:23:00 2016 a root
12	Wed Nov  2 22:27:00 2016 a root
1	Wed Nov  2 21:52:00 2016 a root
[root@iZ28jaak5nnZ ~]# atm 12
-bash: atm: command not found
[root@iZ28jaak5nnZ ~]# atrm 12
[root@iZ28jaak5nnZ ~]# atq
7	Fri Nov 11 00:00:00 2016 a root
6	Thu Nov  3 03:00:00 2016 a root
10	Wed Nov  2 22:16:00 2016 a root
11	Wed Nov  2 22:23:00 2016 a root
1	Wed Nov  2 21:52:00 2016 a root

```bash
[root@iZ28jaak5nnZ ~]# at now+1hour
at> echo 'hello'
at> <EOT>
job 12 at Wed Nov  2 22:27:00 2016
[root@iZ28jaak5nnZ ~]# atq
7	Fri Nov 11 00:00:00 2016 a root
6	Thu Nov  3 03:00:00 2016 a root
10	Wed Nov  2 22:16:00 2016 a root
11	Wed Nov  2 22:23:00 2016 a root
12	Wed Nov  2 22:27:00 2016 a root
1	Wed Nov  2 21:52:00 2016 a root
[root@iZ28jaak5nnZ ~]# atrm 12
[root@iZ28jaak5nnZ ~]# atq
7	Fri Nov 11 00:00:00 2016 a root
6	Thu Nov  3 03:00:00 2016 a root
10	Wed Nov  2 22:16:00 2016 a root
11	Wed Nov  2 22:23:00 2016 a root
1	Wed Nov  2 21:52:00 2016 a root
```

# 0X02 使用crontab命令
* 这里的配置分成六段

分---时---日---月---周---命令

## 创建周期任务
使用任何一个用户登陆到系统之后，就可以执行`crontab -e`就进入了vi的编辑器模式，然后我们来编辑这个文件就可以创建/修改周期任务了。
```bash
15  10 1 10 * echo 'hello' > /tmp/hello		# 在每个10月1号10点15分执行命令
15  10 1 *  * echo 'hello' > /tmp/hello		# 在每个1号10点15分执行命令
15  10 * *  * echo 'hello' > /tmp/hello		# 在每个10点15分执行命令
15  *  * *  * echo 'hello' > /tmp/hello		# 在每个15分执行命令
*/3 *  * *  * echo 'hello' > /tmp/hello		# 每3分钟执行命令
```
退出保存之后就可以按照这个时间来执行命令了。

## 查看周期任务
使用`crontab -l`查看该用户的周期任务
```bash
[root@iZ28jaak5nnZ ~]# crontab -l
15  10 1 10 * echo 'hello' > /tmp/hello
15  10 1 *  * echo 'hello' > /tmp/hello
15  10 * *  * echo 'hello' > /tmp/hello
15  *  * *  * echo 'hello' > /tmp/hello
*/3 *  * *  * echo 'hello' > /tmp/hello
```

## 删除周期任务
可以使用`crontab -r`删除当前用户所有的周期任务。

## 管理周期任务
每个用户都可以使用`crontab -e`来管理自己的周期任务，然而root用户可以使用`crontab -u`来管理其他用户的周期任务。只要加一个-u选项即可，参数后面接上要管理的用户就可以了。然后还是和上面的操作一样，只是多了一个这个参数而已。
