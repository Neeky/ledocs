## 概要

今天晚上在看一下开源的 Python 项目的源代码，看到它把一个函数当“键”来向字典中保存值；通常我都是用字符串来做键，第一次看到有人用函数做键的。

```python
In [1]: def main(): 
   ...:     return 0 
   ...:                                                                         

In [2]: d = {main:None}                                                         

In [3]: d                                                                       
Out[3]: {<function __main__.main()>: None}
```
![sqlpy](static/2020-30/sqlpy-hash.jpg)

---

## 原理
一个对象可以做字典的键就只有一个条件哪就是这个对象是可以 hash 的，并且在对象的整个生命周期中这个对象的 hash 值不会变。

```python
In [4]: hash(main)                                                              
Out[4]: 8782083164802

In [5]: hash(main)                                                              
Out[5]: 8782083164802
```

哈哈哈，真的不会变，长见识了！

---

