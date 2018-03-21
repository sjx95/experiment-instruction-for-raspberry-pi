# Hello RPi

由于 Raspbian 是一个基于 Debian 定制的通用 Linux 系统，因此可以用任何通用的语言来编写代码，包括但不限于：

* C/C++
* Python2/Python3
* Java
* GoLang
* Shell Script

这里用几个非常简单的例子介绍一下最基本的编译方法。

## C/C++

C/C++ 是树莓派中常用语言之一，由于 Linux 及其 GNU/Linux 中的大量组件都是由 C/C++ 编写并导出了链接库，因此 C/C++ 非常适合用作树莓派的开发。此外， C/C++ 执行效率很高，能够充分利用硬件性能。

首先检查编译器是否存在及版本，使用命令 `g++ --version`，若返回信息如下，则说明正在使用 6.3 版本的 gcc。

```
pi@raspberrypi:~ $ g++ --version
g++ (Raspbian 6.3.0-18+rpi1+deb9u1) 6.3.0 20170516
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```



使用前文所述的 NANO 编辑器新建一个文件并写入如下内容：

```cpp
// Hello, RPi.
#include <iostream>
using namespace std;

int main() {
    cout << "Hello, RPi." << endl;
    return 0;
}
```



然后使用命令 `g++ hello.cpp -o hello`进行编译，注意 Linux 系统中不要求程序具有扩展名。如果没有输出任何内容，那么说明 "0 Error, 0 Warning"，编译成功。接下来通过 `./hello`运行编译好的代码，如果没有意外的话：

```
pi@raspberrypi:~ $ ./hello 
Hello, RPi.
```

## Python3

Python 也是树莓派中的一种常用语言，尽管具有弱类型、低效率的缺点，但由于它的库很丰富，还可以无缝集成 C/C++ 编写的代码，因此也得到了广泛的应用。

注意：无缝集成意味着可以先用 Python 进行开发，然后进行性能分析，对于关键部分用 C/C++ 重写。



