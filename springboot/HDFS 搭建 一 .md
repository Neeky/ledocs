## 概要
一步一步搭建一个分布式的 HDFS 集群。

资源规划，我有 5 台虚拟机打算两台用来做 namenode 三台用来做 datanode 。

|ip|主机名|角色|
|------|-----|-----|
|9.134.87.234 | nn01 | namenode|
|9.134.84.114 | nn02 | secondarynamenode|
|9.134.87.86  | dn01 | datanode |
|9.134.87.224 | dn02 | datanode |
|9.134.87.210 | dn03 | datanode |

![](static/2021-02/dawson-mccormick-BIMFgSyf1oc-unsplash.jpg)

---

## 第一步 配置基础环境
1、配置 dns 解析，这里使用配置 `/etc/hosts` 文件的方式。
```ini
9.134.87.234	nn01
9.134.84.114	nn02
9.134.87.86     dn01
9.134.87.224	dn02
9.134.87.210	dn03
```
2、安装 java8，注意这里最好是使用 java8 从官方文档来看不知道什么时候才会支持 java11 。
```bash
yum -y install java-1.8.0-openjdk-devel
```
3、配置 JAVA_HOME
```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.tl2.x86_64/jre/
```
4、创建 hadoop 用来保存数据的目录。
```bash
mkdir /data/hadoop -p
```

在安装与配置 hadoop 环境的时候要明白两件重要的事；1、我们要保证所有的系统环境与软件环境要一样，机器一多最怕一个机器一个样子这样就不好管了。 2、为了保证 hadoop 在所有机器上的软件包和配置文件是一样的，我们可以采用在一台机器上配置好之后 scp 到其它机器的方式来处理。

---

## 第二步 安装配置 namenode 
1、解压安装包到 `/usr/local/`。
```bash
tar -xvf hadoop-3.3.1.tar.gz -C /usr/local/
```
2、导出 hadoop 相关的命令(配置 /etc/profile)。
```bash
export PATH=/usr/local/hadoop-3.3.1/bin/:$PATH
```

etc/hadoop 目录下还有几个文件要配置一下

3、hadoop-env.sh 配置。
```bash
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.tl2.x86_64/jre/
```
4、core-site.xml 配置
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://nn01:8020</value>
    </property>

    <property>
        <name>hadoop.tmp.dir</name>
        <value>/data/hadoop</value>
    </property>
</configuration>
```
5、hdfs-site.xml 配置
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>

    <property>
        <name>dfs.namenode.secondary.http-address</name>
        <value>nn02:9868</value>
    </property>
</configuration>
```
6、启动 namenode 
```bash
hdfs namenode -format my-first-cluster
hdfs --daemon start namenode

ps -ef | grep java
root     23289     1  0 16:21 ?        00:00:16 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.tl2.x86_64/jre//bin/java -Dproc_namenode -Djava.net.preferIPv4Stack=true -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dyarn.log.dir=/usr/local/hadoop-3.3.1/logs -Dyarn.log.file=hadoop-root-namenode-nn01.log -Dyarn.home.dir=/usr/local/hadoop-3.3.1 -Dyarn.root.logger=INFO,console -Djava.library.path=/usr/local/hadoop-3.3.1/lib/native -Dhadoop.log.dir=/usr/local/hadoop-3.3.1/logs -Dhadoop.log.file=hadoop-root-namenode-nn01.log -Dhadoop.home.dir=/usr/local/hadoop-3.3.1 -Dhadoop.id.str=root -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.namenode.NameNode

netstat -ltnp | grep java
tcp        0      0 9.134.87.234:8020       0.0.0.0:*               LISTEN      23289/java          
tcp        0      0 0.0.0.0:9870            0.0.0.0:*               LISTEN      23289/java
```
其中的 9870 端口用于 web-ui ，8020 用于标识一个 hdfs 文件系统。

---

## 第三步 复制文件
刚才我们已经配置好了 namenode 现在我们要把 namename 机器上的配置文件和程序都复制到其它机器上去。
```bash
scp -rp /usr/local/hadoop-3.3.1 root@nn02:/usr/local/
scp -rp /usr/local/hadoop-3.3.1 root@dn01:/usr/local/
scp -rp /usr/local/hadoop-3.3.1 root@dn02:/usr/local/
scp -rp /usr/local/hadoop-3.3.1 root@dn03:/usr/local/
```
这样我们就保证了所有机器上的 hadoop 版本和配置是一样的。

---

## 第四步 启动 datanode
在三台 datanode 机器上执行如下命令用于启动 datanode 。
```bash
hdfs --daemon start datanode

ps -ef | grep java
root     28588     1  0 16:40 ?        00:00:10 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.tl2.x86_64/jre//bin/java -Dproc_datanode -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=ERROR,RFAS -Dyarn.log.dir=/usr/local/hadoop-3.3.1/logs -Dyarn.log.file=hadoop-root-datanode-dn01.log -Dyarn.home.dir=/usr/local/hadoop-3.3.1 -Dyarn.root.logger=INFO,console -Djava.library.path=/usr/local/hadoop-3.3.1/lib/native -Dhadoop.log.dir=/usr/local/hadoop-3.3.1/logs -Dhadoop.log.file=hadoop-root-datanode-dn01.log -Dhadoop.home.dir=/usr/local/hadoop-3.3.1 -Dhadoop.id.str=root -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.datanode.DataNode

netstat -ltpn | grep java
tcp        0      0 127.0.0.1:40716         0.0.0.0:*               LISTEN      28588/java          
tcp        0      0 0.0.0.0:9864            0.0.0.0:*               LISTEN      28588/java          
tcp        0      0 0.0.0.0:9866            0.0.0.0:*               LISTEN      28588/java          
tcp        0      0 0.0.0.0:9867            0.0.0.0:*               LISTEN      28588/java 
```
---

## 第五步 启动 secondarynamenode
```bash
hdfs --daemon start secondarynamenode
```
---

## 第六步 检查
在 hdfs 中创建目录并上传文件
```bash
hdfs dfs -mkdir /test-dir/
hdfs dfs -put /usr/bin/ifconfig /test-dir/
hdfs dfs -put hadoop-3.3.1.tar.gz /test-dir/
```

`http://9.134.87.234:9870/explorer.html#/test-dir` 可以在 namenode 的 web-ui 上看到如下内容。

![](static/2021-02/hadoop-hdfs.jpeg)

---


