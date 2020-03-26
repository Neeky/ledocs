## 概要
由于 pip 默认会使用其在国外的服务器，所以我们在用 pip 安装软件包的时候就会比较卡，在网络不佳的情况有更有可能安装失败。 为了解决这个问题我们可以使用国内的镜像服务器，目前腾讯云、阿里云都提供了 pip 的镜像。

![pip3](static/2020-13/pip3.png)

google-adsense

---

## 配置 pip 使用腾讯云镜像
pip3 的配置可以通过 pip3 命令完成，不需要手工编辑配置文件。
```bash
pip3 config set global.index-url  https://mirrors.cloud.tencent.com/pypi/simple
pip3 config set global.trusted-host  mirrors.cloud.tencent.com
```

---

## 配置 pip 使用阿里云镜像
```bash
pip3 config set global.index-url  https://mirrors.aliyun.com/pypi/simple
pip3 config set global.trusted-host  mirrors.aliyun.com
```
---

## 配置文件的位置
pip 用户级别的配置文件会保存在用户的家目录下。
```bash
cat ~/.config/pip/pip.conf
[global]                                                                                         
index-url = https://mirrors.cloud.tencent.com/pypi/simple                                        
trusted-host = mirrors.cloud.tencent.com
```

---


