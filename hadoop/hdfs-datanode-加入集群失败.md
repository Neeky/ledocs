## 背景
新建一个 hdfs 环境，在启动 datanode 的时候报错，始终没有办法把任何一个 datanode 加入到集群。

---

## core-site.xml 配置
测试环境所以相对简单，最配置最必要的值。
```
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://9.135.73.65:9000</value>
    </property>
</configuration>
```

---

## 报错信息
datanode 的报错信息如下
```
2022-10-09 21:34:26,360 ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: Initialization failed for Block pool BP-1868907714-127.0.0.1-1665322391176 (Datanode Uuid b014d6e9-a26e-47a0-9fb2-8366f21bb05c) service to hdfs.namenode.com/9.135.73.65:9000 Datanode denied communication with namenode because hostname cannot be resolved (ip=9.135.73.34, hostname=9.135.73.34): DatanodeRegistration(0.0.0.0:9866, datanodeUuid=b014d6e9-a26e-47a0-9fb2-8366f21bb05c, infoPort=9864, infoSecurePort=0, ipcPort=9867, storageInfo=lv=-57;cid=CID-f84c39fe-59d8-49bd-a023-ded448332d5c;nsid=1154260677;c=1665322391176)
        at org.apache.hadoop.hdfs.server.blockmanagement.DatanodeManager.registerDatanode(DatanodeManager.java:1147)
        at org.apache.hadoop.hdfs.server.blockmanagement.BlockManager.registerDatanode(BlockManager.java:2566)
        at org.apache.hadoop.hdfs.server.namenode.FSNamesystem.registerDatanode(FSNamesystem.java:4235)
        at org.apache.hadoop.hdfs.server.namenode.NameNodeRpcServer.registerDatanode(NameNodeRpcServer.java:1578)
        at org.apache.hadoop.hdfs.protocolPB.DatanodeProtocolServerSideTranslatorPB.registerDatanode(DatanodeProtocolServerSideTranslatorPB.java:101)
        at org.apache.hadoop.hdfs.protocol.proto.DatanodeProtocolProtos$DatanodeProtocolService$2.callBlockingMethod(DatanodeProtocolProtos.java:33760)
        at org.apache.hadoop.ipc.ProtobufRpcEngine2$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine2.java:604)
        at org.apache.hadoop.ipc.ProtobufRpcEngine2$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine2.java:572)
        at org.apache.hadoop.ipc.ProtobufRpcEngine2$Server$ProtoBufRpcInvoker.call(ProtobufRpcEngine2.java:556)
        at org.apache.hadoop.ipc.RPC$Server.call(RPC.java:1093)
        at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:1043)
        at org.apache.hadoop.ipc.Server$RpcCall.run(Server.java:971)
        at java.security.AccessController.doPrivileged(Native Method)
        at javax.security.auth.Subject.doAs(Subject.java:422)
        at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1878)
        at org.apache.hadoop.ipc.Server$Handler.run(Server.java:2976)
```
namenode 的报错信息
```
2022-10-09 21:33:46,320 INFO org.apache.hadoop.ipc.Server: IPC Server handler 2 on default port 9000, call Call#1 Retry#0 org.apache.hadoop.hdfs.server.protocol.DatanodeProtocol.registerDatanode from 9.135.73.34:57036: org.apache.hadoop.hdfs.server.protocol.DisallowedDatanodeException: Datanode denied communication with namenode because hostname cannot be resolved (ip=9.135.73.34, hostname=9.135.73.34): DatanodeRegistration(0.0.0.0:9866, datanodeUuid=b014d6e9-a26e-47a0-9fb2-8366f21bb05c, infoPort=9864, infoSecurePort=0, ipcPort=9867, storageInfo=lv=-57;cid=CID-f84c39fe-59d8-49bd-a023-ded448332d5c;nsid=1154260677;c=1665322391176)
2022-10-09 21:33:51,328 WARN org.apache.hadoop.hdfs.server.blockmanagement.DatanodeManager: Unresolved datanode registration: hostname cannot be resolved (ip=9.135.73.34, hostname=9.135.73.34)
```
---

## 分析与解决
从 namenode 的日志来看也还比较明显，说是 9.135.73.34 没有办法解析对应的域名，我检查了一下我的 /etc/hosts 确实没有配置(测试环境就没有搞 dns 服务了)；配置好 /etc/hosts 后解决问题。这个只要在 namenode 节点上配置就行。

---
