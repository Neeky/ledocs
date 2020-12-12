## 概要
对于自定义类型的实例如何优雅的转化为字典？假设我们定义了如下的 Person 类
```python
class Person(object):
    """
    """

    name = ''
    age = 16

    def __init__(self,name:str='tom',age:int=16):
        self.name = name
        self.age = age

    def say_hello(self):
        """
        """
        print(f"Hello my name is {self.name}.")

tom = Person(name="tom",age=18)
```
如何优雅的把 tom 实例转化为下面这样的字典呢？
```python
{
    'name': 'tom', 
    'age': 18
}
```
![sqlpy](static/2020-46/to_dict.jpg)

---


## 土办法
最直接的办法就是给类加上一个方法，由这个方法来负责转换。
```python
class Person(object):
    """
    """

    name = ''
    age = 16

    def __init__(self,name:str='tom',age:int=16):
        self.name = name
        self.age = age

    def say_hello(self):
        """
        """
        print(f"Hello my name is {self.name}.")

    def to_dict(self):
        return {
            'name':self.name,
            'age': self.age
        }
```
这个办法的缺点就是使用方要记住 Person 类的 api `to_dict`，无形中加大了使用方的学习成本。
```python
p = Person(name="tom",age=18)
p.say_hello()

d = p.to_dict()
```

---

## 好的 API
好的 API 应该让人用起来像是在使用内置对象，也就说用起来要像下面这段代码一样自然。
```python
p = Person(name="tom",age=18)
p.say_hello()

d = dict(p)
```
内置函数 dict 支持把可迭代的对象转换成字典，也就是说我们只要让自己定义的类支持迭代就行了。

---


## 实现
添加 `__iter__` 和 `__next__` 方法让对象支持迭代。
```python
#!/usr/bin/evn python3

class Person(object):
    """
    """

    name = ''
    age = 16

    def __init__(self,name:str='tom',age:int=16):
        self.name = name
        self.age = age

    def say_hello(self):
        """
        """
        print(f"Hello my name is {self.name}.")

    def __iter__(self):
        return next(self)

    def __next__(self):
        yield ('name',self.name)
        yield ('age',self.age)


if __name__ == "__main__":
    p = Person(name="tom",age=18)
    p.say_hello()
    d = dict(p)
    print(d)

```
运行效果如下。
```bash
python3 main.py

Hello my name is tom.
{'name': 'tom', 'age': 18}
```

---
