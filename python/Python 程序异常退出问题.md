## 背景
之前一个段时间用 Python 写了一个后台服务，但是程序总是跑着跑着就异常退出了；由于不能稳定的复现一时就没有找到原因。

最近把原因理清楚了是由 `a > 1` 这样的一个测试条件引发的异常，正常情况下这个不会有问题，因为 a 是一个整数。而这个值是由一个外部系统的状态决定的，它有可能会是 None 。

出问题的时候整个程序差不多就是这样。

```python
In [1]: a = None                                                                

In [2]: a > 1                                                                   
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-2-878db1e46c45> in <module>
----> 1 a > 1

TypeError: '>' not supported between instances of 'NoneType' and 'int'
```

由于有未处理的异常，所以整个程序退出了。

![sqlpy](static/2020-30/sqlpy-ppc.jpg)

---

## 如何发现问题
发现问题的思路也比较直接，就是在函数调用栈的最低层进行异常处理，并记录异常和堆栈信息到日志文件；这样就算进程退出了，我们也还能堆栈中找到它退出的原因。一个简化的版本大概是这样的。

```python
#!/usr/bin/env python3

import logging


def exec_fun():
    return None > 0


def main():
    """
    """
    exec_fun()


if __name__ == "__main__":
    # 解决方案就是给主函数加上异常处理
    try:
        main()
    except Exception as err:
        logging.basicConfig(filename="/tmp/err.log")
        logging.exception(err)
```

运行后可以看到如下异常日志。

```bash
python3 py-raise.py 


cat /tmp/err.log 
ERROR:root:'>' not supported between instances of 'NoneType' and 'int'
Traceback (most recent call last):
  File "py-raise.py", line 19, in <module>
    main()
  File "py-raise.py", line 13, in main
    exec_fun()
  File "py-raise.py", line 7, in exec_fun
    return None > 0
TypeError: '>' not supported between instances of 'NoneType' and 'int'

```

堆栈信息非常清楚的指出了哪一行有问题。

---

## 代码重用
下次如果还是要做同样的事，我又再写一次 `try except`？ 太 low 了吧，看来我要造一个可重用的轮子。轮子的代码如下，我已经把它发布到了 pip 和 github ，我给项目起了一个名字叫 ppc (Python Posix Component) 。

```python
"""
实现通用的异常处理功能
"""

__ALL__ = ['catch', 'catch_all']


def catch_all(log_file=None):
    """
    装饰器工厂函数

    捕捉所有异常并把它写入到 log_file 指定的文件中去，
    如果 log_file == None 那么会在文件 /tmp/ppc-execeptions-YYYY-MM-DDTHH:MM:SS.ssssss.log
    中记录异常。

    Parameters
    ----------
    log_file: str
        日志文件的全路径，如果为 None 就在 /tmp/ 下创建异常日志

    Return
        desc: function
        返回一个装饰器

    """
    def desc(fun):
        """
        给 fun 加上 try except 块

        Parameters
        ----------
        fun: function
            要加上 try except 块的顶层函数
        """
        import logging
        import functools
        from datetime import datetime

        nonlocal log_file

        @functools.wraps(fun)
        def inner_main(*args, **kwargs):
            """
            """
            nonlocal log_file
            if not log_file:
                log_file = "/tmp/ppc-execeptions-{0}.log".format(
                    datetime.now().isoformat())
            logging.basicConfig(filename=log_file)

            try:
                fun(*args, **kwargs)
            except Exception as err:
                logging.exception(err)

        return inner_main

    return desc


catch = catch_all
```

---

## 用法
第一步 安装 python-posix-component 。
```bash

pip3 install python-posix-component

```
第二步 使用 python-posix-component 解决这个问题。
```python
#!/usr/bin/env python3

from ppc.exceptions import catch_all # 第一步、导入 catch_all 装饰器 


def exec_fun():
    return None > 0


@catch_all("/tmp/err.log") # 第二步、装饰入口点函数
def main():
    """
    """
    exec_fun()


if __name__ == "__main__":
    main()

```

---

## github 与 pypi

github 源码地址: [https://github.com/Neeky/ppc](https://github.com/Neeky/ppc)

pypi 包地址: [https://pypi.org/project/python-posix-component](https://pypi.org/project/python-posix-component)

---




