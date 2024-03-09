
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

### 1.3 命名空间 

using 关键字简化命名空间。

头文件中不能有 using namespace 

### 1.4 输入输出

cout, cin

### 1.45 const 关键字

const 不是常量，变量依然在栈区。

### 1.5 auto 

使用 auto 需要知道 auto 推断出来是什么，避免意想不到的 bug.

1. auto 会忽略值类型的 const, 保留 修饰指向对象的 const.

```cpp
int main() {
    const int i = 100;
    auto i2 = i; // i2 类型 int 
    std::cout << i2 << std::endl;
    i2++;
    std::cout << i2 << std::endl;

    int j;
    const int* const pj = &j; 
    auto j2 = pi; // j2 的类型 const int* 
    return 0;
}

```

2. auto 只能推断类型，不能推断引用，要使用引用必须自己加上 & 

   boost 库 可以查看 auto 的类型:

```cpp
#include <boost/type_index.hpp>
#include <iostream>
int main() {
    const int i = 100;
    auto i2 = i;
    std::cout << type_id_with_cvr<decltype(i2)>().pretty_name() << std::endl;
    return 0;
}
```

3. auto 推断引用的时候，会直接把引用替换成引用的对象。

```cpp
int i = 100;
const int& refi = i;
auto i2 = refi; // 等价于 auto i2 = i;
auto& i3 = refi; // 等价于 auto& i3 = i;
```

4. auto 有 & 时，所有 const 都会得到保留

## 1.6 静态变量 指针 引用

变量的存储位置：静态变量，栈区，堆区。

静态编译区在编译时就已经确定好地址。

```cpp
#include <iostream>

unsigned test() {
    static unsigned cnt = 0; // 这一行代码在编译时确定 cnt = 0, 然后就被忽略了。
    return ++cnt;
}

int main() {
    test();
    test();
    unsigned cnt = test();
    std::cout << cnt << std::endl; // 输出3
    return 0;
}
```

## 1.7 左值 右值 左值引用 右值引用 

左值：有地址属性的对象, 可以放在等号左边(也可以放在右边)

右值：不是左值的对象是右值。不能操作地址的对象。永远只能在等号右边。

临时对象是右值。

```cpp
int a = 10; // a 左值，10右值
int b = (a + 1); // b 左值, a + 1 右值
// 注意 (a + 1) 有地址，但是不能修改，没有地址属性.
++a = 10; // ++a 是左值
b = a++; // a++ 是右值
```
1. 普通左值引用,eg `int &i` 就是i的别名, 无法绑定常量对象, 只能绑定左值。
2. const 左值引用可以绑定左值和右值, 对常量对象起别名。
3. 


## 8. 多线程

并发: 每个cpu执行一个线程，不抢占资源。

并行: 用一个处理器执行多个线程，不同时间轮换执行  

在C++11中，进程是容器，线程是最小执行单元。

一个程序是一个进程，main 函数是主线程, 一个线程执行时可以创建另一个线程（子线程），主线程和子线程是平级的, 主线程结束会强制结束所有子线程。

### 8.1 



