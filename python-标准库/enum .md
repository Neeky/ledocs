## Python 自定义枚举存在的问题
要实现自定义枚举比较简单，通常的做法就是定义几个全局的“只读”变量。
```python
#!/usr/bin/env python3

# 三原色
RED = 1
GREEN = 2
BLUE = 4

if __name__ == "__main__":
    print(f"RED {RED}")
    print(f"GREEN {GREEN}")
    print(f"BLUE {BLUE}")
```

运行效果。

```bash
python3 main.py 

RED 1
GREEN 2
BLUE 4
```
可以看到这种传统的实现方式在 Python 中还是比较脆弱的，因为这里面我们约定“大写”的变更名不要去更新它的值，但是约定并不能保证没有人去改它。

```python
#!/usr/bin/env python3

# 三原色
RED = 1
GREEN = 2
BLUE = 4

if __name__ == "__main__":
    # 以大写命名的变量并不是只读的
    RED = 100
    GREEN = 200
    BLUE = 400

    print(f"RED {RED}")
    print(f"GREEN {GREEN}")
    print(f"BLUE {BLUE}")

```

运行效果。

```bash
python3 main.py 

RED 100
GREEN 200
BLUE 400
```

![sqlpy.com](static/2020-22/python-sqlpy.jpg)

google-adsense

---


## enum 真正的只读枚举
标准库 enum 中定义的枚举类型是真正只读的，并且输出格式也程序员也比较友好。
```python
#!/usr/bin/env python3

import enum

# 三原色
class Colors(enum.Enum):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # enum 中定义的输出格式一更加友好
    print(Colors.red)

    # 更新枚举值是会报错
    Colors.red = 100

```

运行效果。

```bash
Colors.red
Traceback (most recent call last):
  File "main.py", line 15, in <module>
    Colors.red = 100
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/enum.py", line 378, in __setattr__
    raise AttributeError('Cannot reassign members.')
AttributeError: Cannot reassign members.
```

google-adsense

---

## 比较与迭代
前文说到怎么通过 `enum` 标准库来定义自己的枚举类型，现在来看一下枚举类型还支持哪些操作。

1、每一个枚举项都包含两个属性 `name`,`value` 前者用于引用的枚举项的名字，后者是枚举的值。
```python
#!/usr/bin/env python3

import enum

# 三原色


class Colors(enum.Enum):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # 打印枚举项的类型
    print(f"type of Colors.red = {type(Colors.red)}")

    # 打印枚举项
    print(f"str present of Colors.red = {Colors.red} ")

    # 打印打印枚举项的 name 和 value 属性
    print(
        f"Colors.red.name = {Colors.red.name}  Colors.red.value = {Colors.red.value}")

```

运行效果。

```bash
python3 main.py 
type of Colors.red = <enum 'Colors'>
str present of Colors.red = Colors.red 
Colors.red.name = red  Colors.red.value = 1
```

---

2、有了前面对枚举项的了解之后我们就可以看比较操作了。比较操作又可以细分成两个 2.1、同一性比较，通过这个比较可以知道是不同一个枚举项。 2.2、相等性比较，通过这个比较可以知道是不是枚举项的哪个值。

```python
#!/usr/bin/env python3

import enum

# 三原色


class Colors(enum.Enum):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # 同一性比较 is
    # 返回 True
    print(Colors.red is Colors.red)

    # 相等性比较
    # 返回 False，Colors.red 并不是一个整数，不能直接这样比，而是比较 Value 值
    print(Colors.red == 1)

    # 返回 True
    print(Colors.red.value == 1)
```

运行结果。

```bash
python3 main.py 

True
False
True
```
---

3、迭代
```python
#!/usr/bin/env python3

import enum

# 三原色


class Colors(enum.Enum):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # 支持迭代
    for item in Colors:
        print(item)
```

运行效果。

```bash
python3 main.py 

Colors.red
Colors.green
Colors.blue
```

google-adsense

---


## IntEnum
如果想枚举项直接以 Int 方式暴露出来的话，就可以使用 `IntEnum` 来实现，也就是我最常用的一个场景。
```python
#!/usr/bin/env python3

import enum

# 三原色


class Colors(enum.IntEnum):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # 因为继承自 IntEnum 所以这个时候 Colors.red 就是对应的整数值
    # 打印 True
    print(Colors.red == 1)
    print(Colors.red | Colors.green | Colors.blue)
```

运行效果。

```bash
python3 main.py

True
7
```

---

## IntFlag
这个也是一个比较常用的实现，它是为了按位操作而专门设计的，虽然功能上 IntEnum 也能实现，但是 IntFlag 在打印上更加友好。
```python
#!/usr/bin/env python3

import enum

# 三原色


class Colors(enum.IntFlag):
    red = 1
    green = 2
    blue = 4


if __name__ == "__main__":
    # 
    print(Colors.red == 1)
    print(Colors.red | Colors.green | Colors.blue)
```

运行效果。

```bash
python3 main.py 

True
Colors.blue|green|red
```

---

## 官方文档
[enum 官方文档](https://docs.python.org/3.8/library/enum.html) 。

---