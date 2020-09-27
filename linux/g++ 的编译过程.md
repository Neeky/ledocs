## 概要
之前在 windows 上开发 C++ ，vs 过于强大搞的我一直都没有去了解过 C++ 程序从源文件编译成可执行文件的过程。整个大学四年过去了这个也没有影响到我写代码，工作有几年了，现在的看法成熟了不少，想多了解一下 C++ 在 linux 上的编译过程。

![sqlpy](static/2020-37/g++.jpg)

---

## 示例代码
还是用一个经典的 hello world 为例子吧，源代码保存到 `main.cpp` 文件中，源代码如下。
```c++
#include<iostream>

int main()
{
    using namespace std;
    cout<<"hello world"<<endl;
    cin.get();
}
```
一行命令从源代码到可以执行程序。
```bash
ll
总用量 4
-rw-r--r--. 1 root root 106 9月  27 20:41 main.cpp

# 编译
g++ main.cpp -o main

# 得到可执行文件
ll -h
总用量 16K
-rwxr-xr-x. 1 root root 8.9K 9月  27 20:53 main
-rw-r--r--. 1 root root  106 9月  27 20:41 main.cpp

# 执行
./main                                                                    
hello world

```
这个虽然一行命令解决了问题，但是不利于我们了解其背后的过程。

google-adsense

---

## 第一步 预处理
预处理会把 `#include xxx` 直接替换成对应头文件的内容。
```bash
# -E 只做预处理
g++ -E main.cpp -o main.i 

ll -h
总用量 416K
-rw-r--r--. 1 root root  106 9月  27 20:41 main.cpp
-rw-r--r--. 1 root root 411K 9月  27 20:58 main.i  # 可以看到头文件被插入之后文件的大小变大了
```
查看 main.i 文件的最后几行。
```bash
# 查看 main.i 最后几行
tail -n 17 main.i
  extern wostream wclog;




  static ios_base::Init __ioinit;


}
# 2 "main.cpp" 2

int main()
{
    using namespace std;
    cout<<"hello world"<<endl;
    cin.get();
}
```

---

## 第二步 汇编
把 C++ 语言的代码转换成汇编语言。
```bash
# 汇编
g++ -S main.i
# 
ll
总用量 420
-rw-r--r--. 1 root root    106 9月  27 20:41 main.cpp
-rw-r--r--. 1 root root 420808 9月  27 20:58 main.i
-rw-r--r--. 1 root root   1881 9月  27 21:03 main.s
```
查询 main.s 的前几行
```bash
        .file   "main.cpp"                                                                       
        .local  _ZStL8__ioinit                                                                   
        .comm   _ZStL8__ioinit,1,1                                                               
        .section        .rodata                                                                  
.LC0:                                                                                            
        .string "hello world"                                                                    
        .text                                                                                    
        .globl  main                                                                             
        .type   main, @function                                                                  
main:                                                                                            
.LFB971:                                                                                         
        .cfi_startproc 
```

---

## 第三步 编译
编译之后的产物是库文件
```bash
g++ -c main.s  # 生成 main.o 文件

ll
总用量 424
-rw-r--r--. 1 root root    106 9月  27 20:41 main.cpp
-rw-r--r--. 1 root root 420808 9月  27 20:58 main.i
-rw-r--r--. 1 root root   2784 9月  27 21:11 main.o
-rw-r--r--. 1 root root   1881 9月  27 21:03 main.s
```
库文件已经是二进制格式的了(机器码)。
```bash
hexdump main.o | head 
0000000 457f 464c 0102 0001 0000 0000 0000 0000
0000010 0001 003e 0001 0000 0000 0000 0000 0000
0000020 0000 0000 0000 0000 0720 0000 0000 0000
0000030 0000 0000 0040 0000 0000 0040 000f 000e
0000040 4855 e589 00be 0000 bf00 0000 0000 00e8
0000050 0000 be00 0000 0000 8948 e8c7 0000 0000
0000060 00bf 0000 e800 0000 0000 00b8 0000 5d00
0000070 55c3 8948 48e5 ec83 8910 fc7d 7589 83f8
0000080 fc7d 7501 8127 f87d ffff 0000 1e75 00bf
0000090 0000 e800 0000 0000 00ba 0000 be00 0000
```
---

## 第四步 链接
链接的产物是可执行文件
```bash
g++ main.o -o main-execute  

./main-execute 
hello world
```

---