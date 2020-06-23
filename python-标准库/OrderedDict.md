## OrderedDict 要解决的问题
OrderedDict 能记键的写入次序，它结合了列表和字典两大数据结构的优势，基于它可以方便的实现一些算法，如 LRU 缓存淘汰算法。

![sqlpy](static/2020-26/sqlpy-orderedict.jpg)

---

## 创建 OrderedDict
和其它的字典类型一样，直接调用类就可以创建 OrderedDict 的实例。
```python
In [1]: from collections import OrderedDict                                     

In [2]: o = OrderedDict()                                                       

In [3]: o['a']=1                                                                

In [4]: o['b']=2                                                                

In [5]: o.popitem(last=False)                                                   
Out[5]: ('a', 1)

In [6]: o.popitem(last=False)                                                   
Out[6]: ('b', 2)
```

google-adsense

---


## popitem 方法
popitem 方法的原型如下 `popitem(last:bool=True) -> object`，默认返回最后写入到字典的键值对，如果想返回最先写入的值，传入参数 False 就行。
```python
In [1]: from collections import OrderedDict                                     

In [2]: orders = OrderedDict()                                                  

In [3]: orders['a'] = 'hello'                                                   

In [4]: orders['b'] = 'world'                                                   

In [5]: orders.popitem()                                                        
Out[5]: ('b', 'world')
```

---

## move_to_end 方法
move_to_end 方法的原型如下 `move_to_end(key:hashable) -> object`。
```python
In [1]: from collections import OrderedDict                                     

In [2]: orders = OrderedDict()                                                  

In [3]: orders['a'] = 'hello'                                                   

In [4]: orders['b'] = 'world'                                                   

In [5]: orders                                                                  
Out[5]: OrderedDict([('a', 'hello'), ('b', 'world')])

In [6]: orders.move_to_end('a')                                                 

In [7]: orders                                                                  
Out[7]: OrderedDict([('b', 'world'), ('a', 'hello')])
```

google-adsense

---


## LRU 缓存淘汰算法
基于 OrderedDict 实现一个缓存淘汰算法。
```python
from collections import OrderedDict


class LRUCaching(OrderedDict):
    """实现 LUR 缓存淘汰算法.
    """

    def __init__(self, size: int = 7, *args, **kwargs):
        self.size = size
        OrderedDict.__init__(self, * args, **kwargs)

    def __getitem__(self, k: str) -> object:
        """每一次访问都把它移动后最后
        """
        # 第一步 索引出值
        v = OrderedDict.__getitem__(self, k)

        # 第二步 移动到最后面
        self.move_to_end(k)

        return v

    def __setitem__(self, k: str, v: object) -> object:
        """
        """
        # 第一步 如果键存在就先移动到最后面
        if k in self:
            self.move_to_end(k)

        # 第二步 更新键值
        OrderedDict.__setitem__(self, k, v)

        # 第三步 如果已经大于 LRU 缓存的大小就删除最老的数据项
        if len(self) > self.size:
            oldest = next(iter(self))
            del self[oldest]

def main():
    lru = LRUCaching(size=3)
    lru['a'] = 1
    lru['b'] = 2
    lru['c'] = 3
    print(lru)

    lru['d'] = 4
    print(lru)


if __name__ == "__main__":
    main()
```
运行效果如下。
```python
python3 main.py

LRUCaching([('a', 1), ('b', 2), ('c', 3)])
LRUCaching([('b', 2), ('c', 3), ('d', 4)])
```

---
