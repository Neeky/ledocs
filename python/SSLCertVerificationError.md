## 背景
最近在用 requests 写一个小功能，刚发 get 请求就遇到了问题，网址比较敏感我就不发出来了。
```python
import requests
url = "www.xxx.com.cn/xxx/?top=1"
response = requests.get(url) 

SSLError: HTTPSConnectionPool(host='www.xxx.com.cn', port=443): Max retries exceeded with url: /xxx/?top=1 (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1108)')))
```

![ssl-error](static/2020-14/ssl-error.png)

google-adsense

---


## 原因分析与解决方案
从报错的内容可以看出证书校验出错了，原来这个网站太野了，证书是它自己发给自己的(自签证书)，这证书谁认呀！没有办法我的程序还只能认不然就玩不下去了，那让 requests 不验证证书不就行了，多大点事呀！
```python
response = requests.get(url,verify=False) # verify = False requests 便不在验证证书。

response                                                                
<Response [200]>
```
---
