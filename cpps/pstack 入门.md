## pstack 

pstack 用来打印进程的函数调用堆栈，通过它我们可以非常方便的看出来程序目前做些什么(再执行哪些函数)。在 linux 下作为 gdb 软件包的一部分，也就是说可以通过如下的方式进行安装。

```bash
yum -y install gdb 
```
pstack 只是一个连接它指向了 gstack 。
```bash
ll /usr/bin/pstack
lrwxrwxrwx. 1 root root 6 6月  16 2020 /usr/bin/pstack -> gstack
```
gstack 只是一个 bash 脚本。
```bash
head /usr/bin/gstack 
#!/bin/sh

if test $# -ne 1; then
    echo "Usage: `basename $0 .sh` <process-id>" 1>&2
    exit 1
fi

if test ! -r /proc/$1; then
    echo "Process $1 not found." 1>&2
    exit 

... ...

```

![npm-schema](static/2020-46/pstack.jpg)

---

## pstack 单线程
直接用 pstack 看进程的函数堆栈信息，会让人不知所云。如果结合程序源码来看的话会好许多。下面是一个循环打印 hello word 的程序。
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
    //thread t(hello);
    //t.join();
    hello();
    return 0;
}
```
用 pstack 查看进程的堆栈信息。
```bash
pstack ${pid}
#0  0x00007fa119942208 in nanosleep () from /lib64/libc.so.6
#1  0x00007fa11994213e in sleep () from /lib64/libc.so.6
#2  0x0000000000400921 in hello() ()
#3  0x0000000000400933 in main ()
```

---

## pstack 多线程

如果程序只有一个主线程，看起来格式还比较友好，如果程序有多个线程，相对来讲会乱一点点。现在我把上面的代码改一下，让它在一个子线程中打印 hello word 。
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
    return 0;
}
```
用 pstack 查看进程的堆栈信息。
```bash
pstack ${pid}
Thread 2 (Thread 0x7fe5b3d02700 (LWP 1419)):
#0  0x00007fe5b3dcb238 in nanosleep () from /lib64/libc.so.6
#1  0x00007fe5b3dcb13e in sleep () from /lib64/libc.so.6
#2  0x0000000000400e01 in hello() ()
#3  0x0000000000401107 in void std::__invoke_impl<void, void (*)()>(std::__invoke_other, void (*&&)()) ()
#4  0x0000000000400f60 in std::__invoke_result<void (*)()>::type std::__invoke<void (*)()>(void (*&&)()) ()
#5  0x00000000004015ae in decltype (__invoke((_S_declval<0ul>)())) std::thread::_Invoker<std::tuple<void (*)()> >::_M_invoke<0ul>(std::_Index_tuple<0ul>) ()
#6  0x0000000000401584 in std::thread::_Invoker<std::tuple<void (*)()> >::operator()() ()
#7  0x0000000000401568 in std::thread::_State_impl<std::thread::_Invoker<std::tuple<void (*)()> > >::_M_run() ()
#8  0x00007fe5b4721b73 in execute_native_thread_routine () from /lib64/libstdc++.so.6
#9  0x00007fe5b49fc2de in start_thread () from /lib64/libpthread.so.0
#10 0x00007fe5b3dfee83 in clone () from /lib64/libc.so.6
Thread 1 (Thread 0x7fe5b4e2e740 (LWP 1418)):
#0  0x00007fe5b49fd7cd in __pthread_timedjoin_ex () from /lib64/libpthread.so.0
#1  0x00007fe5b4721df7 in std::thread::join() () from /lib64/libstdc++.so.6
#2  0x0000000000400e30 in main ()
```

1、多线程的情况下主线程的 id 是 1 ，当程序是单线时 pstack 并不会打开线程号。


---