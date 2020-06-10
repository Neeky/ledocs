## re
`正则表达式` 可以用形式化的语法描述文本匹配模式，模式又被正则表达式引擎编译成指令；执行指令并提供一个字符串作为输入，就可以知道给定的输入有没有与模式相匹配。 文字比较空洞还是直接看例子吧。

![sqlpy](static/2020-24/sqlpy-re-01.jpg)

google-adsense

---

## search 查找文本中的模式
查找一段文本中有没有出现 `sql` 这个字符串。

```python
In [1]: s ="I like sql"                                                         

In [2]: 'sql' in s                                                              
Out[2]: True
```

可以看到根本没有用到 `re` 这个库就把问题给解决了；但是没有 bug 吗？

```python
In [3]: s = "I like www.sqlpy.com"                                              

In [4]: 'sql' in s                                                              
Out[4]: True
# 这样的太不应该了，我想要的只是 sql 而不是 xxxsqlxx。
```

用 re 就没有 bug 了。

```python
In [1]: import re                                                               

In [2]: p = r"\bsql\b"                                                          

In [3]: s = "I like sql"                                                        

In [4]: re.search(p,s)                                                          
Out[4]: <re.Match object; span=(7, 10), match='sql'>

In [5]: s = "I like www.sqlpy.com"                                              

In [6]: re.search(p,s)           # 可以看到不在有匹配                                               

In [7]:
```

原因在于用 re 我们把可以把自己要找到内容描述的更加清楚，`'sql' in s` 讲的是 s 中是否包含`sql` 这三个字符，而 `r"\bsql\b"` 描述的是 'sql' 这个单词。

---

## compile 编译
上面的代码还有一个问题，`re.search(p,s)` 运行到这里的时候就会对模式进行编译，运行几次就编译几次，对于反复用到的模式提前编译是一个好的选择。下面是两种处理方式的性能比较。
```python
#!/usr/bin/evn python3

import re
import time

# 提前把用到的模式编译好
global_pattern = re.compile(r"\bsql\b")


def fun_a():
    """在每次用的时候都编译模式
    """
    p = r"\bsql\b"
    s = "I like sql"
    return re.search(p, s)


def fun_b():
    """使用提交编译好的模式
    """
    global global_pattern
    s = "I like sql"
    return global_pattern.search(s)


def run(fun, times=1000):
    start_at = time.time()
    for t in range(times):
        fun()

    complete_at = time.time()
    print(
        f"function {fun.__name__} execute {times} times cost {complete_at - start_at}")


if __name__ == "__main__":
    run(fun_a, 10000)
    run(fun_b, 10000)
```
运行结果。
```bash
python3 main.py 
function fun_a execute 10000 times cost 0.013822078704833984
function fun_b execute 10000 times cost 0.006231784820556641
```
可以看到提前编译要比即时编译快 100% 。

google-adsense

---

## findall 查询所有匹配
通过 search 只能知道有没有匹配，如果有多处匹配我们怎么提取出来呢？
```python
In [1]: import re                                                               

In [2]: s = "ab abb abbb abbbb"                                                 

In [3]: p = r"ab*"                                                              

In [4]: re.search(p,s)                                                          
Out[4]: <re.Match object; span=(0, 2), match='ab'>

In [5]: for m in re.findall(p,s): 
   ...:     print(m) 
   ...:                                                                         
ab
abb
abbb
abbbb

In [6]: type(re.findall(p,s))                                                   
Out[6]: list
```
可以看到 findall 返回的是一列表而列表的元素是字符串，如果有大量的结果被匹配到，比如几千几亿个，这个 list 是非常吃内存的，理节约内存的方式是使用生成器。
```python
In [7]: for m in re.finditer(p,s): 
   ...:     print(m) 
   ...:                                                                         
<re.Match object; span=(0, 2), match='ab'>
<re.Match object; span=(3, 6), match='abb'>
<re.Match object; span=(7, 11), match='abbb'>
<re.Match object; span=(12, 17), match='abbbb'>


In [8]: type(re.finditer(p,s))                                                  
Out[8]: callable_iterator
```

google-adsense

---

## 重复与贪婪

正则表达式对重复的描述包含 5 种形式。

1、`ab*` 说的是 'a' 后面的 'b' 可以是零个也可以是任意多个。 2、`ab+` 说的是 'a' 后面的 'b' 至少是一个。 3、`ab?` 说的是 'a' 后面的 'b' 要么没有要么只有一个。4、`ab{5}`说的是 'a' 后面的 'b' 一定要是 5 个，当然这里的 5 也可以改在其它的数值，视需求而定。 5、`ab{5,8}` 说的是可以是 5 个到 8 个之间([5,8]开边都是闭区间)。

默认情况下正则表达式工作在贪婪模式，也就是说它会倾向于匹配出更多的字符串，看一下面的例子。

```python
In [1]: import re                                                               

In [2]: s = "abbbb"                                                             

In [3]: p = "ab*"                                                               

In [4]: re.search(p,s)                                                          
Out[4]: <re.Match object; span=(0, 5), match='abbbb'>
```
怎么让它匹配出尽可能少的字符串呢？答案是在正则表达式的结尾处加上一个`?`号，看下面的代码。
```python
In [5]: p = "ab*?"                                                              

In [6]: re.search(p,s)                                                          
Out[6]: <re.Match object; span=(0, 1), match='a'>
```

---



