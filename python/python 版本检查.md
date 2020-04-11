## 背景
希望程序只运行在 python-3.6 以上的版本上，那如何对当前的 python 环境做检查呢！

![python-version-info](static/2020-15/version-info.png)

google-adsense

---

## 实现方式
python 的版本信息保存在 `sys.version_info` ，要检查版本就是要检查这个值。
```python
import sys

def is_version_ok():
   """当 python 版本号大于等于 3.6 的时候返回 True。
   """
   marjor,minor,*_ = sys.version_info
   return (marjor,minor) >= (3,6)

```

验证程序是否正常。

```python
ipython
Python 3.8.0 (v3.8.0:fa919fdf25, Oct 14 2019, 10:23:27) 
Type 'copyright', 'credits' or 'license' for more information
IPython 7.8.0 -- An enhanced Interactive Python. Type '?' for help.

In [1]: import sys 
   ...:  
   ...: def is_version_ok(): 
   ...:    """当 python 版本号大于等于 3.6 的时候返回 True。 
   ...:    """ 
   ...:    marjor,minor,*_ = sys.version_info 
   ...:    return (marjor,minor) >= (3,6) 
   ...:                                                                         

In [2]: is_version_ok()                                                         
Out[2]: True
```



---