
## 背景
我们希望在调用函数或方法的时候能把相关的参数、函数或方法所在的层次结构也打印到日志里去，让我们先从一下简单的例子开始。

先看一下没有日志时的代码。
```python
class Person:
    def __init__(self,name):
        self.name = name

    def say(self):
        print(f"Hello this is  {self.name} .")


if __name__ == "__main__":
    p = Person("tom")
    p.say()

```

好现在我们用最原始的方式加上日志。

```python
import logging
logging.basicConfig(level=logging.DEBUG)

class Person:
    def __init__(self,name):
        logging.info(f"enter Person.__init__ with args self={self},name={name} .")
        self.name = name
        logging.info(f"leave Person.__init__ .")

    def say(self):
        logging.info(f"enter Person.say with args self={self} .")
        print(f"Hello this is  {self.name} .")
        logging.info(f"enter Person.say with args self={self} .")


if __name__ == "__main__":
    p = Person("tom")
    p.say()
```
运行效果。
```bash
python3 main.py

INFO:root:enter Person.__init__ with args self=<__main__.Person object at 0x7ff5f80a9400>,name=tom .
INFO:root:leave Person.__init__ .
INFO:root:enter Person.say with args self=<__main__.Person object at 0x7ff5f80a9400> .
Hello this is  tom .
INFO:root:enter Person.say with args self=<__main__.Person object at 0x7ff5f80a9400> .
```

![logger](static/2021-01/logger.jpg)

---


## 存在的问题
一个明显的问题就是通用的日志打印和业务逻辑混在一起，让代码的可阅读性变差；另一个就是代码层次结构是日志内容的一部分，不方便维护。

---

## 解决办法
把通用的日志打印逻辑和函数的层次结构实现成日志模块的一部分。为了达到这个目标我们约定所有所日志入口都是 logger。

1、对于函数的层次结构交由 logger 和它的子 logger 对象来表现。 2、实现日志打印功能的装饰器。
```python
import logging

from functools import wraps

root_logger = logging.getLogger("exsample")
root_logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s"))
root_logger.addHandler(stream_handler)

def logger_instance_method(fun):
    """
    方法的日志装饰器

    fun:
    ----
        要被装饰的实例方法
    returns:
    --------
        装饰之后的实例方法
    """
    @wraps(fun)
    def inner(*args,**kwargs):
        """
        """
        self,*_ = args
        self.logger.debug(f"enter bound method {self.__class__.__name__}.{fun.__name__} with args self = {self} args = {args} kwargs = {kwargs} .")
        result = fun(*args,**kwargs)
        self.logger.debug(f"leave bound method {self.__class__.__name__}.{fun.__name__} with args self = {self}.")
        return result
    return inner
```

运行效果。
```bash
python3 main.py

2021-06-20 15:04:15,838 - exsample.Person - MainThread - DEBUG - enter bound method Person.say with args self = Person[instance-id=140542001975744,name=tom] args = (Person[instance-id=140542001975744,name=tom],) kwargs = {} .
Hello this is  tom .
2021-06-20 15:04:15,839 - exsample.Person - MainThread - DEBUG - leave bound method Person.say with args self = Person[instance-id=140542001975744,name=tom].
```

---

## 进阶方案
当前的解决方案还有一个问题，就是对日志级别支持的不好，默认情况下都是打 debug ，下一步我们让装饰器支持参数，以解决这个问题。
```python
import logging

from functools import wraps

root_logger = logging.getLogger("exsample")
root_logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter("%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s"))
root_logger.addHandler(stream_handler)

def desc_instance_method(logger_level=logging.DEBUG):
    """
    实现实例方法的日志装饰器
    
    logger_level:
    ------------
        日志级别
    result:
    -------
        方法装饰器
    """
    def logger_instance_method(fun):
        """
        fun:
        ----
            要被装饰的实例方法
        returns:
        --------
            装饰之后的实例方法
        """
        @wraps(fun)
        def inner(*args,**kwargs):
            """
            """
            self,*_ = args
            self.logger.setLevel(logger_level)
            self.logger.debug(f"enter bound method {self.__class__.__name__}.{fun.__name__} with args self = {self} args = {args} kwargs = {kwargs} .")
            result = fun(*args,**kwargs)
            self.logger.debug(f"leave bound method {self.__class__.__name__}.{fun.__name__} with args self = {self}.")
            return result

        return inner

    return logger_instance_method

class Person:
    logger = root_logger.getChild("Person")

    def __init__(self,name):
        self.name = name

    def __str__(self):
        return f"{self.__class__.__name__}[instance-id={id(self)},name={self.name}]"

    __repr__ = __str__

    # INFO 级别不再打印通用的 DEBUG 日志。
    @desc_instance_method(logging.INFO)
    def say(self):
        print(f"Hello this is  {self.name} .")


if __name__ == "__main__":
    p = Person("tom")
    p.say()
```

运行效果(可以看到所有的 DEBUG 级别的日志就这样被关掉了)。
```python
python3 main.py

Hello this is  tom .
```

---




