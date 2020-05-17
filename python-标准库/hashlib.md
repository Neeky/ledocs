## hashlib
标准库 `hashlib` 专门用于生成数据指纹，其中包含有许多种不同的算法；因为他们提供的接口是一致的，这里就只有最常用的 sha256 来讲解。

---

## 创建加密算法对象
hashlib 把各种加密算法封装成了一个个的类，我们要使用特定的算法，直接创建对象就行。
```python
In [1]: import hashlib                                                          

In [2]: sha256_instance = hashlib.sha256()                                      

In [3]: sha256_instance.digest()                                                
Out[3]: b"\xe3\xb0\xc4B\x98\xfc\x1c\x14\x9a\xfb\xf4\xc8\x99o\xb9$'\xaeA\xe4d\x9b\x93L\xa4\x95\x99\x1bxR\xb8U"
```
以上代码创建了一个空(b'')字节的 sha256 指纹提取对象，也可以直接把要计算指纹值的数据传递进构造函数。
```python
In [1]: import hashlib                                                          

In [2]: sha256_instance = hashlib.sha256(b'')                                   

In [3]: sha256_instance.digest()                                                
Out[3]: b"\xe3\xb0\xc4B\x98\xfc\x1c\x14\x9a\xfb\xf4\xc8\x99o\xb9$'\xaeA\xe4d\x9b\x93L\xa4\x95\x99\x1bxR\xb8U"
```
在实际的编程中我们会使用第二种方式，这样方便一些。

googl-adsense

---

## digest 提取指纹值
加密对象的 `digest()` 方法会返回数据的指纹值。
```python
In [3]: sha256_instance.digest()                                                
Out[3]: b"\xe3\xb0\xc4B\x98\xfc\x1c\x14\x9a\xfb\xf4\xc8\x99o\xb9$'\xaeA\xe4d\x9b\x93L\xa4\x95\x99\x1bxR\xb8U"
```

---

## update 更新数据
加密对象的 `update()` 函数实际上就是做了一个简单的数据拼接，所以我感觉这个函数名起的不太友好。
```python
In [1]: import hashlib                                                          

In [2]: s_a = hashlib.sha256(b'ab')                                             

In [3]: s_a.update(b'cd')                                                       

In [4]: s_a.digest()                                                            
Out[4]: b'\x88\xd4&o\xd4\xe63\x8d\x13\xb8E\xfc\xf2\x89W\x9d \x9c\x89x#\xb9!}\xa3\xe1a\x93o\x03\x15\x89'

In [5]: s_b = hashlib.sha256(b'abcd')                                           

In [6]: s_b.digest()                                                            
Out[6]: b'\x88\xd4&o\xd4\xe63\x8d\x13\xb8E\xfc\xf2\x89W\x9d \x9c\x89x#\xb9!}\xa3\xe1a\x93o\x03\x15\x89'
```
可以看出 `update` 就是把两段字节流拼接一下。

---

## 注意事项
不管是加密算法的构造函数，还是 update 函数，其接受的参数都是字节流。

---