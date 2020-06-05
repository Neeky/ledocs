## ChainMap 要解决的问题
现在我们有两个字典 users 和 customers ，两个字典的格式是一样的，其中的键代表姓名，值代表年龄。
```python
    users = {
        'tim': 20,
        'tom': 19,
    }

    customers = {
        'bob': 100,
        'tim': 200
    }

```
现在我们以这两个字典为数据库，根据姓名查询年龄。如果给定的姓名没有被记录就返回 None，有记录就返回对应的年龄，如果两个字典都有记录就以 customers 的为准，也就是说在 customers 中的内容有更高的优先级。

![sqlpy.com](static/2020-22/python-sqlpy.jpg)

---

## 基于函数的解决方案
逻辑并不复杂，用一个函数就能解决。
```python
#!/usr/bin/env python3


def query_proxy(name, *args):
    """ 
    """
    for d in args:
        if name in d:
            return d[name]
    else:
        return None


if __name__ == "__main__":
    users = {
        'tim': 20,
        'tom': 19,
    }

    customers = {
        'bob': 100,
        'tim': 200
    }

    tom_age = query_proxy('tom', customers, users)
    print(f"tom's age {tom_age}")  # 19

    tim_age = query_proxy('tim', customers, users)
    print(f"tim's age {tim_age}")  # 200
```

运行效果。

```
python3 main.py 
tom's age 19
tim's age 200
```

可以看到 `query_proxy` 的实现是按次序检查各个字典，所以谁有更高的优先级，在调用时它就应该放在更前面的位置，这也就是调用时 customers 在 users 之前的原因。

google-adsense

---

## 基于 ChainMap 的解决方案
由于 `ChainMap` 已经为我们实现了上面的逻辑，所以我们直接用就好，不是特别有必要自己造轮子了。下面来看一下使用 ChainMap 代码要怎么写。

```python
#!/usr/bin/env python3

from collections import ChainMap

if __name__ == "__main__":
    users = {
        'tim': 20,
        'tom': 19,
    }

    customers = {
        'bob': 100,
        'tim': 200
    }

    # 使用 ChainMap 我们不用自己定义函数了
    # 和我们自己定义函数一样，ChainMap 也是谁优先谁就放在前面
    cm = ChainMap(customers, users)

    tom_age = cm['tom']
    print(f"tom's age {tom_age}")  # 19

    tim_age = cm['tim']
    print(f"tim's age {tim_age}")  # 200

```

运行效果。

```
python3 main.py 

tom's age 19
tim's age 200
```
可以看到 ChainMap 把多个字典链接成了一个字典给我们查询用。

---

## ChainMap 的高级用法
如果单单只解决上面的这个问题，那么 ChainMap 就没有存在的必要了，ChainMap 能进标准库的一个重要原因在于它可以写。
```python
#!/usr/bin/env python3

from collections import ChainMap

if __name__ == "__main__":
    users = {
        'tim': 20,
        'tom': 19,
    }

    customers = {
        'bob': 100,
        'tim': 200
    }

    # 向 ChainMap 对象添加新的值
    cm = ChainMap(customers, users)
    cm['neeky'] = 10000
    print(f"users {users}")
    print(f"customers {customers}")

```

运行效果。

```bash
python3 main.py

users {'tim': 20, 'tom': 19}
customers {'bob': 100, 'tim': 200, 'neeky': 10000}
```

可以看到新的键 `neeky` 已经被写进去了。

google-adsense

---

除了更新键，更新的数据源还可以是一个新的字典，这个时候我们要使用到 `new_child` 方法。

```python
#!/usr/bin/env python3

from collections import ChainMap

if __name__ == "__main__":
    users = {
        'tim': 20,
        'tom': 19,
    }

    customers = {
        'bob': 100,
        'tim': 200
    }

    # 向 ChainMap 对象添加新的值
    cm = ChainMap(customers, users)
    others = {
        'neeky': 10000
    }
    # 返回新的 ChainMap 对象
    cm = cm.new_child(others)
    print(cm)
```

运行效果。

```bash
python3 main.py 
ChainMap({'neeky': 10000}, {'bob': 100, 'tim': 200}, {'tim': 20, 'tom': 19})
```

---


## 经典场景
`ChainMap` 的一个典型用例是处理命令行参数与默认参数，也就说执行命令时传递的参数有更高的优先级。如 mysql 命令，在没有指定 --port 的情况下它默认取 3306 ,如果有指定就使用指定的值。这不正好是两个字典的吗？
```python
#!/usr/bin/env python3

from collections import ChainMap
import argparse

if __name__ == "__main__":
    parser = argparse.ArgumentParser('moke-mysql')
    parser.add_argument('--port', default=3306)
    args = parser.parse_args()

    defaults = {'port': 3306}
    cmd_args = {'port': args.port}

    configs = ChainMap(cmd_args, defaults)

    print(f"using port {configs['port']}")

```
运行效果。
```bash
python3 main.py
using port 3306

python3 main.py --port=3333
using port 3333
```
这样就简简单单的实现了命令行参数优先。

---

## 官方文档
[ChainMap 官方文档](https://docs.python.org/3.8/library/collections.html#collections.ChainMap) 。