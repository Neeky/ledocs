## 背影
之前把 C++ 都还给了老师，但是学校又没有退我的学费。毕业后阴差阳错当了一个 MySQL-DBA ，MySQL-8.0.x 又是 C++ 写的，不懂点源码，过几年都不好出去见人。于是这段时间又把之前的 C++ 捡了起来，今天给 C++ 和 Python 跑了一个分，单单从性能上来讲 C++ 是真香啊！

![sqlpy](static/2020-37/C++-vs-Python3.jpg)

---

## 测试用例
分别用 Python 和 C++ 实现一个计算斐波那契数列第 n 位的函数看哪个快。为了尽可能的控制变量，我把 C++ 的实现做成了 Python 的外部模块。

---

## C++ 版本
由于要让 Python 直接调用 C++ 所以会多出一些非功能代码。
```c++
#include <Python.h>
/*
 * 计算斐波那契数列的第 n 位 。
*/
int fib(int index)
{
    if (index == 1 || index == 2)
    {
        return 1;
    }
    return fib(index - 1) + fib(index - 2);
}

/*
 * 以下代码是为了方便 Python 直接导入才加的，与功能无关。
*/
static PyObject *py_fib(PyObject *self, PyObject *args)
{
    int x = 0, result = 0;
    PyArg_ParseTuple(args, "i", &x);
    result = fib(x);
    return Py_BuildValue("i", result);
}

static PyMethodDef fib_methods[] = {
    {"fib", py_fib, METH_VARARGS, "fib"},
    {0, 0, 0, 0}};

static struct PyModuleDef fib_module = {
    PyModuleDef_HEAD_INIT,
    "fib",
    "a simple module",
    -1,
    fib_methods};

PyMODINIT_FUNC PyInit_fib(void)
{
    return PyModule_Create(&fib_module);
}

```
编译成共享库文件，让 Python 可以直接导入。
```bash
gcc -fPIC -I /usr/local/python-3.8.2/include/python3.8/ --shared -o fib.so fibs.cpp

-rw-r--r--. 1 root root  644 9月  12 15:16 fibs.cpp                                                
-rwxr-xr-x. 1 root root 8584 9月  12 15:17 fib.so   # 库文件生成了
```

---

## Python 版本
同样的逻辑用 Python 重写一次，并添加一个简单的跑分函数。
```python
import time

def fib_py(x):
    if x in (1,2):
        return 1
    else:
        return fib_py(x - 1) + fib_py(x - 2)                                             

def bench_mark(fun,args):
    start_at = time.time()
    fun(args) 
    end_at = time.time()
    print(end_at - start_at)
```

---

## 跑分
从这个测试结果来看 C++ 要比 Python 快 50 几倍。
```python
from fib import fib # 导入 C++ 的实现

# 求第 40 位 C++ 用时 0.6s
bench_mark(fib,40)                                                                         
0.6236903667449951

# 求第 40 位 Python 用时 31.4s
bench_mark(fib_py,40)                                                                     
31.4614417552948
```

|**C++ 耗时**|**Python 耗时**|
|------------|---------------|
| 0.6s       |  31.s|

---
