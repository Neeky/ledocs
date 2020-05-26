## 背景
之前对 mysql-connector-python 这个驱动程序中 `MySQLCachingSHA2PasswordAuthPlugin` 类的实现一直不太理解，今天终于明白了；希望可以讲清理。


---

## 类的层次结构
`BaseAuthPlugin` 类是所有授权插件的基类，定义了所有授权插件的对外接口 `auth_response`。至于密码地加密逻辑都交由子类实现。
```python
class BaseAuthPlugin(object):
    """Base class for authentication plugins
    """
    requires_ssl = False
    plugin_name = ''

    def __init__(self, auth_data, username=None, password=None, database=None,
                 ssl_enabled=False):
        """Initialization"""
        self._auth_data = auth_data
        self._username = username
        self._password = password
        self._database = database
        self._ssl_enabled = ssl_enabled

    def prepare_password(self):
        """Prepares and returns password to be send to MySQL
        """
        raise NotImplementedError

    def auth_response(self):
        """Returns the prepared password to send to MySQL
        """
        if self.requires_ssl and not self._ssl_enabled:
            raise errors.InterfaceError("{name} requires SSL".format(
                name=self.plugin_name))
        return self.prepare_password()

```

---

`MySQLCachingSHA2PasswordAuthPlugin` 是 MySQL-8.0 的默认授权插件，今天我们主要分析它的实现。

```python
class MySQLCachingSHA2PasswordAuthPlugin(BaseAuthPlugin):
    """Class implementing the MySQL caching_sha2_password authentication plugin

    Note that encrypting using RSA is not supported since the Python
    Standard Library does not provide this OpenSSL functionality.
    """

    requires_ssl = False
    plugin_name = 'caching_sha2_password'
    perform_full_authentication = 4
    fast_auth_success = 3

    def _scramble(self):
        """ Returns a scramble of the password using a Nonce sent by the server.
        """
        if not self._auth_data:
            raise errors.InterfaceError("Missing authentication data (seed)")

        if not self._password:
            return b''

        password = self._password.encode('utf-8') \
            if isinstance(self._password, UNICODE_TYPES) else self._password

        if PY2:
            password = buffer(password)  # pylint: disable=E0602
            try:
                auth_data = buffer(self._auth_data)  # pylint: disable=E0602
            except TypeError:
                raise errors.InterfaceError("Authentication data incorrect")
        else:
            password = password
            auth_data = self._auth_data

        hash1 = sha256(password).digest()
        hash2 = sha256()
        hash2.update(sha256(hash1).digest())
        hash2.update(auth_data)
        hash2 = hash2.digest()
        if PY2:
            xored = [ord(h1) ^ ord(h2) for (h1, h2) in zip(hash1, hash2)]
        else:
            xored = [h1 ^ h2 for (h1, h2) in zip(hash1, hash2)]
        hash3 = struct.pack('32B', *xored)

        return hash3

    def prepare_password(self):
        if len(self._auth_data) > 1:
            return self._scramble()
        elif self._auth_data[0] == self.perform_full_authentication:
            return self._full_authentication()
        return None

    def _full_authentication(self):
        """Returns password as as clear text"""
        if not self._ssl_enabled:
            raise errors.InterfaceError("{name} requires SSL".format(
                name=self.plugin_name))

        if not self._password:
            return b'\x00'
        password = self._password

        if PY2:
            if isinstance(password, unicode):  # pylint: disable=E0602
                password = password.encode('utf8')
        elif isinstance(password, str):
            password = password.encode('utf8')

        return password + b'\x00'
```

google-adsense

---

## 问题
由父类的实现可以看到，对外的接口只有 `auth_response` 而它又是调用的 `prepare_password` 所以子类中只要去实现这个方法就行了。但是我们看到在子类中 `prepare_password` 事实上只是一个 if - else 角色。

---

何时会走 if 流程： 当 client 回第一个 auth-response 包的时候就会走这个流程来加密密码。

---

何时会走 if-else 流程： 给定的用户第一次连接数据库，也就是这个用户在 MySQL-Server 上没有连接成功的记录时，MySQL-Server 会要求 client 再发一次密码；又由于这个时候 ssl 已经建立发送的字节流就不用加密了，直接发字节串就行。

---

何时会走 else 流程：在这次连接请求之前给定的用户已经成功的建立过与 MySQL-Server 的连接，这种情况下 MySQL-Server 就不会要求对用户密码进行二次验证，所以 `prepare_password` 直接返回了 None.

---

## 其它
我写了一个程序不依赖任何连接驱动就能完成登录 MySQL 的操作，可以用来感受纯 TCP 网络编程。


---





