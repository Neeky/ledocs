## 背景
在 python 中测试(比较)一个对象是不是 None 可以有两种不同的写法，一个很自然的想法就是这两个写法下，哪一个的效率会更高一些呢？

![is-vs-equall](static/2020-8/is-vs-equal.png)
google-adsense

---


## 怎么才能知道哪个写法更好
就目前来讲有两个可行的方案，第一个比较笨重那就是每个操作各自执行 n 次，看哪个的用时更加短。 第二个就是直接分析字节码

--- 

## 方案一、直接比较耗时
   
第一步测试 is None 写法的耗时

```python
def main():
    for x in range(10000000):
        None is None
    
if __name__ == "__main__":
    main()
```

```bash
python3 -m cProfile use-is.py 
         4 function calls in 0.321 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.321    0.321 use-is.py:2()
        1    0.321    0.321    0.321    0.321 use-is.py:2(main)
        1    0.000    0.000    0.321    0.321 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler'objects}
```

第二步测试 == None 写法的耗时

```python
def main():
    for x in range(10000000):
        None == None
    
if __name__ == "__main__":
    main()     
```

```bash
python3 -m cProfile use-equal.py 
         4 function calls in 0.389 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.389    0.389 use-equal.py:1()
        1    0.389    0.389    0.389    0.389 use-equal.py:1(main)
        1    0.000    0.000    0.389    0.389 {built-in method builtins.exec}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler'objects}
```
   
---

## 方案二、分析字节码

方案二比较硬核，直接比较两种写法在字节码上的差异
```python
In [4]: import dis                                                              

In [5]: def use_is(): 
   ...:     return None is None 
   ...:                                                                         

In [6]: dis.dis(use_is)                                                         
  2           0 LOAD_CONST               0 (None)
              2 LOAD_CONST               0 (None)
              4 COMPARE_OP               8 (is) # 注意这里是 is
              6 RETURN_VALUE

In [7]: def use_equal(): 
   ...:     return None == None 
   ...:                                                                         

In [8]: dis.dis(use_equal)                                                      
  2           0 LOAD_CONST               0 (None)
              2 LOAD_CONST               0 (None)
              4 COMPARE_OP               2 (==) # 注意这里是 ==
              6 RETURN_VALUE
``` 

通常来讲一看字节码多数情况下可能发现性能问题，但是这次比较特别它们两个的字节码是一样的，所以不能直观的看出来；好在我们的方案一，是可以说明问题的。

---


## 结论
结论：is None 这个写法的效率要优于 == None，0.321s:0.389s

---

