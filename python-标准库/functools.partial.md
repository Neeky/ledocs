## functools.partial
`functools.partial` 接收一个函数，并返回一个新的函数，与装饰器不同的是`functools.partial`可以传递更多的参数。 

```python
def partial(func, /, *args, **keywords):
    def newfunc(*fargs, **fkeywords):
        newkeywords = {**keywords, **fkeywords}
        return func(*args, *fargs, **newkeywords)
    newfunc.func = func
    newfunc.args = args
    newfunc.keywords = keywords
    return newfunc
```

那 `functools.partial` 的目的是什么呢？答案是代码重用，是面向过程编程的代码重用。

![partial](static/2020-15/partial.png)

google-adsense

---

## 求和函数一
假设有一个需求是给对给定的任意多个整数进行求和，python 代码可以这样
```python
#!/usr/bin/evn python3

import functools


def fun_sum(*args):
    """实现一下简单的求和函数
    """
    return sum(args)

def main():
    print(fun_sum(1, 2, 3, 4))


if __name__ == "__main__":
    main()

```
运行结果如下。
```python
python3 main.py 
10
```

---

## 新需求

不管求出来的和是多少，都要在这个的基础之上加上 100 再返回。

解决方案一、在调用 fun_sum 的时候多加一个 100 的参数
```python
#!/usr/bin/evn python3

import functools


def fun_sum(*args):
    """实现一下简单的求和函数
    """
    return sum(args)


def main():
    print(fun_sum(1, 2, 3, 4, 100))#把 100 直接加在后面


if __name__ == "__main__":
    main()

```

运行效果。

```bash
python3 main.py 
110
```
这样是可以解决问题，时间一长 100 这个常量鬼自己当初是怎么来的。

---

增加一个新的函数把这个 + 100 的功能特异化一下。
```python
#!/usr/bin/evn python3

import functools


def fun_sum(*args):
    """实现一下简单的求和函数
    """
    return sum(args)


def fun_sum_plus_100(*args):
    """对 *args 求和，并在和的基础之上再加上 100
    """
    return sum(args) + 100


def main():
    print(fun_sum_plus_100(1, 2, 3, 4))


if __name__ == "__main__":
    main()

```

执行效果。

```bash
python3 main.py 
110
```

---


## 总结问题
如果使用第一个解决方案，遇到 + 100 的场景越多，那么 100 这个常量就重复出现的次数就越多；并且时间一长还有可能忘记了当初为什么要这么做。使用第二个方案虽然语义上更加明确，但是我们又多定义了一个函数。并且这个函数只是屏蔽一下另一个函数要多传递一个参数的场景，感觉上用了牛刀。



---

## 使用 functools.partial
下面看一下 functools.partial 如何改进代码。

```python
#!/usr/bin/evn python3

import functools


def fun_sum(*args):
    """实现一下简单的求和函数
    """
    return sum(args)

fun_sum_plus_100 = functools.partial(fun_sum, 100)


def main():
    print(fun_sum_plus_100(1, 2, 3, 4))


if __name__ == "__main__":
    main()

```

运行效果。

```bash
python3 main.py 
110
```

---


## 总结

`functools.partial` 用于在哪些通用的方法上加上更多的参数，以让它适更加特殊的场景。就是有点像面向对象编程中的定义子类。

---







