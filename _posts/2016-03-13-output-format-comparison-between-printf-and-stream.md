---
layout: post
title: "常用输出格式归纳：printf 和 stream"
subtitle: "output format comparison between c-style printf and cpp-style stream"
create-date: 2016-03-13
update-date: 2016-03-14
header-img: ""
author: "Mensu"
tags:
    - 归纳
    - C / C++
---

> The article was initially posted on **{{ page.create-date | date: "%Y-%m-%-d" }}**.

说起输出格式的控制，C 中用得最多的是各种 printf ，例如 `printf`、`fprintf`、`sprintf` ，声明于 `<stdio.h>` ，调用时必须通过字符串设置格式  
而 C++ 中一般使用各种 stream ，例如 `ostream`、`ofstream`、`ostringstream` ，声明于 `<iostream>`、`<fstream>`、`<sstream>` 等，使用 stream 单纯地进行默认输出时十分简洁，而控制格式时则要通过自身的成员函数或者 `<iomanip>` 中的流控制符（stream manipulator），显得略微繁琐

下面按照需求进行归纳

## 整数 n 进制

printf：

~~~c
// C code
int num = 10;

printf("十六进制小写：%x"
    "\n十六进制大写：%X"
    "\n八进制：%o"
    "\n十进制：%d"
    "\n", num, num, num, num);
~~~

stream：

~~~cpp
// C++ code
using std::cout;
using std::endl;
using std::dec;
using std::oct;
using std::hex;
int num = 10;

cout << "十六进制：" << hex << num << '\n'
    << "八进制：" << oct << num << '\n'
    << "十进制：" << dec << num << endl;
~~~

或者用流控制符 `std::setbase(int __base)` 设置 n 进制。注意，如果传入的不是 8、10、16，则输出**十进制**

~~~cpp
// C++ code
using std::cout;
using std::endl;
using std::setbase;
int num = 10;

cout << "十六进制：" << setbase(16) << num << '\n'
    << "八进制：" << setbase(8) << num << '\n'
    << "十进制：" << setbase(10) << num << endl;
~~~

stream 输出格式控制中有两个重要的函数

- 立 flag 控制符 `std::setiosflags(ios_base::fmtflags __mask)`
- cout 的成员函数 `std::cout.setf(ios_base::fmtflags __fmtfl)`

它们效果相同，通过传入参数设置相应的输出格式。参数是在 ios 类中定义的，我把它们叫做 ios 参数。需要传入多个参数时，用按位或运算符 `|` 连接

取消设置则使用

- `std::resetiosflags(ios_base::fmtflags __mask)`
- `std::cout.unsetf(ios_base::fmtflags __fmtfl)` 

例如，ios 参数 `ios::uppercase` 使十六进制的字母部分大写

~~~cpp
// C++ code
using std::cout;
using std::endl;
using std::hex;
using std::setiosflags;
using std::resetiosflags;
using std::ios;
int num = 10;

// 立 flag 控制符
cout << "十六进制大写：" << hex << setiosflags(ios::uppercase) << num << '\n'
    << "取消：" << resetiosflags(ios::uppercase) << num << endl;

// 成员函数
cout.setf(ios::uppercase);
cout << "十六进制大写：" << hex << num << '\n';

cout.unsetf(ios::uppercase);
cout << "取消：" << num << endl;
~~~

## 最小宽度、左右对齐、填补、正号

printf 的填补只能在右对齐的情况下用 '0' 补左边的空白

~~~c
// C code
double num = 4.0;
printf("最小宽度为11，右对齐：%11g%11g"
    "\n最小宽度为11，左对齐：%-11g%-11g"
    "\n最小宽度为11，右对齐，左边空白补0：%011g%11g"
    "\n", num, num, num, num, num, num);

// 应用：输出时间格式 12:08:03
int hour = 12, minute = 8, second = 3;
printf("最小宽度为2，右对齐，如果宽度不足2，则在左边补0："
        "%02d:%02d:%02d"
        "\n", hour, minute, second);
~~~

相比之下，stream 的填补更加灵活，可以自定义填补字符，而且 “左右对齐” 和 “填补字符” 可以自由组合

**注意**，stream 在设置最小宽度时使用的  

- 流控制符 `std::setw(int __n)`
- 成员函数 `std::cout.width(std::streamsize __wide)`

**仅对下一个输入有效**

~~~cpp
// C++ code
using std::cout;
using std::endl;
using std::setw;
using std::right;
using std::left;
using std::setfill;
using std::resetiosflags;
using std::ios;
double num = 4.0;

// 注意，对于浮点数，stream 们的默认输出相当于 printf 的 %g
cout << "最小宽度为11，右对齐：" << setw(11) << num << setw(11) << num << '\n'
    << "最小宽度为11，左对齐：" << left
                            << setw(11) << num << setw(11) << num << '\n'
                            << resetiosflags(ios::left)
    
    << "最小宽度为11，右对齐，左边空白补0：" << right
                                        << setfill('0') << setw(11) << num
                                        << setfill(' ') << setw(11) << num << '\n'
                                        << resetiosflags(ios::right)
                                        
    << "最小宽度为11，左对齐，右边空白补*：" << left
                                        << setfill('*') << setw(11) << num
                                        << setfill(' ') << setw(11) << num << endl
                                        << << resetiosflags(ios::left);

// 应用：输出时间格式 12:08:03
int hour = 12, minute = 8, second = 3;
cout << "最小宽度为2，右对齐，如果宽度不足2，则在左边补0："
    << setfill('0')
    << setw(2) << hour << ':'
    << setw(2) << minute << ':'
    << setw(2) << second << endl
    << setfill(' ');
~~~

printf 用`+`输出正号，顺序上，空格？

- 正号+
- 补零0 / 左对齐-

自由组合

~~~c
// C code
// 应用：避免输出 “+-6”
int a = 2, b = 3, c = -6;
printf("平面上某直线的一般方程：%dx%+dy%+d=0"
    "\n正号 补0 最小宽度：%+07d%0+7d"
    "\n正号 左对齐 最小宽度：%+-7d%-+7d%d"
    "\n", a, b ,c, 1, 2, 3, 4, 5);
~~~

stream 使用 ios 参数 `ios::showpos`

~~~cpp
// C++ code
// 应用：避免输出 “+-6”
using std::cout;
using std::endl;
using std::setiosflags;
using std::resetiosflags;
using std::ios;
int a = 2, b = 3, c = -6;
cout << "平面上某直线的一般方程："
    << a << 'x'
    << setiosflags(ios::showpos)
    << b << 'y' << c << "=0" << endl
    << resetiosflags(ios::showpos);
~~~

## 小数

printf 中，想指定有效数字来显示小数，就必须去掉多余的0

~~~c
// C code
double small = 0.003406000400001;
double big = 42.213;

// 7位有效数字，去掉多余的0，输出普通计数法和科学计数法中较短的
printf("7位有效数字，去掉多余的0：small = %.7g, big = %.7g", small, big);

// 普通计数法
printf("\n普通计数法，小数点后7位（包括多余的0）：small = %.7f, big = %.7f", small, big);

// 科学计数法，大写E决定指数符号大写
printf("\n科学计数法，小数点后7位（包括多余的0）：small = %.7e, big = %.7E"
        "\n", small, big);

// 动态控制最小宽度、精度
int leastWide = 10, precision = 7;
printf("动态控制最小宽度、精度：%+-*.*f"
        "\n", leastWide, precision, small);
~~~

stream：

~~~cpp
// C++ code
using std::cout;
using std::endl;
using std::setprecision;

double small = 0.003406000400001;
double big = 42.213;

// 有效数字，去掉多余的0（默认），输出普通计数法和科学计数法中较短的
cout << "7位有效数字，去掉多余的0：\n  "
    << setprecision(7)
    << "small = " << small << ", big = " << big << endl
    << setprecision(6);

// 有效数字，显示多余的0
using std::showpoint;
using std::resetiosflags;
using std::ios;
cout << "7位有效数字，显示多余的0：\n  "
    << setprecision(7) << showpoint
    << "small = " << small << ", big = " << big << endl
    << resetiosflags(ios::showpoint) << setprecision(6);

// 普通计数法
using std::fixed;
cout << "普通计数法，小数点后7位（包括多余的0）：\n  "
    << fixed << setprecision(7)
    << "small = " << small << ", big = " << big << endl
    << resetiosflags(ios::fixed) << setprecision(6);

// 科学计数法
using std::scientific;
using std::uppercase;
cout << "科学计数法，小数点后7位（包括多余的0）：\n  "
    << scientific << setprecision(7)
    << "small = " << small
    << uppercase
    << ", big = " << big << endl
    << resetiosflags(ios::scientific|ios::uppercase) << setprecision(6);
    
// 动态控制最小宽度、精度
using std::left;
using std::setw;
using std::showpos;
int leastWide = 10, precision = 7;
cout << "动态控制最小宽度、精度："
    << showpos << left << fixed
    << setprecision(precision) << setw(leastWide)
    << small << endl
    << resetiosflags(ios::showpos|ios::left|ios::fixed) << setprecision(6);

~~~
