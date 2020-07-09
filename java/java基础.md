[TOC]

## 概述

这块主要写的是java的一些基础的特性

主要包括，java的特性、三大特征继承多态封装

类的初始化、反射、泛型。。。

等等，一些基础的东西，像集合、线程这种考点较多的准备单独分一章





### 1. Java 类初始化的一个过程

**初始化一般遵循3个原则**：

* 静态对象（变量）优先于非静态对象（变量）初始化，静态对象（变量）只初始化一次，而非静态对象（变量）可能会初始化多次
* 父类优先于子类进行初始化
* 按照成员变量的定义顺序进行初始化

**加载顺序**

* 父类的静态变量、静态代码块
* 子类的静态变量、静态代码块
* 父类的实例变量、普通语句块
* 父类的构造函数
* 子类的实例变量、普通语句块
* 子类的构造函数



### 2. 数据类型

#### 基本数据类型

| 类型    | 初始化值 | 字节 |
| ------- | -------- | ---- |
| byte    | 0        | 8    |
| short   | 0        | 16   |
| int     | 0        | 32   |
| long    | 0L       | 64   |
| float   | 0.0f     | 32   |
| double  | 0.0d     | 64   |
| char    | '\u0000' | 16   |
| boolean | false    | ~    |

> boolean 只有两个值：true、false，可以使用1bit来存储，但是没有明确规定具体大小。JVM会在编译时期将boolean类型的数据转换为int，使用1来表示true，0表示false。JVM支持boolean数组，但是是通过读写byte数组来实现。



#### 包装类型

自动装箱拆箱

```java
Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
int y = x;         // 拆箱 调用了 X.intValue()
```

>  ps: 这不知道怎么考

#### 缓存池

`new Integer(123) `与` Integer.valueOf(123) `的区别在于：

* `new Integer(123)` 每次都会创建一个新的`Integer`对象
* `Integer.valueOf(123)`，会使用缓存池中的对象

`Integer.valueof()`方法会先从缓存池中获取，如果没有再新建。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
```

**基本类型对应的缓冲池：**

### 3. String

#### 内部实现

String被声明为`final`，因此无法被继承。

Java8中，String内部是 char[]

```jav
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

Java9中，String内部是 byte[]，同时使用`coder`来标识编码

```jav
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final byte value[];
    /** The identifier of the encoding used to encode the bytes in {@code value}. */
    private final byte coder;
}
```

String为什么是不可变的：内部实现是被final修饰的数组，初始化后无法修改，而且String内部也没有对此修改的方法。

#### 不可变的好处

1. **可以缓存 hash值**： string不可变，对应hash值也不变，计算+比较都方便
2. String Pool 的需要： 如果一个对象已经被创建了，就会从 String pool中取得引用。
3. 保证线程安全

#### String，StringBuffer and StringBuilder

1. String不可变，StringBuffer和StringBuilder可变
   1. String线程安全，StringBuilder不是线程安全， StringBuffer是线程安全，内部使用 synchronized

#### String Pool

字符串常量池中保存着所有字符串字面量，这些字面量实在编译时确定的。还可以使用String.intern()方法在运行时过程将字符串添加到String pool中。

在java7之前，String pool被放在运行时常量池内。在Java7， String pool被移动到堆中。



### 4. 关键字

#### final

修饰属性：使其不可变，如果修饰的是引用类型，还是可以修改其引用对象的。

修饰方法：该方法不能被子类重写。

修饰类：该类不能被继承

#### static

1. 静态变量：属于类，在内存中只存一份，对所有的实例都共享

2. 静态方法：

   > 在类加载的时候就存在，不依赖于任何实例。所以静态方法必须被实现。
   >
   > 只能访问所属类的静态属性和静态方法

3. 静态语句块：在类初始化的时候运行一次

4. 静态内部类：非静态内部类依赖



### 5. Object



### 6. 继承



### 7. 异常



### 8. 泛型



第一行是无法通过编译的，这个例子是用来假设举例的。
数组是协变的，而泛型不是。不能实例化泛型类型的数组,除非type参数是一个无界通配符。

```java
new List<String>[3]; is illegal
new List<?>[3]; is legal
```

如果java中的的泛型是协变的，那么就会出现，在`lsa`中`List<String>`类型对象中添加` li(List<Integer>)`, 这样会导致`ClassCastException`。
虽然他们的父类都是object，但是在jvm中最终进行转换的时候，还是会把泛型**擦除(*erasure* )**转换成的实际类型。`List<Integer>`转换成`List<String>`自然是转换错误了。


```java
List<String>[] lsa = new List<String>[10]; // illegal
Object[] oa = lsa;  // OK because List<String> is a subtype of Object
List<Integer> li = new ArrayList<Integer>();
li.add(new Integer(3));
oa[0] = li; 
String s = lsa[0].get(0);
```

> 参考：https://www.ibm.com/developerworks/java/library/j-jtp01255/index.html



### 9. 反射



### 10. 注解



### 11. 特性

#### Java 区别于C++的一些特性

* Java是纯粹的面向对象的语言
* Java通过`JVM`实现跨平台
* Java没有指针，引用可以理解为安全指针
* Java支持自动垃圾回收  ` gc`
* Java不支持多重继承，但是可以通过实现多个接口
* Java不支持操作符重载
* Java 内置了线程的支持