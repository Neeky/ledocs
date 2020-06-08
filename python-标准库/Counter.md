## Counter 要解决的问题
Counter 类就和它的名字一样是用来计数的，并且能非常方便的给出排名靠前的 n 个。

比如说我现在有一个名单列表 `['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']` 我想找出出现次数最多的两个名字，这个就是典型的 Counter 类的使用场景了。

![sqlpy](static/2020-22/sqlpy-0608-b.jpg)

用 Counter 解决上面的问题。

```python
#!/usr/bin/env python3

from collections import Counter


def main():
    names = ['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']
    ctr = Counter(names)
    print(ctr.most_common(2))


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 
[('tom', 3), ('jerry', 2)]
```

---

## Counter 的学习方法
总的来看就包含两个部分 1、怎么创建出 Counter 对象，2、使用好 Counter 对象的方法。

google-adsense

---

## 创建 Counter 对象
有两个创建 Counter 对象的常用套路，第一个是基于列表创建对象，第二个是基于字典创建对象。

1、基于列表创建对象。
```python
#!/usr/bin/env python3

from collections import Counter


def main():
    names = ['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']
    ctr = Counter(names)
    print(ctr)
    print(ctr.most_common(2))


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 
Counter({'tom': 3, 'jerry': 2, 'bob': 1})
[('tom', 3), ('jerry', 2)]
```

---

2、基于字典创建 Counter 对象，这个场景有一个要注意的地方，字典的键就是你要统计的对象，字典的值就是它出现的次数。
```python
#!/usr/bin/env python3

from collections import Counter


def main():
    #names = ['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']
    ctr = Counter({'tom': 3, 'jerry': 2, 'bob': 1})
    print(ctr)
    print(ctr.most_common(2))


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 
Counter({'tom': 3, 'jerry': 2, 'bob': 1})
[('tom', 3), ('jerry', 2)]
```

google-adsense

---


## update 方法
Counter 对象的 update 方法用于更新计数器，这可能是最常用的一个方法了。
```python
#!/usr/bin/env python3

from collections import Counter


def main():
    #names = ['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']
    ctr = Counter({'tom': 3, 'jerry': 2, 'bob': 1})
    print(ctr)
    print(ctr.most_common(2))

    ctr.update({'bob': 10})  # 告诉 ctr 'bob' 又出现了 10 次。
    print(ctr)
    print(ctr.most_common(2))


if __name__ == "__main__":
    main()

```

运行效果。

```bash
python3 main.py 

Counter({'tom': 3, 'jerry': 2, 'bob': 1})
[('tom', 3), ('jerry', 2)]

Counter({'bob': 11, 'tom': 3, 'jerry': 2})
[('bob', 11), ('tom', 3)]
```

---

## most_common 方法

most_common 用于返回出现次数最多的那 n 项。
```python
#!/usr/bin/env python3

from collections import Counter


def main():
    names = ['tom', 'jerry', 'tom', 'bob', 'tom', 'jerry']
    ctr = Counter(names)
    print(ctr.most_common(2)) # 返回最常出现的 2 项


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 
[('tom', 3), ('jerry', 2)]
```

---

