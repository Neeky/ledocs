## 概要
当 MySQL 客户端连接到 MySQL 服务端时，服务端发送一个握手包，根据服务端版本的不同，数据包也有所差异，本文只讨论 MySQL-8.0.x 版本。

---

## 握手数据包的格式
握手包 payload 部分的格式如下。
|**长度(字节)**|**内容**|
|-------------|--------|
|1            | 协议的版本号 ([0a] protocol version)|
|string[NUL]  | 以 `\x00`作为结尾的字符串 (server version)|
|4            | 连接 ID (connection id)| 
|8            | 授权数据 (auth-plugin-data-part-1)|
|1            | 占位   |
|2            | 特性标识位低 16 位 (capability flags)|
|1            | 字符集 (character set)|
|2            | 状态标识位 (status flags)|
|2            | 特性标识位高 16 位 (capability flags)|
|1            | 授权标识长度 (length of auth-plugin-data)|
|string[10]   | 占位   |
|string[NUL]  | 授权数据第二部分|
|string[NUL]  | 授权插件名 |


google-adsense

---

## 用 Python 解析握手数据包

```python
import struct
import logging

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s - %(name)s - %(threadName)s - %(levelname)s - %(message)s")


def read_string(packet, end=None, size=None):
    """从 MySQL 数据库中读取字符串
    返回余下的字节串，和被读到了字符串
    """
    if end is None and size is None:
        # end 和 size 都没有指定时就报错。
        raise ValueError('read_string function needs either end or size')

    if end is not None:
        # 如果指定了 end
        try:
            index = packet.index(end)
        except ValueError as err:
            logging.warning(err)
        return packet[index + 1:], packet[0:index]
    elif size is not None:
        # 如果指定了 size
        return packet[size + 1:], packet[0:size + 1]


def read_int(buf):
    """解包字节序列，返回其所代表的整数。
    """
    try:
        if isinstance(buf, int):
            return buf
        length = len(buf)
        if length == 1:
            return buf[0]
        elif length <= 4:
            tmp = buf + b'\x00'*(4-length)
            return struct.unpack('<I', tmp)[0]
        tmp = buf + b'\x00'*(8-length)
        return struct.unpack('<Q', tmp)[0]
    except:
        raise

def parse_handshake(packet):
    """解析 MySQL 握手包
    """
    logging.info(f"packet len {len(packet)}, packet {packet}.")
    # 解析 MySQL协议的版本号
    res = {}
    res['protocol'] = struct.unpack('<xxxxB', packet[0:5])[0]
    (packet, res['server_version_original']) = read_string(
        packet[5:], end=b'\x00')
    logging.info(f"protocol version {res['protocol']}")

    #
    (res['server_threadid'],
     auth_data1,
     capabilities1,
     res['charset'],
     res['server_status'],
     capabilities2,
     auth_data_length
     ) = struct.unpack('<I8sx2sBH2sBxxxxxxxxxx', packet[0:31])
    res['server_version_original'] = res['server_version_original'].decode()
    logging.info(f"packet-info {res}")
    logging.info(f"auth_data_length {auth_data_length}")

    # packet 的剩余部分
    packet = packet[31:]
    logging.info(f"remaining packet {packet}")

    # 合并特性标识
    capabilities = read_int(capabilities1 + capabilities2)
    auth_data2 = b''

    SECURE_CONNECTION = 1 << 15
    PLUGIN_AUTH = 1 << 19

    if capabilities & SECURE_CONNECTION:
        size = max(13, auth_data_length - 8) if auth_data_length else 13
        logging.info(f"auth-data-part-2 length {size}")
        auth_data2 = packet[0:size]
        packet = packet[size:]
        if auth_data2[-1] == 0:
            # 如果字符串以 \x00 结尾，就把 \x00 去掉
            auth_data2 = auth_data2[:-1]
            logging.info(f"auth-data-2 {auth_data2}")
    logging.info(f"remaining packet {packet}")

    if capabilities & PLUGIN_AUTH:
        # 解析授权插件
        packet, res['auth_plugin'] = read_string(
            packet, end=b'\x00')
        res['auth_plugin'] = res['auth_plugin'].decode('utf-8')
    else:
        # 用 mysql_native_password 插件兜底
        res['auth_plugin'] = 'mysql_native_password'

    logging.info(f"remaining packet {packet}")
    res['auth_data'] = auth_data1 + auth_data2
    res['capabilities'] = capabilities

    logging.info(res)
    return res
```

```bash
python3 parser-handshake.py 

2020-05-07 15:44:13,298 - root - MainThread - INFO - packet len 78, packet b'J\x00\x00\x00\n8.0.20\x00\x0b\x00\x00\x00\x05?r66p\x029\x00\xff\xff\xff\x02\x00\xff\xc7\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x1e\\<PRz\\\x03pNcr\x00caching_sha2_password\x00'.
2020-05-07 15:44:13,298 - root - MainThread - INFO - protocol version 10
2020-05-07 15:44:13,298 - root - MainThread - INFO - packet-info {'protocol': 10, 'server_version_original': '8.0.20', 'server_threadid': 11, 'charset': 255, 'server_status': 2}
2020-05-07 15:44:13,298 - root - MainThread - INFO - auth_data_length 21
2020-05-07 15:44:13,298 - root - MainThread - INFO - remaining packet b'\x1e\\<PRz\\\x03pNcr\x00caching_sha2_password\x00'
2020-05-07 15:44:13,298 - root - MainThread - INFO - auth-data-part-2 length 13
2020-05-07 15:44:13,298 - root - MainThread - INFO - auth-data-2 b'\x1e\\<PRz\\\x03pNcr'
2020-05-07 15:44:13,298 - root - MainThread - INFO - remaining packet b'caching_sha2_password\x00'
2020-05-07 15:44:13,298 - root - MainThread - INFO - remaining packet b''
2020-05-07 15:44:13,298 - root - MainThread - INFO - {'protocol': 10, 'server_version_original': '8.0.20', 'server_threadid': 11, 'charset': 255, 'server_status': 2, 'auth_plugin': 'caching_sha2_password', 'auth_data': b'\x05?r66p\x029\x1e\\<PRz\\\x03pNcr', 'capabilities': 3355443199}
```
---

## 其它
更多的信息可以参考官方对[握手数据包的描述](https://dev.mysql.com/doc/internals/en/connection-phase-packets.html#packet-Protocol::Handshake)。