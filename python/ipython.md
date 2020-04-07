## 概要
之前一直是直接使用 python 解释器，然而 python 解释器整体上就处于“也不是不能用”的状态；总的来讲就是让人去适用工具，比如说要退出一定要执行`exit()`函数才行。直到后来我遇到了 ipython 这家伙真香。

![ipython](static/2020-15/ipython.png)

google-adsense

---


## 官方解析器不人性
下面举几个官方解释器不太人性化的例子。

1、有时候突然想知道自己当前是在哪个目录下(由于 python 包的导入规则，在 debug 的时候常常会想要知道自己当前在哪个目录)，然而官方的解释器要你编程来实现。
```python
python3
Python 3.8.0 (v3.8.0:fa919fdf25, Oct 14 2019, 10:23:27) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import os
>>> os.getcwd()
'/Users/jianglexing'
>>> 
```
ipython 在这方面做了比较在的改进，它直接直接打命令行。
```python
ipython
Python 3.8.0 (v3.8.0:fa919fdf25, Oct 14 2019, 10:23:27) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.8.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: pwd                                                                     
Out[1]: '/Users/jianglexing'
```

---

2、就是算是想退出解释器也不方便，官方的解释器要调用 `exit()` 函数才行。
```python
python3
Python 3.8.0 (v3.8.0:fa919fdf25, Oct 14 2019, 10:23:27) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
```
ipython 支持直接 `exit` 就行了。
```python
ipython
Python 3.8.0 (v3.8.0:fa919fdf25, Oct 14 2019, 10:23:27) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.8.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: exit
```

---

## 安装 ipython
如果你也想体验一下 `ipython` 带来的快乐，可以通过如下命令安装 ipython 。
```bash
pip3 install ipython
```

---