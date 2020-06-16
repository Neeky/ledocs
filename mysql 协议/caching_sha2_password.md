## 背景
在连接 MySQL 数据库时，用户输入的密码如何传输到 MySQL-Server。是明文传输还是密文传输，如果是密文的话那客户端又是怎么加密的密码呢？

再不刻意指定的情况下密码在网络中传输的是密文，并且加密方式也迭代了好向次了。随着 MySQL-8.0.x 的到来，默认加密算法也从之前的 `mysql_native_password` 更新到现在的 `caching_sha2_password`。

![sqlpy](static/2020-25/sqlpy-password.jpg)

---


## 连接数据库的大致过程
当 MySQL-Client 建立到 MySQL-Server 的 TCP 连接之后，MySQL-Server 会向 MySQL-Client 发送一个叫 `Initial Handshake Packet` 的数据包，这个数据包里面包含了一个叫 `auth-data` 的数据。

现在可以把 auth-data 看成是一个完全随机的字节串，客户端在加密密码时就会用这个串做为“盐值”。

google-adsense

---


## caching_sha2_password 算法
caching_sha2_password 加密密码时的逻辑如下。
```python

class MySQLCachingSHA2PasswordAuthPlugin(object):
    def prepare_password(self,password,auth_data):
        """加密密码返回指纹
        """
        hash1 = sha256(password).digest()
        hash2 = sha256()
        hash2.update(sha256(hash1).digest())
        hash2.update(auth_data)
        hash2 = hash2.digest()

        xored = [h1 ^ h2 for (h1, h2) in zip(hash1, hash2)]
        hash3 = struct.pack('32B', *xored)

```


---
