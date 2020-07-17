## 背景
Python 代码在运行的时候报了一个异常。
```python
In [1]: from collections import namedtuple                                                                                     

In [2]: Person = namedtuple("Person","age name")                                                                               

In [3]: p = Person({'name':'tom','age':16})                                                                                    
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-3-0f14cd889ec8> in <module>
----> 1 p = Person({'name':'tom','age':16})

TypeError: __new__() missing 1 required positional argument: 'name'
```

![sqlpy](static/2020-29/sqlpy-nametuple.jpg)

---


## 分析
虽然自以为 namedtuple 已经了非常熟悉了，但是这几天写代码还是大意了。从报错来看要传 name 关键字参数，这个就比较明确了，要对 dict 进行解包才行。
```python
In [4]: p = Person(**{'name':'tom','age':16})

In [5]: p.name                                                                                                                 
Out[5]: 'tom'

In [6]: p.age                                                                                                                  
Out[6]: 16
```

---

## 总结
本来不想记录这个事的，但是这已经是我第三次犯这样的错，事不过三呀！还是决定记一下加深一下印象。

---