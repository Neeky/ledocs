## 背景
最近在学习一个新的项目，它用到了 zookeeper 来发布服务，所以顺手把 zookeeper 的安装方式也学习了下。


---

## 第一步 下载安装包
直接从 apache 下载最新的稳定版本。
```bash
cd /tmp/
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.6.1/apache-zookeeper-3.6.1-bin.tar.gz
```

---

## 第二步 安装依赖
zookeeper 由 java 编写而成，所以要把 java 运行环境安装上。
```
yum -y install java-11-openjdk
```

---


## 第二步 解压安装
zookeeper 以二进制的方式发现的程序包，所以解压就能用。
```bash
tar -xvf apache-zookeeper-3.6.1-bin.tar.gz -C /usr/local/
cd /usr/local/

useradd zookeeper
chown -R zookeeper:zookeeper apache-zookeeper-3.6.1-bin

ln -s apache-zookeeper-3.6.1-bin zookeeper

echo 'export PATH=/usr/local/zookeeper/bin/:$PATH' >> /etc/profile
source /etc/profile
```

google-adsense

---

## 第四步 创建配置文件
创建配置文件 `/usr/local/zookeeper/conf/zoo.cfg` 并写入如下内容。
```cfg
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
```

## 第五步 启动
不推荐使用 `root` 启动服务，在这里就用之前创建 `zookeeper` 用户来启动服务。
```bash
sudo su zookeeper

zkServer.sh start

/usr/bin/java
ZooKeeper JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```

---

## 验证
可以通过客户端是否能正常连接`zookeeper`来验证安装是否成功。
```bash
# zookeeper 对应的几个端口是否监听上了
netstat -ltnp | grep java 

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp6       0      0 :::36312                :::*                    LISTEN      9543/java           
tcp6       0      0 :::2181                 :::*                    LISTEN      9543/java           
tcp6       0      0 :::8080                 :::*                    LISTEN      9543/java 

# 直接使用 zookeeper 的客户端进行连接
zkCli.sh

2020-05-12 04:37:10,419 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1420] - Session establishment complete on server localhost/127.0.0.1:2181, session id = 0x10003c3c32e0001, negotiated timeout = 30000
```

---

## 故障排除
如果遇到了问题可以直接到`/usr/local/zookeeper/logs`目录下看日志，基本上都能解决了。

---