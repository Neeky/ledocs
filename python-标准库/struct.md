## 概要
标准库 `struct` 用于把 Python 的数据打包成 C 语言的结构体。常见的应用场景是解析 TCP 字节流、解析二进制文件。

![sqlpy](static/2020-26/sqlpy-struct-02.jpg)

---

## struct.pack 打包
`struct.pack` 函数用于把 Python 数据打包成结构体(表现上来看就是字节流);第一个参数用于指定格式，后面的参数用于指定数据。
```python
In [1]: import struct                                                           
# 把数值 8 转换为字节码
In [2]: struct.pack("I",8)    # 默认的字节顺序是小端                                                  
Out[2]: b'\x08\x00\x00\x00'

In [3]: struct.pack(">I",8)                                                     
Out[3]: b'\x00\x00\x00\x08'
```
默认的字节序是“小端”，也就是说权重小的放前面。

google-adsense

---


## struct.unpack 解包
unpack 是 pack 的逆运算。
```python
In [1]: import struct                                                           

In [2]: data = b'\x08\x00\x00\x00'                                              

In [3]: struct.unpack("I",data)                                                 
Out[3]: (8,)
```

---

## 格式串详解
常用格式字符的意义如下。

|**格式**|**C 语言类型**       |**Python 类型**     |**标准大小(字节)**|
|-------|--------------------|-------------------|----------------|
| x     | pad byte           | no value          |                |
| c     | char               | bytes of length 1 | 1              |
| b     | signed char        | integer           | 1              |
| B     | unsigned char      | integer           | 1              |
| ?     | _Bool              | bool              | 1              |
| h     | short              | integer           | 2              |
| H     | unsigned short     | integer           | 2              |
| i     | int                | integer           | 4              |
| I     | unsigned int       | integer           | 4              |
| l     | long               | integer           | 4              |
| L     | unsigned long      | integer           | 4              |
| q     | long long          | integer           | 8              |
| Q     | unsigned long long | integer           | 8              |
| f     | float              | float             | 4              |
| d     | double             | float             | 8              |
| s     | char[]             | bytes             |                |
| p     | char[]             | bytes             |                |
| P     | void *             | integer           |                |


---


---