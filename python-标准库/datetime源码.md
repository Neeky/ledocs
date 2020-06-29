## 背景
最近和标准库的 datetime 打交道比较多，也就有了想看一下它的源码的冲动。

![sqlpy](static/2020-27/sqlpy-datetime-source-code.jpg)

---

## 各个类之间的继承关系
整个模块的继承关系非常简单，如下图。

![sqlpy](static/2020-27/sqlpy-datetime-uml.jpg)

也就是说只有两组简单的继承关系，没有组合与混入。

google-adsense

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

## time 类的实现
time 类的实现模式和 date 类是一样的。
```python
# 以下只是源码中的关键部分，并不完整。
class time:
    __slots__ = '_hour', '_minute', '_second', '_microsecond', '_tzinfo', '_hashcode', '_fold'

    def __new__(cls, hour=0, minute=0, second=0, microsecond=0, tzinfo=None, *, fold=0):
        self = object.__new__(cls)
        self._hour = hour
        self._minute = minute
        self._second = second
        self._microsecond = microsecond
        self._tzinfo = tzinfo
        self._hashcode = -1
        self._fold = fold
        return self

    @classmethod
    def fromisoformat(cls, time_string):
        if not isinstance(time_string, str):
            raise TypeError('fromisoformat: argument must be str')

        try:
            return cls(*_parse_isoformat_time(time_string))
        except Exception:
            raise ValueError(f'Invalid isoformat string: {time_string!r}')

    @property
    def hour(self):
        """hour (0-23)"""
        return self._hour
```

google-adsense

---

## datetime 类的实现
```python
class datetime(date):
    __slots__ = date.__slots__ + time.__slots__

    def __new__(cls, year, month=None, day=None, hour=0, minute=0, second=0,
                microsecond=0, tzinfo=None, *, fold=0):
        self = object.__new__(cls)
        self._year = year
        self._month = month
        self._day = day
        self._hour = hour
        self._minute = minute
        self._second = second
        self._microsecond = microsecond
        self._tzinfo = tzinfo
        self._hashcode = -1
        self._fold = fold
        return self

    @property
    def hour(self):
        """hour (0-23)"""
        return self._hour

    @classmethod
    def now(cls, tz=None):
        t = _time.time()
        return cls.fromtimestamp(t, tz)

    def date(self):
        return date(self._year, self._month, self._day)

    def time(self):
        return time(self.hour, self.minute, self.second, self.microsecond, fold=self.fold)
```

---

## timedelta 类的实现
```python
class timedelta:
    __slots__ = '_days', '_seconds', '_microseconds', '_hashcode'

    def __new__(cls, days=0, seconds=0, microseconds=0,
                milliseconds=0, minutes=0, hours=0, weeks=0):
        self = object.__new__(cls)
        self._days = d
        self._seconds = s
        self._microseconds = us
        self._hashcode = -1
        return self

    @property
    def days(self):
        return self._days

    @property
    def seconds(self):
        return self._seconds

    @property
    def microseconds(self):
        return self._microseconds
```
---

## 总结
从 datetime 模块可以官方类中基本都不定义 `__init__` 方法，而是把大量的逻辑都放在了 `__new__` 方法上，就我现在(2020-06)的水平还看出这样做的高明之处。

另外一个特点是官方非常少的使用普通方法，转而大量的使用 `__xx__` 这样的魔术方法，使用官方的数据结构与 Python 数据模型有非常好的契合。

---
