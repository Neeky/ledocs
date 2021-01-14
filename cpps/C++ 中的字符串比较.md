## 背景
对于一个被 python 宠爱坏了的程序员，突然下定决定要找回自己的 C++ ，并开始学习时哪哪都会觉得 Python 理人性。今天看到 C/C++ 中的字符串比较，还好我仔细的看了，要不然就是一个坑。

![cpp-string](static/2021-01/cpp-string.jpg)

---

## C 中的字符串比较
C 中的字符串是基于数组的，而数组的本质上是一块内存的头指针；所以如果直接比较两个字符串，实际上是比较的这两个字符串的地址，和我想要的完全不是一个东西。
```c++
#include<iostream>
#include<string>
#include<cstring>

using namespace std;

int main()
{
    char tom[8] = "tom";
    char tim[8] = "tom";
    if (tom == tim ){ // 由于两个字符串的地址完全不一样，所以这里的 if 测试一定是 false.
        printf("equal .");
    }
    else{
        printf("not equal.");
    }

    return 0;
}

```
运行时的效果。
```bash
./main 
not equal
```
现代的编译器还是比较智能的针对上面这种明显的“不正常”，在其编译的时候是会有告警的。
```bash
/private/tmp/ups/main.cpp:11:13: warning: array comparison always evaluates to false [-Wtautological-compare]
    if (tom == tim ){ // 由于两个字符串的地址完全不一样，所以这里的 if 测试一定是 false.
            ^
1 warning generated.
```
---

如果真要在 C 语言中比较两个字符串，这个时候我们要用到`cstring`头文件中声明的 `strcmp` 方法；些方法接收两个地址，当两个字符串相等的时候返回 0。
```c++
#include<iostream>
#include<string>
#include<cstring>

using namespace std;

int main()
{
    char tom[8] = "tom";
    char tim[8] = "tom";
    if (strcmp(tom,tim) == 0){ // 当给定的两个字符串的值相等的时候返回 0.
        printf("equal .");
    }
    else{
        printf("not equal.");
    }

    return 0;
}

```
运行时的效果如下。
```bash
./main
equal .
```
这个版本的比较方法编译时就不会再有告警了。
```bash
Starting build...
Build finished successfully.

Terminal will be reused by tasks, press any key to close it.
```
---

## C++ 中的字符串比较
C++ 要比 C 友好一点点，对于字符串这种常用的东西，还是抽象出了一个字符串类型 `string` 专门来做这个事。`string` 重载了比较运算符，所以用起来就方便多了。

```C++
#include<iostream>
#include<string>
#include<cstring>

using namespace std;

int main()
{
    string tom = "tom";
    string tim = "tom";
    if (tom == tom){ // C++ 的 string 对象可以直接进行比较.
        printf("equal .");
    }
    else{
        printf("not equal.");
    }

    return 0;
}

```

---