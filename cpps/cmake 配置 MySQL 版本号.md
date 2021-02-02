## 背景

MySQL 源码包的“根目录”下保存着一个叫`MYSQL_VERSION` 的文本文件，它用来指定 MySQL 的版本号，以 MySQL-8.0.23 为例，其内容如下。
```python
MYSQL_VERSION_MAJOR=8
MYSQL_VERSION_MINOR=0
MYSQL_VERSION_PATCH=22
MYSQL_VERSION_EXTRA=
```
MySQL 是直接读这个文件的内容来指定版本号嘛？虽然最终的效果和直接读文件差不多，不过事实上是在 cmake 来帮助下完成地。下面我们我们来看它是怎么一步步进化成这样的。

![cmake-version](static/2021-01/cmake-version.jpg)

---

## 之前的实现方法
之前的做法比较简单直接，就是把版本号声明成变量(通常是在头文件中声明)，下面以 hello-world 项目为例子。
```bash
tree .
.
├── hello-world.cpp # 源文件
└── hello-world.h   # 头文件
```
`hello-world.h` 文件的内容如下。
```c++
#ifndef hello_world_h_

#define HELLO_WORLD_VERSION_MAJOR 1
#define HELLO_WORLD_VERSION_MINOR 0

#endif
```
`hello-world.cpp` 文件的内容如下。
```c++
#include<iostream>
#include "hello-world.h"

using namespace std;
int main()
{
    // 直接打印版本号别的什么都不做
    cout<<HELLO_WORLD_VERSION_MAJOR<<"."<<HELLO_WORLD_VERSION_MINOR<<endl;
    return 0;
}
```

---

## 存在的问题
我觉得最大的问题就是在于源代码文件无法封闭，也就是说改一个版本号我要去改源代码；后面大家的想法是头文件中的内容根据配置动态生成。下面我用 cmake 来演示一下。

---

## cmake 的实现方法

cmake 用来配置这个也比较方便，主要是因为 cmake 配置文件本身就有指定软件的版本号，像下面这个配置文件把软件的版本号配置成了 1.0 。
```cmake
cmake_minimum_required(VERSION 3.0)
project(HELLO_WORLD VERSION 1.0)
```
---

那么剩下的就是告诉 cmake 把版本号“填”入哪个头文件模板，这个也只是一行 cmake 指令的事。
```cmake
cmake_minimum_required(VERSION 3.0)
project(HELLO_WORLD VERSION 1.0)

configure_file(hello-world.h.in src/hello-world.h) # 用项目目录下的 hello-world.h.in 文件作为模板生成  src/hello-world.h 这个头文件。
```
---

`hello-world.h.in` 模板文件的内容如下。
```c++
#ifndef HELLO_WORLD_H_

#define HELLO_WORLD_VERSION_MAJOR @HELLO_WORLD_VERSION_MAJOR@
#define HELLO_WORLD_VERSION_MINOR @HELLO_WORLD_VERSION_MINOR@

#endif
```
---

完整的 cmake 配置。
```cmake
cmake_minimum_required(VERSION 3.0)
project(HELLO_WORLD VERSION 1.0)

configure_file(hello-world.h.in src/hello-world.h)

add_executable(hello-world-cmake src/hello-world.cpp)

target_include_directories(hello-world-cmake PUBLIC
                           "${PROJECT_BINARY_DIR}/src"
                           )
```
项目的文件分布如下。
```tree
tree .
.
├── CMakeLists.txt # 
├── hello-world.h.in
└── src
    └── hello-world.cpp

1 directory, 3 files
```
---

编译
```bash
mkdir build
cd build/

cmake ../
-- The C compiler identification is GNU 8.3.1
-- The CXX compiler identification is GNU 8.3.1
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
-- Configuring done
-- Generating done
-- Build files have been written to: /opensource/cpps/build

cmake --build .
Scanning dependencies of target hello-world-cmake
[ 50%] Building CXX object CMakeFiles/hello-world-cmake.dir/src/hello-world.cpp.o
[100%] Linking CXX executable hello-world-cmake
[100%] Built target hello-world-cmake
```
检查
```bash
./hello-world-cmake
1.0
```

---

## 注意
MySQL 并没有像我这样把版本号直接写到 CMakeLists.txt ，它把 CMakeLists.txt 给封闭了，把版本号的放到了 `MYSQL_VERSION` 这个文件中去。
它在 CMakeLists.txt 中加入了这么两行，总的效果是一样的。
```cmake
INCLUDE(mysql_version)

CONFIGURE_FILE(${CMAKE_SOURCE_DIR}/include/mysql_version.h.in
               ${CMAKE_BINARY_DIR}/include/mysql_version.h )
```

---
