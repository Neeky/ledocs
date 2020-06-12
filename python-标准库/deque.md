## deque
deque 是标准库中对链表的实现，与数组相比内存的使用效率上会更高一些；另外由于不用移动元素，不管在链表的前面还是后面，接入还是删除元素都非常高效。

![sqlpy](static/2020-24/sqlpy-deque.jpg)

---

## 创建 deque 对象
使用 deque 的第一步就是创建 deque 对象，构造函数的原型如下`deque([iterable[, maxlen]]) --> deque object`，接收一个可迭代对象作为参数，并有一个可选的 maxlen 参数。
```python
In [1]: from collections import deque                                                                                          

In [2]: dq = deque('ABCD')                                                                                                     

In [3]: dq                                                                                                                     
Out[3]: deque(['A', 'B', 'C', 'D'])
```

---

## append 方法
append 方法用于向队列的尾部(right)插入元素，同时也提供了一个叫 `appendleft` 的方法用于向队头插入元素。
```python
In [5]: dq                                                                                                                     
Out[5]: deque(['A', 'B', 'C', 'D'])

In [6]: dq.append('T')                                                                                                         

In [7]: dq                                                                                                                     
Out[7]: deque(['A', 'B', 'C', 'D', 'T'])

In [8]: dq.appendleft('H')                                                                                                    

In [9]: dq                                                                                                                    
Out[9]: deque(['H', 'A', 'B', 'C', 'D', 'T'])
```
google-adsense

---

## extend 方法
append 一次只能向队列添加一个元素，如果想用 append 来添加多个元素，那么要自己写循环才行，像下面的代码。
```python
In [12]: dq = deque()                                                                                                          

In [13]: for i in range(3): 
    ...:     dq.append(i) 
    ...:                                                                                                                       

In [14]: dq                                                                                                                    
Out[14]: deque([0, 1, 2])
```
extend 内部帮我们实现上面的逻辑，用不自己写循环了，用起来会方便一些。
```python
In [15]: dq.extend(range(3))                                                                                                   

In [16]: dq                                                                                                                    
Out[16]: deque([0, 1, 2, 0, 1, 2])
```
同样的 extend 也有与之对应的 extendleft 方法。

---

## len 与 count 方法
len 用于统计队列中有多少个元素，count 用于统计给定的元素出现了多少次。
```python
In [19]: dq                                                                                                                    
Out[19]: deque([0, 1, 2, 0, 1, 2])

In [20]: len(dq)           # 统计元素的个数                                                                                                    
Out[20]: 6

In [21]: dq.count(2)       # 统计队列里有多少个 2                                                                                               
Out[21]: 2
```

google-adsense

---

## pop 方法
pop 用于移除最右边的元素，popleft 用于移除最左边的元素。
```python
In [22]: dq = deque(range(5))                                                                                                  

In [23]: dq                                                                                                                    
Out[23]: deque([0, 1, 2, 3, 4])

In [24]: dq.pop()                                                                                                              
Out[24]: 4

In [25]: dq                                                                                                                    
Out[25]: deque([0, 1, 2, 3])

In [26]: dq.popleft()                                                                                                          
Out[26]: 0

In [27]: dq                                                                                                                    
Out[27]: deque([1, 2, 3])
```

---

## clear 方法
clear 方法用于清空整个队列。
```python
In [28]: dq = deque(range(4))                                                                                                  

In [29]: dq                                                                                                                    
Out[29]: deque([0, 1, 2, 3])

In [30]: dq.clear()                                                                                                            

In [31]: dq                                                                                                                    
Out[31]: deque([])
```

google-adsense

---

## rotate 方法
rotate 用于移位，参数是要移动的次数，负数表示向前，正数据表现向后。
```python
In [32]: dq = deque('ABCD')                                                                                                    

In [33]: dq                                                                                                                    
Out[33]: deque(['A', 'B', 'C', 'D'])

In [34]: dq.rotate(-1)         # 各个元素的索引位置都向前移动了一位，如果是第一个元素，它向前移就会被移动到最后                                               

In [35]: dq                                                                                                                    
Out[35]: deque(['B', 'C', 'D', 'A'])
```

---

## 实战用法 1 tail 功能
linux 有一个命令叫 tail ，它可以用于只打印文件的最后几行，用 deque 这个数据结构可以简单的实现这个功能。
```python
#!/usr/bin/evn python3

import argparse
from collections import deque


def tail():
    parser = argparse.ArgumentParser('deque-tail')
    parser.add_argument('tfile')
    tfile = parser.parse_args().tfile

    # 只保留 7 行
    lines = deque(maxlen=7)
    with open(tfile) as f:
        lines.extend(f)

    for line in lines:
        print(line, end='')


if __name__ == "__main__":
    tail()
```
运行效果。
```bash
ll >/tmp/a.log

python3 deque-tail.py /tmp/a.log 
drwxr-xr-x  319 jianglexing  wheel   10208  6 10 10:53 node_modules
-rw-r--r--    1 jianglexing  wheel  141267  6 10 10:53 package-lock.json
-rw-r--r--    1 jianglexing  wheel     300  6 10 10:53 package.json
drwxr-xr-x    2 root         wheel      64  6  9 18:52 powerlog
drwxr-xr-x    4 jianglexing  wheel     128  6 10 10:37 projects
drwxr-xr-x    9 jianglexing  wheel     288  6 10 11:19 ts-project-a
drwx------    3 jianglexing  wheel      96  6 10 11:43 vmware-jianglexing
```

---

## 实战用法 2 round-robin 调度
假设现在有三条任务队列第一条排着 "ABC" 三个任务，第二条排着 "DE" 两个任务，第三条排着 "F" 一个任务。作为任务的处理方可以先处理第一条队列里的内容，当这条队列处理完之后再去处理下一条队列中的任务；这样的处理方式就会造成后面队列中的任务等待时间比较久。

还有一种方法就是先从第一条队列里取一个任务，处理完成之后去第二条队列里取一个任务，心些类推。这样的话后面的队列也就不要等到前面的队列都处理完才会得到处理了。而这样的调度方法就叫 round-robin 。

下面看一下用 deque 怎么实现。

```python
#!/usr/bin/evn python3
from collections import deque


def round_robin():
    jobsa = "ABC"
    jobsb = "DE"
    jobsc = "F"

    # 生成任务的迭代器
    iters = [iter(jobsa), iter(jobsb), iter(jobsc)]

    dq = deque(iters)

    while len(dq):
        try:
            yield next(dq[0])
            # 移动到下一个迭代器
            dq.rotate(-1)
        except StopIteration:
            dq.popleft()


if __name__ == "__main__":
    for job in round_robin():
        print(job, end=' ')
    print()

```

运行效果。

```bash
python3 round-robin.py 
A D F B E C 
```


---