## 概要
一直也没有用 C++ 写过什么大项目，顶多不超过 3 个文件，最近想让自己的代码工程化一点，于是整起了 cmake 。现在鸡也要用一下牛刀，我要给自己的 hello-world 程序也配上 cmake 。

![sqlpy](static/2020-37/cmake-001.jpg)

---

## 第一步 创建源文件
整个项目的结构如下。
```bash
# 创建一个 cpps 的目录，用于保存我们的文件
mkdir cpps
cd cpps

# 创建程序源文件 main.cpp
touch main.cpp

```

main.cpp 内容如下。
```C++                                                      
#include<iostream>

int main()
{
    using namespace std;
    cout<<"hello cmake."<<endl;
}
```

---


## 第二步 添加 CMakeLists.txt 文件
添加 CMakeLists.txt 文件，其内容如下。
```cmake                                                        
cmake_minimum_required(VERSION 3.7.1) 

project(hello-world)

set(SOURCE_FILES main.cpp)

message(STATUS "This is BINARY dir " ${PROJECT_BINARY_DIR})
message(STATUS "This is SOURCE dir " ${PROJECT_SOURCE_DIR})

add_executable(hello-world ${SOURCE_FILES})
```
---

## 第三步 添加专门用于编译的目录
为了不要“污染”项目目录，cmake 推荐在一个叫 `build` 的子目录下完成编译相关的工作。
```bash

# 创建 build 目录
mkdir build
cd build/
```

---

## 第四步 编译
```bash
# 根据父目录中定义的 CMakeLists.txt 文件的定义生成  MakeFile
cmake ..                                                                   
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- This is BINARY dir /tmp/cpps/build   # 编译完成之后的二进制文件会被保存到这目录
-- This is SOURCE dir /tmp/cpps         # 源文件的目录
-- Configuring done
-- Generating done
-- Build files have been written to: /tmp/cpps/build

## 编译
cmake --build .                                                            
Scanning dependencies of target hello-world
[ 50%] Building CXX object CMakeFiles/hello-world.dir/main.cpp.o
[100%] Linking CXX executable hello-world
[100%] Built target hello-world
```
---

## 第四步 执行程序
编译完成之后我们就可以执行试一下了。
```bash
./hello-world 
hello cmake.
```

---


## cmake 语法解读
声明最低的 cmake 版本号，如果系统上的 cmake 程序小于这个版本号就会报错。
```cmake
cmake_minimum_required(VERSION 3.7.1) 
```
给项目指定起一个名字
```cmake
project(hello-world)
```
指定要编译哪些文件
```cmake
set(SOURCE_FILES main.cpp)
```
打印给定的信息，告诉编译程序的那个人，编译完成之后可执行程序保存在了哪里
```cmake
message(STATUS "This is BINARY dir " ${PROJECT_BINARY_DIR})
message(STATUS "This is SOURCE dir " ${PROJECT_SOURCE_DIR})
```
给可执行文件起名字
```cmake
add_executable(hello-exec ${SOURCE_FILES})
```
---