---
title: Python中Virtualenv和pip的简单用法
date: 2017-08-18 00:19
tags:
  - Python
  - pip
  - virtualenv
---

# 0X00 安装环境
我们在Python开发和学习过程中需要用到各种库，然后在各个不同的项目和作品里可能用的版本还不一样，正因为有这种问题的存在才催生了`virtualenv`的诞生。virtualenv可以在电脑上创建一个虚拟环境，可以针对每一个项目创建一个虚拟环境，这样就不用担心各个不同的项目用不同版本的库的时候出现的冲突了。 ** 下面的内容只适用于Linux/OSX，未经Windows环境测试 **

要使用这个功能还是需要安装，安装virtualenv肯定就得直接用pip安装了，`pip install virtualenv`就可以轻松装上了。装好之后我们就可以来测试一波了。

# 0X01 初始化一个空的工作环境
首先在一个空的环境中执行`virtualenv --no-site-packages test_env`，就是在当前目录创建一个名为test_env的虚拟环境。这里`--no-site-packages`参数是指不从全局的Python中携带任何第三方库。就比如说你在全局Python中安装了xxx库，在不用这个参数来创建虚拟环境时，虚拟环境中也会带着这个库；但是加上了这个参数，虚拟环境中就是一个纯净的Python，没有这些库。
```bash
root in ~ λ virtualenv --no-site-packages test_env
New python executable in /root/test_env/bin/python
Please make sure you remove any previous custom paths from your /root/.pydistutils.cfg file.
Installing setuptools, pip, wheel...done.
```
然后可以通过`source test_env/bin/activate`可以进入（激活）到这个虚拟环境里去。进入到虚拟环境中之后，通常情况下你的命令提示符最前面会出现一个括号，括号里面写着你虚拟环境的名字。
> 这里说是虚拟环境，其实一切都是真实的。只是说你在激活了这个环境，在这个环境下用pip安装的库都放在 `test_env` 中。

也可以通过`deactivate`来退出这个环境。

# 0X02 批量导出和安装库
比如我们开发了一个项目，里面用到了pymongo/requests/flask/pymysql等等等等十几二十个库，还要指定特定的版本，那么当把一个项目从机器A迁移到机器B的时候就会很麻烦。需要手动记录每个库和版本，还要逐个去安装，非常麻烦。所以针对这个问题pip已经有了非常完善的解决方案。

```bash
(test_env) root in ~ λ pip freeze > requirements.txt  # 导出已安装的库
```
这个命令可以导出当前环境中安装好的所有第三方库，并且是以一个标准的格式导出的。所以一般一个标准的python项目的根目录都会有这个名为`requirements.txt`的依赖文件。

既然可以一次性导出，那么必然可以一次性安装喽。通过这种方式就可以将上面导出的特定版本的所有库一次性全装上。配合virtualenv可以快速的部署一个Python项目，并且不会搞乱其他的Python项目环境。
```bash
(test_env_1) root in ~ λ pip install -r requirements.txt
```

