## PathLike 
PathLike 是由 python 的内建函数 `open` 引出来的一个概念;通常我们转递给 open 函数的参数，是一个字符串形式的文件路径，但是 open 函数支持接收一个实现了 PathLike 接口的对象。函数的原型如下。

```python
open(file, mode='r', buffering=-1, encoding=None, errors=None, newline=None, closefd=True, opener=None)¶
```

file is a path-like object giving the pathname (absolute or relative to the current working directory) of the file to be opened or an integer file descriptor of the file to be wrapped. (If a file descriptor is given, it is closed when the returned I/O object is closed, unless closefd is set to False.)

![pathlike](static/2020-46/path-like.jpg)

---

## PathLike 接口
PathLike 接口上定义了魔术方法 `__fspath__` 看起来象这样。
```python
class PathLike:
    def __fspath__(self)->str:
        pass
```
---

## 一般用法
一般我们要打开一个文件都是直接向 open 传字符串
```python
#!/usr/bin/evn python3

if __name__ == "__main__":
    with open("/tmp/error.log",'w') as error_log_file:
        error_log_file.write("this an error message\n")
```

---

## PathLike 写法
```python
#!/usr/bin/evn python3

import os

class ErrorLog(os.PathLike):
    def __fspath__(self):
        return "/tmp/error.log"


if __name__ == "__main__":
    el = ErrorLog()
    with open(el,'w') as error_log_file:
        error_log_file.write("this an error message\n")
```

---

## PathLike 的优点
直接定死字符串的形式拓展性不强，PathLike 给了我们一个可以动态返回路径的机会(在我上面的程序中没有体现)。

---