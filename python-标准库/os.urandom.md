## os.urandom 
os.urandom 的原型是 `os.urandom(size:int) -> bytes` 返回值是一个完全随机的字节串。
```python
In [1]: import os                                                                                                              

In [2]: os.urandom(4)                                                                                                          
Out[2]: b'E"\x92w'

In [3]: os.urandom(4)                                                                                                          
Out[3]: b'P\xc6\xc5p'

In [4]: os.urandom(4)                                                                                                          
Out[4]: b'\xacs$\xa0'
```

![sqlpy](static/2020-26/sqlpy-urandom.jpg)

---

## os.urandom 的应用场景
1、加密算法的随机“盐值”。

---