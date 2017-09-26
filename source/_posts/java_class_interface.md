---
title: Java 抽象类和接口 理解抽象类和接口
date: 2015-11-21 23:59
tags:
  - Java
  - 面向对象
---


# 0X00抽象类&接口简介
抽象类

	abstract 抽象修饰符——抽象就是为了让子类集成的，并不能直接实现一个对象
    抽象类中所有抽象方法都要在子类中实现
    拥有抽象方法的类必须声明为抽象类
    抽象类可以有非抽象的方法

接口

	interface 接口修饰符——接口是为了让类实现的
    变量默认是public static final并且不能改变
    方法默认是public abstract并且不能改变
    接口不实现方法

# 0X01抽象类和接口的区别
	抽象类可以实现方法细节，接口不能
	抽象类的变量可以是各种类型的，接口不能
	抽象类可以有静态代码块和静态方法，接口不能
	一个类可以实现多个接口，而只能继承自一个抽象类
	继承可以理解成“是不是”，接口可以理解成“有没有”

# 0X02举个例子

有一个接口CanFly
```java
public interface CanFly {

    public abstract void fly();

}
```

有一个抽象类Bird
```java
public abstract class Bird {

    int age;

    void eat(){
        System.out.println("I can eat insect~");
    }

}
```

有一个Sparrow类继承自Bird
```java
public class Sparrow extends Bird implements CanFly{

    public void fly(){
        System.out.println("I can fly");
    }

}
```

有一个抽象类Airplane
```java
public abstract class Airplane {

    double price;//价格

    void Crash(){   //坠毁
        System.out.println("This airplane is crashed!");
    }

}
```

有一个Jian_10类继承自Airplane
```java
public class Jian_10 extends Airplane implements CanFly{

    public void fly(){
        System.out.println("I can fly");
    }

}
```

有一个包含主方法的类来测试
```java
public class Main {

    public static void main(String[] args) {
        Jian_10 A_0 = new Jian_10();//实例化A_0号战机
        Sparrow xiaoMing = new Sparrow();//没错，这只麻雀叫小明

        //我们都能飞
        A_0.fly();
        xiaoMing.fly();

        //小明吃饭了
        xiaoMing.eat();

        //战机坠毁了
        A_0.Crash();
    }

}
```

运行结果是这样的
```
I can fly
I can fly
I can eat insect~
This airplane is crashed!
```

# 0X03粗略解释
大概是这么回事：
Airplane和Bird是两个抽象类，Jian_10和Sparrow分别继承自他们，所以子类可以直接调用父类的方法。且Jian_10和Sparrow还有接口CanFly 。然后Jian_10和Sparrow实现了接口CanFly中声明的fly方法（必须实现）。
如果以后想要修改Airplane和Bird两个父类的方法的时候，比如我不想让Bird吃东西了或者Airplane不会坠毁了，就只需要修改Airplane和Bird中相应的方法。
>一个类只能继承自一个类&抽象类，但是可以实现多个接口


比如，Airplane和Bird有很多相同的方法，但是实现不尽相同，我们就可以把这些方法放到一个接口中。
