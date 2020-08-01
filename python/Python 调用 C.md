## 背景
近期准备从源码的角度更深入的学习下 MySQL，所以又重操旧业看起了 C/C++ 。可能是对 Python 太喜欢了吧，突然想到 Python 怎么才能调用我的 C/C++ 代码呢？在 Python 官方网站上“沉迷”了半天，终于把程序跑通了。

Python 官方文档 [Extending and Embedding](https://docs.python.org/3.8/extending/index.html) 说了好多。虽然我看懂了，但是我觉得它没有讲清楚为什么要这样做。

![sqlpy](static/2020-31/sqlpy-c.jpg)

---

## 跨越语言的鸿沟
如果用一句话来概括就是“我们需要在 Python 和 C/C++ 之间有一个翻译”。

第一件事就是要做数据类型的翻译，例如把 Python 的 int 类型翻译成 C/C++ 的 int 类型，这两个 int 的差别还是比较大的，理论上只要你机器的内存无穷大 Python 的 int 就能无穷大，而 C/C++ int 永远都是 4 个字节。

第二件事就是要做句法上的翻译，例如执行 `import xxx` 导入 C/C++ 库时，我们要在 C/C++ 层面导出哪些对象。不幸的是整个翻译程序都要我们自己实现，幸运的是 Python 为我们提供了必要的基础设施(Py开头的所有对象)实现起来也不是特别难。

---

## 例子
我们要用 C 语言实现一个叫 `sqlpy` 的模块，其中的 plus_one 函数返回一个比传入参数大 1 的值。整个模块可以被 Python 直接导入使用，使用方感知不到它的实现语言是 Python 还是 C/C++ ，最终的效果像这样。

```python
In [1]: import sqlpy                                                                             
                                                                                                 
In [2]: sqlpy.plus_one(118)                                                                      
Out[2]: 119
```

google-adsense

---

## 第一步 用C实现功能
第一步用 C 语言实现我们模块的主要功能“返回一个比较输入值大 1 的新值”。
```c
int plus_one(int x)
{
    return x + 1;   
}
```

---

## 第二步 给函数添加翻译
添加一个翻译它能同时理解 Python 和 C/C++ 的函数调用，把 Python 想表达的意思翻译成 C/C++，然后把得到的执行结果翻译成 Python 能看的懂的形式。
```c
#include <Python.h>

int plus_one(int x)
{
    return x + 1;   
}

// Python 中一切都是对象，所以翻译函数的参数，返回也就都是 PyObject 类型的指针
static PyObject *py_plus_one(PyObject *self,PyObject *args) 
{
    int x = 0,result = 0;
    // 参数数据类型转换
    PyArg_ParseTuple(args,"i",&x);
    result = plus_one(x);
    // 返回值数据类型转换
    return Py_BuildValue("i",result);
}
```

---

## 第三步 定义模块中的函数列表
由于我们只有一个函数，所以整个列表就只有一个元素。
```c
// 定义模块中要导出的函数列表
/*
 * {导出时 python 看到的函数名,翻译函数的名字,参数的传递方式,函数的文档字符串}
 * {0,0,0,0} 可以看成是个哨兵值，用来表示结尾
*/
static PyMethodDef sqlpy_methods[] = {
{"plus_one",py_plus_one,METH_VARARGS,"plus one"},
{0,0,0,0}
};
```

---

## 第四步 给 import 添加翻译
当模块被导入时实际上就是执行 `PyInit_模块名` 这个函数，所以我们要在这个函数里面返回一个 Python 模块。
```c
/*
 * 定义模块的数据结构 {PyModuleDef_HEAD_INIT,模块名,文档字符串,-1,模块要导出的函数列表}
*/
static struct PyModuleDef sqlpy_module = {
PyModuleDef_HEAD_INIT,
"sqlpy",
"a simple module",
-1,
sqlpy_methods
};

/*
 * 创建模块实现，PyInit_模块名 。
 * 当 python 中 import xxx 时就调用着对应的 PyInit_xxx 函数。
*/
PyMODINIT_FUNC PyInit_sqlpy(void)
{
    return PyModule_Create(&sqlpy_module);
}
```

---

## 第五步 编译模块
要把我们用 C/C++ 编写的模块编译成 xxx.so 或 xxx.ddl 这样的库文件，然后 python 就可以像导入 xxx.py 一样的导入这些模块了。

完整的 C 代码如下。
```c
#include <Python.h>

int plus_one(int x)
{
    return x + 1;   
}

static PyObject *py_plus_one(PyObject *self,PyObject *args) 
{
    int x = 0,result = 0;
    PyArg_ParseTuple(args,"i",&x);
    result = plus_one(x);
    return Py_BuildValue("i",result);
}

static PyMethodDef sqlpy_methods[] = {
{"plus_one",py_plus_one,METH_VARARGS,"plus one"},
{0,0,0,0}
};

static struct PyModuleDef sqlpy_module = {
PyModuleDef_HEAD_INIT,
"sqlpy",
"a simple module",
-1,
sqlpy_methods
};

PyMODINIT_FUNC PyInit_sqlpy(void)
{
    return PyModule_Create(&sqlpy_module);
}
```

编译成库文件

```bash
# 编译
gcc -fPIC -I /usr/local/python-3.8.2/include/python3.8/ -o sqlpy.so -shared sqlpy.c

# 查看库文件是否有生成
ll | grep sqlpy
-rw-r--r--. 1 root root  564 8月   1 17:48 sqlpy.c                                               
-rwxr-xr-x. 1 root root 8560 8月   1 21:39 sqlpy.so
```
---

## 使用

经过上面这么多的操作步骤，我们的模块终于可以用了。
```python
In [1]: from sqlpy import plus_one                                                               
                                                                                                 
In [2]: plus_one(10085)                                                                          
Out[2]: 10086
```

如果你想把 C/C++ 模块以源码的方式发行，当用户在 `pip3 install 你的模块` 时自动的编译 C/C++ 模块也是可以的，我在这里就不行了，看官方的文档吧。

google-adsense

---