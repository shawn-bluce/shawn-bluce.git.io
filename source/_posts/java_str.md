---
title: Java中的字符串
date: 2017-02-13 14:48
tags:
  - Java
  - 字符串
---

> String类中每一个看起来会修改String值得方法，实际上都是创建了一个全新的String对象，以包含修改后的字符串内容。而最初的String对象则丝毫未动。   ---《Java编程思想》第13章

# 0X00 String常量池
如果使用常用的方式定义两个内容完全一样的字符串，那么Java使用常量的方式，也就是说第二个字符串并没有生成一个对象而是引用了之前的字符串，导致他们的本质是一样的，所以当使用`==`判断两个字符串对象是否是同一个对象的时候，会显示是同一个对象。但是如果我们每次声明一个字符串的时候使用`new String()`的方式，则会每次创建一个String对象，两者就不是同一个对象了。
```Java
public class Main {
    public static void main(String args[]) {
		// 两个相同的字符串引用自同一处
        String str1 = "hello,world";
        String str2 = "hello,world";
        System.out.println(str1 == str2);

		// 这两个字符串就是每次生成一个新对象
        String str3 = new String("hello,world");
        String str4 = new String("hello,world");
        System.out.println(str3 == str4);
    }
}
```

# 0X01 StringBuilder
字符串的拼接在Java中非常方便，但常用的使用`+`来拼接字符串效率很低，在需要拼接的次数不是很多的时候不会影响多少效率，但当需要拼接的字符串数量很多的时候就需要使用`StringBuilder`来拼接。该类中有一个`append()`的方法，就是将一个字符串连接到本对象的字符串后面。下面我们来对比一下这两个拼接方法的速度差异。
```Java
public class Main {
    public static void main(String args[]) {

        String string = "hello,world";
        long start = new Date().getTime();
		// 循环连接1W次
        for (int i = 0; i < 10000; i++){
            string += "hello,world";
        }

        long middle = new Date().getTime();

        StringBuilder stringBuilder = new StringBuilder("hello,world");
		// 循环连接1000W次
        for (int i = 0; i < 10000000; i++){
            stringBuilder.append("hello,world");
        }
        long end = new Date().getTime();

        System.out.println("使用+连接耗时: " + (middle - start));
        System.out.println("使用StringBuilder连接耗时: " + (end - middle));
    }
}
```
执行结果如下
```bash
使用+连接耗时: 850
使用StringBuilder连接耗时: 263
```
速度差异很明显，使用加号连接只连接一万次就耗时800多毫秒，而使用StringBuilder即使连接一千万次也只需要200多毫秒。

该类中还有其他的方法`insert()指定位置插入` / `replace()分片赋值` / `substring()提取子字符串` / `reverse()翻转字符串` / `toString()返回字符串`
使用演示
```Java
public class Main {
    public static void main(String args[]) {
        StringBuilder stringBuilder = new StringBuilder("0123456789");
        stringBuilder.insert(5, "hello,world");	// 插入字符串
        System.out.println(stringBuilder.toString());

        stringBuilder = new StringBuilder("0123456789");
        stringBuilder.replace(3, 5, "hello,world");	// 分片赋值
        System.out.println(stringBuilder.toString());

        stringBuilder = new StringBuilder("0123456789");
        stringBuilder.substring(3, 5);	// 提取子字符串
        System.out.println(stringBuilder.toString());

        stringBuilder = new StringBuilder("0123456789");
        stringBuilder.reverse();	// 翻转字符串
        System.out.println(stringBuilder.toString());
    }

}
```

# 0X02 String的其他方法
String类中有很多方法，这里有几个常用的
```Java
public class Main {
    public static void main(String args[]) {
        String string;
        boolean bool;

        string = "hello,world";
        bool = string.equals("hello,world");    // 判断字符串是否相同
        System.out.println(bool);

        string = "hello,world";
        bool = string.contains("hello");    // 检查字符串中是否有另一个字符串
        System.out.println(bool);

        string = "hello,world";
        bool = string.startsWith("he"); // 是否以某个字符串开头
//        bool = string.endsWith("ld"); // 结尾
        System.out.println(bool);

        string = "hello,world";
        string = string.replace("world", "java");   // 字符串搜索替换
        System.out.println(string);

        string = "hello,world";
        string = string.toUpperCase();  // 全转成大写
//        string = string.toLowerCase();    // 小写
        System.out.println(string);

        string = "  hello,world   ";
        string = string.trim(); // 去掉字符串两头的空白
        System.out.println(string);
        
    }
}
```

# 0X03 String的正则方法
简单的正则匹配可以直接使用String类中的方法，比如查看字符串是否符合某正则表达式的`matches()`和切割字符串的`split()`。
```Java
public class Main {
    public static void main(String args[]) {
        String string = "java0python00cpp";
        System.out.println(string.matches("[\\w\\d]+"));	// 检测是否能匹配

        System.out.println();

        String[] strings = string.split("\\d+");	// 按正则表达式分割字符串，返回字符串数组
        for(int i = 0; i < strings.length; i++){
            System.out.print(strings[i] + "  ");
        }
    }
}
```
