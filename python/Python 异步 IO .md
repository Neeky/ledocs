
## Python 异步 IO
通常用异步 IO 用于服务器端的网络编程，但是磁盘文件的操作也是可以进行异步 IO 的，本文将会介绍怎么用 Python 进入异步读写文件。

要想开启异步 IO 的功能，我们就不能使用高层次的 `open` 函数，而是要使用更加低层的 `os.open` 函数。

![sqlpy.com](static/2020-22/python-sqlpy.jpg)

---


## open 与 os.open 的关系
`open` 函数是对 `os.open` 函数的封装，如果想使用异步 IO 就只能通过 os.open 来实现。

1、用 open 实现向一个文件写入四字节数据。
```python
In [1]: sync_file_path = '/tmp/sync.log'                                                                                       

In [2]: sync_file = open(sync_file_path,'bw')                                                                                  

In [3]: data = b'ABCD'                                                                                                         

In [4]: chunck_length = sync_file.write(data) # write 函数的返回值就是完成写入的大小，这里 data 只有 4 字节所以返回 4.

In [5]: print(chunck_length)                
4

In [6]: 
```
sync.log 的指纹是。
```bash
ll /tmp/sync.log 
-rw-r--r--  1 jianglexing  wheel  4  6  4 10:25 /tmp/sync.log

md5 /tmp/sync.log 
MD5 (/tmp/sync.log) = cb08ca4a7bb5f9683c19133a84872ca7
```
---

2、同样的逻辑用 os.open 来实现。
```python
In [1]: import os                                                                                                              

In [2]: data = b'ABCD'                                                                                                         

In [3]: async_file_path = '/tmp/async.log'                                                                                     

In [4]: fileno = os.open(async_file_path,os.O_CREAT | os.O_WRONLY)                                                             

In [5]: os.write(fileno,data)                                                                                                  
Out[5]: 4
```
async.log 的指纹值。
```bash
ll /tmp/async.log 
-rwxr-xr-x  1 jianglexing  wheel  4  6  4 10:29 /tmp/async.log

md5 /tmp/async.log 
MD5 (/tmp/async.log) = cb08ca4a7bb5f9683c19133a84872ca7
```

---

## 同步 IO 与异步 IO 的区别
到目前为止我们都还是只用了同步 IO 的功能，不管是使用 open 还是 os.open，在开始使用异步 IO 之前我们先来看一下异步 IO 与同步 IO 的区别。

1、同步 IO 的情况下 `write` 还是阻塞的，也就是说要完成所有的写入之后它才会返回，返回值就是写入的字节数；这个字节数和要写入的数据长度是一样长的，原因是它要写完才返回。

2、异步 IO 的情况下 `write` 是不阻塞的，也就是说 write 会更加快的返回，它不要等到完全把数据写完；这个时候 write 的返回值就不是数据的大小了，而是有多数字节完成了写入。

---


## 同步 IO vs 异步 IO
1、采用同步 IO 的方式向文件中写入 4G 数据。 
```python
In [1]: sync_file_path = '/tmp/sync.log'                                                                                       

In [2]: data = b'ABCD' * 1024 * 1024 * 1024                                                                                    

In [3]: sync_file = open(sync_file_path,'bw')                                                                                  

In [4]: sync_file.write(data)                                                                                                  
Out[4]: 4294967296

In [5]: sync_file.close() 
```
```bash
ll /tmp/sync.log 
-rw-r--r--  1 jianglexing  wheel  4294967296  6  4 10:56 /tmp/sync.log

md5 /tmp/sync.log 
MD5 (/tmp/sync.log) = 6a628b7c7be4251d724217b78b52ef6c
```
可以看到当 4G 数据都写完之后 `write` 函数返回。

---

2、采用异步 IO 的方式向文件中写入 4G 数据。
```python
In [1]: import os                                                                                                              

In [2]: async_file_path = '/tmp/async.log'                                                                                     

In [3]: data = b'ABCD' * 1024 * 1024 * 1024                                                                                    

In [4]: async_file = os.open(async_file_path,os.O_WRONLY | os.O_NONBLOCK)                                                      

In [5]: os.write(async_file,data)                                                                                              

Out[5]: 2147483647

In [7]: 2147483647 /1024/1024/1024                                                                                             
Out[7]: 1.9999999990686774
```
可以看到只写了 1.9G 数据就返回了，为了能把数据完整的写完，我们就要加上其它的逻辑来保证。
```python
import os
# 异步写入的目标文件
async_file_path = '/tmp/async.log'
# 4G 数据
data = b'ABCD' * 1024 * 1024 * 1024


def async_write_all(file_path, data):
    """实现一个异步写入所有数据的函数
    """
    # 总长度
    total_length = len(data)
    # 已经完成写入的长度
    writed_length = 0

    #
    print("start write .")
    try:
        fileno = os.open(file_path, os.O_WRONLY | os.O_NONBLOCK)

        # 由于 write 不保证全部写完成，所以要应用程序来保证。
        while total_length != writed_length:
            # 当两个长度相等后说明写入完成了
            writed_length = writed_length + \
                os.write(fileno, data[writed_length:])
            progress = writed_length / total_length
            print(f"writing {progress:.3} .")

    except IOError as err:
        pass
    finally:
        os.close(fileno)

    print("complete write .")


async_write_all(async_file_path, data)

```
运行效果。
```bash
python3 main.py 
start write .
False
writing 0.5 .
writing 1.0 .
writing 1.0 .
complete write .
```


---