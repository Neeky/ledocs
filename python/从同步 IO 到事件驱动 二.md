## select 存在的问题
前文 [从同步 IO 到事件驱动 一](/blogs/10802075)  介绍了 select，但是 select 有一个问题，那就是最多只支持 1024 句柄，虽然可以通过自己编译操作系统的方式把这个限制调大，那为什么不直接使用没有限制的 poll 或 epoll 来做事件循环呢？

![sqlpy](static/2020-24/sqlpy-reactor.jpg)


---


## poll 的服务端代码实现
功能方面和前文是一样的，所以这里不多做介绍了，直接上代码。
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
    '%(asctime)s - %(name)s  - %(levelname)s - %(message)s')
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

    def process_event(self):
        """事件内部只会调用这一个函数
        """
        raise NotImplementedError('')


class TimerServer(EventHandler):
    """
    """
    logger = logger.getChild("TimerServer")

    def __init__(self):
        """
        """
        logger = self.logger.getChild("__init__")
        logger.info("初始化 TimeServer 实例")

        self._sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self._sock.setblocking(False)
        self._sock.bind(('127.0.0.1', 3333))
        self._loop = None

        logger.info(f"TimeServer 实例的 fileno 为 {self.fileno()}")
        logger.info("初始化 TimeServer 实例完成")

    def fileno(self):
        return self._sock.fileno()

    def wants_read(self):
        logger = self.logger.getChild("wants_read")
        logger.info("注册可读事件")
        self._loop.modify(self.fileno(), select.POLLIN)

    def wants_write(self):
        logger = self.logger.getChild("wants_write")
        logger.info("注册可写事件")
        self._loop.modify(self.fileno(), select.POLLOUT)

    def read(self):
        """
        """
        logger = self.logger.getChild("read")

        message, addrs = self._sock.recvfrom(1024)
        message = message.decode('utf8')

        logger.info(f"收到 {addrs} 发来的 {message}")
        self._client = addrs

    def write(self):
        now = str(datetime.now()).encode('utf8')
        self._sock.sendto(now, self._client)

        self._client = None

    def process_event(self, fileno=None, event=None):
        """
        """
        logger = self.logger.getChild("process_event")
        if self.fileno() != fileno:
            logger.warning("return ")
            return

        if event & select.POLLIN:
            self.read()
            self.wants_write()

        if event & select.POLLOUT:
            self.write()
            self.wants_read()

    def add_to_event_loop(self, loop: select.poll):
        """
        """
        logger = self.logger.getChild("add_to_event_loop")
        logger.info("把 TimeServer 实例关联到事件循环")
        if self._loop == loop:
            return

        self._loop = loop
        loop.register(self.fileno(), select.POLLIN)
        logger.info("TimeServer 实例注册可读事件完成")


def main():
    """
    """
    loop = select.poll()
    ts = TimerServer()
    ts.add_to_event_loop(loop)
    # 记录事件与处理器的对应关系
    handlers = {}
    handlers[ts.fileno()] = ts
    i = 0
    while True:
        for fileno, event in loop.poll():

            logger.info(f"fileno {fileno}  event {event} i = {i}")

            handlers[fileno].process_event(fileno, event)
            logger.info("处事器调用完成 .")
            i = i + 1


if __name__ == "__main__":
    main()

```

google-adsense

---

## 运行效果
1、先运行起服务端。
```bash
python3 poll-timeserver.py 
2020-06-14 15:36:05,536 - time-server.TimerServer.__init__  - INFO - 初始化 TimeServer 实例
2020-06-14 15:36:05,537 - time-server.TimerServer.__init__  - INFO - TimeServer 实例的 fileno 为 3
2020-06-14 15:36:05,537 - time-server.TimerServer.__init__  - INFO - 初始化 TimeServer 实例完成
2020-06-14 15:36:05,537 - time-server.TimerServer.add_to_event_loop  - INFO - 把 TimeServer 实例关联到事件循环
2020-06-14 15:36:05,537 - time-server.TimerServer.add_to_event_loop  - INFO - TimeServer 实例注册可读事件完成
```
2、运行客户端。
```bash
python3 timeclient.py 
2020-06-14 15:36:19.233660
```
3、这时服务端看到的日志如下。
```bash
2020-06-14 15:36:19,233 - time-server  - INFO - fileno 3  event 1 i = 0
2020-06-14 15:36:19,233 - time-server.TimerServer.read  - INFO - 收到 ('127.0.0.1', 64702) 发来的 time-client
2020-06-14 15:36:19,233 - time-server.TimerServer.wants_write  - INFO - 注册可写事件
2020-06-14 15:36:19,233 - time-server  - INFO - 处事器调用完成 .
2020-06-14 15:36:19,233 - time-server  - INFO - fileno 3  event 4 i = 1
2020-06-14 15:36:19,233 - time-server.TimerServer.wants_read  - INFO - 注册可读事件
2020-06-14 15:36:19,234 - time-server  - INFO - 处事器调用完成 
```

---


