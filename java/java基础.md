[TOC]

## 概述

**本文非原创，主要是抄的，自己加了些理解。仅用于个人记录！查看！**

这块主要写的是java的一些基础的特性

主要包括，java的特性、三大特征继承多态封装

类的初始化、反射、泛型。。。

等等，一些基础的东西，像集合、线程这种考点较多的准备单独分一章





### 1. Java 类初始化的顺序

类加载的过程：加载、连接、初始化。

加载的过程涉及到类加载器，`ClassLoader`。

* `Bootstrap ClassLoader`：启动类加载器，`jvm`自带，c++ 编写，用来加载 `java`核心库`java.*`
* `Extension ClassLoader`:  扩展类加载器，由`java`实现的 独立于`jvm`, 加载扩展库
* `App ClassLoader`: 系统类加载器 java编写，加载程序所在的目录，如`user.dir`所在的位置的class
* `CustomClassLoader`: 用户自定义类加载器,`java`编写,用户自定义的类加载器,可加载指定路径的`class`文件

双亲委派？

类加载的时候先让自己的父加载器加载，如果父找不到，一直往上查找，直到最顶层的 bootstrap ClassLoader，最顶层的父类还是找不到，那就 `ClassNotFoundException`了。

作用：

1. 防止重复加载
2. 防止核心类被篡改



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

| 类型    | 初始化值 | bit  |
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

缓存池

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

`Java8`中，String内部是 char[]

```java
public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
    private int hash;
}
```

`Java9`中，String内部是 byte[]，同时使用`coder`来标识编码

```java
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
2. String线程安全，StringBuilder不是线程安全， StringBuffer是线程安全，内部使用 synchronized
#### String Pool

字符串常量池中保存着所有字符串字面量，这些字面量实在编译时确定的。还可以使用`String.intern()`方法在运行时过程将字符串添加到String pool中。

在java7之前，String pool被放在运行时常量池内。在Java7， String pool被移动到堆中。

#### 参考文章

> [在Java虚拟机中，字符串常量到底存放在哪](https://juejin.im/post/5c3d3121e51d4551741171fe)

### 4. 关键字

#### final

修饰属性：使其不可变，如果修饰的是引用类型，还可以修改其引用对象的。

修饰方法：该方法不能被子类重写。

修饰类：该类不能被继承

#### static

1. 静态变量：属于类，在内存中只存一份，对所有的实例都共享

2. 静态方法：

   > 在类加载的时候就存在，不依赖于任何实例。所有静态方法必须被实现。
   >
   > 只能访问所属类的静态属性和静态方法

3. 静态语句块：在类初始化的时候运行一次

4. 静态内部类：非静态内部类依赖外部实例。静态内部类只能访问外部类的静态变量和方法。

5. 静态导包：使用静态变量和方法时，无须指定 ClassName ,简化代码

6. 初始化顺序：见上文



### 5. Object

```java
public final native Class<?> getClass();
public native int hashCode();
public boolean equals(Object obj) {return (this == obj);}
protected native Object clone() throws CloneNotSupportedException;
public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());}
public final native void notify();
public final native void notifyAll();
public final native void wait(long timeout) throws InterruptedException;
public final void wait(long timeout, int nanos) throws InterruptedException;
public final void wait() throws InterruptedException {wait(0);}
protected void finalize() throws Throwable {}
```

#### equals()

1. 等价关系：自反性、对称性、一致性、传递性、和null比较为false
2. 实现
   * 检查引用是否相等
   * 检查类型是否一致
   * 对object进行类型转换
   * 判断关键域是否相等



#### hashCode()

hashCode() 返回哈希值。

> 在覆盖 equals() 方法时应当总是覆盖 hashCode() 方法，保证等价的两个对象哈希值也相等。
>
> HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。
>
> 理想的哈希函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求了哈希函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。
>
> R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。并且一个数与 31 相乘可以转换成移位和减法：`31*x == (x<<5)-x`，编译器会自动进行这个优化。

 

#### toString()

getClass().getName() + "@" + Integer.toHexString(hashCode())

默认返回类名 + @ + hashCode的无符号十六进制表示



#### clone()

1. 类实现cloneable接口才能调用clone()接口，不然会抛 CloneNotSupportedException
2. 浅拷贝
3. 深拷贝
4. 最好使用手写的构造函数或者工厂来实现clone的需求



### 6. 继承

#### 访问权限

private、protected、public、default

#### 抽象类与接口

1. 抽象类

   抽象类和抽象方法都使用abstract关键字进行声明。如果一个类中包换抽象方法，该类就必须被申明为抽象类。

   抽象类和普通类最大的区别就是，抽象类不能被实例化只能被继承。

2. 接口

   * 接口是抽象类的延伸
   * 接口可以继承接口
   * 接口的成员（字段、方法）默认都是pubilc的，而且不允许定义为private或protected。接口的字段默认是static和final的
   
   ```java
   public interface DemoInterface {
       // 常量 == 默认是static和final的
       String NAME = "123";
       // 默认方法
       default void defaultMethod() {
           System.out.println("defaultMethod");
       }
       // 需要实现的方法
       void needImplMethod();
   }
   
   public class DemoInterfaceImpl implements DemoInterface {
       @Override
       public void needImplMethod() {
           String name = DemoInterface.NAME;
           System.out.println(name);
           new DemoInterfaceImpl().defaultMethod();
       }
   }
   ```
   
   
   
3. 两者比较

   * 从设计上看，抽象类提供了一种 **is-a**  的关系，需要满足子类对象必须能够替换所有父类对象的 “**里氏替换原则**”。而接口更像是一种 **like-a** 关系，它只是提供一种方法实现契约，不要求接口和实现类有这种 is-a 强关系。
   * 从使用上来看，一个类只能继承一个抽象类，但是可以实现多个接口。
   * 接口的字段只能是static和final的。接口的成员只能是public。
   * 使用时，接口可以让不想关的类都实现一个方法。但是抽象类主要是，把几个相关类的共性代码抽取出一个抽象类。

#### 参考

[Abstract Methods and Classes](https://docs.oracle.com/javase/tutorial/java/IandI/abstract.html)

[Java 的接口和抽象类详解](https://juejin.im/entry/576d0e73816dfa0055cb6e07)



####  重写与重载

1. 重写（@Override）

   存在于继承体系中，指的是子类实现了一个与父类在方法声明上完全相同的一个方法。

   为了满足**里氏替换原则**，重写有以下三个限制：

   * 子类方法的访问权限必须大于等于父类方法
   * 子类方法的返回类型必须是父类方法返回类型或者是其子类
   * 子类方法抛出的异常类型必须是父类抛出异常类型或者是其子类

2.  重载

   在同一个类中，存在多个相同方法名称，但是参数类型、个数、顺序至少有一个不同。

   如果只有返回值不同，不算是重载。

### 7. 异常

`Throwable` 可以用来表示任何可以作为异常抛出的类，分为两种： **Error** 和 **Exception**。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

* 受检异常：需要使用 try catch 语句捕获并进行处理，并且可以从异常中回复
* 非受检异常：是程序运行时错误，例如除0会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。





### 8. 泛型

#### 什么是泛型？

泛型提供一种编译时期的类型安全。防止开发过程中不停的类型转换，确保只能把正确类型放入集合，避免运行时出现 `ClassCastException`。

##### 类型擦除

泛型是通过类型擦除来实现的。编译器在编译时期擦除了所有类型相关的信息，所以在运行时不存在任何类型相关信息。例如`List<String>` 在运行时仅仅使用`List`代替。

#### 限定通配符和非限定通配符

非限定：< ? >

限定通配符：

* <? extends T>它通过确保类型必须是T的子类来设定类型的上界
* <? super T>它通过确保类型必须是T的父类来设定类型的下界



#### eg:

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

每个类都有个一Class对象，包含了类有关信息。当编译一个新类时，会产生一个同名的.class文件，该文件内容保存着Class对象。

类加载，相当于Class对象的加载，类在第一次使用时才动态加载到`JVM`中。也可以使用`Class.froName()`这种方式来控制类的加载。

反射可以提供运行时的类信息，并且这个类可以在运行时才加载进来，甚至在编译时期该类的.class	

* Field :  读取修改字段
* Method：invoke() 调用方法
* Constructor:  使用newInstance()创建新的对象

优点： 可扩展、ide开发工具、调试

缺点： 性能开销、安全限制、内部暴露

Class 和 java.lang.reflect 一起对反射提供支持，java.lang.reflect 类库主要包含以下三个类：

### 10. 注解

#### 什么是注解？

Java注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能

#### 注解的用处？

1. 生成文档。 @param @return
2. 在编译时进行格式检查。 @Override
3. 跟踪代码依赖性，实现配置文件的功能。使用注解配置

#### 注解的原理

注解本质是一个继承了 Annotation 的特殊接口，其具体实现类是Java运行时动态生成的代理类。而我们通过反射获取注解信息的时候，返回的是Java运行时生成的动态代理对象 $Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用 AnnotationInvocationHandler 的invoke方法。 该方法会从memberValues这个map中索引出对应的值。而memberValues的来源是Java 的常量池。

#### 元注解

* @Documented – 注解是否将包含在JavaDoc中
* @Retention – 什么时候使用该注解
  * source 在编译阶段丢弃（@Override, @SuppressWarnings）
  * class 在类加载的时候丢弃。
  * runtime 始终不会丢弃。
* @Target – 注解用于什么地方（构造器、成员变量、方法、包、参数、类）
* @Inherited – 是否允许子类继承该注解



### 11. 特性

#### Java 区别于C++的一些特性

* Java是纯粹的面向对象的语言
* Java通过`JVM`实现跨平台
* Java没有指针，引用可以理解为安全指针
* Java支持自动垃圾回收  ` gc`
* Java不支持多重继承，但是可以通过实现多个接口
* Java不支持操作符重载
* Java 内置了线程的支持