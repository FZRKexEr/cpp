
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

### 1.6 静态变量 指针 引用

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

### 1.7 左值 右值 左值引用 右值引用 

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
3. 右值引用 `int &&c = a + 1;`

### 1.8 move 函数

`int&& rrefi = std::move(i);`

把 i 从左值变成右值，失去地址属性，不能再被使用。

所有临时对象都是右值。

### 1.9 可调用对象

如果一个对象使用调用运算符`()`, 并且里面可以放参数，这个对象就是可调用对象。

可调用对象最重要的用法是给另一个函数作为参数。

可调用对象分类：

1. 函数
    ```cpp
    #include <iostream>

    int test(int i) {
        std::cout << i << std::endl;
        return i;
    }

    using pf_type = int(*)(int);

    void my_func(pf_type pf, int i) {
        pf(i);
    }
    void my_func2(int(*pf)(int), int i) {
        pf(i);
    }

    int main() {
        my_func(test, 100);
        my_func2(test, 100);
        return 0;
    }

    ```
2. 仿函数 functor
    ```cpp
    class Test {
    public:
        void operator()(int i) {
            std::cout << i << std::endl;
            std::cout << "operator()(int i)" << std::endl;
        }
    };

    int main() {
        Test t;
        t(100);
        return 0;
    }
    ```
3. lambda表达式

    格式：最少 `[]{}`, 完整格式 `[]()->ret{}` 

    ```cpp
    int main() {
        []{
            std::cout << "hello" << std::endl;
        }(); // 后面这个括号不是定义中的 (), 而是表示调用这个匿名函数
        return 0;
    }
    ```
    
    1. `[]` 捕获列表
       - `[]` 不捕获
       - `[=]` 按值捕获
       - `[&]` 按引用捕获
       - `[&, i]` i按值捕获, 其他变量按引用捕获
       - `[=, &i]` i按引用捕获，其他变量按值捕获
       - `[i]` 单独捕获 i 的值
       - `[&i]` 单独捕获 i 的引用   
    2. `()` 表达式的参数

        ```cpp
        int main() {
            [](int num){
                std::cout << num << std::endl;
            }(100); // 输出 100
            return 0;
        }
        ```
    lambda表达式最常用的用法是给普通函数做参数.  

    ```cpp
    void my_fun(int(*pf)(int), int num) {
        pf(num);
    }

    int main() {
        my_fun([](int num)->int {
            std::cout << num << std::endl;
            return num;
        }, 100);
        return 0;
    }
    ```
    注意，如果lambda表达式传递给函数指针，不能有任何捕获, 这是函数指针的固有缺陷。使用C++11 的 function 可以解决这个问题。 
    ```cpp
    #include <iostream>
    #include <functional>

    using func_type = std::function<int(int)>; // C++11, 和函数指针唯一的区别是这个可以捕获

    void my_fun(func_type func, int num) {
        func(num);
    }

    int main() {
        int a = 101;
        my_fun([a](int num)->int {
            std::cout << num << std::endl;
            std::cout << a << std::endl;
            std::cout << "lambda" << std::endl;
            return num;
        }, 100);
        return 0;
    }
    ```

## 2 类
### 2.1 构造函数，析构函数

面向对象：按人类思维编码, 类有自己的函数，不需要类外的函数来操作类。

面向过程：按机器运行编码, C语言 struct 不能有函数，C++ class, struct 可以。面向过程编程有从天而降的函数，来完成各种事情。比如在结构体外定义函数操作结构体。c语言开发的大型项目，也模拟了面向对象的过程。

构造函数类型：

1. 普通构造函数 
2. 复制构造函数, 用另一个对象来初始化对应的内存
    ```cpp
    class Test {
    public:
        Test(const Test& other) : a(other.a), b(other.b) {}
        int a;
        int b;
    };
    ```
3. 移动构造函数
4. 默认构造函数 `Test() {}`

析构函数，类删除的时候调用析构函数，一般什么都不需要干 `~Test() {}`

```cpp
class Test {
public:
    Test(int a_, int b_) : a(a_), b(b_), p(new int(8)) {}
    ~Test() { // 析构函数, 释放指针, 避免内存泄露
        delete p;
    }
    int a;
    int b;
    int* p;
};
```

几乎所有类都需要构造函数，析构函数未必要

### 2.2 this 关键字，常成员函数，常对象

const指针:
```cpp
const int* p; // 指向的类型是 const int, 指针可以随意修改指向的对象
int const * p; // 指向的类型是 const int, 指针可以随意修改指向的对象
int * const p; // 指向的类型是 int, 指针不能随意修改指向的对象
const int* const p; // 指向const int, 且指针不能修改
``` 

常成员函数：不能修改成员变量，可以理解为把this指针指向对象用const修饰的函数。

```cpp
void output() const { // const 表示不希望 output() 函数修改类成员变量
    //...
}
```

注意：
1. 常成员函数不能调用普通函数
2. 常对象不能调用普通函数
3. 能加const就加!!! 少出很多bug
4. 常对象 优先调用常成员函数，普通对象优先调用普通成员函数，同名常成员函数和普通成员函数可以重载。(没啥用, 一般不写两个)

### 2.3 inline, mutable, default, delete 关键字

inline 建议编译器将函数放在需要调用的地方，不进行压栈操作，提高效率。在类中直接实现函数，默认会加inline关键字。

mutable 关键字让变量永远可以被修改，即使处于常函数中。mutable 是一种万不得已的写法。

default 关键字，使用系统默认的函数。推荐写出来更直观。

```cpp
class Test {
public:
    Test() = default;
    Test(const Test& test) = default;
    ~Test() = default;
    Test& operator = (const Test& test) = default;
private:
    int a;
};
```

delete 关键字:

```cpp
class Test {
public:
    Test() = delete; // 指定系统不默认生成普通构造函数
    Test(const Test& test) = delete; // 不默认生成复制构造函数
    Test& operator = (const Test& test) = delete; 
    ~Test() = delete; // 析构函数一般不会这样做, 但是可以。
private:
    int a;
};
```
### 2.4 friend 关键字

friend 让类或者函数能访问私有成员。会破坏封装性，尽量不要用，只在必须使用友元的算符重载时使用。

### 2.5 重载运算符

to do 


### 2.6 继承

```cpp
class Spear {
public:
    Spear(std::string a, std::string b) : name(a), icon(b) {}
protected: // 外界无法访问，public 继承后可以访问
    std::string name;
    std::string icon;
};

class IceSpear : public Spear { // 只有 public 继承 有用
public:
    IceSpear(std::string a, std::string b, std::string c) : Spear(a, b), ice(c) {}
    void Output() {
        std::cout << name << std::endl;
    }
private:
    std::string ice;
};
```

```cpp
Spear* pSpear = new IceSpear(); // 也是可以的，因为子类先调用父类的构造函数。父类构造完毕已经匹配了。
```

父类的构造函数和析构函数都在子类之前运行。

### 2.7 虚函数

父类的函数指针可以指向子类对象。

多态：父类的函数指针可以调用子类的成员函数。例如，对于开火这个动作，只需要写让父类开火，就会自动调用子类的开火动作。

虚函数: 加上 virtual 关键字的函数, 会优先调用子类的同名函数

注意：子类虚函数定义需要和父类完全一样, 否则子类虚函数会被当成新的成员函数，没有报错信息。

C++ 11 override 能强制让子类父类虚函数必须保持一致，如果不一致会编译失败。

```cpp
class Spear {
public:
    Spear(std::string a, std::string b) : name(a), icon(b) {}
    virtual void OpenFire() { // 使用 virtual 关键字会优先调用子类的 OpenFire
        std::cout << "Spear::OpenFire()" << std::endl;
    }
protected: // 外界无法访问，public 继承后可以访问
    std::string name;
    std::string icon;
};

class IceSpear : public Spear { // 只有 public 继承 有用
public:
    IceSpear(std::string a, std::string b, std::string c) : Spear(a, b), ice(c) {}
    void Output() {
        std::cout << name << std::endl;
    }
    virtual void OpenFire() override { // 如果父类 OpenFire 已经加上 virtual 了，子类会自动加上，但这里还是写上 virtual
        std::cout << "IceSpear::OpenFire()" << std::endl;
    }
private:
    std::string ice;
};

int main() {
    Spear* pSpear = new IceSpear("a", "b", "c");
    pSpear -> OpenFire();
    delete pSpear; // 有 new 一定要delete !
    return 0;
}
```

析构函数必须是虚函数，避免内存泄漏。

静态绑定：编译时，就知道函数的地址。非虚函数就是静态绑定。

动态绑定：程序运行时才知道函数的地址,编译时只知道找到函数地址的方法。虚函数是动态绑定。

### 2.8 静态成员变量 静态成员函数

静态成员变量只能在类外初始化。因为构造函数在运行时才会执行，而静态变量需要在编译时就分配地址。

静态成员函数可以用类名直接访问，不需要对象 (也可以用对象)。

静态成员函数。可以返回静态成员变量，可以用类名调用，不需要对象。

```cpp
class Test {
public:
    static int GetINF() {
        return INF;
    }
    int func() {
        return 1;
    }
    static int func2() {
        return 1;
    }
    int value = 100;
private:
    static const int INF;
};

const int Test::INF = 1000000000;

int main() {
    std::cout << Test::GetINF() << std::endl; // ok
    std::cout << Test::value << std::endl; // 对非静态成员变量的错误用法
    std::cout << Test::func() << std::endl; // 没有对象不能调用
    std::cout << Test::func2() << std::endl; // ok
    return 0;
}
```
### 2.9 纯虚函数

基类的纯虚函数可以被忽略掉，有纯虚函数的基类不生成对象。

```cpp
class Spear {
public:
    Spear(std::string a, std::string b) : name(a), icon(b) {}
    virtual void OpenFire() const = 0; // 纯虚函数 不需要实现 , 纯虚函数： = 0
protected: // 外界无法访问，public 继承后可以访问
    std::string name;
    std::string icon;
};

class IceSpear : public Spear { // 只有 public 继承 有用
public:
    IceSpear(std::string a, std::string b, std::string c) : Spear(a, b), ice(c) {}
    void Output() {
        std::cout << name << std::endl;
    }
    virtual void OpenFire() const override { // virtual 可以只在父类定义，override 只在子类定义
        std::cout << "IceSpear::OpenFire()" << std::endl;
    }
private:
    std::string ice;
};

int main() {
    Spear* pSpear = new IceSpear("a", "b", "c"); // 注意不能 new Spear, 因为 Spear 是有纯虚函数的基类
    pSpear -> OpenFire();
    delete pSpear; // 有 new 一定要delete !
    return 0;
}
```
### 2.10 RTTI (Run Time Type Identification)

检查基类指针指向的实际的派生类

RTTI 是C++判断指针或者引用实际类型的唯一方式。

typeid用法, 使用 typeid 父类和子类必须要有虚函数(父类有了，子类自动会有)

```cpp
int main() {
    Spear* pSpear = new IceSpear("a", "b", "c");
    pSpear -> OpenFire();

    std::cout << typeid(*pSpear).name() << std::endl; // typeid(*Pointer).name() 返回指针对象的真实名称，这里是 IceSpear

    delete pSpear; // 有 new 一定要delete !
    return 0;
}
```

dynamic_cast 用法, 把基类指针转换成子类指针

```cpp
int main() {
    Spear* pSpear = new IceSpear("a", "b", "c");
    pSpear -> OpenFire();

    std::cout << typeid(*pSpear).name() << std::endl; // typeid(*Pointer).name() 返回指针对象的真实名称，这里是 IceSpear

    IceSpear* pIceSpear = dynamic_cast<IceSpear*>(pSpear);  // 把基类指针转换成子类指针
    pIceSpear -> IamIceSpear();
    std::cout << pIceSpear << " " << pSpear; // 这两个指针地址相同, 只delete 一次
    delete pSpear; // 有 new 一定要delete !
    return 0;
}
```

重要用法，判断类型

```cpp
int main() {
    Spear* pSpear = new IceSpear("a", "b", "c");

    std::cout << typeid(*pSpear).name() << std::endl; // typeid(*Pointer).name() 返回指针对象的真实名称，这里是 IceSpear
    std::cout << typeid(IceSpear).name() << std::endl;

    if (typeid(*pSpear) == typeid(IceSpear)) {
        IceSpear* pIceSpear = dynamic_cast<IceSpear*>(pSpear);
        if (pIceSpear) { // 不是 none 就转换成功
            std::cout << "cast IceSpear success" << std::endl;
        }
    }

    delete pSpear; // 有 new 一定要delete !
    return 0;
}
```
### 2.11 多继承


子类继承多个基类

除了借口模式，不建议使用。

### 2.12 虚继承

### 2.13 移动构造函数 移动复制运算符

`&&` 右值引用

```cpp
class MemoryBlock
{
public:

    // Simple constructor that initializes the resource.
    explicit MemoryBlock(size_t length)
        : _length(length)
        , _data(new int[length])
    {
        std::cout << "In MemoryBlock(size_t). length = "
                  << _length << "." << std::endl;
    }

    // Destructor.
    ~MemoryBlock()
    {
        std::cout << "In ~MemoryBlock(). length = "
                  << _length << ".";

        if (_data != nullptr)
        {
            std::cout << " Deleting resource.";
            // Delete the resource.
            delete[] _data;
        }

        std::cout << std::endl;
    }

    // Copy constructor.
    MemoryBlock(const MemoryBlock& other)
        : _length(other._length)
        , _data(new int[other._length])
    {
        std::cout << "In MemoryBlock(const MemoryBlock&). length = "
                  << other._length << ". Copying resource." << std::endl;

        std::copy(other._data, other._data + _length, _data);
    }

    // Copy assignment operator.
    MemoryBlock& operator=(const MemoryBlock& other)
    {
        std::cout << "In operator=(const MemoryBlock&). length = "
                  << other._length << ". Copying resource." << std::endl;

        if (this != &other)
        {
            // Free the existing resource.
            delete[] _data;

            _length = other._length;
            _data = new int[_length];
            std::copy(other._data, other._data + _length, _data);
        }
        return *this;
    }

    // Move constructor.
    MemoryBlock(MemoryBlock&& other) noexcept
        : _data(nullptr)
        , _length(0)
    {
        std::cout << "In MemoryBlock(MemoryBlock&&). length = "
                  << other._length << ". Moving resource." << std::endl;

        // Copy the data pointer and its length from the
        // source object.
        _data = other._data;
        _length = other._length;

        // Release the data pointer from the source object so that
        // the destructor does not free the memory multiple times.
        other._data = nullptr;
        other._length = 0;
    }

    // Move assignment operator.
    MemoryBlock& operator=(MemoryBlock&& other) noexcept
    {
        std::cout << "In operator=(MemoryBlock&&). length = "
                  << other._length << "." << std::endl;

        if (this != &other)
        {
            // Free the existing resource.
            delete[] _data;

            // Copy the data pointer and its length from the
            // source object.
            _data = other._data;
            _length = other._length;

            // Release the data pointer from the source object so that
            // the destructor does not free the memory multiple times.
            other._data = nullptr;
            other._length = 0;
        }
        return *this;
    }

    // Retrieves the length of the data resource.
    size_t Length() const
    {
        return _length;
    }

private:
    size_t _length; // The length of the resource.
    int* _data; // The resource.
};


using namespace std;

int main()
{
    // Create a vector object and add a few elements to it.
    vector<MemoryBlock> v;
    v.push_back(MemoryBlock(25));
    v.push_back(MemoryBlock(75));

    // Insert a new element into the second position of the vector.
    v.insert(v.begin() + 1, MemoryBlock(50));
}
```


## 8. 多线程

并发: 每个cpu执行一个线程，不抢占资源。

并行: 用一个处理器执行多个线程，不同时间轮换执行  

在C++11中，进程是容器，线程是最小执行单元。

一个程序是一个进程，main 函数是主线程, 一个线程执行时可以创建另一个线程（子线程），主线程和子线程是平级的, 主线程结束会强制结束所有子线程。

### 8.1 



