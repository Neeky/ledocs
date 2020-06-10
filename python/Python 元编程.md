## Python 元编程要解决的问题
python 元编程目的是为了让代码更加的灵活，实现的手段就是用代码生成代码，用代码修改代码。

---

## 元编程的基础
第一条、在 Python 的世界里一切都是对象，对象是类的实例，而类是元类的实例，元类也是类。

第二条、创建实例由类的 `__new__` 方法完成，创建完成之后交由 `__init__` 初始化。

为了一步步加大难度，在这里我们先会用 `__new__` 修改实例的创建过程(实现单例模式)，然后再使用 `__new__` 为类动态的添加属性。

---

## 用元编程实现单例
由于“实例”是由类的 `__new__` 方法创建的，如果 `__new__` 的逻辑只有在第一次运行时才会创建实例，其它情况都是直接返回第一次运行时所创建的实例，这样的话不就完成单例模式了吗？

```python
#!/usr/bin/evn python3

class Singleton(object):

    # 单例模式的那个实例会交由这个变量名引用
    _instance = None

    def __new__(cls, *args):
        """
        """
        if cls._instance is None:
            # 只有在第一次运行里 cls._instance 才会是 None
            print("Singleton.__new__ function create instance.")
            cls._instance = super().__new__(cls)

        return cls._instance


def main():
    s0 = Singleton()
    s1 = Singleton()

    if id(s0) == id(s1) == id(Singleton._instance):
        print("is the same")


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 

Singleton.__new__ function create instance.
is the same
```

可以看到 ·Singleton.__new__ function create instance.` 只打印了一次，说明只创建了一次实例，`id(s0) == id(s1) == id(Singleton._instance)` 三个对象的 id 相同也证明了这一点。

---

## 第一阶段总结
1、可以看到元编程并不难，只要理解了“实例”是由“类”的 `__new__` 方法创建的，这一个关键点就行了。至于是如何创建的这个就交给解释器自己解决吧，这也就是我们在创建实例是直接使用 `super().__new__(cls)` 的原因，套娃就行了。

2、上面的例子中我们的日志格式是 [类名].[方法名] [日志内容] 按这个格式打印日志是可以比较容易的找到有问题的函数，那后面我们自己写的类也都这样干吧。

---

## 用传统写法给类加上日志
假设我们有两个类一个是 Person,另一个是 Student 都要给他们的方法上加日志，在不用元类的时候可以这样实现。
```python
#!/usr/bin/evn python3

import logging


# 配置日志格式
lgr = logging.getLogger("sqlpy")
lgr.setLevel(logging.DEBUG)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(name)s %(message)s')
stream_handler.setFormatter(formatter)
lgr.addHandler(stream_handler)


class Person(object):
    # 日志对象
    logger = lgr.getChild("Person")
    # 名字
    name = ""

    def __init__(self, name="tom"):
        self.name = name

    def say_hello(self):
        logger = self.logger.getChild("say_hello")
        logger.info(f"My name is {self.name} .")


class Student(Person):
    # 日志对象
    logger = lgr.getChild("Student")
    # 专业名
    major = 0

    def __init__(self, name="tom", major=16):
        self.major = major
        super().__init__(name)

    def say_hello(self):
        logger = self.logger.getChild("say_hello")
        logger.info(f"My name is {self.name}. My major is {self.major} .")


def main():
    p = Person()
    p.say_hello()

    s = Student()
    s.say_hello()


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 

sqlpy.Person.say_hello My name is tom .
sqlpy.Student.say_hello My name is tom. My major is 16 .
```

---

## 传统写法存在的问题
传统写法的一个问题在于，要用日志功能就要 "显示的在类上加一个属性"，可不可以把这个去掉呢？我们没有必要一直重复下去。

想想“类”是元类的实例，我们只要在元类创建类时把 logger 加上去就行了，这样就一步到位了。

---

## 元类 type 
type 类是所有类的元类，也就是说默认情况下所有的类都由 type 创建，可以通过下面的代码验证。

```python
#!/usr/bin/evn python3

import logging


# 配置日志格式
lgr = logging.getLogger("sqlpy")
lgr.setLevel(logging.DEBUG)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(name)s %(message)s')
stream_handler.setFormatter(formatter)
lgr.addHandler(stream_handler)


class Person(object):
    # 日志对象
    logger = lgr.getChild("Person")
    # 名字
    name = ""

    def __init__(self, name="tom"):
        self.name = name

    def say_hello(self):
        logger = self.logger.getChild("say_hello")
        logger.info(f"My name is {self.name} .")


if __name__ == "__main__":
    p = Person()

    print(f"实例 p 由 {type(p)} 创建")
    print(f"实例 {Person} 由 {type(Person)} 创建")

```

运行效果。
```python
python3 main.py 
实例 p 由 <class '__main__.Person'> 创建
实例 <class '__main__.Person'> 由 <class 'type'> 创建
```

也就是说如果我们想在创建类的时候做一点其它的手脚，只要实现一个 type 的子类，并把我们的类标记成用这个子类创建，而不是用默认的 type 创建。

---

## 用元类实现为类添加日志对象
你应该已经想到了，就是重写一个 `type.__new__` 方法。
```python
#!/usr/bin/evn python3

import logging


# 配置日志格式
lgr = logging.getLogger("sqlpy")
lgr.setLevel(logging.DEBUG)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(name)s %(message)s')
stream_handler.setFormatter(formatter)
lgr.addHandler(stream_handler)


class LoggerMeta(type):
    """
    """
    def __new__(cls, classname, bases, attrs):
        """
        classname  要创建的类名
        bases      类的基类
        attrs      类的属性
        """
        # lgr.info(
        #    f"cls = {cls} classname = {classname} bases = {bases} attrs = {attrs}")

        # 添加日志对象
        attrs.update({'logger': lgr.getChild(classname)})
        return super().__new__(cls, classname, bases, attrs)


class Person(object, metaclass=LoggerMeta):
    # 名字
    name = ""

    def __init__(self, name="tom"):
        self.name = name

    def say_hello(self):
        logger = self.logger.getChild("say_hello")
        logger.info(f"My name is {self.name} .")


if __name__ == "__main__":
    p = Person()
    p.say_hello()
```

运行效果。

```bash
python3 main.py 
sqlpy.Person.say_hello My name is tom .
```

---

## 总结
元类算是 Python 的黑魔法，是通往代码定义代码的大门。

---