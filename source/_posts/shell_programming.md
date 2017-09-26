---
title: Shell编程入门笔记  新手教程
date: 2016-10-02 18:03
tags:
  - Linux
  - Shell
---

# 0X00 hello,world
从一本*The C Programming Language*开始，我们就开始了几十年的'hello,world'之路。从那以后，机会所有的教程都从输出一句'hello,world'开始，这次也不例外。
```bash
#!/bin/bash

echo "hello,world"
```
这里的第一行是注释，这个注释是很特殊的，他会告诉系统我们使用哪个解释器来运行下面的代码，这里我们用的是`/bin/bash`，当然Python的代码就要加上`#!/usr/bin/python`。
第二行就是输出一句'hello,world'。`echo`就是输出语句。
```
[root@mail shell]# chmod +x test.sh
[root@mail shell]# ./test.sh
hello,world
```

## 运行脚本
执行之前要给脚本一个x权限，也就是执行权限。然后直接运行就行了。还有一种运行方式是`/bin/bash test.sh` 这样就是执行bash这个命令，将test.sh作为参数传进去，这样就可以不必写第一行的解释器声明。但是建议使用第一种方式执行脚本。

# 0X01 使用变量
既然是编程，那一定会有变量。Shell编程里的变量和C、Java不同，我们不需要声明一个变量就能直接赋值，想下面这样。
```bash
#!/bin/bash

str="hello,world"
echo $str
```
> 这里需要注意一点，我们在写一些代码的时候，可能习惯了像这样使用操作符`str = "hello,world"`,也就是在操作符两端加上空格。但是在Shell编程里这样做是被禁止的，加了空格就会导致语法错误。所以Shell编程里的空格限制是很严格的。

给一个新的变量赋值的时候我们可以直接写变量名，但是我们调用这个变量的时候要给变量名前面加上一个`$`符号，就像上面我写的那样。当然最好写成下面这种形式`${str}`因为这样会更加清晰的显示出变量名。

## 只读变量
在Shell中有一种变量叫‘只读变量’，顾名思义，这种变量的值不会被改变，是固定的，我们这样来声明一个只读变量`readonly str`。只读变量之前也是可以随意更改的，只是在后面给它加上了一个只读属性而已，就像下面这样。
```bash
#!/bin/bash

str="hello,world"
readonly str
str="hey,world"
echo str
```
就会报错：'./hello.sh: line 5: str: readonly variable'

## 删除变量
当我们不再使用一个变量的时候，可以把这个变量删除掉
```bash
#!/bin/bash

str="hello,world"
unset str
echo str
```
这里什么都不会输出，因为并没有str这个变量。在Shell中输出一个并不存在的变量不会有提示。

# 0X02 执行一行命令
既然是Shell编程，那么执行命令是最重要的事情了。所以在Shell编程里执行命令也是非常简单的，直接把要执行的命令写到这里就行了。
```bash
#!/bin/bash

lscpu  #这个命令是查看CPU相关信息的
```

# 0X03 字符串
字符串可以用单引号包起来，也可以用双引号包起来。单引号包起来的字符串会原封不动，会忽略转义字符和变量；双引号包起来的字符串会识别转义字符和变量。
```bash
#!/bin/bash

a="hello"
str1='$a, world'  #单引号字符串
str2="$a, world"  #双引号字符串
echo $str1
echo $str2
```

输出是这样的
```bash
[root@mail shell]# ./hello.sh
$a, wrld        #可以看到这里没有识别到变量
hello, world    #这里是识别到了变量的
```

## 字符串拼接
Shell里拼接字符串的语法非常简陋，直接把两个字符串变量写在一起就行了。
```bash
#!/bin/bash

str1="hello,"
str2="world"
echo $str1$str2  #就这么简单粗暴
```
这样就可以输出一个'hello,world'了

## 获取长度
在Shell编程里获取字符串长度不是通过一个len方法或者.length属性获取，而是通过下面这种并不直观的方式获取长度。
```bash
#!/bin/bash

str="hello,world"
str_len=${#str}
echo $str_len
```

## 部分截取
在Shell编程里我们可以截取一个字符串中的某一段，只需要两个参数，一个来指定开始位置，一个来指定结束位置。
```bash
#!/bin/bash

str="hello,world"
echo ${str:1:4}
```
这样可以截取str字符串从1到4的部分。是一个闭区间，从0开始计数。

## 查找位置
我们经常会需要从一段字符串里找到某个字符出现的位置，可以通过下面的方法来查找。
```bash
#!/bin/bash

str="hello,world"
index=`expr index "$str" lo` #查找l或者o这个字符首次出现在字符串的哪个位置
```


# 0X04 搞个数组
既然是编程，那么当然要有数组这个最基本的数据结构了。但是Shell只支持一维数组，并不支持二维和多维数组。

```bash
#!/bin/bash

array_str=("hello" "hey" "nihao")
echo ${array_str[2]} #定义的时候用的是小括号，调用的时候是大括号
echo ${array_str[@]} #这里的一个@表示数组里的所有内容
```

## 获取数组的长度
获取数组长度的方式和获取字符串长度的方式差不太多。
```bash
#!/bin/bash

array_str=("hello" "hey" "nihao")
length=${#array_str[@]} #获取数组长度
echo $length
length=${#array_str[2]} #获取数组中某个元素的长度
echo $length
```

# 0X05 别忘了注释
编程的时候给关键代码加上注释是一个非常好的习惯。Shell编程里只支持单行注释，不支持多行注释。单行注释是这样的
```bash
#!/bin/bash

str="hello"
#str="world"
echo $str
```
这样输出的结果是'hello'而不是'world'因为哪一行被注释掉了，并不会执行。
那么我们需要多行注释怎么办呢？其实也不是不可以，我们可以用一个诡异的方式来实现多行注释：把需要注释掉的代码改写成一个函数，只要我们在后面不去调用这个函数，那不就和被注释掉是一样的效果了嘛。关于函数的问题下面会说的。

# 0X06 Shell参数
我们的Shell脚本经常是需要传入参数进来的，那么应该怎么传进来呢？我们有下面一段代码：
```bash
#!/bin/bash

echo "1. $1"
echo "2. $2"
echo "3. $3"
echo "all is $#"
echo "pid is $$"
```
我们运行一下这段代码，并传入两个参数
```bash
[root@mail shell]# ./hello.sh hello world
1. hello
2. world
3. 
all is 2
pid is 22017
```
> $1 $2 $3 这些参数表示：传入的第几个参数
> $# 表示传入的参数总数
> $$ 表示这个脚本运行的PID  （进程号）

# 0X07 运算符
运算符示例
```bash
#!/bin/bash

val=`expr 1 + 1` #注意空格问题
echo $val
```
最常见的 `+` `-` `*` `/` `=` `==` 全都有的，不过注意的一点是，运算符两边一定要加空格，一定。

## bool布尔运算
`!`  非
`-o` 或
`-a` 且
-eq 是‘判断是否相等，相等则true’
```bash
#!/bin/bash

a=1
b=2

if [ $a -eq $b -o $a == 1 ]
then
    echo "ok"
else
    echo "not ok"
fi
```
这段代码输出的是'ok'因为虽然$a和$b并不相等，但是后面的`-o`表示或，或后面的`$a == 1`成立了，所以还是输出了ok

## 逻辑运算
`&&` and
`||` or
下面这段Demo代码和上面的效果几乎是一样的
```bash
#!/bin/bash

a=1
b=2

if [[ $a -eq $b || $a == 1 ]] #这里比上面多了一组中括号
then
    echo "ok"
else
    echo "not ok"
fi
```

## 字符串运算符
`=`  这里不是赋值，是判断两个字符串是否相等
`!=` 这里是判断不相等
`-n` 检测字符串长度，不为0则返回true
`-z` 检测字符串长度，为0则返回true
`str` 检测字符串是否为空，这里的str表示一切字符串或者变量

## 运算符总结：

### 算术运算符
```

运算符	说明	                                        举例
+	     加法	                                    `expr $a + $b` 结果为 30。
-	     减法	                                    `expr $a - $b` 结果为 -10。
*	     乘法	                                    `expr $a \* $b` 结果为  200。
/	     除法	                                    `expr $b / $a` 结果为 2。
%	     取余	                                    `expr $b % $a` 结果为 0。
=	     赋值	a=$b 将把变量 b 的值赋给 a。
==	     相等。用于比较两个数字，相同则返回 true。	[ $a == $b ] 返回 false。
!=	     不相等。用于比较两个数字，不相同则返回 true。[ $a != $b ] 返回 true。
```
### 关系运算符
```
运算符	说明	                                      举例
-eq	   检测两个数是否相等，相等返回 true。	 [ $a -eq $b ] 返回 false。
-ne	   检测两个数是否相等，不相等返回 true。[ $a -ne $b ] 返回 true。
-gt	   检测左边的数是否大于右边的，如果是，则返回 true。[ $a -gt $b ] 返回 false。
-lt	   检测左边的数是否小于右边的，如果是，则返回 true。[ $a -lt $b ] 返回 true。
-ge	   检测左边的数是否大等于右边的，如果是，则返回 true。[ $a -ge $b ] 返回 false。
-le	   检测左边的数是否小于等于右边的，如果是，则返回 true。[ $a -le $b ] 返回 true。
```

### 布尔运算符
```
运算符	说明	                                             举例
!	     非运算，表达式为 true 则返回 false，否则返回 true。[ ! false ] 返回 true。
-o	    或运算，有一个表达式为 true 则返回 true。          [ $a -lt 20 -o $b -gt 100 ] 返回 true。
-a	    与运算，两个表达式都为 true 才返回 true。	         [ $a -lt 20 -a $b -gt 100 ] 返回 false。
```

### 逻辑运算符
```
运算符	说明	      举例
&&	    逻辑的 AND	[[ $a -lt 100 && $b -gt 100 ]] 返回 false
||	    逻辑的 OR	 [[ $a -lt 100 || $b -gt 100 ]] 返回 true
```

### 字符串运算符
```
运算符	说明	                                   举例
=	     检测两个字符串是否相等，相等返回 true。  [ $a = $b ] 返回 false。
!=	    检测两个字符串是否相等，不相等返回 true。[ $a != $b ] 返回 true。
-z	    检测字符串长度是否为0，为0返回 true。    [ -z $a ] 返回 false。
-n	    检测字符串长度是否为0，不为0返回 true。  [ -n $a ] 返回 true。
str   	检测字符串是否为空，不为空返回 true。	   [ $a ] 返回 true。
```

### 文件测试运算符
```
操作符	 说明	                                             举例
-b file	检测文件是否是块设备文件，如果是，则返回 true。	   [ -b $file ] 返回 false。
-c file	检测文件是否是字符设备文件，如果是，则返回 true。	 [ -c $file ] 返回 false。
-d file	检测文件是否是目录，如果是，则返回 true。	         [ -d $file ] 返回 false。
-f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。[ -f $file ] 返回 true。
-g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	 [ -g $file ] 返回 false。
-k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。[ -k $file ] 返回 false。
-p file	检测文件是否是具名管道，如果是，则返回 true。	     [ -p $file ] 返回 false。
-u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	 [ -u $file ] 返回 false。
-r file	检测文件是否可读，如果是，则返回 true。	           [ -r $file ] 返回 true。
-w file	检测文件是否可写，如果是，则返回 true。	           [ -w $file ] 返回 true。
-x file	检测文件是否可执行，如果是，则返回 true。	         [ -x $file ] 返回 true。
-s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。[ -s $file ] 返回 true。
-e file	检测文件（包括目录）是否存在，如果是，则返回 true。[ -e $file ] 返回 true。
```

# 0X08 条件判断语句
## if-else
Shell 编程里的if语句和平时接触的编程语言有点区别
1. if后面要加一个中括号，中括号之间是条件，而且中括号前后都要有一个空格；
2. if调节写好之后要写一个then表示接下来执行什么
3. then结束之后如果没有别的条件了的话就接一个if的逆字符串fi来表示if语句结束
4. 如果还有其他条件那就用elif，然后也是一个中括号里写条件，then后面接要执行的语句
5. 最后的else可写可不写，表示上面的条件没达成的话要执行的语句。
6. if语句的最后一定是一个fi，表示if语句的结束

```bash
#!/bin/bash

if [ $1 == "1" ]
then
    echo "input is 1"
elif [ $1 == "2" ]
then
    echo "input is 2"
elif [ $1 == "3" ]
then
    echo "input is 3"
else
    echo "input is big"
fi
```
运行起来是这样的，比如我运行的参数是`./hello.sh 2 `，那么就会输出'2'

## case
跟if一起的一般还有一个switch-case语句，但是Shell里没有switch这个关键字，但是功能是一样的。`case $1 in`表示的是$1这个变量去对比下面的选项。下面的每一个选项都要加一个回括号，然后写上要执行的语句，再加上两个连续的分号，表示这一段结束，最后的`* )`表示匹配其他全部没有匹配到的可能。
```bash
#!/bin/bash

case $1 in
    "1" )
        echo "word is 1"
        ;;
    "2" )
        echo "word is 2"
        ;;
    * )
        echo "word is other"
        ;;
esac
```

# 0X09 几种循环
## for
在Shell编程里有我们非常熟悉的for循环，但是语法个其他编程语言还是有一定的出入。
在Shell里用for循环的话，要给每一段循环体加上`do...done`，表示这之间的代码是循环体，注意for的一行后面没有冒号，像Python的话就会有个冒号。
```bash
#!/bin/bash

for a in "hello,wrld"	#循环输出字符串中的每个字符
do
    echo $a
done

for a in 1 2 3 4 5      #循环输出列表中的每一个数据
do
    echo $a
done
```
`in`关键字前面是一个a后面是一堆数据，我们可以理解成把a从后面的一堆数据上走一遍，也就是说相当于Python代码中这样
```python
#!/usr/bin/python

for i in ['123', '234', '345', '456']:
	print i
```
## while
```bash
#!/bin/bash

while [[ $1 == "2" ]]
do
    echo "hello,world"
done
```
这段代码如果我是这样运行`./hello.sh 2` 那么因为`$1`是2所以就会进入死循环，否则就什么都不会输出。
while循环就是说，如果while后面的表达式成立，那么就执行一次循环。因为我这段代码没有改变$1的值，所以才会出现要么无输出要么死循环的情况。我们改成下面这种情况就好了，这意思是当我运行这个脚本，就执行10次。
```bash
#!/bin/bash

a=0
while [[ $a -lt 10 ]]  #这里的 -lt 的意思是 '当左边的值小于右边的值就返回true'
do
    a=$a+1
    echo "hello,world"
done
```
## until
until循环就相当于C和Java中的do-while循环，表示一个‘直到’的效果。下面就是说，执行循环体里的内容，一直到$a的变量大于右边的数值的时候才停止，所以才输出了11次'hello,world'
```bash
#!/bin/bash

a=0
until [[ $a -gt 10 ]]; 
do
    a=$a+1
    echo "hello,world"
done
```

# 0X0A 写个函数
Shell里的函数有两种定义方式，效果是一样的。
## 方案1
方案一是直接写函数名，后面接一个括号，然后大括号里写上函数体就好了。
```bash
myFunction(){
	echo "hello,world"
}
```
## 方案2
方案二是`function name()`这样定义，先声明这是一个函数，然后写上函数名，最后也是一对小括号，大括号里写函数体。
```bash
function name(){
	echo "hello,world"
}
```

Shell的函数也是有返回值的，不过返回值的类型很少不像Java中可以返回数字字符串甚至返回对象，在Shell中只能返回数字，还只能是0~255的。变量只有8个二进制位，就算你返回了一个255以上的数字也会返回出溢出之后的数字的。返回值是不可以赋值的，想使用函数的返回值的话要用`$?`，这个符号代表上一次调用的函数的返回值。
```bash
#!/bin/bash

function my(){
    return $1
}

my 1
echo $?
my 10
echo $?
my 256 
echo $?
my -2
echo $?
my 123.123
echo $?
```
这里前两个可以正常输出，256因为溢出了所以输出不正常，负数和浮点数不支持。

# 0X0B test测试
test测试可以测试各种数据，包括数字、字符串、文件。使用方法如下
```bash
#!/bin/bash

if test -e /etc/passwd #判断是否存在这个文件
then
	echo "ok"  #存在则输出ok
else
	echo "no"  #不存在则输出no
fi
```

## 针对数字的测试
-eq	等于则为真
-ne	不等于则为真
-gt	大于则为真
-ge	大于等于则为真
-lt	小于则为真
-le	小于等于则为真

## 针对字符串的测试
=	等于则为真
!=	不相等则为真
-z 字符串	字符串的长度为零则为真
-n 字符串	字符串的长度不为零则为真

## 针对文件的测试
-e 文件名	如果文件存在则为真
-r 文件名	如果文件存在且可读则为真
-w 文件名	如果文件存在且可写则为真
-x 文件名	如果文件存在且可执行则为真
-s 文件名	如果文件存在且至少有一个字符则为真
-d 文件名	如果文件存在且为目录则为真
-f 文件名	如果文件存在且为普通文件则为真
-c 文件名	如果文件存在且为字符型特殊文件则为真
-b 文件名	如果文件存在且为块特殊文件则为真
