## os.dup2 
`os.dup2` 函数可以让两个不同的文件描述符共享同一个文件，这样不管向哪个文件描述符进行写入，最终都会到同一个文件。希望我后面的文字可以讲清楚这件事。

![string](static/2020-46/os-dup.jpg)

---

## 文件描述符
linux 系统会为每一个打开的文件关联上一个整数，这个整数就叫文件描述符。python 中我们通常不用这个整数来操作文件，而是用 open 函数返回的对象来完成文件的读写。只有在需要更加精细控制的情况下才会直接使用文件描述符。至于怎么通过文件描述符来操作文件这又是另一个故事了。

---


## 默认情况
默认情况下不同文件对象的读写是独立的，如下两个文件的写入互不影响。
```python
target_file_full_path = "/tmp/target.log"
hiden_file_full_path = "/tmp/hiden.log"


with open(target_file_full_path,'w') as tareget_file:
    tareget_file.write("this is target file object\n")

with open(hiden_file_full_path,'w') as hiden_file:
    hiden_file.write("this is hiden file object\n")
```
可以直接用 cat 来查看内容。
```bash
> cat /tmp/target.log 
this is target file object

> cat /tmp/hiden.log 
this is hiden file object
```

---


## 复制文件描述符
`os.dup2` 可以让两个文件对象共用同一个文件对象，达到的效果就是不管向哪个文件对象写入，最终都是写到同一个文件。
```python
import os 

target_file_full_path = "/tmp/target.log"
hiden_file_full_path = "/tmp/hiden.log"

target_file = None
hiden_file = None
try:
    target_file = open(target_file_full_path,'w')
    hiden_file = open(hiden_file_full_path,'w')

    print(f"fileno of target file = {target_file.fileno()}") # 3
    print(f"fileno of hiden file = {hiden_file.fileno()}")   # 4

    # 复制文件描述符 。
    # 让对 hiden_file 的写入，实际上写到 /tmp/target.log 中去 。
    os.dup2(target_file.fileno(),hiden_file.fileno())

    print(f"fileno of target file = {target_file.fileno()}") # 3
    print(f"fileno of hiden file = {hiden_file.fileno()}")   # 4

    target_file.write("this is target file object\n")
    hiden_file.write("this is hiden file object\n")

except Exception as err:
    pass

finally:
    if hasattr(target_file,'close'):
        target_file.close()
    
    if hasattr(hiden_file,'close'):
        hiden_file.close()

```
查看文件内容
```bash
cat /tmp/target.log 
this is target file object # 来自 target_file.write
this is hiden file object  # 来自 hiden_file.write

cat /tmp/hiden.log # 空文件
```

---

## 为什么会有 os.dup2 
一个常见的情况是代码里面用了 print 来打印日志(shadowsocket)，程序又是后台运行的，如果父进程退出那么 stdin,stdout,stderr 就都关闭了；这种情况下执行到 print 就会报异常，进一步使得程序退出。为了解决这样的问题就可以使用 `os.dup2` 来移花接木了。

当然针对这个种情况最后的方法应该还是使用 logging 模块来处理日志。

---


## 其它
shadowsocket 中的源代码
```python
def freopen(f, mode, stream):
    oldf = open(f, mode)
    oldfd = oldf.fileno()
    newfd = stream.fileno()
    os.close(newfd)
    os.dup2(oldfd, newfd)
```

man7 中关于 dup 的解释
```python
The dup() system call creates a copy of the file descriptor oldfd,
using the lowest-numbered unused file descriptor for the new
descriptor.

After a successful return, the old and new file descriptors may be
used interchangeably.  They refer to the same open file description
(see open(2)) and thus share file offset and file status flags; for
example, if the file offset is modified by using lseek(2) on one of
the file descriptors, the offset is also changed for the other.

The two file descriptors do not share file descriptor flags (the
close-on-exec flag).  The close-on-exec flag (FD_CLOEXEC; see
fcntl(2)) for the duplicate descriptor is off.
```

---

