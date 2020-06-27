## time
Python 标准库里面的 `time` 主要用于完成两件事情 1、获取当前的时间 2、线程休眠。

![sqlpy](static/2020-27/sqlpy-time.jpg)

---

## 获取当前时间
在 python 中想获取当前的时间两种方式 

第一种 `time.time()` 取得当前的时间戳。
```python
In [1]: import time                                                             

In [2]: time.time()                                                             
Out[2]: 1593271351.171198
```

第二种 `datetime.now()` 。
```python
In [3]: from datetime import datetime                                           

In [4]: print(datetime.now())                                                   
2020-06-27 23:23:20.509571
```

---

## 线程休眠
在 Python 中要想让线程休眠只要 `time.sleep(seconds)`就行。
```python
In [5]: # 休息 3 秒                                                             

In [6]: time.sleep(3)
```

---