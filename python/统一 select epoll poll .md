## 背景
在解决 C10k 、C100k 问题时，不可避免的要用到 epoll、poll、select 三者之一。然而 linux 上最好的是 epoll ，这个在 mac 上又没有，所以我在 mac 退而求其次使用 poll，但对于 windows 而言又只能用 select 。

如果我在代码里写死了用 epoll 那么它是可以在 linux 上运行起来了，但是它在我的开发机(mac)上运行不起来，为了一次性的解决这种平台相关的不兼容，我决定写一个抽象层。一方面它可以隔离 epoll、poll、select 这三个物理上的实现，另一方面它又对“反应器模式”友好。

![sqlpy](static/2020-25/sqlpy-epoll-select.jpg)

---


## 统一事件循环
1、第一步 定义事件枚举统一事件 ID。
```python
import enum

class UEvent(enum.IntFlag):
    """event mask
    """
    poll_none = 0x00
    poll_in = 0x01
    poll_out = 0x04
    poll_err = 0x08
    poll_hup = 0x10
    poll_nval = 0x20
```

google-adsense

---

2、第二步 封装 select 让它看起来像 epoll 。
```python
class SelectLoop(object):
    """封装 select 事件循环
    """

    def __init__(self):
        """
        """
        # 可读句柄列表
        self._reads = []
        # 可写句柄列表
        self._writes = []
        # 异常句柄列表
        self._execes = []

    def register(self, fileno: int, mode: UEvent):
        """注册事件
        """
        if mode & UEvent.poll_in:
            self._reads.append(fileno)

        if mode & UEvent.poll_out:
            self._writes.append(fileno)

        if mode & UEvent.poll_err:
            self._execes.append(fileno)

    def unregister(self, fileno: int):
        """取消事件
        """
        if fileno in self._reads:
            self._reads.remove(fileno)

        if fileno in self._writes:
            self._writes.remove(fileno)

        if fileno in self._execes:
            self._execes.remove(fileno)

    def modify(self, fileno: int, mode: UEvent):
        """
        """
        self.unregister(fileno)
        self.register(fileno, mode)

    def close(self):
        """
        """
        pass

    def poll(self):
        """
        """
        r, w, e = select.select(self._reads, self._writes, self._execes)
        for fileno in r:
            yield fileno, UEvent.poll_in

        for fileno in w:
            yield fileno, UEvent.poll_out

        for fileno in e:
            yield fileno, UEvent.poll_err

```
---

3、第三步 实现工厂类隔离三个不同的物理实现。
```python
# 配置日志
import logging
logger = logging.getLogger('Uloop')
logger.setLevel(logging.INFO)
stream_handler = logging.StreamHandler()
formatter = logging.Formatter(
    '%(asctime)s - %(name)s  - %(levelname)s - %(message)s')
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)

class Uloop(object):
    """封装 epoll,poll,select 为一个事件循环
    """
    logger = logger.getChild("Uloop")

    def __init__(self):
        """
        """
        logger = self.logger.getChild("__init__")
        # self._loop 事件循环的低层实现方法
        # self._impl 实现的方法名
        if hasattr(select, 'epoll'):
            self._loop = select.epoll()
            self._impl = 'epoll'
        elif hasattr(select, 'poll'):
            self._loop = select.poll()
            self._impl = 'poll'
        elif hasattr(select, 'select'):
            self._loop = SelectLoop()
            self._impl = 'select'
        else:
            raise Exception(
                "can't find any available functions in select  model .")

        logger.info(f"使用 {self._impl} 实现")

        # self._handers 保存句柄与处理器之间的对应关系
        # {fileno:EventHandler}
        self._handlers = {}

        # self._last_time
        self._last_time = time.time()

        # self._periodic_callbacks 周期性回调函数列表
        self._periodic_callbacks = []

        # 是否停止
        self._stoping = False

        # poll 的超时时间
        self.timeout = 10

    def add(self, fileno, mode: UEvent, eventhandler):
        """注册文档句柄到事件循环
        """
        logger = self.logger.getChild("add")

        if not isinstance(fileno, int):
            # fileno 不是句柄号，就调用 fileno 方法
            fileno = fileno.fileno()

        #
        logger.info(f"添加 fileno = {fileno} 到事件循环 .")

        # 记录句柄与处理器的关系
        self._handlers[fileno] = (fileno, eventhandler)
        # 注册事件
        self._loop.register(fileno, mode)

    def remove(self, fileno):
        """从事件循环中移除句柄
        """
        logger = self.logger.getChild("remove")

        if not isinstance(fileno, int):
            fileno = fileno.fileno()

        if fileno in self._handlers:
            self._loop.unregister(fileno)
            del self._handlers[fileno]
            logger.info(f"从事件循环中移除 {fileno}")

    def modify(self, fileno, mode: UEvent):
        """修改句柄注册的事件
        """
        logger = self.logger.getChild("modify")

        if not isinstance(fileno, int):
            fileno = fileno.fileno()

        if fileno in self._handlers:
            # 如果在
            self._loop.modify(fileno, mode)
            logger.info(f"{fileno} 将关注 {mode} 事件")

    def add_periodic(self, callback):
        """添加周期性回调函数
        """
        self._periodic_callbacks.append(callback)

    def remove_periodic(self, callback):
        """移除周期性回调函数
        """
        self._periodic_callbacks.remove(callback)

    def stop(self):
        self._stoping = True

    def close(self):
        self._loop.close()

    def poll(self, timeout=10):
        """
        """
        return [(self._handlers[fileno][1], fileno, event)for fileno, event in self._loop.poll(timeout)]

    def run(self):
        """启动事件循环
        """
        logger = self.logger.getChild("run")
        while not self._stoping:
            # 只要没有设置 self._stopping == True 就一直执行下去

            # 接收事件
            has_error = False
            try:
                events = self.poll(self.timeout)
            except (OSError, IOError) as err:
                has_error = True
                logging.error(str(err))

            # 处理事件
            for handler, fileno, event in events:
                logger.info(f"新事件 event = {event}  fileno = {fileno} .")
                try:
                    handler.process_event(fileno, event)
                except Exception as err:
                    logging.error(str(err))
                    logging.exception(str(err))

            # 处理周期性任务和异常
            now = time.time()
            if has_error or (now - self._last_time) > self.timeout:
                for callback in self._periodic_callbacks:
                    callback()
                self._last_time = now

```
google-adsense

---

## 使用 Uloop 编写事件驱动的程序
文章的后半部分会给出用 Uloop 实现 UDP-Server 和 TCP-Server ，在这里先定义一下 C/S 之间的协议。第一步 C 向 S 发布几个字节的数据； 第二步 S 在收到 C 发来的数据后把自己主机的当前时间发给 C 。

---

## UDP服务端的实现
服务端程序使用了反应器模式，整个服务端是一个反应器，针对事件循环产生的事件做出反应。
```python
import socket
from datetime import datetime

class EventHandler(object):
    """
    """
    def fileno(self):
        raise NotImplementedError('')

class EchoServer(EventHandler):
    """
    """
    def __init__(self):
        self._sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self._sock.setblocking(False)
        self._sock.bind(('0.0.0.0', 3333))
        self.client_addrs = None
        self.message = None
        
    def fileno(self):
        return self._sock.fileno()

    def add_to_event_loop(self, loop: Uloop):
        """
        """
        loop.add(self.fileno(), UEvent.poll_in, self)
        self.loop = loop

    def read(self):
        """
        """
        message, addrs = self._sock.recvfrom(1024)
        self.message = message
        self.client_addrs = addrs
        self.loop.modify(self.fileno(), UEvent.poll_out)

    def write(self):
        self._sock.sendto(str(datetime.now()).encode(
            'utf8'), self.client_addrs)
        self.message = None
        self.client_addrs = None
        self.loop.modify(self.fileno(), UEvent.poll_in)

    def process_event(self, fileno, event):
        if fileno != self.fileno():
            return
        if event & UEvent.poll_in:
            self.read()
        elif event & UEvent.poll_out:
            self.write()

es = EchoServer()
loop = Uloop()
es.add_to_event_loop(loop)
loop.run()
```

google-adsense

---


## TCP 服务端的实现
一个 reactor 模式的 TCP 服务端。
```python
import socket
from datetime import datetime

class EventHandler(object):
    def fileno(self):
        raise NotImplementedError('')

    def add_to_event_loop(self, loop: Uloop):
        raise NotImplementedError('')

    def process_event(self, fileno, event):
        raise NotImplementedError('')

class ClientHandler(EventHandler):
    """
    """
    logger = logger.getChild("ClientHandler")

    def __init__(self, sock: socket.socket):
        self._sock = sock

    def __del__(self):
        logger = self.logger.getChild("__del__")
        if hasattr(self._sock, 'close'):
            logger.info("ClientHandler 垃圾回收")
            self._sock.close()

    def fileno(self):
        return self._sock.fileno()

    def add_to_event_loop(self, loop: Uloop):
        loop.add(self.fileno(), UEvent.poll_in, self)
        self.loop = loop

    def read(self):
        logger = self.logger.getChild("read")
        message = self._sock.recv(1024)
        message = message.decode('utf8')
        logger.info(f"收到数据 '{message}' ")
        self.loop.modify(self.fileno(), UEvent.poll_out)

    def write(self):
        logger = self.logger.getChild("write")
        now = str(datetime.now()).encode('utf8')
        self._sock.send(now)
        self.loop.modify(self.fileno(), UEvent.poll_in)

    def process_event(self, fileno, event):
        if fileno != self.fileno():
            return
        #
        if event & UEvent.poll_hup:
            self.loop.remove(self.fileno())
        elif event & UEvent.poll_in:
            self.read()
        elif event & UEvent.poll_out:
            self.write()

class TcpServerHandler(EventHandler):
    logger = logger.getChild("TcpServerHandler")

    def __init__(self):
        self._sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self._sock.setblocking(False)
        self._sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
        self._sock.bind(('127.0.0.1', 3333))
        self._sock.listen(128)

    def __del__(self):
        if hasattr(self._sock, 'close'):
            self._sock.close()

    def fileno(self):
        return self._sock.fileno()

    def add_to_event_loop(self, loop: Uloop):
        loop.add(self.fileno(), UEvent.poll_in, self)
        self.loop = loop

    def read(self):
        logger = self.logger.getChild("read")
        sock, adrrs = self._sock.accept()
        logger.info(f"收到了来自 {adrrs} 的连接.")
        client = ClientHandler(sock)
        client.add_to_event_loop(self.loop)

    def process_event(self, fileno, event):
        if fileno != self.fileno():
            return
        self.read()


tsh = TcpServerHandler()
loop = Uloop()
tsh.add_to_event_loop(loop)
loop.run()
```
运行效果。

1、服务端日志。
```bash
python3 tcp-server.py 
2020-06-15 15:34:16,139 - Uloop.Uloop.__init__  - INFO - 使用 poll 实现
2020-06-15 15:34:16,139 - Uloop.Uloop.add  - INFO - 添加 fileno = 3 到事件循环 .
2020-06-15 15:34:28,692 - Uloop.Uloop.run  - INFO - 新事件 event = 1  fileno = 3 .
2020-06-15 15:34:28,692 - Uloop.TcpServerHandler.read  - INFO - 收到了来自 ('127.0.0.1', 55339) 的连接.
2020-06-15 15:34:28,692 - Uloop.Uloop.add  - INFO - 添加 fileno = 4 到事件循环 .
2020-06-15 15:34:33,898 - Uloop.Uloop.run  - INFO - 新事件 event = 1  fileno = 4 .
2020-06-15 15:34:33,898 - Uloop.ClientHandler.read  - INFO - 收到数据 'hello sock' 
2020-06-15 15:34:33,899 - Uloop.Uloop.modify  - INFO - 4 将关注 4 事件
2020-06-15 15:34:33,899 - Uloop.Uloop.run  - INFO - 新事件 event = 4  fileno = 4 .
2020-06-15 15:34:33,899 - Uloop.Uloop.modify  - INFO - 4 将关注 1 事件
```
2、客户端日志。
```bash
import socket
In [1]: sock = socket.socket(socket.AF_INET,socket.SOCK_STREAM)                 

In [2]: sock.connect(('127.0.0.1',3333))                                       

In [3]: sock.send(b'hello sock')                                               
Out[3]: 10

In [4]: sock.recv(1024)                                                        
Out[4]: b'2020-06-15 15:34:33.899256'
```
google-adsense

---

## 安装 posix-pythonic-tool
为了下一次写代码的时候可以直接使用 Uloop，我把它发布到了 PyPI，要使用 Uloop 要先安装。
```bash
pip3 install posix-pythonic-tool
```
---

## 使用 posix-pythonic-tool
完成安装后导入就能使用了。
```python
In [1]: from ppt.eventloop import UEvent,Uloop                                  

In [2]:
```
---

## 源代码

源代码托管在 gitub [posix-pythonic-tool](https://github.com/Neeky/posix-pythonic-tool) 。

---