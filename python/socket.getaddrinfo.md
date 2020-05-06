## 背景
在网络编程中的一个常见问题就是把代码写的太死了，比如下面的代码。
```python
#!/usr/bin/env python3

import socket
import logging

logging.basicConfig(level=logging.INFO)


def get_mysql_version(host, port):
    """连接上给定的 MySQL 实例，并返回 MySQL-Server 的版本号。
    """
    logging.info("start.")
    logging.info(
        f"call function get_mysql_version with host {host} port {port}.")

    # 连接到给定的 MySQL-Server
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((host, port))

    # 解析版本号
    data = client.recv(1024)
    version = data[5:11].decode('utf8')

    logging.info("complete.")
    return version


if __name__ == "__main__":
    version = get_mysql_version('172.16.192.100', 3306)

    logging.info(f"MySQL-Server is {version}")
```
运行效果。
```bash
python3 mysql-client.py

INFO:root:start.
INFO:root:call function get_mysql_version with host 172.16.192.100 port 3306.
INFO:root:complete.
INFO:root:MySQL-Server is 8.0.20
```
上面的代码直接把 IP 写死了，如果服务端的 IP 发生了变化，那么代码就要做相应的改动。解决办法是什么呢？答案是 DNS 解析。

google-adsense

---

## 使用 socket.getaddrinfo
如果服务端有提供域名接入，一切都好处理了。上面例子中“172.16.192.100”对应的域名是“sqlstudio”,从 ping 的输出也可以看出来。
```bash
ping sqlstudio
PING sqlstudio (172.16.192.100): 56 data bytes
64 bytes from 172.16.192.100: icmp_seq=0 ttl=64 time=0.301 ms
64 bytes from 172.16.192.100: icmp_seq=1 ttl=64 time=0.587 ms
```
`socket.getaddrinfo` 用于完成 DNS 解析与套节字参数生成。

```python
In [1]: import socket                                                                          

In [2]: socket.getaddrinfo("sqlstudio","mysql")                                                
Out[2]: 
[(<AddressFamily.AF_INET: 2>,
  <SocketKind.SOCK_DGRAM: 2>,
  17,
  '',
  ('172.16.192.100', 3306)),
 (<AddressFamily.AF_INET: 2>,
  <SocketKind.SOCK_STREAM: 1>,
  6,
  '',
  ('172.16.192.100', 3306))]
```

使用 socket.getaddrinfo 改写代码。

```python
#!/usr/bin/env python3

import socket
import logging

logging.basicConfig(level=logging.INFO)


def get_mysql_version(host, port):
    """连接上给定的 MySQL 实例，并返回 MySQL-Server 的版本号。
    """
    logging.info("start.")
    logging.info(
        f"call function get_mysql_version with host {host} port {port}.")

    # 获取地址信息列表
    addrinfos = socket.getaddrinfo(host, port, 0, socket.SOCK_STREAM)
    # 使用列表中的第一项创建套节字
    family, socket_type, proto, _, addrs = addrinfos[0]

    # 连接到给定的 MySQL-Server
    client = socket.socket(family, socket_type, proto)
    client.settimeout(3)
    client.connect(addrs)

    # 解析版本号
    data = client.recv(1024)
    version = data[5:11].decode('utf8')

    logging.info("complete.")
    return version


if __name__ == "__main__":
    version = get_mysql_version('sqlstudio', 'mysql')

    logging.info(f"MySQL-Server is {version}")


```

运行效果。

```bash
python3 mysql-client.py 

INFO:root:start.
INFO:root:call function get_mysql_version with host sqlstudio port mysql.
INFO:root:complete.
INFO:root:MySQL-Server is 8.0.20
```

---

## 注意事项
一个比较常见的问题是，当无法完成 DNS 解析时，socket.getaddrinfo 会报异常，所以也要把这个异常情况考虑进去。
```python
#!/usr/bin/env python3

import sys
import socket
import logging

logging.basicConfig(level=logging.INFO)


def get_mysql_version(host, port):
    """连接上给定的 MySQL 实例，并返回 MySQL-Server 的版本号。
    """
    logging.info("start.")
    logging.info(
        f"call function get_mysql_version with host {host} port {port}.")
    try:
        # 获取地址信息列表
        addrinfos = socket.getaddrinfo(host, port, 0, socket.SOCK_STREAM)
        # 使用列表中的第一项创建套节字
        family, socket_type, proto, _, addrs = addrinfos[0]
    except IOError as err:
        logging.exception(err)
        logging.error(err)
        sys.exit()

    # 连接到给定的 MySQL-Server
    client = socket.socket(family, socket_type, proto)
    client.settimeout(3)
    client.connect(addrs)

    # 解析版本号
    data = client.recv(1024)
    version = data[5:11].decode('utf8')

    logging.info("complete.")
    return version


if __name__ == "__main__":
    version = get_mysql_version('sqlstudio123', 'mysql')

    logging.info(f"MySQL-Server is {version}")

```
由于 `sqlstudio123` 无法完成解析，所以会引发异常，运行效果如下。
```bash
python3 mysql-client.py 

INFO:root:start.
INFO:root:call function get_mysql_version with host sqlstudio123 port mysql.
ERROR:root:[Errno 8] nodename nor servname provided, or not known
Traceback (most recent call last):
  File "mysql-client.py", line 18, in get_mysql_version
    addrinfos = socket.getaddrinfo(host, port, 0, socket.SOCK_STREAM)
  File "/Library/Frameworks/Python.framework/Versions/3.8/lib/python3.8/socket.py", line 914, in getaddrinfo
    for res in _socket.getaddrinfo(host, port, family, type, proto, flags):
socket.gaierror: [Errno 8] nodename nor servname provided, or not known
ERROR:root:[Errno 8] nodename nor servname provided, or not known
```

---