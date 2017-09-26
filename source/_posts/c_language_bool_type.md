---
title: C语言中的Bool类型
date: 2015-03-17 12:12
tags:
    - C语言
    - Bool
---

　　最近在网上看到有的说法里是没有bool类型的，不过以前在书上好像看到过相关的介绍，就特意找出来了那本书《C Primer Plus》，确定了C语言里确实存在bool类型。C语言是在C99标准中添加的bool类型。
>bool类型是以英国数学家 *George Boole* 命名的，是他开发了用线性代数表示并解决逻辑问题的系统。

　　在C语言中我们使用 _Bool 来定义bool类型的变量

　　下面定义了一个_Bool类型的变量，并把(1 == 3)的计算值赋值给test
```c
#include <stdio.h>

int main ()
{
    _Bool test;
    test = (1 == 3);
    return 0;
}
```

　　下面能证明bool类型变量的特点
　　　　只有0和1两个值
　　　　只有0赋值给bool类型时，bool才为0
```c
#include <stdio.h>

int main ()
{
    _Bool test;
    int i;
    for (i = -10; i < 10; i++)
    {
        test = i;
        if (test)
            printf ("true\n");
        else
            printf ("false\n");
    }
    return 0;
}
```

 　　最后我们证明一下bool类型比int类型占的内存要少

```c
#include <stdio.h>
#include <stdlib.h>

int main ()
{
    int myint;
    _Bool mybool;
    int memint;
    int membool;

    memint = sizeof(myint);
    membool = sizeof(mybool);

    printf ("int   = %d\n", memint);
    printf ("_Bool = %d\n", membool);
    return 0;
}
```
