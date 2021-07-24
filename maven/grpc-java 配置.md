## 背景
最近要用 java 开发一套 C/S 架构的服务端程序的部分小功能；由于之前好长一段时间的主力语言是 python 所以这个对我来说是一个陌生的环境。光是配置是一个 grpc-java 就用了我好长的一段时间，现在回想起来这整个过程中并没有哪一个是特别困难的，走了这么多弯路主要还是因为自己向入为主了。

![](static/2021-02/andrea-de-santis-iw-idGUJzHQ-unsplash.jpg)

---

## 约定大于配置
对于一个 maven 项目来说什么样的文件保存在一个什么样的位置是有差明确约定的，java 程序员之间的这种心照不宣，对于我这边从其它语言转过来的就有点坑了；不过还好有助于自己长记性。

对于 proto 文件要保存在 `src/main/proto` 目录下，下面我把经过这几天踩坑之后总结出的实例记一下方便之后快速上手。

---

## 第一步创建 maven 项目
创建一个 grpc-exsameple 的项目，创建完成之后之后 pom.xml 和 目录结构如下。
```pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sqlpy</groupId>
    <artifactId>grpc-exsameple</artifactId>
    <version>1.0-SNAPSHOT</version>


</project>
```
空项目结构
```bash
tree .
.
├── grpc-exsameple.iml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   └── resources
    └── test
        └── java

```
一个空项目不会有什么问题，这里就不检查了。

---

## 第二步 添加 proto 文件
所有的 proto 文件都要保存在 `src/main/proto` 目录下，这里添加一个 `Person.proto` 的文件用于保存信息。
```proto3
syntax="proto3";
option java_package="com.sqlpy.grpcexsample.pb";
option java_multiple_files=true;

message Person {
  string name = 1;
  int32 id = 2;
}
```
新目录结构如下。
```bash
├── grpc-exsameple.iml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   ├── proto
    │   │   └── Person.proto
    │   └── resources
    └── test
        └── java

```

---

## 第三步 添加主类
添加主类 `com.sqlpy.grpcexsample.App` 类，其它代码如下(这一步的目的是为了确认我们的项目可以正常的跑起来)。
```java
package com.sqlpy.grpcexsample;

public class App {
    public static void main(String[] args) {
        System.out.println("hello grpc exsamples.");
    }
}
```
运行效果如下。
```bash
hello grpc exsamples.

Process finished with exit code 0
```
当前的目标结构如下。
```bash
├── grpc-exsameple.iml
├── pom.xml
├── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── sqlpy
    │   │           └── grpcexsample
    │   │               └── App.java
    │   ├── proto
    │   │   └── Person.proto
    │   └── resources
    └── test
        └── java

```

---

## 第四步 配置grpc-java
1、添加 grpc 相关的依赖
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
2、添加编译 proto 的插件
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
                    <protocArtifact>com.google.protobuf:protoc:3.17.2:exe:${os.detected.classifier}</protocArtifact>
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
编译 proto 生成代码 。
```bash
mvn clean package && tree src

src
├── main
│   ├── java
│   │   └── com
│   │       └── sqlpy
│   │           └── grpcexsample
│   │               ├── App.java
│   │               └── pb
│   │                   ├── Person.java
│   │                   ├── PersonOrBuilder.java
│   │                   └── PersonOuterClass.java
│   ├── proto
│   │   └── Person.proto
│   └── resources
└── test
    └── java
```

---

## 第五步 使用 grpc-java 生成的代码
把主类的代码改成如下的内容。
```java
package com.sqlpy.grpcexsample;

import com.sqlpy.grpcexsample.pb.Person;

public class App {
    public static void main(String[] args) {
        System.out.println("hello grpc exsamples.");
        System.out.println(Person.class);
    }
}
```
执行效果如下。
```bash
hello grpc exsamples.
class com.sqlpy.grpcexsample.pb.Person

Process finished with exit code 0
```
在这里可以正常的使用 grpc-java 生成的代码，我就认为 grpc-java 配置成功了。

---

