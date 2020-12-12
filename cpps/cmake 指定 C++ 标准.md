## 背景
在用 cmake 编译 c++ 项目的时候出错了，报错信息如下。
```bash
/Users/jianglexing/repos/cpp-students/cpp_primer_plus/chapter4/src/4_9.cpp:8:5: warning: 'auto' type specifier is a C++11 extension [-Wc++11-extensions]
    auto us = "这是一个测试字符串.";
    ^
1 warning generated.
```
CMakeLists.txt 文件的内容如下。
```cmake
project(neeky)
add_executable(neeky src/4_9.cpp)
```
![npm-schema](static/2020-46/c++11.jpg)

---

## 分析
从报错来看提示已经非常明显了，auto 是 C++ 11 里的内容，如果我们要使用 auto 的话，要告诉 cmake 要使用 c++ 11 标准。这要求我们在 cmake 文件中加一行。
```cmake
project(neeky)
add_definitions(-std=c++11)
add_executable(neeky src/4_9.cpp)
```
---

---

## 验证
编译一下以检查是否有错。

```bash
cmake ../ && make
-- The C compiler identification is AppleClang 12.0.0.12000032
-- The CXX compiler identification is AppleClang 12.0.0.12000032
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working C compiler: /Library/Developer/CommandLineTools/usr/bin/cc - skipped
-- Detecting C compile features
-- Detecting C compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: /Library/Developer/CommandLineTools/usr/bin/c++ - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Warning (dev) in CMakeLists.txt:
  No cmake_minimum_required command is present.  A line of code such as

    cmake_minimum_required(VERSION 3.18)

  should be added at the top of the file.  The version specified may be lower
  if you wish to support older CMake versions for this project.  For more
  information run "cmake --help-policy CMP0000".
This warning is for project developers.  Use -Wno-dev to suppress it.

-- Configuring done
-- Generating done
-- Build files have been written to: /Users/jianglexing/repos/cpp-students/cpp_primer_plus/chapter4/build
Scanning dependencies of target neeky
[ 50%] Building CXX object CMakeFiles/neeky.dir/src/4_9.o
[100%] Linking CXX executable neeky
[100%] Built target neeky
```
看样子是 OK 了。

---

