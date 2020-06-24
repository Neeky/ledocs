## 背景
最近到了一个新的地方，接手了一些新环境。之前我们的数据库是指支持 instant 算法来加列的，但是现在用的是官方的 MySQL-5.7 所以支持不了； 没办法也就只能退回到 percona-toolkit 时代了。

本来以为是老朋友了，编译安装一个 percona-toolkit 不是分分钟的事，没想到出错了；看情况一时半会解决不了，那我还是直接使用 yum 安装吧。

![sqlpy](static/2020-26/sqlpy-percona-install.jpg)

---


## 配置 yum 源
腾讯云同步了 percona 的 yum 源镜像，国外我不知道，国内的话使用腾讯云网速还是非常不错的，官方的源在国内是真的不行。
```bash
cd /etc/yum.repos.d/
touch percona.repo
```

percona.repo 配置文件的内容如下。

```ini
[percona-release-x86_64]
name = Percona Original release/x86_64 YUM repository
baseurl = http://mirrors.tencent.com/percona/yum/release/$releasever/RPMS/x86_64
enabled = 1
gpgcheck = 0

[percona-release-noarch]
name = Percona Original release/noarch YUM repository
baseurl = http://mirrors.tencent.com/percona/yum/release/$releasever/RPMS/noarch
enabled = 1
gpgcheck = 0

[percona-release-sources]
name = Percona Original release/sources YUM repository
baseurl = http://mirrors.tencent.com/percona/yum/release/$releasever/SRPMS
enabled = 0
gpgcheck = 0

```

google-adsense

---

## 安装 percona-toolkit
配置好了 yum 源安装自然就是分分钟的事了。
```bash
yum -y install percona-toolkit                                       
```
---

## 使用官方源
如果你是 percona 官方源的粉丝，你也可以这样配置官方源。
```bash
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
```

---
