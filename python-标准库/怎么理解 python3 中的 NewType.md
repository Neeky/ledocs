## 背景

python3 为了增强类型提示，在标准库中新加了一个 typing 库，其中 `NewType` 部分的例子最简单，但是也最能理解官方要表达什么。以至于成了我多年以来的一块心病，今天早上晨读突然就顿悟了。啪的一下很快啊！


![newtype](static/2021-01/new-type.jpg)

---

## 例子

这里以一个从 html 文档中抽取 title 值的函数做例子，先来看一下没有类型提示、文档注释的版本。
```python
from bs4 import BeautifulSoup

def get_title(document):
    soup = BeautifulSoup(document,"html5lib")
    return soup.title.text
```
这样的话使用这个函数的人就要看源代码才能明白函数的功能。


---

## 加文档注释

为了代码的使用方好过一点，决定还是加几行文档注释(numpy 风格)。
```python
def get_title(document):
    """
    返回 html 文档的 title 值
    
    document:
    --------
        html 文档字符串

    return:
    -------
        str
    """
    soup = BeautifulSoup(document,"html5lib")
    return soup.title.text
```

虽然注释是有了，但是新的问题是我们的注释比代码都长，感觉自己不是来写代码的是来写注释的。

---


## 普通类型提示
为了不要让我们的主要精力花在写注释上，决定换一种技术方案，那就是写类型提示。
```python
def get_title(document:str) -> str:
    soup = BeautifulSoup(document,"html5lib")
    return soup.title.text
```

这下是简了，除了告诉别人函数接收一个 str 返回一个 str 之外，其它的信息就太少了，不方便使用方理解函数的功能。

---

## 使用 NewType

上面的代码有一个毛病就是 `str` 的语义是不明确的，也就是说看不出为 document 是一个怎样的 str ？它有没有格式？如果有，格式是怎样的？为了把语义搞的更加明确我们可以用 `NewType` 。

```python
from typing import NewType

from bs4 import BeautifulSoup

Html = NewType("Html",str)

def get_title(document:Html) -> str:
    """
    返回 html 文档的 title 值
    """
    soup = BeautifulSoup(document,"html5lib")
    return soup.title.text
```

---

## NewType 背后的原理

从上面代码中我们可以看到 `Html` 这个对象是因为我们需要一个比 `str` 更加具体的描述才创造出来的。也就是说它没有什么功能性的需求在里面， `def f(x): return x` 就是没有任何功能的典型代表。

1、NewType 本身是函数
```python
In [1]: from typing import NewType

In [2]: type(NewType)
Out[2]: function
```

2、由 NewType 创建出来的对象是一个把输入参数直接返回的函数。
```python
In [1]: from typing import NewType

In [2]: Html = NewType("Html",str)

In [3]: s = "hello world"

In [4]: s == Html(s)
Out[4]: True

In [5]: def f(x): return x # Html 的源码和这个函数是一样的

In [6]: s == f(s)
Out[6]: True

In [7]: type(f)
Out[7]: function

In [8]: type(Html)
Out[8]: function

# 可以看到 Html 和 函数 f 的字节码是一样的。
In [9]: from dis import dis

In [10]: dis(f)
  1           0 LOAD_FAST                0 (x)
              2 RETURN_VALUE

In [11]: dis(Html)
1823           0 LOAD_FAST                0 (x)

```

---
