## Python-3.10 新特性

Python 一直以来都没有 `switch case` 语句，使得我们只能用字典来模拟，令人高兴的是 Python-3.10  语法上有了原生的支持。

![is-vs-equall](static/2021-01/match-case.jpg)

---

## 例子

假设有这么一个需求吧，我们要把 1 转换成 `ONE` ，0 转换成 `ZERO` 其它的任何东西都转化为 `OTHERS` 。

---

## 最原始写法
没有 `switch case` 语句，最原始的方法就是使用 `if else` 来模拟。
```python
#!/usr/bin/env python3


def number_to_string(number:int):
    """
    把数据转换成字符串
    """
    if number ==1:
        return "ONE"
    elif number == 0:
        return "ZERO"
    else:
        return "OTHERS"

if __name__ == "__main__":
    s = number_to_string(100)
    print(s)
```
---

## 字典写法
如果情况比较多的话上面写法的 `if else` 语句就会比较多，为了尽可能的简化代码，我们可以使用字典来辅助实现。
```python
#!/usr/bin/env python3

num_to_str_mapping = {
    1:"ONE",
    0:"ZERO"
}

def number_to_string(number:int):
    """
    把数据转换成字符串
    """
    return num_to_str_mapping.get(number,"OTHERS")

if __name__ == "__main__":
    s = number_to_string(100)
    print(s)
```

---

## python-3.10 写法
新的语法和 `switch case` 的写法差不多，也就是说它简洁不到哪里去了，但重要的是 Python 支持了新的语法。下面看用新的语法怎么写。
```python
#!/usr/bin/env python3


def number_to_string(number:int):
    match number:
        case 1:
            return "ONE"
        case 0:
            return "ZERO"
        case _:
            return "OTHERS"

if __name__ == "__main__":
    s = number_to_string(100)
    print(s)
```

---


## 评介

也许是我习惯了字典的写法吧，并没有感受到新语法对我有多大的吸引力。 详细信息可以参考官方文档 [结构化模式匹配](https://docs.python.org/3.10/whatsnew/3.10.html#pep-634-structural-pattern-matching) 。

---