---
title: 《Java编程思想》笔记
tags:
- Java
categories:
- Java
comments: true
date: 2016-02-29 18:10:51
updated:
desc: 《Java编程思想》笔记
---

P140 到底是用组合还是继承，一个最清晰的判断办法就是问一问自己是否需要从新类向基类进行向上转型。

P144 “覆盖”只是在某方法是基类的接口的一部分时才会出现。即，必须能将一个对象向上转型为它的基本类型并调用相同的方法。private无法覆盖父类的同名方法。

P151 Java中除了static方法和final方法（private方法属于final方法）之外，其他所有的方法都是后期绑定。后期绑定：在运行时根据对象的类型进行绑定，也叫动态绑定或运行时绑定，相对的是前期绑定。

P172 接口可以像类一样，在声明时，interface关键字前面添加public关键字。如果不添加public关键字，则它只具有包访问权限，只能在同一包内可用。接口成员变量都是**static和final的**。

```java
interface Instrument {
  // Compile-time constant:
  int VALUE = 5; // static & final
  // Cannot have method definitions:
  void play(Note n); // Automatically public
  void adjust();
}
```

只能有一个public
![](http://7xp4nq.com1.z0.glb.clouddn.com/image/blog/Screen%20Shot%202016-01-27%20at%209.10.34%20AM.png?attname=&e=1456541707&token=HscLHyfEc-LCGJUC9i88_1qLjOjnb23TXXg3UeZF:5-Im4B8hdNDJGKDYoI4tNhzgrcE
P179)

 使用接口的核心原因：1.为了能够向上转型为多个基类型；2.防止客户端程序员创建该类的对象（同抽象类）

# 内部类

P191 当生成一个内部类的对象时，此对象与制造它的外围对象之间就有了一种联系，它能访问其外围对象的**所有成员**，而不需要任何特殊条件。此外，内部类还拥有其外围类的所有元素的访问权

P193 想要告知某些其他对象，去创建其某某个内部类的对象，必须在new表达式中提供对其外部类对象的引用，需要使用.new语法。
在拥有外部类对象之前是不可能创建内部类对象的。这是因为内部类对象会暗暗地连接到创建它的外部类对象上。
如果创建的是嵌套类（静态内部类），那么它就不需要对外部对象的引用。

```java
public class DotNew {
  public class Inner {}
  public static void main(String[] args) {
    DotNew dn = new DotNew();
    DotNew.Inner dni = dn.new Inner();
  }
}
```
P194 内部类向上转型为其基类，尤其是转型为一个接口，有用。
因为此内部类是某个接口的实现，能够完全不可见，并且不可用。所得到的是指向基类或者接口的引用，所以能够**很方便地隐藏实现细节**。（内部类可以是private，实现一个接口，外围类可以返回内部类的实例，但除了外围类，没人知道内部类的具体实现）

P195 protected成员，同包的类也可访问（protected也给与了包访问权）

P198 如果定义一个匿名内部类，并且希望它使用一个在其外部定义的对象，那么编译器会要求其参数引用是final的。

P199 工厂方法？

P201 将内部类声明为static，通常称为嵌套类。普通的内部类对象隐式地保存了一个引用，指向创建它的外围类对象，然而当内部类是static时，就不是这样了。
嵌套类和普通内部类之间：

* 要创建嵌套类的对象，并不需要其外围类的对象
* 不能从嵌套类的对象中访问非静态的外围类对象
* 普通的内部类不能有static数据和static字段（IntelliJ中普通内部类中可以有static成员变量），也不能包含嵌套类。但是嵌套类可以包含这些
* 嵌套类没有特殊的this引用可以链接到外围类对象

```java
public class Test {
    public static void main(String args[]) {
        Test test = new Test();
        Test.Destination destination = test.new Destination();
        destination.getClassName();
    }

     class Destination {
        public void getClassName() {
            System.out.println(this.getClass().toString());
        }
    }
}
```
输出： class com.vinc\.www\.Test$Destination


```java
public class Test {
    public static void main(String args[]) {
        Test test = new Test();
        Test.Destination destination = test.new Destination();
        destination.getClassName();
    }

     class Destination {
        public void getClassName() {
            System.out.println(Test.this.getClass().toString());
        }
    }
}
```
输出：class com.vinc.www\.Test


```java
public class Test {
    public static void main(String args[]) {
        Test test = new Test();
        Test.Destination destination = new Destination();
        destination.getClassName();
    }

     static class Destination {
        public void getClassName() {
            System.out.println(Test.this.getClass().toString()); //报错
        }
    }
}
```
报错

P203 一个内部类被嵌套多少层并不重要，它能够透明地访问所有它所嵌入的外围类的所有成员


P220 `Arrays.asList()`底层表示的是数组，因此不能调整尺寸。`add()` `delete()`会引发错误

```java
List<Integer> list = Arrays.asList(16, 17, 18, 19, 20);
list.set(1, 99); // OK -- modify an element
// list.add(21); // Runtime error because the
// underlying array cannot be resized.
```

以下代码，只输出0

```java
List<Integer> list = Arrays.asList(16, 17, 16, 19, 20);
List<Integer> old = new ArrayList<Integer>();
old.addAll(list);
System.out.println(old.indexOf(16));
```


