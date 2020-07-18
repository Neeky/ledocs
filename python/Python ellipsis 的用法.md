## 背景
在 Python 的基本类型中单例模式的值有三个 None 类型的 `None` ,NotImplemented 类型的 `NotImplemented`, Ellipsis 类型的 `...` 。

None 已经用的烂大街了，NotImplemented 也比较常用，唯独 `...` 在江湖上只知它是三巨头之一，但不知其用法。

![sqlpy](static/2020-29/sqlpy-ellipsis.jpg)

---

## Ellipsis
Ellipsis 在 python 中代表“省略”，用现在的流形语来表达就是“老铁，不要在意这些细节！”。哪什么时候要告诉别人不要在意这些细节呢？其中的一个场景就是随机值。


---


## 用于文档测试
假设我们编写了一个类，要想知道这个有没有语法层面的错误，只要简单的调用一下就能测试出来。为了把这个测试自动化，于是做成了文档测试。
```python
#!/usr/bin/evn python3

class Person(object):
    """人类类型
    Parameters:
    ----------
        name: str
        age: int

    Return:
    ------
    
    >>> Person()
    <main.Person object at 0x7ff36c1ca250>
    """

    name = ''
    age = 0

    def __init__(self, name: str = 'tom', age: int = 10) -> 'Person':
        """初始化
        """
        self.name = name
        self.age = age

    def say_hello(self) -> str:
        """返回打招呼信息
        """
        return f"Hello My name is {self.name} ."


```
当我们运行测试用例时会报错，原因是每次创建的对象，它的内存地址并不等于测试用例中指定的哪个，而我们的用例上写死了。诚然这个问题用 unittest 可以解决，但是这个不是这里要讲的。

```bash
python3 -m doctest main.py -v
Trying:
    Person()
Expecting:
    <main.Person object at 0x7ff36c1ca250>
**********************************************************************
File "/private/tmp/main.py", line 12, in main.Person
Failed example:
    Person()
Expected:
    <main.Person object at 0x7ff36c1ca250>
Got:
    <main.Person object at 0x7fe4e078ac70>
3 items had no tests:
    main
    main.Person.__init__
    main.Person.say_hello
**********************************************************************
1 items had failures:
   1 of   1 in main.Person
1 tests in 4 items.
0 passed and 1 failed.
***Test Failed*** 1 failures.
```

google-adsense

哪如何才能告诉 doctest 这位老铁不要在意返回值细节呢？答案是加上 `Ellipsis` 这个指令，改造后的代码如下。
```python
#!/usr/bin/evn python3


class Person(object):
    """人类类型
    Parameters:
    ----------
        name: str
        age: int

    Return:
    ------

    >>> Person() #doctest: +ELLIPSIS
    <main.Person object at 0x...>
    """

    name = ''
    age = 0

    def __init__(self, name: str = 'tom', age: int = 10) -> 'Person':
        """初始化
        """
        self.name = name
        self.age = age

    def say_hello(self) -> str:
        """返回打招呼信息
        """
        return f"Hello My name is {self.name} ."
```
运行测试用例这下可以通过了。
```python
python3 -m doctest main.py -v
Trying:
    Person() #doctest: +ELLIPSIS
Expecting:
    <main.Person object at 0x...>
ok
3 items had no tests:
    main
    main.Person.__init__
    main.Person.say_hello
1 items passed all tests:
   1 tests in main.Person
1 tests in 4 items.
1 passed and 0 failed.
Test passed.
```

---

## 其它
如果我们是为模块添加测试用例，那么可以这样做，会方便一些。
```python
#!/usr/bin/evn python3


class Person(object):
    """人类类型
    Parameters:
    ----------
        name: str
        age: int

    Return
    ------

    >>> Person() #doctest: +ELLIPSIS
    <...Person object at 0x...>
    """

    name = ''
    age = 0

    def __init__(self, name: str = 'tom', age: int = 10) -> 'Person':
        """初始化
        """
        self.name = name
        self.age = age

    def say_hello(self) -> str:
        """返回打招呼信息
        """
        return f"Hello My name is {self.name} ."


if __name__ == "__main__":
    # 因为在模块在被 import 的时候 __name__ 直接等于 模块名 不等于 “__main__” ，所以在作为模块被导入时并不会执行测试用例
    # 如果想执行测试用例直接执行模块就行
    import doctest
    doctest.testmod()
```

---
