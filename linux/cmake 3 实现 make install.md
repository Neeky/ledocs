## 背景
记得刚开始玩 linux 的时候，编译安装软件都让人感觉是一件特别神奇的事。一个 `make && make install` 就完成了，并没有看到什么 gcc/g++ 呀。学生时代太年轻，见识浅薄，以为自己一定不会去写这种界面都没有的程序。

谁又能想的到，一个人的命运啊，当然要靠自我奋斗；也要考虑到历史的进程。还没有毕业 windows phone 平台就日薄西山了，注定我成不了一个 windows phone 平台的程序员小哥。

感觉命运又和我开了一次 180 度的大玩笑，那个想写手机 app 的少年一度轻看字符界面的程序，工作几年后发现自己边字符界面的程序都没的开发，只能开发没有界面的程序(linux 的守护进程运行在后台没有界面)。

![sqlpy](static/2020-41/cmake-03.jpg)

---

## 有些快乐来的突然
最近在看 MySQL 的源代码，发现所有和编译相关的事都交给了 cmake 来处理。经过多次白嫖 [www.cmake.org](www.cmake.org) 之后，发现我悟了。我也能给你整一个支持 `make && make install` 的发行包了。

虽然我用的是 cmake 和老师当年手写 MakeFile 是不能同日而语的(手写 MakeFile 要难些)。


---

## 一个最简单的项目
要想让自己的 c/c++ 支持 `make && make install` 非常简单，像我下面要说的这个 hello world 项目四行代码就行了。

项目结构如下
```bash
tree hello/

hello/
├── CMakeLists.txt
└── src
    └── main.cpp

1 directory, 2 files
```
`main.cpp` 比较简单就是一个 hello-world 。
```c++
#include<iostream>

int main()
{
    using namespace std;
    cout<<"hello world"<<endl;
    cin.get();
}
```
cmake 的配置文件 `CMakeLists.txt` 更加简单只有四行。
```cmake
cmake_minimum_required(VERSION 3.0)
project(hello)
add_executable(hello src/main.cpp)
install(TARGETS hello DESTINATION bin)
```

对就是这么简单下面就可以开始玩编译了。

```bash
# 创建用于编译的子目录
mkdir build
cd build

# 用 cmake 生成 MakeFile
cmake ../
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
-- Build files have been written to: /root/hello/build

# 查看生成的内容
ll
总用量 28
-rw-r--r--. 1 root root 13895 10月 18 21:16 CMakeCache.txt
drwxr-xr-x. 5 root root   280 10月 18 21:16 CMakeFiles
-rw-r--r--. 1 root root  2337 10月 18 21:16 cmake_install.cmake
-rw-r--r--. 1 root root  7424 10月 18 21:16 Makefile

# 编译
make
Scanning dependencies of target hello
[ 50%] Building CXX object CMakeFiles/hello.dir/src/main.cpp.o
[100%] Linking CXX executable hello
[100%] Built target hello

# 安装
make install
[100%] Built target hello
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/bin/hello

# 运行
/usr/local/bin/hello                                                     
hello world

```

---
