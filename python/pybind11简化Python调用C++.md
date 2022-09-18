

## pybind11 介绍
可以把 pybind11 看成是一个胶水，它可以把 C/C++ 语言定义的对象，方便的导出成 python 认识的格式，这样 python 就能直接用了

![pip3](static/2021-02/casey-horner-ygmZUdAzeBw-unsplash.jpg)

![pip3](static/2022-12/pybind11.jpeg)

google-adsense

---

## 第一步 实现业务功能并导出 example 模块
在这里我们假设业务功能就是一个简单的加法函数，并把这个 add 方法放到 example 模块里； src/example.cpp 文件的内容如下。
```cpp

#include <pybind11/pybind11.h>
namespace py = pybind11;

int add(int i, int j) {
    return i + j;
}

PYBIND11_MODULE(example, m) {
    m.doc() = "pybind11 示例"; // 模块文档字符串
    m.def("add", &add, "一个简单的加法函数");
}

```

---

## 第二步 把功能打包成 python 包
为了方便使用我们最好配置一下 setup.py 把上面的 C/C++ 代码打包成 python 包， setup.py 文件的内容如下。
```python

import sys
from pybind11 import get_cmake_dir
from pybind11.setup_helpers import Pybind11Extension, build_ext
from setuptools import setup

__version__ = "0.0.1"

ext_modules = [
    Pybind11Extension("example",
        ["src/example.cpp"],
        define_macros = [('VERSION_INFO', __version__)],
        ),
]

setup(
    name="example",
    version=__version__,
    author="neeky",
    author_email="neeky@live.com",
    description="A test project using pybind11",
    long_description="",
    ext_modules=ext_modules,
    extras_require={"test": "pytest"},
    cmdclass={"build_ext": build_ext},
    zip_safe=False,
    python_requires=">=3.6",
    install_requires=['pybind11==2.10.0']
)
```
现在整个项目的结构如下
```bash

tree .
.
├── setup.py
└── src
    └── example.cpp

1 directory, 2 files
```
---

## 第三步 打包安装
现在我们可以把刚才的 C/C++ 代码打包成 Python 包，并安装。
```bash

# 打包
python3 setup.py sdist

# 安装
pip3 install dist/example-0.0.1.tar.gz

Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple
Processing ./dist/example-0.0.1.tar.gz
...
Running setup.py install for example ... - 
```
---

## 第四步 体验 C/C++ 写的模块
现在可以用 python 代码一样来，使用刚才的 C/C++ 代码了。
```python

In [1]: import example

In [2]: example.__file__
Out[2]: '/usr/local/python-3.10.4/lib/python3.10/site-packages/example.cpython-310-x86_64-linux-gnu.so'

In [3]: example.add(100, 100)
Out[3]: 200
```

可以看到对于用 C/C++ 实现的模块，不再是我们熟悉的 .py 文件，而是一个动态连接库文件。

---