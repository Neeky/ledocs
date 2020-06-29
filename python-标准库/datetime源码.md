## 背景
最近和标准库的 datetime 打交道比较多，也就有了想看一下它的源码的冲动。

![sqlpy](static/2020-27/sqlpy-datetime-source-code.jpg)

---

## 各个类之间的继承关系
整个模块的继承关系非常简单，如下图。

![sqlpy](static/2020-27/sqlpy-datetime-uml.jpg)

也就是说只有两组简单的继承关系，没有组合与混入。

---


## date 类的实现
构造函数(`__new__`)的大致逻辑。
```python
# 以下只是源码中的关键部分，并不完整。
class date:
    __slots__ = '_year', '_month', '_day', '_hashcode'

    def __new__(cls, year, month=None, day=None):
        self = object.__new__(cls)
        self._year = year
        self._month = month
        self._day = day
        self._hashcode = -1
        return self
    
    @classmethod
    def fromtimestamp(cls, t):
        "Construct a date from a POSIX timestamp (like time.time())."
        y, m, d, hh, mm, ss, weekday, jday, dst = _time.localtime(t)
        return cls(y, m, d)

    @property
    def year(self):
        """year (1-9999)"""
        return self._year
```
date 类的源码有三个亮点 1、使用了 `__slots__` 来节约内存  2、提供了静态构造函数 `fromtimestamp`  3、以属性的方式暴露内部数据。

---