## 概要
今天运行一段别人发给我的代码，一运行就报错了，错误信息如下。
```python
In [1]: from Crypto.Cipher import DES                                           
---------------------------------------------------------------------------
ModuleNotFoundError                       Traceback (most recent call last)
<ipython-input-1-7887bce5dbd1> in <module>
----> 1 from Crypto.Cipher import DES

ModuleNotFoundError: No module named 'Crypto'
```
因为是第一次遇到缺少这个库，所以在这里把解决方案也给记一下。

![sqlpy](static/2020-27/sqlpy-crypto.jpg)

---


## 解决方案
看报错就是我们有 python 包没有安装，所以导入失败；要从报错信息中找出是缺少哪个包并不是一个容易的事，这个只能靠平时的积累了。

这次 google 了一下，发现是要安装 pycrypto 这个包。
```bash
pip3 install pycrypto
Looking in indexes: https://mirrors.cloud.tencent.com/pypi/simple
Collecting pycrypto
  Downloading https://mirrors.cloud.tencent.com/pypi/packages/60/db/645aa9af249f059cc3a368b118de33889219e0362141e75d4eaf6f80f163/pycrypto-2.6.1.tar.gz (446kB)
     |████████████████████████████████| 450kB 556kB/s 
Installing collected packages: pycrypto
  Running setup.py install for pycrypto ... done
Successfully installed pycrypto-2.6.1
WARNING: You are using pip version 19.2.3, however version 20.1.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.

```
验证。
```python
In [1]: from Crypto.Cipher import DES                                           

In [2]:                                                                         

In [2]: exit
```

---

