## 概要
python-2 的年代，对异常的处理要有一定的技巧，不然还真不容易找到程序在哪里抛出了异常。看下面的例子。

```python
#!/usr/bin/env python
# coding:utf8


def inner_error():
    """
    用于引发一个除 0 异常 .
    """
    return 1/0


def warp_fun():
    """
    调用引发异常的函数，并处理异常。
    """
    try:
        inner_error()
    except Exception as err:
        raise err


if __name__ == "__main__":
    warp_fun()
```

看起来没有毛病，但是从异常的堆栈信息来看，并不能知道在哪里引发了除 0 异常。
```bash
python main.py 
Traceback (most recent call last):
  File "main.py", line 23, in <module>
    warp_fun()
  File "main.py", line 19, in warp_fun
    raise err
ZeroDivisionError: integer division or modulo by zero
```

![try-except](static/2020-34/sqlpy-python-try.jpg)

---

## 解决 python-2 的问题
要解决 Python-2 版本下的这个问题，要求我们改进写法，就是要把
```python
raise err
```
改成
```python
raise
```

完整的代码如下。
```python
#!/usr/bin/env python
# coding:utf8


def inner_error():
    """
    用于引发一个除 0 异常 .
    """
    return 1/0


def warp_fun():
    """
    调用引发异常的函数，并处理异常。
    """
    try:
        inner_error()
    except Exception as err:
        raise  # 直接 raise 后面不再接异常对象


if __name__ == "__main__":
    warp_fun()
```
运行效果如下，这下可以知道真正引发异常的是哪一行了。
```python
python main.py 
Traceback (most recent call last):
  File "main.py", line 23, in <module>
    warp_fun()
  File "main.py", line 17, in warp_fun
    inner_error()
  File "main.py", line 9, in inner_error
    return 1/0
ZeroDivisionError: integer division or modulo by zero
```

---

## python-3 的改进
python-3 来说两种写法都行，都能定位到最终抛异常的代码。
```python
#!/usr/bin/env python
# coding:utf8


def inner_error():
    """
    用于引发一个除 0 异常 .
    """
    return 1/0


def warp_fun():
    """
    调用引发异常的函数，并处理异常。
    """
    try:
        inner_error()
    except Exception as err:
        raise err


if __name__ == "__main__":
    warp_fun()

```
运行效果如下。
```bash
python3 main.py 
Traceback (most recent call last):
  File "main.py", line 23, in <module>
    warp_fun()
  File "main.py", line 19, in warp_fun
    raise err
  File "main.py", line 17, in warp_fun
    inner_error()
  File "main.py", line 9, in inner_error
    return 1/0
ZeroDivisionError: division by zero
```


---

