## 概要
python-3.8 引入了两个新的语法，一个是赋值表达式，另一个是仅位置参数；下面一个个的来说一下它们分别是什么解决什么问题。

![python-3.8-new-syntax.png](static/2020-12/python-3.8-new-syntax.png)

google-adsense

---

## 赋值表达式
假设我们要检测一个数组的长度，当它的长度超过 3 时就报错。
```python
#!/usr/bin/env python3

lst = [1, 2, 3, 4]

if len(lst) > 3:
    print(f"lst 太长了，最多只能 3 个，但是现在有了 {len(lst)}")

```
功能上是实现了，但是这个段代码也有一定的问题，那就是`len(lst)`被调用了两次，现在我打算在不引入新语法的情况下把`len`调用的次数降到一次。
```python
#!/usr/bin/env python3

lst = [1, 2, 3, 4]
lst_len = len(lst)

if lst_len > 3:
    print(f"lst 太长了，最多只能 3 个，但是现在有了 {lst_len}")
```
现在用新语法来表达看有多么的优雅。
```python
#!/usr/bin/env python3

lst = [1, 2, 3, 4]

if (lst_len := len(lst)) > 3:
    print(f"lst 太长了，最多只能 3 个，但是现在有了 {lst_len}")
```
新语法存在的一些问题 1、由于是新语法自动格式化代码的工具还没有跟上，会格式化错误` (lst_len: = len(lst)) ` “:”应该和“=”在一起的，但是工具识别错误。

---


## 仅位置参数
对参数的传方式进行了控制，模拟了已经存在于 Python 项目的 C 语言风格；人话是不可能说清楚的，看代码吧。定义一个接收四个参数的加法函数，那下面三种调用方式都正确。
```python
#!/usr/bin/env python3


def add_fun001(a, b, c=0,d=0):
    return a + b + c + d


print(add_fun001(1, 1))
print(add_fun001(1, 1, 0,0))
print(add_fun001(1, 1, c=0,d=0))
```
仅位置参数为限制参数的传递方式提供了可能，下面我们通过仅关键写参数来限制住第三种传递，也就是说让 `c` 只支持位置传递与关键字传递，`d`只支持关键字传递呢？
```python
#!/usr/bin/env python3

def add_fun001(a, b, / , c=0, *, d=0):
    return a + b + c

print(add_fun001(1, 1)) # 可以成功，c 和 d 都使用默认值 0
print(add_fun001(2, 2, 2, d=2)) # 可以成功，c 可以支持位置传递
print(add_fun001(3, 3, 3, 3))   # 报错，d 只支持关键字的方式传递
```
执行效果
```bash
python3 main.py 
2
6
Traceback (most recent call last):
  File "main.py", line 10, in <module>
    print(add_fun001(3, 3, 3, 3))
TypeError: add_fun001() takes from 2 to 3 positional arguments but 4 were given

```

",/" 后面的参数会支持“位置传递”和“关键字传递”

",*" 后面的参数只支持“关键字传递”

---



