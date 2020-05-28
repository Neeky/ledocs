## 正则表达式分组捕获要解决的问题
今天看到了正则表达式的分组捕获语法，感觉还是比较实用的，至少可以少写点代码了。假设有这样一个串 `# Time: 2020-05-27T00:00:36.070424+08:00` 如果我们要用正则提取出日期部分可以这样做。
```python
In [1]: import re                                                               

In [2]: s = "# Time: 2020-05-27T00:00:36.070424+08:00"                          

In [3]: date_pattern = r"\d{4}-\d{2}-\d{2}"                                     

In [4]: re.findall(date_pattern,s)                                              
Out[4]: ['2020-05-27']

In [5]: start_at , *_ = re.findall(date_pattern,s)                              

In [6]: start_at                                                                
Out[6]: '2020-05-27'
```
可以看到要得到时期值并不容易。

---

## 使用分组捕获
使用正则表达式的分组功能可以让我们更加简单的提取出目标子串。
```python
In [1]: import re                                                               

In [2]: date_pattern = r"(\d{4}-\d{2}-\d{2})"                                   

In [3]: s = "# Time: 2020-05-27T00:00:36.070424+08:00"                          

In [4]: re.search(date_pattern,s).group(0)                                      
Out[4]: '2020-05-27'

```
只要把想要提取的内容用`()`括起来就算是分好一个组了，如果整个表达式只有一个组，也可以不写`()`。

google-adsense

```python
In [1]: import re                                                               

In [2]: s = "# Time: 2020-05-27T00:00:36.070424+08:00"                          

In [3]: date_pattern = r"\d{4}-\d{2}-\d{2}"                                     

In [4]: re.search(date_pattern,s).group(0)                                      
Out[4]: '2020-05-27'
```
并且用组 “0” 来整个模式串匹配到的结果。

---

## 命名分组
默认组的名字是下标，就像上面说到的 “0” 就是下标，python 中还可以给组起名字；假设我们现在要提取日期和时区，那么我们就可以分成两个组。
```python
In [5]: date_pattern = r"(\d{4}-\d{2}-\d{2})\S*\s*(\d{2}:\d{2})$"               

In [6]: re.search(date_pattern,s).groups()      # groups() 返回所有捕获到的内容                                
Out[6]: ('2020-05-27', '08:00')

In [7]: re.search(date_pattern,s).group(0)      # group(0) 返回整个串匹配到的内容                        
Out[7]: '2020-05-27T00:00:36.070424+08:00'

In [8]: re.search(date_pattern,s).group(1)      # group(1) 返回第一个组匹配到的内容                            
Out[8]: '2020-05-27'

In [9]: re.search(date_pattern,s).group(2)                                      
Out[9]: '08:00'
```
现在我不想用下标了，那么我只要在表达式中定义好组名就行，语法像这样 `(?<group_name>)` 。

```python
In [13]: date_pattern = r"(?P<start_at>\d{4}-\d{2}-\d{2})\S*\s*(?P<zone>\d{2}:\d{2})$"                                         

In [14]: re.search(date_pattern,s).groups()                                                                                    
Out[14]: ('2020-05-27', '08:00')

In [15]: re.search(date_pattern,s).groupdict()                                                                                 
Out[15]: {'start_at': '2020-05-27', 'zone': '08:00'}
```

可以看到命名分组是分组捕获的一个扩展，它不会影响前都的功能，用起来感觉更加友好。

---


## findall 与 search
findall 也是可以识别分组的，search 就识别不了了，话说 search 也没有打算识别分组吧。
```python
In [17]: re.findall(date_pattern,s)                                                                                            
Out[17]: [('2020-05-27', '08:00')]

In [18]: re.search(date_pattern,s)                                                                                             
Out[18]: <re.Match object; span=(8, 40), match='2020-05-27T00:00:36.070424+08:00'>
```

---
