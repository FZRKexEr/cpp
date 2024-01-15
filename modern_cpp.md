
# Modern Cpp

## 0. C++ 开发环境

### 0.1 编译过程

1. 第一步：从 .cpp 到 .o 文件
    1. 预处理：把 include 替换成 头文件，对所有宏处理。
    2. 编译：把 .cpp 变成 .s 汇编文件，最耗时的阶段。
    3. 汇编：把 .s 变成 .o 二进制文件。

2. 第二步：把多个 .o 文件合成 一个可执行文件

```sh
g++ -c filename1.cpp filename2.cpp filename3.cpp
g++ filename1.o filename2.o filename3.o -o executablefile
```

### 0.2 静态库和动态库

库：源代码的实现部分（cpp文件）被编译成的二进制文件是库。源码只提供头文件（调用接口）。这样别人就不知道源代码是怎么实现的了。



## 1. 基本特性

### 1.1 程序执行过程

栈内存和堆内存

静态变量

### 1.2 new 

new 是动态分配内存的主要方式，对类对象，需要有默认构造函数。

用 delete 回收内存。

```cpp
#include <iostream>

int main() {
    
    int *p = new int[100]();

    std::cout << p[20] << std::endl;

    delete[] p;
    return 0;
}
```

