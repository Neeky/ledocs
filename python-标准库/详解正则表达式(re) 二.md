## 概要
今天遇到一个非常有意思的问题，“re.match 和 re.search 都能解决问题的情况下，哪个更加写法在执行上的效率更加高呢？”

---

## 两者各自的使用场景 
re.math 在官方文档上有如下描述：
```bash
Try to apply the pattern at the start of the string, returning a match object, or None if no match was found.
```
re.search 在官方文档上有如下描述：
```bash
Scan through string looking for a match to the pattern, returning a match object, or None if no match was found
```
可以看到两者的使用场景是不一样的，re.match 用于测试给定的模式是不是出现在字符串的开头，re.search 用于测试字符串中是不是包含给定的模式。

看样子只要在正则表达式前加上一个`^`，就能让 re.search 完成 re.match 的功能了；但是问题在于标准库的开发者单独出一个 re.match 是不是有专门做了一些起优化呢？

google-adsense

---

## 性能比较
判断一个字符串是不是以"this"开头，这个用例来测试两个函数的效率。
```python
import re
from datetime import datetime


def using_match(s=""):
    p = "this"
    start_at = datetime.now()
    for i in range(800000):
        re.match(p, s)
    end_at = datetime.now()
    print(f"using match cost {end_at - start_at} (s)")


def using_search(s=""):
    p = "^this"
    start_at = datetime.now()
    for i in range(800000):
        re.search(p, s)
    end_at = datetime.now()
    print(f"using search cost {end_at - start_at} (s)")


def main():
    """
    """
    s = "this is a string."
    using_search(s)
    using_match(s)


main()

```
输出结果如下。
```bash
# 执行过许多次，这里只取其中的一次。
using search cost 0:00:00.781091 (s)
using match cost 0:00:00.773531 (s)
```

---

## 结论
经过多次的执行，比较耗时，得出的结论是“在比较是不是以给定的模式开头”的这个一个场景下 re.match 要优于 re.search。

|**函数**|**耗时**  |**执行次数**|
|---------|--------|----------|
|re.search| 0.78(s)|80w       |
|re.math  | 0.77(s)|80w       |

---

