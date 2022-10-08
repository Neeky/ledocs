## 背景
开发环境上要用到一些 hdfs 的功能，于是看了下文档自己搞了一个伪分布式的；现在记一下这个过程防止以后用的时候可以直接用。

---

## 第一步 下载 hadoop
```
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz
```
---

## 第二步 解压
```bash
tar -xvf hadoop-3.3.4.tar.gz -C /usr/local/
cd /usr/local/
ln -s hadoop-3.3.4 hadoop
```

---

## 第三步 修改配置文件
etc/hadoop/core-site.xml
```xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
---
etc/hadoop/hdfs-site.xml
```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```
---

## 第四步 配置 ssh 免密
```bash
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```
---

## 第五步 配置进程的启动用户
etc/hadoop/hadoop-env.sh
```
export HDFS_NAMENODE_USER=root
export HDFS_DATANODE_USER=root
export HDFS_SECONDARYNAMENODE_USER=root
export YARN_RESOURCEMANAGER_USER=root
export YARN_NODEMANAGER_USER=root
```
这个还是要配置一下，不然在启动 hdfs 的时候会报这 3 个错。
```
bash sbin/start-dfs.sh 
Starting namenodes on [localhost]
ERROR: Attempting to operate on hdfs namenode as root
ERROR: but there is no HDFS_NAMENODE_USER defined. Aborting operation.
Starting datanodes
ERROR: Attempting to operate on hdfs datanode as root
ERROR: but there is no HDFS_DATANODE_USER defined. Aborting operation.
Starting secondary namenodes [git-sqlpy-com]
ERROR: Attempting to operate on hdfs secondarynamenode as root
ERROR: but there is no HDFS_SECONDARYNAMENODE_USER defined. Aborting operation.
```
---

## 第六步 格式化文件系统并启动hdfs
```bash
bash bin/hdfs namenode -format
bash sbin/start-dfs.sh
```

---

## 第七步 检查进程的状态
```
ps -ef | grep java

root     2786859       1  0 20:42 ?        00:00:06 /usr/local/TencentKona-8.0.10-332/bin/java -Dproc_namenode -Djava.net.preferIPv4Stack=true -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dyarn.log.dir=/usr/local/hadoop-3.3.4/logs -Dyarn.log.file=hadoop-root-namenode-git-sqlpy-com.log -Dyarn.home.dir=/usr/local/hadoop-3.3.4 -Dyarn.root.logger=INFO,console -Djava.library.path=/usr/local/hadoop-3.3.4/lib/native -Dhadoop.log.dir=/usr/local/hadoop-3.3.4/logs -Dhadoop.log.file=hadoop-root-namenode-git-sqlpy-com.log -Dhadoop.home.dir=/usr/local/hadoop-3.3.4 -Dhadoop.id.str=root -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.namenode.NameNode
root     2787031       1  0 20:43 ?        00:00:06 /usr/local/TencentKona-8.0.10-332/bin/java -Dproc_datanode -Djava.net.preferIPv4Stack=true -Dhadoop.security.logger=ERROR,RFAS -Dyarn.log.dir=/usr/local/hadoop-3.3.4/logs -Dyarn.log.file=hadoop-root-datanode-git-sqlpy-com.log -Dyarn.home.dir=/usr/local/hadoop-3.3.4 -Dyarn.root.logger=INFO,console -Djava.library.path=/usr/local/hadoop-3.3.4/lib/native -Dhadoop.log.dir=/usr/local/hadoop-3.3.4/logs -Dhadoop.log.file=hadoop-root-datanode-git-sqlpy-com.log -Dhadoop.home.dir=/usr/local/hadoop-3.3.4 -Dhadoop.id.str=root -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.datanode.DataNode
root     2787302       1  0 20:43 ?        00:00:04 /usr/local/TencentKona-8.0.10-332/bin/java -Dproc_secondarynamenode -Djava.net.preferIPv4Stack=true -Dhdfs.audit.logger=INFO,NullAppender -Dhadoop.security.logger=INFO,RFAS -Dyarn.log.dir=/usr/local/hadoop-3.3.4/logs -Dyarn.log.file=hadoop-root-secondarynamenode-git-sqlpy-com.log -Dyarn.home.dir=/usr/local/hadoop-3.3.4 -Dyarn.root.logger=INFO,console -Djava.library.path=/usr/local/hadoop-3.3.4/lib/native -Dhadoop.log.dir=/usr/local/hadoop-3.3.4/logs -Dhadoop.log.file=hadoop-root-secondarynamenode-git-sqlpy-com.log -Dhadoop.home.dir=/usr/local/hadoop-3.3.4 -Dhadoop.id.str=root -Dhadoop.root.logger=INFO,RFA -Dhadoop.policy.file=hadoop-policy.xml org.apache.hadoop.hdfs.server.namenode.SecondaryNameNode
```