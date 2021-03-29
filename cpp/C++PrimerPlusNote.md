# 第1章 预备知识

## 1.1 C++ 简介

c++ 是一种面向对象OOP 语言， 后面还支持了泛型编程

## 1.2 C++ 简史

诞生于20世纪80年代，贝尔实验室，Bjarne Stroustrup





# 第2章 开始学习C++

## 2.1 进入C++ 

### 2.1.1 main() 函数

主函数么，作为程序的入口

### 2.1.2 C++ 注释

```c++
// 行尾注释

/*
* 段落注释
*/
```

### 2.1.3 C++ 预处理和iostream 文件



。。。略，后面补充，看到第七章了才想起来记笔记





# 第7章 函数——C++的编程模块

## 7.1 复习函数的基本知识

### 7.1.1 定义函数

函数原型

函数实现

函数调用

```c++
typeName functionName(paremeterList)
{
    // content
    return typeName; 
}
```

### 7.1.2 函数原型和函数调用



## 7.2 函数参数和按值传递



## 7.3 函数和数组

引用传递么

## 7.4 函数和二维数组

这个和上面这个差不多，对javaer来说，可以理解为传递的都是一个对象的地址。

​	

## 7.5 函数和C- 风格字符串

以 `'\0'` 空值字符结尾

## 7.6 函数和结构

可以把结构理解为java中的对象么？

## 7.7 函数和string 对象

## 7.8 函数与array对象

传递array 作为函数的参数

[std::array](http://www.cplusplus.com/reference/array/array)

有序、定长、线性存储

有两个模板参数 ：  泛型、长度

```c++
void testArray() {
    array<string, 4> arrays = {"af", "ewr", "gad"};

    cout << "array.size : " << arrays.size() << endl;
    cout << "array.max_size : " << arrays.max_size() << endl;
    cout << "array.empty : " << arrays.empty() << endl;

    arrays[3] = "adfaa";

    for (int i = 0; i < arrays.size(); i++) {
        cout << "array" << i << ": " << arrays.at(i) << endl;
    }
    cout << "array.size : " << arrays.size() << endl;
    cout << "array.max_size : " << arrays.max_size() << endl;

    for (auto iterator = arrays.begin(); iterator != arrays.end(); iterator++) {
        cout << "array: " << *iterator << endl;
    }

    for (auto & array : arrays) {
        cout << "array: " << array << endl;
    }
}
result:
array.size : 4
array.max_size : 4
array.empty : 0
array0: af
array1: ewr
array2: gad
array3: adfaa
array.size : 4
array.max_size : 4
array: af
array: ewr
array: gad
array: adfaa
```





## 7.9 递归

介个就是递归！

<iframe 
src="//player.bilibili.com/player.html?aid=87819206&bvid=BV1K741147NA&cid=150040663&page=1" 
scrolling="no" 
border="10"
frameborder="no"
framespacing="10"
allowfullscreen="true">
</iframe>







## 7.10 函数指针

指向某个函数的指针

这个我在学qt的时候，有一个，信号槽连接的地方，就可以先实现一个函数，然后连接的时候用函数指针指向这个函数。当然也可以直接用匿名函数，省略这个函数指针。不过拆开可以让代码更加易读。

```c++

double fun(int);
double (*fun1)(int);

fun1 = fun;

```



# 第8章 函数探幽

## 8.1 C++内联函数

**What?**

> 函数在编译时，在调用函数时，直接替换为一个该函数的副本。
>
> 相对于普通的函数调用来说，调用时，跳至函数的地址，执行完毕后返回

**How?**

* 在函数声明前加上关键字 `inline`
* 在函数定义前加上关键字 `inline`

通常会省略原型，直接将整个定义放在本应提供原型的地方

**注： 申请将原函数作为内联函数时，编译器不一定会满足。函数过大或者有递归，编译器不会转成内联。**



## 8.2 引用变量

**What?**

> 引用是已定义的变量的别名
>
> 可以作为函数的形参，在函数中直接适用原函数而不是副本



**How?**

适用 `&` 来声明引用

```c++
int name;		
int & alias = name;
```



## 8.3 默认参数

**What?**

> 在声明函数时，给参数定义一个默认值，在调用这个函数时可以省略这个参数
>
> 省略的参数必须在最后。

```c++
// 声明
void defaultParam(int param = 1);

// 调用 
defaultParam();
defaultParam(1);
```



## 8.4 函数重载

函数名相同，参数列表不一致。又称特征标不一致



## 8.5 函数模板

泛型

```c++
template <typename T>
void fun(T param);

```





重载模板

显示具体化



# 第9章 内存模型和名称空间



## 9.1 单独编译

单独编译组件函数文件， 然后将其链接成可执行的程序。

可以将程序分成三个部分

* 头文件：包含结构声明和使用这些结构的函数的原型
  * 函数原型、使用`#define`或`const`定义的符号常量
  * 结构声明、类声明、模板声明、内联函数
* 源代码文件：包含于结构有关的函数的代码
* 源代码文件：包含调用与结构相关的函数的代码



## 9.2 存储持续性、作用域和链接性

* 自动存储持续性：在函数定义中声明的变量。我理解为局部变量，它的生命周期随着函数变化。
* 静态存储持续性：被static关键字修饰的。它们在程序整个运行过程中都存在。我理解为，随着类的生命周期变化。
* 线程存储持续性：用thread_local关键字声明的。其生命周期随着该线程变化。
* 动态存储持续性：用new运算符分配的内存，直到使用delete 运算符将其释放或程序结束为止。





### 9.2.10 存储方案和动态分配内存

动态内存由运算符`new`和`delete`控制

通常，编译器使用三块独立的内存

* 一块用于静态变量（可能再细分）
* 一块用于自动变量
* 一块用于动态存储



**定位new运算符？**

> new 可以自动在 堆（heap） 中找一个满足要求的内存块。
>
> 而定位new运算符，可以指定要使用的位置，不管该内存块是否满足要求。可以用这种特性来自己实现内存管理，或者用来处理需要通过特定地址进行访问的硬件

```c++
char buffer[50];

int main(){
    p1 = new int[20]; // place int array in heap 
    p2 = new (buffer) int[20]; // place int array in buffer2
}
```





## 9.3 名称空间

命名空间

```c++
namespace djx{
    
    
}

using namespace djx;

```



使用using的尽可能缩小范围，使用`::` 或者 局部声明





# 第10章 对象和类



面向对象编程 OOP

* 抽象
* 封装和隐藏数据
* 多态
* 继承
* 代码的可重用性





## 10.1 过程性编程和面向对象编程

## 10.2  抽象和类

c++中的类

* 类声明：以数据成员的方式描述数据部分，以成员函数的方式描述共有接口
* 类方法定义：描述如何实现类成员函数

类声明提供类的结构，方法定义提供细节

C++中类对象默认private

```c++
class World
{
    float mass;		// private by default
    char name[20];	// private by default
 public: 
    void tellall();
    ...
}
```



成员函数定义有两个特征

* 需要使用作用域解析运算符`::` 来标识函数所属类
* 类方法可以访问类的private组件



## 10.3 类的构造函数和析构函数



​		

































