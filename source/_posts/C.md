---
title: C++
date: 2018-10-22 15:51:20
tags:
categories: C++
---

# malloc/free和new/delete的区别
1. malloc/free成对使用，是C语言的函数。new/delete和new\[]/delete[]也是成对使用，是C++语言的操作符。
1. new会先调用构造函数，delete会先调用析构函数，而malloc和free不会。

# C++的const
1. 常量的指针也必须是指向常量的指针，而变量的指针没有限制。

```
const int* p=1; //表示指针指向常量。
int* const p=1; //表示指针是常量。

const int a=1;
int* ap=&a; //出错，const int*不能转成int*。
int b=1;
const int* bp=&b; //正常，int*可以转成const int*。
```

# C和C++的const的区别
1. 作用域不同。C的全局const作用域是整个程序，C++的全局const作用域是当前文件。
1. 在ANSI标准下，C的const表示只读变量，C++
表示的是常量。

```
const int size=5;
int array[size]; //在ANSI标准的C中是错误的，size是变量，虽然是只读。在C++中是正确的。
```

3. C++的const作用范围广，可以作用于函数。