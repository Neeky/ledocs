## 背景
我想我的程序只在 Centos-7 操作系统下运行，如果是别的环境就直接报错退出。

![distro](static/2020-12/distro.png)
google-adsense

---

## 过时的实现方式
在久远的过去对于这个需求可以通过标准库`platform`来实现，但是现在最好不要这么做，因为 python 官方已经把这个标记为过时了。
```python
In [1]: import platform                                                                           
                                                                                      
In [2]: platform.linux_distribution()                                                             
/usr/local/python/bin/ipython:1: DeprecationWarning: dist() and linux_distribution() functions are deprecated in Python 3.5                                                                         
  #!/usr/local/python-3.7.3/bin/python3.7                                                         
Out[2]: ('CentOS Linux', '7.6.1810', 'Core')
```
>虽然自己解析`/etc/system-release`对于这个例子也是可行的，但是这个并不`pythonic`。

---

## distro
就在官方说这个用法已经过时的时候，这个功能的开发者开发了`distro`，以后要实现这样的需求就找`distro`吧。

第一步安装`distro`。
```bash
pip3 install distro
```

第二步使用`distro`
```python
In [1]: import distro                                                                             
                                                                                                  
In [2]: distro.linux_distribution()                                                               
Out[2]: ('CentOS Linux', '7', 'Core') 
```
---

## 总结
之所以这么喜欢`python`就是因为当我写完`import`的时候就成功一半。

---