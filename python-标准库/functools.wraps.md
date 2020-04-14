## functools.wraps
要讲清楚 `functools.wraps` 需要一大段的内容，但是要概括 `functools.wraps` 的作用只要一句 “更新包装函数的元数据，使它看起来像被包装函数”。 下面会通过给函数加上日志为例子来一步步讲清楚 `functools.wraps`。

![wrap](static/2020-16/wrap.png)

google-adsense

---

## 函数是对象
在 Python 中函数也是一个值，只不过这个值可以调用。
```python
def print_message(message):
    """打印 message 参数的值
    """
    print(message)

# 把 print_message 这个函数赋值给 x 这个变量
x = print_message

x('hello world') # 这里会打印出 hello world
```
执行效果。
```bash
python3 main.py

hello world
```

---

## 函数有元数据
函数不但可以执行，和其它所有的对象一样，它还有自己的元数据。
```python
In [1]: def print_message(message): 
   ...:     """打印 message 参数的值 
   ...:     """ 
   ...:     print(message) 
   ...:                                                                                                                        

In [2]: print_message.__name__                                                                                                 
Out[2]: 'print_message'

# 更多的元数据可以通过 dir(print_message) 来查看
```

---

## 函数可以作为参数
既然函数是一个对象，那么它就能像其它对象那样，作为一个参数传递给其它函数；更有甚者一个函数的返回值是另一个函数。现在结合这两点实现一个为函数添加日志的功能。
```python
#!/usr/bin/evn python3
import logging


def log_it_desc(fun):
    """
    """
    def inner(*args, **kwargs):
        logging.warning(f"start {fun.__name__}")
        r = fun(*args, **kwargs)
        logging.warning(f"complete {fun.__name__}")
        return r
    
    # 一个函数的返回值是另一个函数
    return inner


def print_message(message):
    """打印 message 参数的值
    """
    print(message)

# 一个函数作为另一个函数的参数
print_message = log_it_desc(print_message)


if __name__ == "__main__":
    print_message("hello world")

```
运行效果。
```bash
python3 main.py 

WARNING:root:start print_message
hello world
WARNING:root:complete print_message
```
可以看到我们在不改 `print_message` 函数代码的情况下，为 `print_message` 加上了运行时的日志输出功能。

google-adsense

---


## 存在什么问题
问题就是前后两个 `print_message` 事实上是两个不同的对象，所以它们的元数据也就不一样，下面加上打印元数据的代码，用于观察效果。
```python
#!/usr/bin/evn python3
import logging


def log_it_desc(fun):
    """
    """
    def inner(*args, **kwargs):
        logging.warning(f"start {fun.__name__}")
        r = fun(*args, **kwargs)
        logging.warning(f"complete {fun.__name__}")
        return r

    # 一个函数的返回值是另一个函数
    return inner


def print_message(message):
    """打印 message 参数的值
    """
    print(message)

# 打印函数名
print(print_message.__name__)

# 一个函数作为另一个函数的参数
print_message = log_it_desc(print_message)

# 打印函数名
print(print_message.__name__)
```
运行效果。
```bash
python3 main.py 

print_message
inner
```
可以看到两次打印的 `print_message.__name__` 是不一样的，也就映证了两个函数的元数据不一样，那如何方便的把他们改成一样呢？

---

## functools.wraps 更新元数据
可以用 functools.wraps 来更新函数的元数据信息，让它看起来和另一个函数一样。
```python
#!/usr/bin/evn python3
import logging
import functools


def log_it_desc(fun):
    """
    """

    def inner(*args, **kwargs):
        logging.warning(f"start {fun.__name__}")
        r = fun(*args, **kwargs)
        logging.warning(f"complete {fun.__name__}")
        return r

    # 一个函数的返回值是另一个函数
    functools.wraps(fun)(inner)
    return inner


def print_message(message):
    """打印 message 参数的值
    """
    print(message)


print(print_message.__name__)

# 一个函数作为另一个函数的参数
print_message = log_it_desc(print_message)

print(print_message.__name__)
```
运行效果如下。
```python
python3 main.py 

print_message
print_message
```
现在元数据也改了，`inner` 函数伪装的更像 `print_message` 了呢。

---

## 高级语法
使用装饰器语法可以上代码看起来更加的简洁。
```python
#!/usr/bin/evn python3
import logging
import functools


def log_it_desc(fun):
    """
    """
    @functools.wraps(fun)
    def inner(*args, **kwargs):
        logging.warning(f"start {fun.__name__}")
        r = fun(*args, **kwargs)
        logging.warning(f"complete {fun.__name__}")
        return r

    return inner


@log_it_desc
def print_message(message):
    """打印 message 参数的值
    """
    print(message)


if __name__ == "__main__":
    print_message("hello world")
```
运行效果如下。
```bash
python3 main.py 

WARNING:root:start print_message
hello world
WARNING:root:complete print_message
```



