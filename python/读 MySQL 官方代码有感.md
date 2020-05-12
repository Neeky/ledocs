## 背景
暗中观察 MySQL [连接驱动](https://github.com/Neeky/mysql-connector-python-8.0.20/blob/master/lib/mysql/connector/authentication.py) 的源代码时，看到它用到了工厂方法这个设计模式，之所以让我眼前一亮不是因为这个模式有多高大上，而是官方让这个模式以最优雅的姿态，出现了在最适合的位置。

---

## 问题介绍
MySQL 客户端连接 MySQL 服务端时要发送用户名和密码，密码部分在网络上以密文的形式传输；从明文到密文的这个转化由加密算法完成。

作为一个有历史的数据库，加密算法也经过了好几个版本的迭代比较有名的就有 `mysql_native_password` 和 `caching_sha2_password` ，不怎么常用的还有 `sha256_password`，`mysql_clear_password`。

官方把每个加密算法都设计成了一个类。
```python
class BaseAuthPlugin(object):
    plugin_name = ''

class MySQLNativePasswordAuthPlugin(BaseAuthPlugin):
    plugin_name = 'mysql_native_password'

class MySQLClearPasswordAuthPlugin(BaseAuthPlugin):
    plugin_name = 'mysql_clear_password'

class MySQLSHA256PasswordAuthPlugin(BaseAuthPlugin):
    plugin_name = 'sha256_password'

class MySQLCachingSHA2PasswordAuthPlugin(BaseAuthPlugin):
    plugin_name = 'caching_sha2_password'
```

---

## 我的工厂方法
在没有看官方的实现之前我的工厂方法是这样的。
```python
def get_auth_plugin(plugin_name):
    if plugin_name == 'mysql_native_password':
        return MySQLNativePasswordAuthPlugin()
    elif plugin_name == 'caching_sha2_password':
        return MySQLCachingSHA2PasswordAuthPlugin()
    elif plugin_name == 'sha256_password':
        return MySQLSHA256PasswordAuthPlugin()
    elif plugin_name == 'mysql_clear_password':
        return MySQLClearPasswordAuthPlugin()
```
根据 `plugin_name` 的不同返回不同的 BaseAuthPlugin 子类的实例，看起来完成了模式，但是这个函数没有封闭。也就是说当有新的加密算法时，要支持这个新的算法就要改现在的函数。

---

## 官方的做法
官方在这个问题上直接打开了 Python 的黑魔法 ---- 元编程，并且使得函数天然的封闭。
```python
def get_auth_plugin(plugin_name):
    for authclass in BaseAuthPlugin.__subclasses__():
        if authclass.plugin_name == plugin_name:
            return authclass
```
不服不行啊！

---