## 问题

cmake 如何构建出一个 debug 版本的应用程序？

---

## 没有 cmake 之前的做法

在没有 cmake 之前我们可以通过在编译的时候加上 `-g` 选项来让编译器生成一个 debug 版本的应用程序，方便我们用 gdb 调试。假设我们有一个 1 加到 3 的小程序。
```c++
// src/main.cpp

#include<iostream>
using namespace std;

int main()
{
    int sum = 0;
    for (int i = 1;i<=3;i++)
    {
        sum = sum + i;
    }

    cout<<"sum = "<< sum << " .";
    cout<<endl;
    return 0;
}
```
编译一个 debug 版本
```bash
# 编译
g++ -o main-debug -g main.cpp

-rw-rw-r--. 1 jianglexing jianglexing   199 1月  31 14:59 main.cpp
-rwxrwxr-x. 1 jianglexing jianglexing 34504 1月  31 15:02 main-debug

```
如果要一步步的调试运行就交给 gdb 吧。
```bash
# gdb 调试运行
gdb -s main-debug 
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from main-debug...done.
```

![cmake](static/2021-01/camke.jpg)

---

## 用 cmake 怎么做
用 cmake 也差不多，只是把 `-DCMAKE_BUILD_TYPE` 设置为 `Debug` 就行了。
```bash
mkdir build
cd build

cmake .. -DCMAKE_BUILD_TYPE=Debug
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
Scanning dependencies of target main-debug
[ 50%] Building CXX object CMakeFiles/main-debug.dir/src/main.cpp.o
[100%] Linking CXX executable main-debug
```

完整的 cmake 配置文件如下。
```cmake
cmake_minimum_required(VERSION 3.0)
project(helloworld VERSION 1.0)
add_executable(main-debug src/main.cpp)
```

cmake 是内部是怎么做到的呢？从 cmake 的缓存文件中我们可以看到这样一行，可以想到 cmake 内部帮我们把 -g 传递给了 g++ 。

```cmake
//Flags used by the CXX compiler during DEBUG builds.
CMAKE_CXX_FLAGS_DEBUG:STRING=-g
```

---