## string
`string` 这个库在 `python-2` 的年代就已经是标准库中的一员了，主要的功能就是用来做字符串处理的。总的来讲包含三个大的部分，分别是字符串常量、模板、辅助方法。
![string](static/2020-13/string.png)
google-adsense

---

## 字符串常量
string 模块中提供了一些常用的字符串常量，如大小写、数字、空白等等

1、大小写`string.ascii_lowercase`与`string.ascii_uppercase`
```python
In [1]: import string                                                           

In [2]: string.ascii_lowercase                                                  
Out[2]: 'abcdefghijklmnopqrstuvwxyz'

In [3]: string.ascii_uppercase                                                  
Out[3]: 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
```
如果想大小写全都要的话可以对两个常量做一个简单的连接。
```python
In [4]: string.ascii_lowercase + string.ascii_uppercase                         
Out[4]: 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
```
上的写法代码量要多一些`ascii_letters`可以达到同样的效果。
```python
In [5]: string.ascii_letters                                                    
Out[5]: 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
```

2、数字 `string.digits`
```python
In [7]: string.digits                                                           
Out[7]: '0123456789'
```

3、特殊字符`string.punctuation`与`string.whitespace`空白
```python
In [8]: string.punctuation                                                      
Out[8]: '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'

In [9]: string.whitespace                                                       
Out[9]: ' \t\n\r\x0b\x0c'
```
google-adsense

---

## 模板
`string.Template` 的定位与 Jinja2 这样的模板引擎定位不太一样，前者比较简单不支持像 if  else 这样的逻辑用模板表达的。
```python
from string import Template
#创建模板、模板中可以用 ${name} 的方式在运行时求值
s = Template('Hello ${something}')
values = {'something':"world"}
#把变量替换成值
print(s.substitute(**values))   # Hello world
```
---

## 辅助方法
`string.capwords(s, sep=None)`用于大写单词的首字母，不过它已经成为绝唱了(这个模块的最后一个辅助方法)。
```python
import string
s = "space, tab, linefeed, return, formfeed, and vertical tab"
string.capwords(s) # 'Space, Tab, Linefeed, Return, Formfeed, And Vertical Tab'
```
---