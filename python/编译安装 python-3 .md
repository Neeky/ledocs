## 概要
本文介绍如何编译安装 Python-3 ，Python 的运行环境比较有意思，当我们的依赖不足的时候编译安装的 Python 解释器运行一个 `hello world` 是没有问题，当程序用到一些特殊的功能的时候就会报错。比如这个例子 [无法解压 tar.xz](https://github.com/Neeky/dbm-agent#%E5%B8%B8%E8%A7%81%E9%97%AE%E9%A2%98%E8%A7%A3%E7%AD%94) 。为了解决这类问题，我给出的方案就是把要用到的依赖包都安装上。

![python-3.8.2](static/2020-14/python-3.8.2.png)

google-adsense

---


## 第一步 下载源码
通过 wget 在官方下载 python-3.8.2 的源代码
```bash
cd /tmp/
wget https://www.python.org/ftp/python/3.8.2/Python-3.8.2.tar.xz
```

---

## 第二步 安装依赖
依赖包安装的多少将直接影响到 Python 解释器所能支持的功能，根据我多年的经验来看下面这些包可以非常好的覆盖。
```bash
yum -y install gcc gcc-c++ libffi libyaml-devel libffi-devel zlib zlib-devel openssl shadow-utils net-tools \
openssl-devel libyaml sqlite-devel libxml2 libxslt-devel libxml2-devel wget vim mysql-devel 
```

---

## 第三步 编译安装
通过编译源码来安装 Python-3.8.2 。
```bash
tar -xvf Python-3.8.2.tar.xz 
cd Python-3.8.2
./configure --prefix=/usr/local/python-3.8.2 && make && make install

#创建连接并导出环境变量
cd /usr/local
ln -s python-3.8.2 python
echo 'export PATH=/usr/local/python/bin/:$PATH' >> /etc/profile
source /etc/profile
```

---

## 第四步 验证
验证一下 python-3.8.2 是否安装成功。
```bash
python3                                                                                    
Python 3.8.2 (default, Apr  4 2020, 23:10:31)                                                                     
[GCC 4.8.5 20150623 (Red Hat 4.8.5-39)] on linux                                                                  
Type "help", "copyright", "credits" or "license" for more information.                                            
>>>
```

---

## 配置 pip
配置 pip 使用腾讯云的镜像服务。
```bash
pip3 config set global.index-url  https://mirrors.cloud.tencent.com/pypi/simple
pip3 config set global.trusted-host  mirrors.cloud.tencent.com
```

---