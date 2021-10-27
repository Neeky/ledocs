## 背景
最近有幸看到大神 [Aymeric Augustin](https://myks.org/en/) 的一个项目 [websockets](https://github.com/aaugustin/websockets) ，其中用到了 python-3.7 新加的标准库模块 [dataclassses](https://www.python.org/dev/peps/pep-0557/) ，它能让 Python 这本就简洁的语法再简化不少。

![sqlpy](static/2021-02/dimitry-anikin-NN3M8sHfsMU-unsplash.jpg)

---

## 案例一 简化类的定义
以前要是我们定义一个 `Person` 类它可能长的像这样。
```python
#!/usr/bin/evn python3

class Person(object):
    def __init__(self,first_name,last_name,age):
        """
        Parameters
        ----------
        first_name: str
            名字
        last_name: str
            姓氏
        age: int
            年龄
        """
        self.first_name = first_name
        self.last_name = last_name
        self.age = age

    def full_name(self):
        """返回实例对象的全名
        Return
        ------
        str
        """
        return self.last_name + self.first_name

    def __str__(self):
        return f"{self.__class__.__name__}(first_name='{self.first_name}',last_name='{self.last_name}',age={self.age})"

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    print(zhang_san.full_name())
    print(zhang_san)

```
在使用 dataclasses 之后一切都变得简单了。
```python
#!/usr/bin/evn python3

from dataclasses import dataclass

@dataclass
class Person(object):
    first_name: str
    last_name: str
    age: int

    def full_name(self):
        """返回实例对象的全名
        Return
        ------
        str
        """
        return self.last_name + self.first_name

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    print(zhang_san.full_name())
    print(zhang_san)
```
运行效果是一样的。
```bash
python3 main.py

张三
Person(first_name='三', last_name='张', age=18)
```
---

## 案例二 `__init__` 钩子问题
上面的例子中还留了一个尾巴，`full_name` 是一个名词，但是在代码里表现的出来的是一个方法，这个要求客户端代码要执行函数调用才能拿到值。我们现在要把这个函数调用去掉，老的代码我们可以这样写。

方法一。
```python
class Person(object):
    # 省略其它代码

    # 用属性语法
    @property
    def full_name(self):
        """返回实例对象的全名
        Return
        ------
        str
        """
        return self.last_name + self.first_name
```
方法二 。
```python
class Person(object):
    """
    """
    def __init__(self,first_name,last_name,age):
        """
        Parameters
        ----------
        first_name: str
            名字
        last_name: str
            姓氏
        age: int
            年龄
        """

        self.first_name = first_name
        self.last_name = last_name
        self.age = age
        # 对于这种相当简单的逻辑我们直接写在 __init__ 钩子里也没有问题
        # 直接搞成字段
        self.full_name = self.last_name + self.first_name
```
现在的问题是 dataclasses 接管了我们的 `__init__` ，我们不能在这里写代码了；事实上他也考虑到了这些事，为我们提供了 `__post_init__` 这个钩子 。
```python
#!/usr/bin/evn python3

from dataclasses import asdict, dataclass


@dataclass
class Person(object):
    first_name: str
    last_name: str
    age: int

    def __post_init__(self):
        self.full_name = self.last_name + self.first_name

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    print(zhang_san.full_name)
```
运行效果如下。
```bash
python3 main.py

张三
```
---

## 案例三 简化对象转字典
之前我们要把 Python 对象转化字典少不了要写代码，每当对象添加新属性的时候我们的代码可能还要跟着变。
```python
class Person(object):
    """
    """
    #
    # 省略其它代码 
    #

    def asdict(self):
        """返回对象的字典形式
        Return
        ------
        dict
        """
        return {
            'first_name':self.first_name,
            'last_name': self.last_name,
            'age': self.age
        }

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    print(zhang_san.asdict())
```
如果是使用 dataclasses 上面这些转换的逻辑我们都不用写了,只要加一行 `from dataclasses import asdict` 功能就实现了。
```python
#!/usr/bin/evn python3

from dataclasses import asdict, dataclass


@dataclass
class Person(object):
    first_name: str
    last_name: str
    age: int

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    print(asdict(zhang_san))
```
实现上 dataclasses 还有一个把对象转换成元组的方法`astuple` ，用法上和 asdict 是一样的。

---

## 案例四 冻结对象
如果我们想让 Person 类的实例一旦创建出来之后就是只读的，以前的写法我们确实要多加几行代码才能完成。
```python
class Person(object):
    """
    """
    def __init__(self,first_name,last_name,age):
        """
        Parameters
        ----------
        first_name: str
            名字
        last_name: str
            姓氏
        age: int
            年龄
        """

        self.first_name = first_name
        self.last_name = last_name
        self.age = age
        # 第一步我们要在初始化完成之后打上标记
        self._is_inited = True

    # 第二步 检查对于非初始化赋值的情况，我们要报错。
    def __setattr__(self,attr,value):
        if '_is_inited' in self.__dict__:
            raise RuntimeError("read only object .")
        else:
            object.__setattr__(self,attr,value)

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    zhang_san.first_name="四"  
```
运行时效果如下。
```bash
python3 main.py
Traceback (most recent call last):
  File "/private/tmp/pys/main.py", line 82, in <module>
    zhang_san.first_name="四"
  File "/private/tmp/pys/main.py", line 72, in __setattr__
    raise RuntimeError("read only object .")
RuntimeError: read only object .
```
如果是使用 dataclasses 的话这一切都变的简单了，只要给 `dataclass` 装饰器加上一个冻结的参数就行了。
```python
@dataclass(frozen=True)
class Person(object):
    first_name: str
    last_name: str
    age: int

if __name__ == "__main__":
    zhang_san = Person("三","张",18)
    zhang_san.first_name="四"
    print(zhang_san)
```
运行效果如下。
```bash
python3 main.py
Traceback (most recent call last):
  File "/private/tmp/pys/main.py", line 78, in <module>
    zhang_san.first_name="四"
  File "<string>", line 4, in __setattr__
dataclasses.FrozenInstanceError: cannot assign to field 'first_name'
```
---


## 更多好用特性
关于 dataclasses 还有其它的内容，可以参考标准库和PEP。

1、[标准库](https://docs.python.org/3.8/library/dataclasses.html)

2、[PEP 0557](https://www.python.org/dev/peps/pep-0557/)


---

