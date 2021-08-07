## 背景
最近要用 springboot 写一套 c/s 构架的程序；服务端用 java 语言，框架用的 springboot，客户端用 python 没有什么服务框架可用，直接手撸。因为这还是第一次在 springboot 中使用 grpc ，一开始有点慌，毕竟想到 grpc-server 要持续运行，因此应该会有一个线程(或进程)被一直阻塞，刚开始搞 java 和 springboot 还不清楚其中的原因，所以这里只写一些与 grpc相关的内容。

众所周知，项目中的代码是不会出现在这里的，所以我这里用 google 官方的 helloworld 示例来记录一下 sprintboot 和 grpc 是怎么结合的。

---

## 第一步 定义c/s之间的交互方式
官方的的 helloworld 示例中用的比较简单，客户端调用 stub 的 sayHello 方法，并把 HelloRequest 参数传递进去，最后得到一下 HelloReply 类型的返回值，服务端执行的结果就包在这个返回对象里。 proto 文件`src/main/proto/helloworld.proto`的内容如下。

```proto
syntax="proto3";
option java_multiple_files=true;
option java_package="com.example.demogrpc.rpcs";

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

---

## 第二步 配置grpc 
我刚开始搞 java 的时候感觉这个好难呀，总是出问题，现来看这个还是比较简单的；并且这些配置是可以直接复用的，所以我在这里记一下，方便之后直接使用。

build 部分要添加的配置如下
```xml
<build>
    <extensions>
        <extension>
            <groupId>kr.motd.maven</groupId>
            <artifactId>os-maven-plugin</artifactId>
            <version>1.6.2</version>
        </extension>
    </extensions>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocArtifact>com.google.protobuf:protoc:3.17.3:exe:${os.detected.classifier}</protocArtifact>
                <pluginId>grpc-java</pluginId>
                <pluginArtifact>io.grpc:protoc-gen-grpc-java:1.39.0:exe:${os.detected.classifier}</pluginArtifact>
                <clearOutputDirectory>false</clearOutputDirectory>
                <outputDirectory>${project.build.sourceDirectory}</outputDirectory>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                        <goal>compile-custom</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

依赖部分的要添加的配置如下
```xml
<dependencies>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-netty-shaded</artifactId>
        <version>1.39.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-protobuf</artifactId>
        <version>1.39.0</version>
    </dependency>
    <dependency>
        <groupId>io.grpc</groupId>
        <artifactId>grpc-stub</artifactId>
        <version>1.39.0</version>
    </dependency>
    <dependency> <!-- necessary for Java 9+ -->
        <groupId>org.apache.tomcat</groupId>
        <artifactId>annotations-api</artifactId>
        <version>6.0.53</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```
---

## 第三步 编译 proto 文件
我使用的是 `maven` 来管理依赖，所以我只要执行 `mvn compile` maven 也会自动的帮我把 proto 文件给编译了。编译完成之后会生成如下几个类，他们之间关系如下(红色部分的类是要自己实现的，其它类是内框架生成的)。

![](static/2021-02/grpc-java-helloworld.svg)

`GreeterImpl` 负责实现 proto 文件中定义的请求处理逻辑。

`GreeterServer` 负责把 GreeterImpl 包装成服务，并且实例服务启动，停止方法。


---

## 第四步 实现服务端的处理逻辑
`GreeterImpl.java`的完整代码如下：

```java
import com.example.demogrpc.rpcs.GreeterGrpc;
import com.example.demogrpc.rpcs.HelloReply;
import com.example.demogrpc.rpcs.HelloRequest;
import io.grpc.Server;
import io.grpc.ServerBuilder;
import io.grpc.stub.StreamObserver;

import java.io.IOException;
import java.util.concurrent.TimeUnit;
import java.util.logging.Logger;


public class GreeterServer {

    private static final Logger logger = Logger.getLogger(GreeterServer.class.getName());

    // 第一步：通过一个静态的内部类来实现真正的业务逻辑
    static class GreeterServerImpl extends GreeterGrpc.GreeterImplBase {

        @Override
        public void sayHello(HelloRequest request, StreamObserver<HelloReply> responseObserver) {
            //super.sayHello(request, responseObserver);
            String name = request.getName();
            HelloReply relay = HelloReply.newBuilder().setMessage("Hello " + name + "[this is reply from MyGreeterServer]").build();
            responseObserver.onNext(relay);
            responseObserver.onCompleted();
        }
    }


    // 第二步：为我们的服务提供启动和关闭的方法
    Server server = null;

    // 第二步.1 添加启动函数
    public void start() throws IOException {
        //
        int port = 10352;
        this.server = ServerBuilder.forPort(port)
                .addService(new GreeterServerImpl())
                .build()
                .start();

        logger.info("The greatest Server GreeterServer has been started.");

        Runtime.getRuntime().addShutdownHook(new Thread() {
            @Override
            public void run() {
                System.out.println("prepare shutdown server.");
                try {
                    // TODO
                    GreeterServer.this.stop();
                }
                catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
                System.out.println("GreeterServer stoped.");
            }
        });
    }

    // 第二步.2 添加关闭函数
    public void stop() throws InterruptedException {
        if(this.server != null) {
            this.server.shutdown().awaitTermination(30, TimeUnit.SECONDS);
        }
    }

}
```
一开始我以为 springboot 的 main 方法执行完成之后，整个服务就退出了；后来调试了一下发现并不是这么一回事，main 方法执行完成之后服务还在。服务在子线程中完成，grpc 也类似，后台服务也不会阻塞整个进程，同理它也是在一个子线程中执行。

也就是说我们只要在 main 函数中启动 grpc 的服务端就行了，别的都交给框架来处理了。

```java
package com.example.demogrpc;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

import com.example.demogrpc.services.GreeterServer;

@SpringBootApplication
public class DemoGrpcApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoGrpcApplication.class, args);

        GreeterServer server = new GreeterServer();
        try {
            server.start();
        }
        catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    // 这个比较有意思，服务已经后台运行了。所以

}

```
---

## 第五步 启动服务端程序
启动 springboot 服务。
```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.3)

2021-08-06 19:53:47.822  INFO 92545 --- [           main] c.example.demogrpc.DemoGrpcApplication   : Starting DemoGrpcApplication using Java 1.8.0_282 on NEEKYJIANG-MB2 with PID 92545 (/Users/jianglexing/temps/demo-grpc/target/classes started by jianglexing in /Users/jianglexing/temps/demo-grpc)
2021-08-06 19:53:47.824  INFO 92545 --- [           main] c.example.demogrpc.DemoGrpcApplication   : No active profile set, falling back to default profiles: default
2021-08-06 19:53:48.366  INFO 92545 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-08-06 19:53:48.371  INFO 92545 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-08-06 19:53:48.372  INFO 92545 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.50]
2021-08-06 19:53:48.414  INFO 92545 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-08-06 19:53:48.414  INFO 92545 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 561 ms
2021-08-06 19:53:48.646  INFO 92545 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-08-06 19:53:48.653  INFO 92545 --- [           main] c.example.demogrpc.DemoGrpcApplication   : Started DemoGrpcApplication in 1.052 seconds (JVM running for 1.414)
2021-08-06 19:53:48.778  INFO 92545 --- [           main] c.e.demogrpc.services.GreeterServer      : The greatest Server GreeterServer has been started
```

--- 

## 第六步 实现客户端代码
客户端的最小化实现如下。
```python
#!/usr/bin/evn python3

import logging
import argparse
from concurrent import futures

from helloworld_pb2 import HelloRequest,HelloReply
from helloworld_pb2_grpc import GreeterStub

import grpc


with grpc.insecure_channel(f"127.0.0.1:10352") as channel:
    stub = GreeterStub(channel)
    reply = stub.SayHello(HelloRequest(name="dalio"))
    print(reply.message)
```
调用效果
```bash
python3 main.py 
Hello dalio[this is reply from MyGreeterServer]
```
---
