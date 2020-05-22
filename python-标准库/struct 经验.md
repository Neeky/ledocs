## 背景
最近在使用 struct 遇到了一些比较坑的地方，问题非常的隐蔽，还是直接上代码吧。

```python
In [2]: struct.pack("B",1)                                                                                                     
Out[2]: b'\x01'

In [3]: struct.pack("<B",1)                                                                                                    
Out[3]: b'\x01'

In [4]: struct.pack("H",1)                                                                                                     
Out[4]: b'\x01\x00'

In [5]: struct.pack("<H",1)                                                                                                    
Out[5]: b'\x01\x00'
```
可以看到字节的大小端 `<` 对结果没有影响。

---


## 问题
看一下有影响的例子 `B` 本来只要占一个字节，某些情况下就会占两个字节。
```python
In [6]: struct.pack("BH",1,1)                                                                                                  
Out[6]: b'\x01\x00\x01\x00' # 两个

In [7]: struct.pack("<BH",1,1)                                                                                                 
Out[7]: b'\x01\x01\x00'     # 一个
```
可以看到这个时候 `<` 就对结构体的内存布局有了影响。
```python
In [8]: struct.pack("HB",1,1)                                                                                                  
Out[8]: b'\x01\x00\x01'

In [9]: struct.pack("<HB",1,1)                                                                                                 
Out[9]: b'\x01\x00\x01'
```
这样又没有问题。

---

## 最佳实践方案
不管在什么时候都要显示的指定字节的大小端，这样代码就可以预期了。

---

