---
layout:     post
title:      Java 中的 void 是基础数据类型吗
published: True
date: '2015-04-11'
cover_image: '/content/images/2015/4/scaffolding.jpg'
---

Java是面向对象的编程语言，它主要包含两种数据类型，**引用类型 （Reference types）**和**基础数据类型 （Primitive types）**，普通的类（Class）、枚举类型、数组、接口（interface）都属于引用类型，而基础数据类型就是 boolean，byte，short，int，long，char，float，double。偶然的机会，看到有人讨论 void 是否属于 Java 中的基础数据类型。这是一个很有意思的问题，于是查了一些资料，并且自己动手写代码验证了一下。
####Java Language Specification
我首先想到的是最权威的 **Java Language Specification**，但是这里面并没有列出基础数据类型，void 只是出现在了 Java 关键字列表中。
####Java API
我又去查了[Java 的官方 API 文档](http://docs.oracle.com/javase/8/docs/api/)，在 *java.lang* 包里找到了 Void 类，里面是这样说的：
>The Void class is an uninstantiable placeholder class to hold a reference to the Class object representing the Java keyword void.

也就是说，Void 是一个占位类，它持有一个关键字 void 对应的 Class 对象。那么，我们可以认为，void 不是基础数据类型。
####Android API
接着我又查了[Android 的官方 API 文档](http://developer.android.com/reference/java/lang/Void.html)，Google 修改了不少 Java 标准的 API，我们可以看看 Google 的工程师是怎么想的。类的简介中说:
>Placeholder class for the Java keyword void.

但是在解释类成员变量 TYPE 的时候，又是这样描述的：
>The Class object that represents the primitive type void.

这里将 void 称为基础数据类型，那么 void 究竟是不是基础数据类型呢，我又有点疑惑了。
####程序验证
所以我又尝试着自己写程序验证：

```java
public class VoidTest {
    public static void main (String [] args ) {
        System.out.println("Is void primitive type?" + (void.class.isPrimitive() ? "YES" : "NO"));
    }
}
```

输出的结果是 **Is void primitive type? YES**，这里 Java 又告诉我 void 是基础数据类型。那么让我们再来看看 Class 类的文档，找到 isPrimitive() 方法：
>**isPrimitive**
>
public boolean isPrimitive()
>
Determines if the specified Class object represents a primitive type.
>
There are nine predefined Class objects to represent the eight primitive types and void. These are created by the Java Virtual  Machine, and have the same names as the primitive types that they represent, namely *boolean*, *byte*, *char*, *short*, *int*, *long*, *float*, and *double*.
>
These objects may only be accessed via the following public static final variables, and are the only Class objects for which this method returns true.
>
**Returns:**
true if and only if this class represents a primitive type

可以看到，里面说到，有 9 种预先定义的类用来表示 *8 种基础数据类型和 void*。所以，Java 并不把 void 作为基础数据类型，但是不知是什么原因，void 和其它基础数据类型有一些相同的行为。

其实，我们可以抛开这一切，从语言的角度去思考这个问题。基础数据类型会保存一份数据，栈中会为其分配相应的内存空间，但是 void 是方法调用的一个返回结果，也就是空。以后了解更多了，我会从更深入的原理解释这个问题。
