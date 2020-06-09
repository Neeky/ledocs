## 背景
最近在看关于 Python 数据模型部分的文档，在看到 `__new__` 函数的调用时机，发现只用它来定义元类也是真是有点浪费了，可以让它多做点事嘛！比如让它来实现“单例模式”。

![sqlpy](static/2020-22/sqlpy-0609-a.jpg)


---

## \_\_new\_\_ 方法的调用时机
在创建类的实例时先调用 `__new__` 方法把实例创建出来，然后 `__new__` 把实例返回，接下来才是 `__init__` 初始化实例的属性。

要想实现单例，关键的问题就是实例只创建一次，`__init__` 已经晚了，但 `__new__` 的时机正好。

google-adsense

---

## Python 单例模式
下面就用 `__new__` 方法使用一个单例模式。
```python
#!/usr/bin/env python3


class Singleton(object):
    instance = None

    def __new__(cls, *args):
        """
        """
        if cls.instance is None:
            cls.instance = super().__new__(cls)
            cls.instance.message = "hello world"

        return cls.instance


def main():
    s0 = Singleton()
    s1 = Singleton()

    if id(s0) == id(s1):
        print("is the same")


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 

is the same
```
---


## \_\_new\_\_ 的注意事项
`__new__` 在 Python 中一个非常特殊的函数，它的第一个参数是“类”，但是他不需要 `@classmethod` 来修饰自己。从函数原型来看是这样的 `__new__(cls, *args)` 也就是说它没有关键字参数。


---


