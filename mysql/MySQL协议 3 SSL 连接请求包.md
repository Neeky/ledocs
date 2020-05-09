## 背景
在 client 收到 MySQL-Server 发来的握手包后，如果 client 想把启用 ssl 功能那它要先回一个 `SSL Connection Request Packet` 包。

---

## SSL Connection Request 包格式
SSL 连接请求包还是比较简单的，payload 部分的格式如下。

|**长度(字节)**|**内容**|
|-------------|---------|
|4            | 客户端的特性列表(capability flags)|
|4            | 支持最大的包大小(max-packet size) |
|1            | 字符集(character set) |
|string[23]   | 占位|

---

## 用 Python 打包
用 struct 打包 payload 部分。
```python
import struct

def make_ssl_auth_payload(client_flags=0,charset=45,max_allowed_packet=1073741824):
    buf = struct.pack("<I",client_flags) + struct.pack("<I",max_allowed_packet) + struct.pack("<b",charset) + b'\x00' * 23
    return buf

```

---


## 官方文档
[SSL Connection Request](https://dev.mysql.com/doc/internals/en/connection-phase-packets.html#packet-Protocol::SSLRequest) 包格式的明细。

---