## 背景
最近这段时间在看 `<<C++ 并发编程实践>>` 这本书，刚开始写 hello world 就失败了，源码如下。
```c++
#include<iostream>
#include <unistd.h>
#include<thread>


using namespace std;

void hello(){
    int i = 47;
    while (i>=1){
        cout<<"hello word i = "<<i<<endl;
        sleep(1);
        i = i - 1;
    }
}


int main(){
    thread t(hello);
    t.join();
    //hello();
    return 0;
}

```
用 g++ 编译的时候报下面这个错。
```bash
g++ -o neek-multi multi-main.cpp 
/tmp/ccqbPDqG.o：在函数‘std::thread::thread<void (&)(), , void>(void (&)())’中：
multi-main.cpp:(.text._ZNSt6threadC2IRFvvEJEvEEOT_DpOT0_[_ZNSt6threadC5IRFvvEJEvEEOT_DpOT0_]+0x21)：对‘pthread_create’未定义的引用
collect2: 错误：ld 返回 1
```

![npm-schema](static/2020-46/lpthread.jpg)

---

## g++ 的解决办法
加 lpthread 选项。
```bash
g++ -lpthread -o neek-multi multi-main.cpp
```

---


## cmake 的解决办法
```cmake
cmake_minimum_required(VERSION 3.0)
project(nee)
add_definitions(-std=c++11)
add_executable(nee-multi src/multi-main.cpp)
target_link_libraries(nee-multi pthread)
```
这样就可以正常完成编译了。
```bash
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
-- Build files have been written to: /root/cpps/build

make
Scanning dependencies of target nee-singl
[ 25%] Building CXX object CMakeFiles/nee-singl.dir/src/single-main.cpp.o
[ 50%] Linking CXX executable nee-singl
[ 50%] Built target nee-singl
Scanning dependencies of target nee-multi
[ 75%] Building CXX object CMakeFiles/nee-multi.dir/src/multi-main.cpp.o
[100%] Linking CXX executable nee-multi
[100%] Built target nee-multi
```

---
