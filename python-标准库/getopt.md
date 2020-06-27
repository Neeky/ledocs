## getopt
getopt 模块用于完成 C 语言风格的命令行参数处理，这是一个比较“古老”的模块了；虽然 getopt 比较老土，但是人家简单呀，只包含两个函数和一个异常。

```python
# -- 函数
getopt.getopt(args, shortopts, longopts=[])
#    a): args 用于指定命令行参数列表 
#    b): shortopts 用于指定有哪些“短选项”
#    c): longopts 用于指定有哪些“长选项”

getopt.gnu_getopt(args, shortopts, longopts=[])
#    与 getopt.getopt 类似，只不过这个是 gun风格的

# -- 异常
exception getopt.GetoptError
```

![getopt](static/2020-27/sqlpy-getopt.jpg)

---

## 自己解析命令行参数
Python 程序启动后 `sys.argv` 这个变量引用着所有的命令行参数，解析这个变量就能完成对命令行参数的处理。
```python
import sys


def main(args):
    # 假设只有短选项
    args = args[1:]
    args_dict = {}
    for arg in args:
        k = arg[1]
        v = arg[2:]
        args_dict.update({k: v})

    print(args_dict)


if __name__ == "__main__":
    main(sys.argv)
```
运行效果如下。
```bash
python3 main.py -uroot -p123456 -h127.0.0.1 -P3306
{'u': 'root', 'p': '123456', 'h': '127.0.0.1', 'P': '3306'}
```

google-adsense

---

## getopt.getopt 函数
用 getopt.getopt 来处理命令行参数又比自己解析要来的方便一些。先来看一个模拟 mysql 命令行短选项的例子。
```python
import getopt
import sys


def main(args):
    """
    """
    # 模拟 MySQL 的短命令行选项
    result = getopt.getopt(args, 'h:P:u:p:')
    print(result)


if __name__ == "__main__":
    main(sys.argv[1:])
```
运行效果。
```python
python3 main.py -uroot -p123456 -h127.0.0.1 -P3306
([('-u', 'root'), ('-p', '123456'), ('-h', '127.0.0.1'), ('-P', '3306')], [])
```

---

从源码上来看 getopt.getopt 函数的原型是这样的。

```python
getopt.getopt(args, shortopts, longopts=[])
```
shortopts 用于指定短选项字符串，如果选项后要有值那么这个选项就要以":"结尾。

longopts 用于指定长选项数组，数据中的每一项都用来描述一个长选项，如果这个长选项要有值那么它要以"="结尾。

---

下面看一个处理长选项的例子。

```python
import getopt
import sys


def main(args):
    """
    """
    # 模拟 MySQL 的长命令行选项
    longs = ['host=', 'port=', 'user=', 'password=']
    result = getopt.getopt(args, shortopts='h:P:u:p:', longopts=longs)
    print(result)


if __name__ == "__main__":
    main(sys.argv[1:])
```

运行效果。

```bash
python3 main.py --user=root --password=123456 --host=127.0.0.1 --port=3306
([('--user', 'root'), ('--password', '123456'), ('--host', '127.0.0.1'), ('--port', '3306')], [])
```

---


## getopt 实战
写一个相对完成的例子，感受一下用 `getopt` 处理参数代码会有多长(文件名假设为 mysql.py)。
```python
#!/usr/bin/env python3

import sys
import getopt

def print_help():
    help_text="""
mysql.py 使用帮助

    mysql.py 选项参数  位置参数

    选项参数列表
        -u  用户名
        -p  密码
        -h  服务端ip
        -P  服务端端口
        --defaults-file 默认配置文件
    
    位置参数
        数据库名
    """
    print(help_text)
    sys.exit()

if __name__ == "__main__":
    try:
        # 取得命令行参数
        args = sys.argv[1:]
        # 告知 getopt 有哪些短选项
        short = 'u:p:h:P:'
        long = ['defaults-file=',]
        # getopt 会去处理选项参数与位置参数并返回一个列表
        options= getopt.getopt(args,short,longopts=long)
        # 通过 for 循环处理参数
        config = {}
        for k,v in options[0]:
            if k == '-u':
                config['user'] = v
            elif k == '-p':
                config['password'] = v
            elif k == '-h':
                config['host'] = v 
            elif k == '-P':
                config['port'] = v
            elif k == '--defaults-file':
                config['defaults-file'] = v
        # 如果没有传递参数直接打印命令帮助，并退出
        if not config:
            print_help()
    except getopt.GetoptError as err:
        print("{0}".format(err),file=sys.stderr)
        sys.exit(1)

    print(config)
```

运行效果。

```bash
# 不传递参数的情况下打印帮助信息
python3 mysql.py

mysql.py 使用帮助

    mysql.py 选项参数  位置参数

    选项参数列表
        -u  用户名
        -p  密码
        -h  服务端ip
        -P  服务端端口
    
    位置参数
        数据库名

# 传递参数的情况下打印收到的信息
python3 mysql.py -uroot -p123456 -h127.0.0.1 -P3306 performance_schema
{'user': 'root', 'password': '123456', 'host': '127.0.0.1', 'port': '3306'}
```



