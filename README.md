# cpp
C++ 学习笔记

参考资料:

[现代 C++ 教程：高速上手 C++ 11/14/17/20](https://cntransgroup.github.io/EffectiveModernCppChinese/)

[C++ 风格指南](https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/)

[Effective Modern C++](https://cntransgroup.github.io/EffectiveModernCppChinese/)

[Modern CMake](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/)

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

