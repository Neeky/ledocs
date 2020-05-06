## MySQL 应用层协议的包格式
MySQL 数据库的客户端与服务端通信使用的是 MySQL协议(classic)，这是一个二进制协议，数据包的格式如下。

|**payload_length**|**sequence_id**|**payload**|
|------------------|---------------|-----------|
|int<3>            |int<1>         |           |

数据包的最前面三个字节用于描述`payload`部分的长度，第四个字节是一个包的序列号，后面剩下的都是包的内容部分。

---

## 接收 MySQL 的应用层包
当 TCP 连接完成之后 MySQL-Server 会发送一个应用层数据包到 client ，client 用这个包来完成应用层的握手；下面看用 Python 如何处理。
```python
#!/usr/bin/env python3

import sys
import struct
import socket
import logging

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s")


def read_payload(host, port):
    """连接上 MySQL-Server 找接收 MySQL-Server 发来的数据包。
    """
    try:
        # 解析 MySQL-Server 的地址
        addrinfos = socket.getaddrinfo(host, port, 0, socket.SOCK_STREAM)
        logging.info(addrinfos)
        info, *_ = addrinfos
        family, sock_type, proto, _, addr = info

        # 创建套节字，并连接到 MySQL-Server
        sock = socket.socket(family, sock_type, proto)
        sock.settimeout(3)
        sock.connect(addr)

    except IOError as err:
        logging.exception(err)
        sys.exit()

    # 执行到这里说明连接成功了
    try:
        # 读取包头
        packet = bytearray(b'')
        packet_len = 0
        while packet_len < 4:
            chunk = sock.recv(4 - packet_len)
            packet = packet + chunk
            packet_len = packet_len + len(chunk)

        #packet_number = packet[3]
        payload_len, *_ = struct.unpack("<I", packet[0:3] + b'\x00')

        # 读取包体(payload)的内容
        # 由于 payload 可能非常的大，这里一次性的 extend 完所需要的空间，避免之后多次的内存复制。
        rest = payload_len
        packet.extend(bytearray(payload_len))
        packet_view = memoryview(packet)
        packet_view = packet_view[4:]

        while rest:
            read = sock.recv_into(packet, rest)
            if read == 0 and rest > 0:
                # 如果在读取的过程中 TCP 连接被断开就会造成 read == 0
                raise Exception(errno=2013)
            packet_view = packet_view[:read]
            rest = rest - read

        # 返回读取到的 payload
        return packet
    except IOError as err:
        logging.exception(err)


if __name__ == "__main__":
    payload = read_payload("sqlstudio", "mysql")
    logging.info(payload)

```
运行效果。
```bash
python3 client.py 

2020-05-06 15:37:52,895 - root - MainThread - INFO - [(<AddressFamily.AF_INET: 2>, <SocketKind.SOCK_STREAM: 1>, 6, '', ('172.16.192.100', 3306))]
2020-05-06 15:37:52,896 - root - MainThread - INFO - bytearray(b'\n8.0.20\x00#\x00\x00\x00u8b\x1364[\x11\x00\xff\xff\xff\x02\x00\xff\xc7\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x05\x15\x11lsJ\x04\x1d\rr\x7fK\x00caching_sha2_password\x00\x00\x00\x00\x00')
```

google-adsense

---


## 其它

[MySQL 协议](#https://dev.mysql.com/doc/internals/en/integer.html) 中明确的指出了，对于固定长度的整数使用的是小端方式保存的，也就是说在用 `struct` 解包的时候要用“<” 格式。

---



