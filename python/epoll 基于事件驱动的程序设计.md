## epoll 要解决什么问题
相信大家都有被“背诵全文”支配过的的恐惧，这都放学了，还要背完成课文才能走，伤心。老师的处理方式通常是让已经会背的同步举手，在老师允许他开始背诵之后，同学开始背诵课文，老师确认没问题之后，这位同学就可以放学回家了。

其实上面的解决方案就是一个典型的事件驱动式的任务处理，“举手” 就是事件，老师收到这个事件之后就知道“哦，这个同学会背了，去验收一下”。事件驱动的处理方式优化点效率高，下面看一下其它处理方式。

`同步阻塞式` 老师从学号  1 ~ n 逐个验收，如果遇到了不会背的同学，老师也不会主动的去验收下一个同学，而是就卡在这里，走到这个同步会背了，才验收下一个。这样的话大家都不用回家了。

`异步轮询式` 与同步阻塞不同，当遇到还不会的同学时，老师直接跳过他，去验收下一个。这个的话老师就会比较累了，因为它要一遍一遍的确认班上的同学。

`多线程式或多进程式` 这个要求老师会分身术，这样的话他就可以为每一个同学安排一个分身，各个同学之间并行的进行互不干扰。虽然现实生活中做不到，在计算机上是可以的。

---


## epoll 的工作模式
事件驱动程序通常用于 IO 密集型应用的服务端，用我们刚才的例子来对比，老师就是一个服务端，学生就是客户端，举手就是事件。那么关于举手就有两种不同的模式。第一种 只要某个同学他会背诵了就直接举着手，永不放下，直到老师来验收。 第二种 会背的同步只举手一下，然后就把手放心了(想想如果当时老师没有看到他就GG了)。 在 epoll 中第一种模式叫 Level-Trigger 这个也是默认的模式；第二种模式叫 Edge-Trigger 。

第一种实现起来简单，第二种实现起来困难但可以做更加精细的控制。

---

## epoll 的事件
在上面的例子中“事件”只有一个类型，那就是“会背了”老师也只要关注这一类事件就行，但是在服务端 socket 编程里，事件的类就多了比如“客户端 socket 可读”，“客户端 socket 可写”，目前 epoll 中定义的事件类型列表如下。

|**事件名**|**意义**|
|------------------|--------|
|select.EPOLLIN    | socket 可读|
|select.EPOLLOUT   | socket 可写|
|select.EPOLLERR   | socket 有错误|
|select.EPOLLHUP   | 服务端主动关闭了客户端 socket|
|select.EPOLLRDHUP | 客户端主动关闭了 socket |

完整的列表比这个要多，请参考 [epoll 事件列表](https://docs.python.org/3.8/library/select.html) 。

---


## 设计一个基于 epoll 的程序
设计一个 epoll 的服务端程序，目的是用上 epoll 中的常用事件，并感受一下 Level-Trigger 模式持续触发事件是一个什么状况。

1、服务端接收客户端发来的字节流，为了观察 Level-Trigger 持续触发事件，服务端一次只接收一个字节。2、当服务端接收满三个字节后，向客户端发送当前的时间值。3、服务端进行资源回收。

A、客户端连接上服务端，发送三个字节，然后接收服务端发来的字节流。


---

## 服务端源码实现
服务端的源代码如下。
```python
#!/usr/bin/env python3

import socket
import select
import logging
import argparse
from datetime import datetime

NAME = 'epoll-server'
logger = logging.getLogger(NAME)


class EpollServer(object):
    """
    """
    logger = logger.getChild("EpollServer")

    def __init__(self, host="0.0.0.0", port=9090):
        logger = self.logger.getChild("__init__")
        #
        self._host = host
        self._port = port

        #
        logger.info(f"epoll 服务器将会监听在 {self._host}:{self._port} .")
        logger.info("epoll 服务器初始化完成.")

    def event_loop(self, poll):
        """主事件循环
        """
        while True:
            events = poll.poll()
            for fileno, event in events:
                yield fileno, event

    def run(self):
        """
        """
        logger = self.logger.getChild("run")
        logger.info("服务器开始运行.")

        logger.info("准备创建 epoll 对象.")
        epoll = select.epoll()
        connections = {}
        datas = {}

        with socket.socket(socket.AF_INET, socket.SOCK_STREAM, 6) as server_sock:
            # 重用 ip 和 port
            server_sock.setsockopt(
                socket.SOL_SOCKET, socket.SO_REUSEADDR, True)
            server_sock.setsockopt(
                socket.SOL_SOCKET, socket.SO_REUSEPORT, True)

            logger.info(f"epoll 服务器监听在 {self._host}:{self._port} .")
            server_sock.bind((self._host, self._port))
            server_sock.listen(128)

            #
            logger.info("服务端 socket 进入异步模式.")
            server_sock.setblocking(False)

            # 注册 EPOLLIN 事件
            server_fileno = server_sock.fileno()
            epoll.register(server_fileno, select.EPOLLIN)
            logger.info(f"服务端 socket 文件句柄为 {server_fileno}")

            logger.info("正式进入事件循环.")
            for fileno, event in self.event_loop(epoll):
                # 如果是 Server-Socket 说明是有连接进来了
                logger.info(f"新事件 {event} 文件句柄 {fileno}")

                if fileno == server_fileno:
                    # 接收新的连接请求
                    client_sock, addr = server_sock.accept()
                    client_sock.setblocking(False)
                    logger.info(f"收到来自 {addr} 的连接请求.")

                    client_fileno = client_sock.fileno()
                    connections[client_fileno] = client_sock
                    datas[client_fileno] = b''
                    logger.info(f"客户端 socket 句柄为 {client_fileno}")

                    # 注册新连接关注的事件
                    epoll.register(client_fileno, select.EPOLLIN |
                                   select.EPOLLHUP | select.EPOLLERR | select.EPOLLRDHUP)

                    logger.info("客户端 socket 事件注册完成.")
                    logger.info(connections)

                elif event & (select.EPOLLHUP | select.EPOLLRDHUP):
                    # 断开连接时的处理
                    logger.info(f"错误事件处理.")
                    if hasattr(connections[fileno], 'close'):
                        connections[fileno].close()
                        datas[fileno] = b''
                        epoll.unregister(fileno)
                        del connections[fileno]
                        del datas[fileno]
                elif event & select.EPOLLIN:
                    # 新连接可读
                    # 为了观察效果每一次只读一个字节
                    logger.info(f"可读事件处理.")
                    client = connections[fileno]
                    chunck = client.recv(1)
                    datas[fileno] = datas[fileno] + chunck
                    logger.info(f"收到数据 {chunck} .")

                    # 假设客户端只会发送三个字节，收到三个字节后就直接切换成关注客户端的写事件
                    if len(datas[fileno]) == 3:
                        logger.info("所有数据接收完成.")
                        datas[fileno] = b''
                        logger.info("准备关注客户端 socket 的可写事件.")
                        epoll.modify(fileno, select.EPOLLOUT |
                                     select.EPOLLHUP | select.EPOLLERR | select.EPOLLRDHUP)
                        logger.info(epoll)

                elif event & select.EPOLLOUT:
                    # 客户端可写
                    logger.info(f"可写事件处理.")
                    client = connections[fileno]
                    now = str(datetime.now()).encode('utf8')
                    datas[fileno] = now

                    length = client.send(datas[fileno])
                    datas[fileno] = datas[fileno][length:]

                    if len(datas[fileno]) == 0:
                        # 说明发完了、那么单方面关闭连接
                        logger.info(
                            "数据发送完成，准备关闭客户端 socket.")
                        epoll.modify(fileno, select.EPOLLHUP |
                                     select.EPOLLERR | select.EPOLLRDHUP)

                        client_sock = connections[fileno]
                        client_sock.close()
                        logger.info("客户端 socket 清理完成.")
                        logger.info(f"{epoll}")


if __name__ == "__main__":
    # 解析命令行参数
    parser = argparse.ArgumentParser(NAME)
    parser.add_argument('--host', default="0.0.0.0")
    parser.add_argument('--port', default=9090)
    # 配置日志
    logger = logging.getLogger(NAME)
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s')
    logger.setLevel(logging.DEBUG)
    stream_hander = logging.StreamHandler()
    stream_hander.setLevel(logging.DEBUG)
    stream_hander.setFormatter(formatter)
    logger.addHandler(stream_hander)

    #
    eps = EpollServer()
    eps.run()

```

---

## 运行效果
客户端日志。
```python
In [1]: import socket                                                           

In [2]: sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM, 6)             

In [3]: sock.connect(('172.16.192.100',9090))                                   

In [4]: sock.send(b'Now')                                                       
Out[4]: 3

In [5]: sock.recv(1024)                                                         
Out[5]: b'2020-05-29 17:11:23.484237'
```
服务端日志。
```bash
python3 epoll-server.py 
2020-05-29 17:10:22,997 - epoll-server.EpollServer.__init__ - MainThread - INFO - epoll 服务器将会监听在 0.0.0.0:9090 .
2020-05-29 17:10:22,997 - epoll-server.EpollServer.__init__ - MainThread - INFO - epoll 服务器初始化完成.
2020-05-29 17:10:22,997 - epoll-server.EpollServer.run - MainThread - INFO - 服务器开始运行.
2020-05-29 17:10:22,997 - epoll-server.EpollServer.run - MainThread - INFO - 准备创建 epoll 对象.
2020-05-29 17:10:22,998 - epoll-server.EpollServer.run - MainThread - INFO - epoll 服务器监听在 0.0.0.0:9090 .
2020-05-29 17:10:22,999 - epoll-server.EpollServer.run - MainThread - INFO - 服务端 socket 进入异步模式.
2020-05-29 17:10:22,999 - epoll-server.EpollServer.run - MainThread - INFO - 服务端 socket 文件句柄为 4
2020-05-29 17:10:22,999 - epoll-server.EpollServer.run - MainThread - INFO - 正式进入事件循环.
2020-05-29 17:11:07,311 - epoll-server.EpollServer.run - MainThread - INFO - 新事件 1 文件句柄 4
2020-05-29 17:11:07,311 - epoll-server.EpollServer.run - MainThread - INFO - 收到来自 ('172.16.192.1', 52711) 的连接请求.
2020-05-29 17:11:07,312 - epoll-server.EpollServer.run - MainThread - INFO - 客户端 socket 句柄为 5
2020-05-29 17:11:07,312 - epoll-server.EpollServer.run - MainThread - INFO - 客户端 socket 事件注册完成.
2020-05-29 17:11:07,312 - epoll-server.EpollServer.run - MainThread - INFO - {5: <socket.socket fd=5, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=6, laddr=('172.16.192.100', 9090), raddr=('172.16.192.1', 52711)>}
2020-05-29 17:11:23,482 - epoll-server.EpollServer.run - MainThread - INFO - 新事件 1 文件句柄 5
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 可读事件处理.
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 收到数据 b'N' .
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 新事件 1 文件句柄 5
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 可读事件处理.
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 收到数据 b'o' .
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 新事件 1 文件句柄 5
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 可读事件处理.
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 收到数据 b'w' .
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 所有数据接收完成.
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - 准备关注客户端 socket 的可写事件.
2020-05-29 17:11:23,483 - epoll-server.EpollServer.run - MainThread - INFO - <select.epoll object at 0x7feae7a04ed0>
2020-05-29 17:11:23,484 - epoll-server.EpollServer.run - MainThread - INFO - 新事件 4 文件句柄 5
2020-05-29 17:11:23,484 - epoll-server.EpollServer.run - MainThread - INFO - 可写事件处理.
2020-05-29 17:11:23,485 - epoll-server.EpollServer.run - MainThread - INFO - 数据发送完成，准备关闭客户端 socket.
2020-05-29 17:11:23,486 - epoll-server.EpollServer.run - MainThread - INFO - 客户端 socket 清理完成.
```
从日志中可以看到只要有字节可以读取，就会持续的触发 EPOLLIN 。

---