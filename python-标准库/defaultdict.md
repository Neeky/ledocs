## defaultdict 要解决的问题
要知道 defaultdict 要解决什么问题，不防来看一下没有 defaultdict 会发生什么问题。
```python
In [1]: d = dict(a = 1)                                                         

In [2]: d                                                                       
Out[2]: {'a': 1}

In [3]: d['a']                                                                  
Out[3]: 1

In [4]: d['b']                                                                  
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
<ipython-input-4-dee353b142fb> in <module>
----> 1 d['b']

KeyError: 'b'
```
也就是说对于一个普通的字典当索引一个不存在的键时会报 `KeyError` 异常。

![sqlpy](static/2020-24/sqlpy-defaultdict.jpg)


---


## 使用 defaultdict
如果想在键不存在的情况下返回一个默认值，而不是报错的话可以这样。
```python
In [1]: from collections import defaultdict                                     

In [2]: def defaults(): 
   ...:     return 'sqlpy.com' 
   ...:                                                                         

In [3]: d = defaultdict(defaults,a=1)                                           

In [4]: d['a']                                                                  
Out[4]: 1

In [5]: d['b']                                                                  
Out[5]: 'sqlpy.com'
```
可以看出使用 defaultdict 的关键就是其构造函数的第一个参数传入用于产生默认值的函数，注意这个函数是不接收参数的。通常来说这个函数比较短，更多的情况下是使用 lambda 表达式来定义。

```python
In [6]: d = defaultdict(lambda:'sqlpy.com',a=1)                                 

In [7]: d                                                                       
Out[7]: defaultdict(<function __main__.<lambda>()>, {'a': 1})

In [8]: d['a']                                                                  
Out[8]: 1

In [9]: d['b']                                                                  
Out[9]: 'sqlpy.com'
```

google-adsense

---

## 实现自己的 DefaultDict 类
1、根据官方文档来看 defaultdict 是 dict 的一个子类，下面的代码也可以看的出这个继承关系。
```python
In [1]: from collections import defaultdict                                     

In [2]: defaultdict in dict.__subclasses__()                                    
Out[2]: True
```
2、当键不存在时 dict 会去调用 `__missing__` 这个魔术方法，如果我们要实现自己的默认值字典只要重写这个方法就行。
```python
class DefaultDict(dict):
    """实现一个默认值字典，当给定的键不存在时，直接返回 'sqlpy.com'
    """

    def __missing__(self, k):
        return 'sqlpy.com'


d = DefaultDict(a=100)
print(d['a'])
print(d['b'])

```
运行效果如下。
```python
python3 my-default-dict.py 
100
sqlpy.com
```
---


## defaultdict 的替代方案
defaultdict 不怎么出现在代码中的一个原因是 `dict.get` 函数支持指定默认值，在一些不一定要索引的地方就都用的它。
```python
In [1]: d = dict(a = 1)                                                         

In [2]: d.get('a')                                                              
Out[2]: 1

In [3]: d.get('b','sqlpy.com')                                                  
Out[3]: 'sqlpy.com'
```
dict.get 的第一个参数是键，第二个参数是当键不存在时默认的返回值。

---