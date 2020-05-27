## 问题
linux 的哲学就是把一个又一个短小精悍的命令组合起来用以完成工作，例如存在一个名单文件 names.txt 要对其中的行进行去重可以这样做。
```bash
# 文件内容如下
cat names.txt 
tom
tim
tom
jerry

# 数据去重
cat names.txt | sort | uniq
jerry
tim
tom
```

那如何让我们自己写的 Python 程序也支持管道 “|” 和文件重定向 "<" 呢？

---


## 原理
不管是从管道读取还是从文件重定向的数据流读取，本质上都是读的 `sys.stdin` 、下面程序会直接把读到的内容输出。
```python
#!/usr/bin/env python3

import sys
# 从 sys.stdin 读取内容并输出
print(sys.stdin.read())

```
可以看到三行代码就搞定了，事实上最关键的就只有一行 `print(sys.stdin.read())`，运行效果如下。
```bash
cat names.txt | python3 main.py 
tom
tim
tom
jerry

```

google-adsense

---

## 用最原始的方法实现去重
我们已经知道背后的原理了，先写一个去重程序练下手。虽然 Python 并不提倡什么都自己写，这里还是先过一下苦日子，后面才会知道 import 有多爽。
```python
#!/usr/bin/env python3

import sys


def main():
    # sys.stdin 中读到的所有内容会被整合成一个字符串
    content = sys.stdin.read()

    # 自己做切分，切成一行一行的数据
    lines = content.split('\n')

    # 把行保存到集合中，以实现去重的目的
    uset = set()
    for line in lines:
        if line:
            # line 不为空行
            uset.add(line)

    # 输出
    for item in uset:
        print(item)


if __name__ == "__main__":
    main()

```

运行效果。

```bash
cat names.txt | python3 main.py 
tim
jerry
tom
```
目的是实现了就是代码有点长。

---

## 最 pythonic 的写法
上面的写法不太像 Python 的风格，真正的 Python 风格是一个 import 就完成了一大半的工作。我直说了吧 标准库 `fileinput` 就是用来做这个的。这下整个世界都干净了。
```python
#!/usr/bin/env python3

import fileinput


def main():
    # 把行保存到集合中，以实现去重的目的
    uset = set()
    for line in fileinput.input():
        if line:
            # line 不为空行
            uset.add(line.strip())
    # 输出
    for item in uset:
        print(item)


if __name__ == "__main__":
    main()
```
运行效果。
```bash
cat names.txt | python3 main.py 
tim
jerry
tom
```

---

## fileinput 官方文档
官方文档 [fileinput](https://docs.python.org/3.8/library/fileinput.html) 。