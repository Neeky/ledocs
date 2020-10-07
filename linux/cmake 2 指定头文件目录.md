## 背景
之前我们用 g++ 编译项目的时候直接通过 -I 选项来指定头文件的目录，当我们用 cmake 的时候应该怎样才通把头文件目录的信息传递到 g++ 呢？

![sqlpy](static/2020-41/cmake-02.jpg)


---

## 实验项目
头文件 `headers/Person00.h` 包含了 Person 类的声明，`src/Person00.cpp` 文件中对其进行了实现，`src/main.cpp` 是真正的使用方。
```bash
tree .
.
├── CMakeLists.txt
├── headers
│   └── Person00.h
└── src
    ├── Person00.cpp
    └── main.cpp

2 directories, 4 files
```
headers/Person00.h
```c++
#ifndef __Person_H__

#define __Person_H__

class Person
{
private:
    std::string m_name;

public:
    void say_hello();
    Person(std::string name = "");
};

#endif
```
src/Person00.cpp
```c++
#include <iostream>

#include "Person00.h"

void Person::say_hello()
{
    using namespace std;
    cout << "hello my name is " << m_name << endl;
}

Person::Person(std::string name)
{
    m_name = name;
}
```
src/main.cpp
```c++
#include <iostream>
#include "Person00.h"

int main()
{
    //using namespace std;
    Person p("tom");
    p.say_hello();
}
```

---

## 直接使用 g++ 编译
对于这样的一个小项目，直接使用 g++ 编译也非常方便。
```bash
# 编译
g++ -Iheaders src/main.cpp src/Person00.cpp -o nee-headers

# 执行
./nee-headers 
hello my name is tom
```

---

## cmake 编译
cmake 文件的全部内容如下。
```cmake
# 指定 camke 的最小版本
cmake_minimum_required(VERSION 3.0)

# 指定项目名
project(nee-headers)

# 指定项目中的源文件列表
set(files 
    src/main.cpp
    src/Person00.cpp)

# 指定生成的执行程序的名字
add_executable(nee-headers ${files})

# 指定头文件的搜索目录为项目目录下的 headers
target_include_directories(nee-headers
PRIVATE
    ${PROJECT_SOURCE_DIR}/headers)

```
使用 cmake 编译
```bash
# 创建一个专门用来编译的目录
mkdir build
cd build

# 用 cmake 生成 Makefile
cmake ../
-- The C compiler identification is AppleClang 11.0.0.11000033
-- The CXX compiler identification is AppleClang 11.0.0.11000033
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /Users/jianglexing/cpps/learning002/build

# 编译
make
Scanning dependencies of target nee-headers
[ 33%] Building CXX object CMakeFiles/nee-headers.dir/src/main.cpp.o
[ 66%] Building CXX object CMakeFiles/nee-headers.dir/src/Person00.cpp.o
[100%] Linking CXX executable nee-headers
[100%] Built target nee-headers

# 整个过程会生成如下文件
ll -h
total 120
-rw-r--r--   1 jianglexing  staff    13K 10  7 22:53 CMakeCache.txt
drwxr-xr-x  15 jianglexing  staff   480B 10  7 22:54 CMakeFiles
-rw-r--r--   1 jianglexing  staff   5.7K 10  7 22:53 Makefile
-rw-r--r--   1 jianglexing  staff   1.4K 10  7 22:53 cmake_install.cmake
-rwxr-xr-x   1 jianglexing  staff    29K 10  7 22:54 nee-headers

# 执行程序
./nee-headers 
hello my name is tom
```

---
