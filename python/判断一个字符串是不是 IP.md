## 概要
任意给定一个字符串如何判断它是不是一个 IP 地址？

![sqlpy](static/2020-26/sqlpy-pton.jpg)

---

## 方法一 正则表达式
从题目来看是一个典型的模式模匹配问题，我第一个想到的办法也是正则表达式。
```python
import re

ipv4 = re.compile(
    r"(((25[0-5])|(2[0-4]\d)|(1\d{2})|([1-9]\d{1})|\d{1})\.){3}((25[0-5])|(2[0-4]\d)|(1\d{2})|([1-9]\d{0,1})|\d{1})", re.MULTILINE)


def is_ip_using_re(ip: str) -> bool:
    """
    """
    return ipv4.fullmatch(ip) is not None


def main():
    ip = "127.0.0.1"
    print(is_ip_using_re(ip))


if __name__ == "__main__":
    main()

```

运行效果。

```bash
python3 main.py 
True
```

google-adsense

---

## 使用正则存在的问题
诚然这个问题使用正则确实可以解决，如果想要兼容 IPv6 大不了，再加一个正则嘛！但是也应该注意到正则表达式并不是每一个人都会写并且写的对，至于正确且高效就更不说了。

下面我们来看一下不使用正则的方式，socket.inet_pton。


---


## 方法二 socket.inet_pton
`socket.inet_pton` 函数可以把一个 IP 地址转换成字节串，如果是一个合法的 IP 地址那么一定可以转成功，不然就会报异常。利用这个特性我们就可以完成验证。
```python
def is_ip_using_pton(ip: str) -> bool:
    """
    """
    for family in (socket.AF_INET, socket.AF_INET6):
        # 先尝试转换成 IPv4 再转换成 IPv6
        try:
            socket.inet_pton(family, ip)
            # 只要没有报异常说明转换成功
            return True
        except Exception as err:
            pass

    # 没有一个能成功，说明不是一个合法的 IP 地址
    return False

```

---

## 总结
socket.inet_pton 用于验证是不是 IP 还是非常方便的，如果问题的核心是提取而不是验证正则表达式会更加适合一些。

---