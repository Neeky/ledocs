## 概要
不要以为 Python 有自动垃圾回收就不会内存泄漏，本着它有“垃圾回收”我有垃圾代码的精神，现在总结一下三种常见的内存泄漏场景。

![sqlpy](static/2020-31/sqlpy-weakref.jpg)

---

## 无穷大导致内存泄漏
如果把内存泄漏定义成只申请不释放，那么借着 Python 中整数可以无穷大的这个特点，我们一行代码就可以完成内存泄漏了。
```python
i = 1024 ** 1024 ** 1024
```

---

## 循环引用导致内存泄漏
引用记数器 是 Python 垃圾回收机制的基础，如果一个对象的引用数量不为 0 那么是不会被垃圾回收的，我们可以通过 `sys.getrefcount` 来得到给定对象的引用数量。
```python
In [1]: import sys                                                              

In [2]: a = {'name':'tom','age':16}                                             

In [3]: sys.getrefcount(a)   # 由于 getrefcount 内部也会临时的引用 a 所以，使得计数器的值变成了 2 。                              
Out[3]: 2

In [4]: b = a                                                                   

In [5]: sys.getrefcount(a)                                                      
Out[5]: 3
```

先来看一个循环引用的场景。
```python
#!/usr/bin/evn python3

import sys
import time
import threading


class Person(object):
    free_lock = threading.Condition()

    def __init__(self, name: str = ""):
        """
        Parameters
        ----------
        name: str
            姓名

        best_friend: str
            最要好的朋友名
        """
        self._name = name
        self._best_friend = None

    @property
    def best_friend(self, person: "Person"):
        return self._best_friend

    @best_friend.setter
    def best_friend(self, friend: "Person"):
        self._best_friend = friend

    def __str__(self):
        """
        """
        return self._name

    def __del__(self):
        """
        """
        self.free_lock.acquire()
        print(f"{self._name} 要 GG 了，现在释放它的内存空间。")
        sys.stderr.flush()
        self.free_lock.release()


def mem_leak():
    """
    循环引用导致内存泄漏
    """
    zhang_san = Person(name='张三')
    li_si = Person("李四")

    # 构造出循环引用
    # 李四的好友是张三
    li_si.best_friend = zhang_san
    # 张三的好友是李四
    zhang_san.best_friend = li_si


if __name__ == "__main__":
    for i in range(3):
        time.sleep(0.01)
        print(f"{i}")
        mem_leak()

    print("mem_leak 执行完成了.")
    time.sleep(5)

```
运行效果。
```bash
python3 main.py 
0
1
2
mem_leak 执行完成了.
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间。
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间。
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间
```

google-adsense

由于循环引用的存在，使得 mem_leak 函数就行执行完了其内部的局部变量引用计数器也不为 0 ，所以内存得不到及时的释放。释放这部分内存有两个途径 1、 被 Python 内部的循环检测机制发现了； 2、进程退出前的集中释放。

[tracemalloc](https://docs.python.org/3.8/library/tracemalloc.html) 可以在一定程序上帮我们发现问题，在此就不讲怎么用了，我们直接上解决方案。Python 为程序员提供了弱引用，通过这种方式可以不增加对象引用计数器的数值，这成为了我们打破循环引用的一种手段。

```python
In [1]: import sys                                                              

In [2]: import weakref                                                          

In [3]: from main import Person                                                 

In [4]: tom = Person('tom')                                                     

In [5]: sys.getrefcount(tom)                                                    
Out[5]: 2

In [6]: p = weakref.ref(tom)                                                    

In [7]: sys.getrefcount(tom)    # 弱引用不会增加计数器的值                                                
Out[7]: 2
```
现在使用 weakref 技术来改造我们的代码。
```python
#!/usr/bin/evn python3


import sys
import time
import weakref
import threading


class Person(object):
    free_lock = threading.Condition()

    def __init__(self, name: str = ""):
        """
        Parameters
        ----------
        name: str
            姓名

        best_friend: str
            最要好的朋友名
        """
        self._name = name
        self._best_friend = None

    @property
    def best_friend(self, person: "Person"):
        return self._best_friend

    @best_friend.setter
    def best_friend(self, friend: "Person"):
        self._best_friend = weakref.ref(friend)

    def __str__(self):
        """
        """
        return self._name

    def __del__(self):
        """
        """
        self.free_lock.acquire()
        print(f"{self._name} 要 GG 了，现在释放它的内存空间。")
        sys.stderr.flush()
        self.free_lock.release()


def mem_leak():
    """
    循环引用导致内存泄漏
    """
    zhang_san = Person(name='张三')
    li_si = Person("李四")

    # 构造出循环引用
    # 李四的好友是张三
    li_si.best_friend = zhang_san
    # 张三的好友是李四
    zhang_san.best_friend = li_si


if __name__ == "__main__":
    for i in range(3):
        time.sleep(0.01)
        print(f"{i}")
        mem_leak()

    print("mem_leak 执行完成了.")
    time.sleep(5)

```
运行效果。
```bash
python3 main.py 
0
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间。
1
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间。
2
张三 要 GG 了，现在释放它的内存空间。
李四 要 GG 了，现在释放它的内存空间。
mem_leak 执行完成了.
```

可以看到现在一旦函数执行完成，其内部的局部变量的内存就会得到释放，非常的及时。

---

## 外面库内存泄漏

这种情况我也只遇到过一次，之前 mysql-connector-python 的内存泄漏，导致我的程序跑着跑着占用的内存就越来越大；最后我们返的 C 语言扩展禁用之后就没有问题了。

---
