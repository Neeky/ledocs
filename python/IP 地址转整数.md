## 背景
今天有人问我 “为什么数据库中有人推荐使用 int 类型来保存 IP 地址？”。现在(2020看)来看这个东西已经有点过时，一方面是磁盘空间不在那么贵，另一方面是 IPv6 与这条法则不兼容。

下面我们就来看一下把 IPv4 地址转换成整数的原理和收益各是什么。

![static](static/2020-26/sqlpy-ip.jpg)

---

## 转换的原理
一个 IPv4 类的地址共分为四个部分 `0.0.0.0` 然而每一个部分的取值范围都在 `0 ~ 255`；也就是说每一个部分都可以用一个字节来保存，总共写个字节就够了，4 个字节不就是 int 吗？

第一步 把 IP 地址的各个部分转换为一个字节，并拼接它们，那么会得到一个 4 字节的串。
```python
import struct

def aton(ip_address: str) -> bytes:
    result = []
    for i in ip_address.split('.'):
        result.append(struct.pack("!B", int(i)))
    return b''.join(result)
```

第二步 把字节串转换成整数。
```python
In [2]: aton("127.0.0.1")                                                                                                      
Out[2]: b'\x7f\x00\x00\x01'

In [3]: int.from_bytes(b'\x7f\x00\x00\x01','big')                                                                              
Out[3]: 2130706433
```

这样我们就把 IPv4 地址转换成了一个整数，完整的代码如下。

```python
import struct


def aton(ip_address: str) -> bytes:
    result = []
    for i in ip_address.split('.'):
        result.append(struct.pack("!B", int(i)))
    return b''.join(result)


if __name__ == "__main__":
    bts = aton("127.0.0.1")
    print(int.from_bytes(bts, 'big'))

```
运行效果如下。
```bash
python3 main.py 
2130706433
```
---

## 转换的收益限制
如果不做转换可以使用 varchar 来保存 IPv4 地址，这样的话需要 15 (3*4 + 3) 个字节才行；如果转换一下只需要 4 个字节就行了，节约了磁盘空间，可能会多用点 cpu 时间。

今天来说 IPv6 已经是主流，它的长度直接从之前的 4 字节直接涨到了 16 字节；int 不再能满足需求，为了可以统一处理这两种类型的 IP 现在推荐使用 varchar 来保存。

---

## inet_aton 与 inet_ntoa
这一对 IP 是 IPv4 时代的转换函数，目前来看已经过时。

1、inet_aton IP 转数字。
```python
In [4]: socket.inet_aton("127.0.0.1")                                                                                          
Out[4]: b'\x7f\x00\x00\x01'

```

2、inet_ntoa 数字转 IP。
```python
In [5]: socket.inet_ntoa(b'\x7f\x00\x00\x01')                                                                                  
Out[5]: '127.0.0.1'
```

---

## inet_pton 与 inet_ntop
这是一对新的 API ，这对 API 兼容了 IPv4 和 IPv6 。
```python
In [6]: socket.inet_pton(socket.AF_INET6,"5aef:2b::8")                                                                         
Out[6]: b'Z\xef\x00+\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x08'

In [7]: socket.inet_ntop(socket.AF_INET6,b'Z\xef\x00+\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x08')                        
Out[7]: '5aef:2b::8'

In [8]: socket.inet_pton(socket.AF_INET,"127.0.0.1")                                                                           
Out[8]: b'\x7f\x00\x00\x01'

In [9]: socket.inet_ntop(socket.AF_INET,b'\x7f\x00\x00\x01')                                                                  
Out[9]: '127.0.0.1'
```

---


