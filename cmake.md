# CMake 学习笔记

CMake 版本应该比编译器更新，CMake 尽量保持最新。

## 构建项目

经典构建流程

```sh
mkdir build
cd build
cmake ..
make
```

新版构建流程，除非检查对老版本 CMake 兼容性，应该使用新版。

```sh
cmake -S . -B build
cmake --build build
```

