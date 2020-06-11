## 概要
为了搞明白同步 IO 的编程方式和事件驱动的编程方式有什么不同，我设计了一个简单的时间服务器，并提供两种实现方式。这样就可以直观的比较两种编程方式的差异了。

实现一个简单的 C/S 程序，服务端监听在 0.0.0.0 的 3333 端口，当发现有客户端发来数据时，先收取数据，然后再把服务端当前的时间发给客户端。

![sqlpy](static/2020-24/sqlpy-block-io.jpg)

---

## 同步 IO 的服务端代码
以同步 IO 的方式实现的服务端。
```python
#!/usr/bin/env python3

# 客户端通过 UDP 协议向服务端发送信息，服务端把当前的时间值作为返回


import select
import socket
import logging
from datetime import datetime

# 配置日志
logger = logging.getLogger('time-server')
logger.setLevel(logging.INFO)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s')
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)


class TimeServer(object):
    logger = logger.getChild("TimeServer")

    def __init__(self, ip="0.0.0.0", port=3333):
        """
        """
        logger = self.logger.getChild("__init__")
        logger.info(f"TimeServer bind on  ({ip},{port})")
        self._sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self._sock.bind((ip, port))

    def run(self):
        logger = self.logger.getChild("run")
        while True:
            message, addrs = self._sock.recvfrom(1024)
            message = message.decode('utf8')

            logger.info(f"new client coming from {addrs} .")
            logger.info(f"recive message {message} .")

            now = str(datetime.now()).encode('utf8')

            logger.info(f"send {now} to client")
            self._sock.sendto(now, addrs)

            logger.info("complete .")
            logger.info('-'*48)


if __name__ == "__main__":
    ts = TimeServer()
    ts.run()

```

google-adsense

---

## 同步 IO 的客户端代码
采用同步 IO 实现的客户端代码如下。
```python
#!/usr/bin/evn python3

import time
import socket
import argparse


def main():
    parser = argparse.ArgumentParser('time-client')
    parser.add_argument('--host', default='127.0.0.1')
    parser.add_argument('--port', default=3333, type=int)
    args = parser.parse_args()

    client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    message = b'time-client'
    client.sendto(message, (args.host, args.port))
    time_bytes = client.recv(1024)
    print(time_bytes.decode('utf8'))

    client.close()


if __name__ == "__main__":
    main()

```

---

## 运行效果。
1、先运行服务端。
```bash
python3 timeserver.py 
2020-06-11 13:52:38,709 - time-server.TimeServer.__init__ - MainThread - INFO - TimeServer bind on  (0.0.0.0,3333)
```
---

2、运行客户端。
```bash
python3 timeclient.py 
2020-06-11 14:15:21.267058
```
---
3、当收到客户端信息时服务端的日志如下。
```bash
2020-06-11 14:15:21,266 - time-server.TimeServer.run - MainThread - INFO - new client coming from ('127.0.0.1', 55661) .
2020-06-11 14:15:21,267 - time-server.TimeServer.run - MainThread - INFO - recive message time-client .
2020-06-11 14:15:21,267 - time-server.TimeServer.run - MainThread - INFO - send b'2020-06-11 14:15:21.267058' to client
2020-06-11 14:15:21,267 - time-server.TimeServer.run - MainThread - INFO - complete .
2020-06-11 14:15:21,267 - time-server.TimeServer.run - MainThread - INFO - ------------------------------------------------
```

---


## select 
select 是操作系统提供的 API。之前我们想接收数据就要调用 socket 对象的 recv 方法，但是对端并没有发送任何数据的话，我方的 recv 方法就被卡住。 进一步导致整个程序都暂停在了这里，表现出来的样子就是这个程序卡死了。

借用操作系统提供的 select 我们可以在对端已经把数据发送到我方网卡的时候收到一个可读的“事件”，同样在我方网上的写缓存没有满的时候就可以收到可写的“事件”。这就是整个 select 的核心理论，之前是主动的去等 IO ，现在变成了订阅事件通知，当可读的时候就去读取，可写的时候就去写入，没有可执行的 IO 操作时就可以处理其它的事情，避免整个程序卡死。

google-adsense

---


## 事件驱动的服务端代码实现
在事件驱动的场景下服务器的逻辑也只不过是一个“事件处理器”而已，所以代码里都会有一个叫 "EventHandler" 的抽象，在这里我也按这个套路来写。

```python
#!/usr/bin/env python3

# 客户端通过 UDP 协议连接上服务端，查询服务端的当前系统时间


import select
import socket
import logging
from datetime import datetime

# 配置日志
logger = logging.getLogger('time-server')
logger.setLevel(logging.INFO)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s')
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)


class EventHandler(object):
    """定义事件处理器要实现的接口
    """

    def fileno(self):
        # 返回事件处理器的句柄
        raise NotImplementedError('')

    def wants_write(self):
        raise NotImplementedError('')

    def wants_read(self):
        raise NotImplementedError('')

    def write(self):
        raise NotImplementedError('')

    def read(self):
        raise NotImplementedError('')


class TimeServer(EventHandler):
    """实现时间服务器的服务端
    """
    logger = logger.getChild("TimeServer")

    def __init__(self, ip="0.0.0.0", port=3333):
        logger = self.logger.getChild("__init__")
        logger.info(f"TimeServer bind on  ({ip},{port})")
        self._sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self._sock.bind((ip, port))

        # 由前面的约定我们知道，TimeServer 先是接收 Client 发来的信息
        # 所以这里 _wants_read 为标记成了 True
        # 是为了逻辑上对的上。
        self._wants_read = True
        self._wants_write = False

    def fileno(self):
        return self._sock.fileno()

    def wants_read(self):
        """
        """
        return self._wants_read

    def wants_write(self):
        return self._wants_write

    def read(self):
        """
        """
        logger = self.logger.getChild("read")
        message, addrs = self._sock.recvfrom(1024)
        message = message.decode('utf8')
        logger.info(f"recive '{message}' from {addrs}'")

        # 接下来要把服务器的当前信息写回去，所以要把这个设置为 True
        self._wants_write = True
        self._wants_read = False
        self._client = addrs

    def write(self):
        """
        """
        logger = self.logger.getChild("write")
        now = str(datetime.now()).encode('utf8')
        logger.info(f"send {now} to {self._client}")

        self._sock.sendto(now, self._client)

        # 写完了之后就把自己标记成想读，不想写
        self._client = None
        self._wants_read = True
        self._wants_write = False


def loop(handers):
    """实现事件循环
    """
    i = 0
    while True:
        # 按业务逻辑找出想读，或想写的事件处理器
        logger.info(f"start {i}")
        reads = [h for h in handers if h.wants_read() == True]
        writes = [h for h in handers if h.wants_write() == True]
        execs = []
        rs, ws, _ = select.select(reads, writes, execs)

        for r in rs:
            r.read()

        for w in ws:
            w.write()

        logger.info(f"end {i}")
        logger.info(f'-' * 48 + '\n')
        i = i + 1


if __name__ == "__main__":
    ts = TimeServer()
    loop([ts])

```

运行效果。

```bash
python3 timeserver.py 
2020-06-11 15:03:38,766 - time-server.TimeServer.__init__ - MainThread - INFO - TimeServer bind on  (0.0.0.0,3333)
2020-06-11 15:03:38,766 - time-server - MainThread - INFO - start 0
2020-06-11 15:03:40,029 - time-server.TimeServer.read - MainThread - INFO - recive 'time-client' from ('127.0.0.1', 60972)
2020-06-11 15:03:40,029 - time-server - MainThread - INFO - end 0
2020-06-11 15:03:40,029 - time-server - MainThread - INFO - ------------------------------------------------

2020-06-11 15:03:40,030 - time-server - MainThread - INFO - start 1
2020-06-11 15:03:40,030 - time-server.TimeServer.write - MainThread - INFO - send b'2020-06-11 15:03:40.030141' to ('127.0.0.1', 60972)
2020-06-11 15:03:40,030 - time-server - MainThread - INFO - end 1
2020-06-11 15:03:40,030 - time-server - MainThread - INFO - ------------------------------------------------

2020-06-11 15:03:40,030 - time-server - MainThread - INFO - start 2
```

---

## 总结
`s, ws, _ = select.select(reads, writes, execs)` 会逐个检查，reads,writes,execs 这三个处理器列表中的，处理器是否可以读，可以写，产生异常，如果是保存到对应的返回数组里；这里直接检查的套节字，不是按业务逻辑来检查的，所以业务逻辑上的是否读写的检查在话在它的前面。

---