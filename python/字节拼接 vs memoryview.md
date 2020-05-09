## 背景
在网络编程中对字节流的解析通常是一步一步完成的，也就是说要解析出前面的内容才知道后面的数据有多长；那么最后拼接字节流的时候是直接使用`+`运算高效还是使用`memoryview`高效呢？

---

## 测试代码
分别用两种方法实现同样的逻辑，用于比较哪个方法更加高效。
```python
#!/usr/bin/env python3

import socket
import logging
from datetime import datetime

logging.basicConfig(level=logging.INFO)


def using_plus(counts=100):
    """测试使用 + 完成字节拼接所要的时间
    """
    start_at = datetime.now()
    # 定义一个列表用于保存所有的字节流
    byte_list = []
    for i in range(100):
        # 重度测试 100 次

        byte_list.append(b'')
        for j in range(counts):
            # 每次向 byte_list[i] 的后面加上 j 个字节
            byte_list[i] = byte_list[i] + b'\x00' * j
    end_at = datetime.now()

    logging.info(f"using_plus cost {end_at - start_at}")


def using_memory_view(counts=100):
    """测试使用 memoryview 完成字节拼接所要的时间
    """
    start_at = datetime.now()
    # 定义一个列表用于保存所有的字节流
    byte_list = []

    # 一次性计算出总共要拓展多长
    target_length = sum(range(counts))
    logging.info(f"allocate {target_length} bytes.")

    for i in range(100):
        # 创建 bytearray 对象
        byte_list.append(bytearray(b''))
        # 一次性申请后面所要的空间
        byte_list[i].extend(bytearray(target_length))
        # 创建内存视图
        mv = memoryview(byte_list[i])
        for j in range(counts):
            mv[0:j] = b'\x00' * j
            mv = mv[j:]
        # 用完了就尽可能早的释放
        mv.release()
    end_at = datetime.now()

    logging.info(f"using_memory_view cost {end_at - start_at}")


def main():
    using_plus(1000)
    using_memory_view(1000)


if __name__ == "__main__":
    main()

```
运行效果如下。
```bash
python3 main.py

INFO:root:using_plus cost 0:00:04.235193
INFO:root:allocate 499500 bytes.
INFO:root:using_memory_view cost 0:00:00.067734
```
从结果可以看到当 counts = 1000 时，加法运算符的效率要比 memoryview 慢上 700% 。

---

## 更多测试结果
比两个算法进行了多次测试最终的耗时如下。

|**counts值**|**+运算符(耗时)**|**memoryview(耗时)** |
|------------|---------------|--------------------|
|100         |0.003          | 0.003              | 
|200         |0.010          | 0.006              |
|400         |0.067          | 0.020              |
|800         |1.877          | 0.046              |

---


## 结论
单单从性能来说要尽可能的使用 memoryview 来取代 + 法运算符。

---
