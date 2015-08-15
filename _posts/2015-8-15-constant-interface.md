---
layout:     post
title:      对 Constant interface 的一些思考
category:   []
tags: [Tech]
published: True
date: 2015-08-15
summary: Java 中对 Constant interface 的错误使用以及如何更好的定义常量
---

####Constant interface简介
我在开发的过程中，看到了类似于下面这样的代码：
```java
public interface FruitConstants {
     int APPLE = 0;
     int ORANGE = 1;
     int BANANA = 2;
}

public class Breakfast implements FruitConstants {
     public Fruit provideFruit(int type) {
          switch (type) {
          case APPLE:
               return new Apple();
          case ORANGE:
               return new Orange();
          case BANANA:
               return new Banana();
          default:
               return null;
          }
     }
}
```  
上面的代码将我们需要用到常量值定义到 interface 中，而需要使用这些常量的类，则直接实现这个 interface，这样就可以在类中直接访问 interface 中定义的所有常量。很多人第一次看到这种写法，可能会觉得这是一个不错的小技巧。我在公司的代码中看到这种写法时，也觉得不错，甚至打算自己也实践一番。直到后来自己真正需要定义常量，仔细思考了一下这个问题，再加上查阅了 *Effective Java* 的*第 19 条：接口只用于定义类型*，我意识到这种写法存在很大的问题。

####Constant interface 的几个缺点
首先，定义常量接口的一个可能的出发点，是在多个类中需要使用到这些常量，每个类只需要实现此接口即可。那么你很可能会把所有常量定义在一个接口中，而不同的类可以访问所实现接口的所有常量，这无形中扩大了接口（通用的接口概念，不是 interface），你不需要使用所有的常量，但是你却能够访问所有的常量，那些和实现无关的常量会给使用接口的程序员带来很大的困惑。更进一步，由于很可能不存在类型错误，你可能误用常量。

interface 中所有的成员变量（严格来讲是常量）自动都是 *public static final* 的，那么实现类会自动将所有这些成员导出，也就是使用实现类的时候，你也可以没有任何限制的访问它们，对于公开 API 来说，这算是一个比较严重的错误。因此从上一点和这一点来看，interface 天然的属性会导致成员不断向外扩散，你无法施加任何权限保护措施。

从 interface 设计的思想来看，其代表的是一种能力。如果一个类实现了一个 interface，那么表示这个类能够做一件事情或者具有某种功能。比如 JDK 中的 [java.io.Serializable](http://developer.android.com/reference/java/io/Serializable.html) 和 [java.lang.Cloneable](http://developer.android.com/reference/java/lang/Cloneable.html) 接口，都是如此。在某些情况下，你可能需要在代码抽象实现中，将 interface 抽象类型作为方法的参数传入，以达到解耦或者多态的目的。这个时候，如果将常量接口类型传入，逻辑上是讲不通的。

曾有同事和我讨论过此问题，他觉得常量接口没有问题，理由是我们完全可以通过接口名来引用常量，就像普通的类一样。我对此表示不能赞同。当你定义了一个 interface，你默认就给别的程序员做出承诺，我定义了一个接口，你们需要的话可以来实现它。退一步讲，就算你不打算让别人实现你的接口，在注释中严词警告，也无法阻止别人从技术上不小心实现这个接口。这是非常危险的行为。当然，我们在这里忽略了接口的访问权限的问题，是为了简化这里的讨论。代码是有意义的，类型的声明在 Java 语言的环境下，也是具有特定含义的，混用或者故意使用不合适的类型，在功能上可能不会有什么问题，但是对于整个软件或者功能模块的设计来说，是存在潜在的问题的。我不建议为了功能上的一点点便利，而牺牲设计上的正确性。

####如何定义常量

那么，说了那么多常量接口的坏话，提出一种更好的实现方法是必须的。

对于和某个模块或者某个类紧密相关的常量，直接定义到相关的包中或者类中，不要定义成全局的常量。全局常量即使是定义在 class 中，也不比常量接口好多少。我们在使用 Android 的 API 的时候，为什么可以很方便的使用各种常量，就是因为这些常量直接定义在 API 相关的类中，比如 [Service](http://developer.android.com/reference/android/app/Service.html) 在启动的时候在 [onStartCommand()](http://developer.android.com/reference/android/app/Service.html#onStartCommand(android.content.Intent, int, int)) 回调方法里的返回值 START_STICKY, START_NOT_STICKY, START_REDELIVER_INTENT 等。又比如 JDK 中 [Integer](http://developer.android.com/reference/java/lang/Integer.html) 等包装类有表示最大值和最小值的常量，对于 [Integer](http://developer.android.com/reference/java/lang/Integer.html) 是 [Integer.MAX_VALUE](http://developer.android.com/reference/java/lang/Integer.html#MAX_VALUE) 和 [Integer.MIN_VALUE](http://developer.android.com/reference/java/lang/Integer.html#MIN_VALUE)。

而对于某些比较公用的常量，比如文章最开始定义的水果的常量，有好几个地方都想用，那么就直接定义一个普通的 *class* 类型。为了对这个类实行比较严格的控制，你甚至可以将此类定义为 **final** 的，以禁止子类化，同时声明一个默认的私有构造方法，以禁止实例化。这样的话，大家就只能通过 **ClassName.CONSTANT_VALUE** 的方式来引用常量。其实，我是建议一般情况下都这么做的。下面就是对上面的例子的一个改进。
```java
public final class FruitConstants {
     private FruitConstants() {}
     int APPLE = 0;
     int ORANGE = 1;
     int BANANA = 2;
}
```
另外提一点，为了增强可读性，你可以使用十六进制定义这些常量。

不过，用整型定义常量，也有一些不好的地方，比如你无法保证这些整型的数值都是不一样的，也无法进行类型检查，就是说别人可以任意传递一个整型数值过来，而不会检查它是不是你定义的常量类中的数值。还有一些其它原因，会造成一些潜在的问题。这种情况，有一个更好的选择，就是枚举类型，枚举类型的可扩展性也很强，当你把它看作一种类型，而不仅仅是“常量”的时候，能够适应各种编码情况。但是关于枚举就不在这篇文章中详谈了，以后可以专门写一篇这方面的文章。

可能需要注意的一点是，枚举类型毕竟不是基础数据类型，性能方面稍逊于整型，具体的性能差异我没有测试过，不过在一般情况下，更需要考虑的应该是代码的设计，需要自己权衡一下。
