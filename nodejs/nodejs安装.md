## 概要
在 Centos-7 操作系统下安装 nodejs 环境

![nodejs](static/2020-13/nodejs.png)
google-adsense

---

## 自动化安装
运行一下脚本就可以完成自动化安装了
```bash
#切到root用户
sudo su

#下载并解压node安装包
cd /tmp/
wget https://nodejs.org/dist/v12.16.1/node-v12.16.1-linux-x64.tar.xz
tar -xvf node-v12.16.1-linux-x64.tar.xz -C /usr/local/

cd /usr/local/
ln -s node-v12.16.1-linux-x64 node

#导出 PATH 环境变量
echo 'export PATH=/usr/local/node/bin/:$PATH' >> /etc/profile
source /etc/profile
```

---

## 验证是否安装成功
如果可以正常执行 node 命令说明安装成功了
```bash
[root@lestudio local]# node --version
v13.5.0
```

---

## 配置npm使用淘宝源
node 通过 npm 来管理软件包，但是 npm 的服务器在国外，这个就导致了我们安装软件包会比较卡，为了解决这个问题可以使用淘宝的源。
```bash
npm config set registry https://registry.npm.taobao.org
```
---
