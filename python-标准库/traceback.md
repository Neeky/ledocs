## 背景
程序虽然有打印日志，但是遇到异常还是不知道问题在哪里，比如说下面的代码。
```python
#!/usr/bin/env python3
import logging


def fun_div(x, y):
    """实现一个除法功能
    """
    return x / y


def main():
    try:
        # 故意把分母设置为 0
        fun_div(1, 0)
    except Exception as err:
        logging.error(err)


if __name__ == "__main__":
    main()
```
运行效果。
```bash
python3 main.py 
ERROR:root:division by zero
```

![traceback](static/2020-15/traceback.png)

google-adsense

---


## 问题与解决方案
我们只打印了“异常”但是没有打印“异常的堆栈信息”，也就是说通过日志无法定位在哪里触发的异常。为了解决这个问题可以使用 traceback。

```python
#!/usr/bin/env python3
import logging
import traceback


def fun_div(x, y):
    """实现一个除法功能
    """
    return x / y


def main():
    try:
        # 故意把分母设置为 0
        fun_div(1, 0)
    except Exception as err:
        logging.error(err)
        # 打印堆栈信息
        traceback.print_exc()


if __name__ == "__main__":
    main()

```
运行效果。
```python
python3 main.py 
ERROR:root:division by zero
Traceback (most recent call last):
  File "./main.py", line 15, in main
    fun_div(1, 0)
  File "./main.py", line 9, in fun_div
    return x / y
ZeroDivisionError: division by zero
```

---


## 更加优雅的方案
如果是使用 logging 来管理日志，那么一切都更加简单了。
```python
#!/usr/bin/env python3
import logging
import traceback


def fun_div(x, y):
    """实现一个除法功能
    """
    return x / y


def main():
    try:
        # 故意把分母设置为 0
        fun_div(1, 0)
    except Exception as err:
        # 打印异常信息、异常的堆栈信息
        logging.exception(err)


if __name__ == "__main__":
    main()
```
运行效果。
```bash
python3 main.py 
ERROR:root:division by zero
Traceback (most recent call last):
  File "./main.py", line 15, in main
    fun_div(1, 0)
  File "./main.py", line 9, in fun_div
    return x / y
ZeroDivisionError: division by zero
```

---