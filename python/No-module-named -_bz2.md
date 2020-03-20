## No module named _bz2 异常
背景一个新的 python3 环境在我想导入 pandas 包的时候报错了。
```python
import pandas
Traceback (most recent call last):
  File "", line 1, in 
  File "/usr/local/python/lib/python3.7/site-packages/pandas/__init__.py", line 55, in 
    from pandas.core.api import (
  File "/usr/local/python/lib/python3.7/site-packages/pandas/core/api.py", line 24, in 
    from pandas.core.groupby import Grouper, NamedAgg
  File "/usr/local/python/lib/python3.7/site-packages/pandas/core/groupby/__init__.py", line 1, in 
    from pandas.core.groupby.generic import (  # noqa: F401
  File "/usr/local/python/lib/python3.7/site-packages/pandas/core/groupby/generic.py", line 44, in 
    from pandas.core.frame import DataFrame
  File "/usr/local/python/lib/python3.7/site-packages/pandas/core/frame.py", line 88, in 
    from pandas.core.generic import NDFrame, _shared_docs
  File "/usr/local/python/lib/python3.7/site-packages/pandas/core/generic.py", line 71, in 
    from pandas.io.formats.format import DataFrameFormatter, format_percentiles
  File "/usr/local/python/lib/python3.7/site-packages/pandas/io/formats/format.py", line 47, in 
    from pandas.io.common import _expand_user, _stringify_path
  File "/usr/local/python/lib/python3.7/site-packages/pandas/io/common.py", line 3, in 
    import bz2
  File "/usr/local/python/lib/python3.7/bz2.py", line 19, in 
    from _bz2 import BZ2Compressor, BZ2Decompressor
ModuleNotFoundError: No module named '_bz2'
```
![pandas](static/2020-12/pandas.png)

一开始是想，难道是因为我用了最新的 python 版本就这样对我？于是去官网看了一下发现 pandas 是已经支持到 python-3.7 了呀。可是没有 pandas 我还怎么炒股呀， 所以这个事完全不能忍啦！

---

## 分析问题
从日志上看有这么一条关键信息。
```bash
ModuleNotFoundError: No module named '_bz2'
```
看样子是有一个叫“_bz2”的模块没有找到，不过从这个名字来看这个模块是内部用的。与 python 解释器打交道多年，深知它虽然安装简单，但是它有多少功能主要取决于你在编译安装时系统上有多库文件和头文件。 没办法看来解决这个问题要重新安装 python-3.7 的解释器了。

---

## 解决方案
第一步先安装依赖。
```bash
yum -y install bzip2 bzip2-devel

# 根据我多年的使用经验来看，一次性安装如下依赖集，包治百病
# yum -y install openssh openssh-clients gcc gcc-c++ libffi libyaml-devel libffi-devel zlib zlib-devel openssl openssl-devel libyaml sqlite-devel libxml2 libxslt-devel libxml2-devel bzip2 bzip2-devel
        
```
第二步编译安装 python-3.7。
```bash
#编译安装
cd /tmp/
wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
tar -xvf Python-3.7.7.tgz
cd Python-3.7.7
./configure --prefix=/usr/local/python-3.7.7 && make && make install

#导出环境变量
cd /usr/local/
ln -s python-3.7.7 python3
echo 'export PATH=/usr/local/python/bin/:$PATH' >> /etc/profile
source /etc/profile

pip install pandas
```
---

## 检查
检查 pandas 能不能正常导入。
```bash                                                            
[GCC 4.8.5 20150623 (Red Hat 4.8.5-36)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pandas
```

---
