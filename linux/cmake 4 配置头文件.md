## 背景
按 c++ 的风格是把代码的声明和实现分开，声明保留在 .h 文件中，实现在 .cpp 文件中。后面的内容会用一个简单的例子来介绍一下用 cmake 怎么配置。

![sqlpy](static/2020-41/cmake-04.jpg)

---


## 项目的目录结构
整个项目的目录结构如下
```bash
tree headers/

headers/
├── CMakeLists.txt
├── include
│   └── hello.h
├── run.sh
└── src
    ├── hello.cpp
    └── main.cpp

2 directories, 5 files
```
其中 `include/hello.h` 是声明，`src/hello.cpp` 是实现，`src/main.cpp` 是主程序的所在文件。


--- 


## hello.h
```cpp
#ifndef _HELLO_H__

#define _HELLO_H__
class Hello
{
public:
    void hello();
};

#endif
```

---

## hello.cpp
```cpp
src/hello.cpp                                                        
#include<iostream>
#include "hello.h"

void Hello::hello()
{
    using namespace std;
    cout<<"this is Hello.hello function."<<endl;
}
```

---

## main.cpp
```cpp
#include<iostream>
#include "hello.h"

int main()
{
   Hello h;
   h.hello();

}
```

---

## CMakeLists.txt
编写 cmake 构建文件
```cmake
project(headers)

set(SOURCES
    src/hello.cpp
    src/main.cpp)

add_executable(headers 
    ${SOURCES})

target_include_directories(headers
    PUBLIC
    ${PROJECT_SOURCE_DIR}/include)

install(TARGETS
    headers
    DESTINATION
    ${PROJECT_SOURCE_DIR}/bin)
```

---
## 编译安装
编译安装并运行我们的 headers 程序
```bash
mkdir build -p && cd build && cmake ../ && make && make install && cd ../bin && echo '' && ./headers

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
-- Configuring done
-- Generating done
-- Build files have been written to: /root/headers/build
Scanning dependencies of target headers
[ 33%] Building CXX object CMakeFiles/headers.dir/src/hello.cpp.o
[ 66%] Building CXX object CMakeFiles/headers.dir/src/main.cpp.o
[100%] Linking CXX executable headers
[100%] Built target headers
[100%] Built target headers
Install the project...
-- Install configuration: ""
-- Installing: /root/headers/bin/headers

this is Hello.hello function.
```

---


