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









































