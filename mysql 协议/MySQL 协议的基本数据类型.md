## 概要
总的来讲 MySQL 协议中只定义了两种数据类型，一种是“整型”，另一种就是“字符串”；这两个大类下又分别包含了若干子类型。

![mysql-logo](static/2020-22/mysql-logo.jpeg)

---

## 整型
整型包含两个子类 1、固定长度的整型， 2、长度编码的整型。

---


## 固定长度的整型

固定长度的整型就和它的名字一样，数据所占用的长度是固定的，目前协议中定义了 6 种分别是 1字节，2字节，3字节，4字节，6字节，8字节。

如果 MySQL 想要通过 8 字节的固定长度发送一个“1”，那么我们在二进制层面看到的字节流是这样的。
```python
b'\x01\x00\x00\x00\x00\x00\x00\x00' # 字节序是小端
```

可以看到只有第一个字节是“1”其它字节都是 0 ,在网络上发送那么多无用数据，是不划算的；通常对于小的数据我们用小类型发送，针对对这个例子我们可以使用 1字节长度的方式来发送。

```python
b'\x01
```

我总结了一下按固定长度打包的 Python 代码。

|**类型**|**打包代码**|
|-------|-----------|
| 1字节  | `struct.pack('<b',1)`|
| 2字节  | `struct.pack('<h',1)`|
| 3字节  | `struct.pack('<i',1)[0:3]`|
| 4字节  | `struct.pack('<i',1)`|
| 6字节  | `struct.pack('<q',1)[0:6]`|
| 8字节  | `struct.pack('<q',1)`|
|        |                     |

google-adsense

---

## 长度编码的整型
对于长度编码的整型，第一个字节有特殊的意义，含义如下表。

|**第一个字节的值**|**意义**|
|----------------|-------|
|小于251(\xfb)| 这是一个 1 字节长度的整数，它自己的值就代表数的值|
|等于252(\xfc)| 这是一个 2 字节长度的整数，要解码它后面的两个字节才能得到值|
|等于253(\xfd)| 这是一个 3 字节长度的整数，要解码它后面的三个字节才能得到值|
|等于254(\xfe)| 这是一个 8 字节长度的整数，要解码它后面的八个字节才能得到值|

按上面的定义解包代码应该是这样的。

```python
#!/usr/bin/env python3

import struct


def unpack_fix_length_int(buffer):
    """
    """
    if buffer[0] < 251:
        return buffer[0], None
    elif buffer[0] == 252:
        return struct.unpack("h", buffer[1:3])
    elif buffer[0] == 253:
        return struct.unpack("i", buffer[1:4]+b'\x00')
    elif buffer[0] == 254:
        return struct.unpack("q", buffer[1:9])
    else:
        raise ValueError("buffer not a fix_length_int")
```

验证。
```python
def main():
    v128, *_ = unpack_fix_length_int(b'\x80')  # 128
    v256, *_ = unpack_fix_length_int(b'\xfc\x00\x01')
    print(v128)
    print(v256)


if __name__ == "__main__":
    main()

```
运行效果。
```bash
python3 main.py 

128
256
```
可以正常解码。

---

## 字符串类型
字符串类型也有几种实例
|**类型名**|**详细**|
|----|-----|
|固定长度的字符串|string\<fix\>|
|'\x00'结尾的字符串|string\<NUL\>|
|可变长度的字符串|string\<var\>|
|长度编码的字符串|string\<lenenc\>|
|直至包尾的字符串|string\<EOF\>|


---