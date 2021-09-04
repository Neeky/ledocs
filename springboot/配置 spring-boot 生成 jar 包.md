## 背景
[从 maven 到 springboot](https://sqlpy.com/blogs/661345428) 我们介绍了怎么从一个空的 maven 项目，一步步配置成一个 spring-boot 项目。但是还留了一个问题没有讲，那就是当我们把项目编译成 jar 包后并运行不起来。
```bash
# 编译 OK
mvn package

[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.sqlpy:setup-spring-boot >---------------------
[INFO] Building setup-spring-boot 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:resources (default-resources) @ setup-spring-boot ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/jianglexing/temps/setup-spring-boot/src/main/resources
[INFO] skip non existing resourceDirectory /Users/jianglexing/temps/setup-spring-boot/src/main/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:compile (default-compile) @ setup-spring-boot ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /Users/jianglexing/temps/setup-spring-boot/target/classes
[INFO] 
[INFO] --- maven-resources-plugin:3.0.1:testResources (default-testResources) @ setup-spring-boot ---
[INFO] Using 'UTF-8' encoding to copy filtered resources.
[INFO] skip non existing resourceDirectory /Users/jianglexing/temps/setup-spring-boot/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.7.0:testCompile (default-testCompile) @ setup-spring-boot ---
[INFO] Changes detected - recompiling the module!
[INFO] Compiling 1 source file to /Users/jianglexing/temps/setup-spring-boot/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.21.0:test (default-test) @ setup-spring-boot ---
[INFO] 
[INFO] -------------------------------------------------------
[INFO]  T E S T S
[INFO] -------------------------------------------------------
[INFO] Running com.sqlpy.AppTest
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.019 s - in com.sqlpy.AppTest
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-jar-plugin:3.0.2:jar (default-jar) @ setup-spring-boot ---
[INFO] Building jar: /Users/jianglexing/temps/setup-spring-boot/target/setup-spring-boot-1.0-SNAPSHOT.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 2.076 s
[INFO] Finished at: 2021-09-04T13:46:18+08:00
[INFO] ------------------------------------------------------------------------

# 运行时报错
jianglexing@NEEKYJIANG-MB2 setup-spring-boot % java -jar target/setup-spring-boot-1.0-SNAPSHOT.jar
target/setup-spring-boot-1.0-SNAPSHOT.jar中没有主清单属性
```

![](static/2021-02/eberhard-grossgasteiger-UKT28Wi4aZA-unsplash.jpg)

---

## 解决办法
spring 官方提供了一个编译插件，这个插件会帮我们配置 jar 的主类，所以我们只要改一下 POM 文件就行了。
```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.sqlpy</groupId>
  <artifactId>setup-spring-boot</artifactId>
  <version>1.0-SNAPSHOT</version>

  <name>setup-spring-boot</name>
  <url>https://www.sqlpy.com</url>

  <parent>
    <artifactId>spring-boot-starter-parent</artifactId>
    <groupId>org.springframework.boot</groupId>
    <version>2.0.2.RELEASE</version>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
      <version>2.0.2.RELEASE</version>
    </dependency>

    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
</project>
```

---

## spring-boot 编译后生成的 jar 文件
jar 本质上就是一个 zip 文件，就上面的 jar 包，我们解压后可以得到如下的目录结构。
```bash
tree .
.
├── BOOT-INF
│   ├── classes
│   │   └── com
│   │       └── sqlpy
│   │           └── App.class
│   └── lib
│       ├── classmate-1.3.4.jar
│       ├── hibernate-validator-6.0.9.Final.jar
│       ├── jackson-annotations-2.9.0.jar
│       ├── jackson-core-2.9.5.jar
│       ├── jackson-databind-2.9.5.jar
│       ├── jackson-datatype-jdk8-2.9.5.jar
│       ├── jackson-datatype-jsr310-2.9.5.jar
│       ├── jackson-module-parameter-names-2.9.5.jar
│       ├── javax.annotation-api-1.3.2.jar
│       ├── jboss-logging-3.3.2.Final.jar
│       ├── jul-to-slf4j-1.7.25.jar
│       ├── log4j-api-2.10.0.jar
│       ├── log4j-to-slf4j-2.10.0.jar
│       ├── logback-classic-1.2.3.jar
│       ├── logback-core-1.2.3.jar
│       ├── slf4j-api-1.7.25.jar
│       ├── snakeyaml-1.19.jar
│       ├── spring-aop-5.0.6.RELEASE.jar
│       ├── spring-beans-5.0.6.RELEASE.jar
│       ├── spring-boot-2.0.2.RELEASE.jar
│       ├── spring-boot-autoconfigure-2.0.2.RELEASE.jar
│       ├── spring-boot-starter-2.0.2.RELEASE.jar
│       ├── spring-boot-starter-json-2.0.2.RELEASE.jar
│       ├── spring-boot-starter-logging-2.0.2.RELEASE.jar
│       ├── spring-boot-starter-tomcat-2.0.2.RELEASE.jar
│       ├── spring-boot-starter-web-2.0.2.RELEASE.jar
│       ├── spring-context-5.0.6.RELEASE.jar
│       ├── spring-core-5.0.6.RELEASE.jar
│       ├── spring-expression-5.0.6.RELEASE.jar
│       ├── spring-jcl-5.0.6.RELEASE.jar
│       ├── spring-web-5.0.6.RELEASE.jar
│       ├── spring-webmvc-5.0.6.RELEASE.jar
│       ├── tomcat-embed-core-8.5.31.jar
│       ├── tomcat-embed-el-8.5.31.jar
│       ├── tomcat-embed-websocket-8.5.31.jar
│       └── validation-api-2.0.1.Final.jar
├── META-INF
│   ├── MANIFEST.MF
│   └── maven
│       └── com.sqlpy
│           └── setup-spring-boot
│               ├── pom.properties
│               └── pom.xml
└── org
    └── springframework
        └── boot
            └── loader
                ├── ExecutableArchiveLauncher.class
                ├── JarLauncher.class
                ├── LaunchedURLClassLoader$UseFastConnectionExceptionsEnumeration.class
                ├── LaunchedURLClassLoader.class
                ├── Launcher.class
                ├── MainMethodRunner.class
                ├── PropertiesLauncher$1.class
                ├── PropertiesLauncher$ArchiveEntryFilter.class
                ├── PropertiesLauncher$PrefixMatchingArchiveFilter.class
                ├── PropertiesLauncher.class
                ├── WarLauncher.class
                ├── archive
                │   ├── Archive$Entry.class
                │   ├── Archive$EntryFilter.class
                │   ├── Archive.class
                │   ├── ExplodedArchive$1.class
                │   ├── ExplodedArchive$FileEntry.class
                │   ├── ExplodedArchive$FileEntryIterator$EntryComparator.class
                │   ├── ExplodedArchive$FileEntryIterator.class
                │   ├── ExplodedArchive.class
                │   ├── JarFileArchive$EntryIterator.class
                │   ├── JarFileArchive$JarFileEntry.class
                │   └── JarFileArchive.class
                ├── data
                │   ├── RandomAccessData.class
                │   ├── RandomAccessDataFile$1.class
                │   ├── RandomAccessDataFile$DataInputStream.class
                │   ├── RandomAccessDataFile$FileAccess.class
                │   └── RandomAccessDataFile.class
                ├── jar
                │   ├── AsciiBytes.class
                │   ├── Bytes.class
                │   ├── CentralDirectoryEndRecord.class
                │   ├── CentralDirectoryFileHeader.class
                │   ├── CentralDirectoryParser.class
                │   ├── CentralDirectoryVisitor.class
                │   ├── FileHeader.class
                │   ├── Handler.class
                │   ├── JarEntry.class
                │   ├── JarEntryFilter.class
                │   ├── JarFile$1.class
                │   ├── JarFile$2.class
                │   ├── JarFile$JarFileType.class
                │   ├── JarFile.class
                │   ├── JarFileEntries$1.class
                │   ├── JarFileEntries$EntryIterator.class
                │   ├── JarFileEntries.class
                │   ├── JarURLConnection$1.class
                │   ├── JarURLConnection$JarEntryName.class
                │   ├── JarURLConnection.class
                │   ├── StringSequence.class
                │   └── ZipInflaterInputStream.class
                └── util
                    └── SystemPropertyUtils.class
```
`META-INF/MANIFEST.MF` 配置文件的内容如下
```yaml
Manifest-Version: 1.0
Implementation-Title: setup-spring-boot
Implementation-Version: 1.0-SNAPSHOT
Built-By: jianglexing
Implementation-Vendor-Id: com.sqlpy
Spring-Boot-Version: 2.0.2.RELEASE
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.sqlpy.App
Spring-Boot-Classes: BOOT-INF/classes/
Spring-Boot-Lib: BOOT-INF/lib/
Created-By: Apache Maven 3.5.4
Build-Jdk: 1.8.0_282
Implementation-URL: https://www.sqlpy.com
```
可以看到 spring-boot 帮我们把 Main-Class 设置成了它内部的一个包 `org.springframework.boot.loader.JarLauncher` 。

---

在我们不添加编译插件的情况下 MANIFEST.MF 的内容是这样的。
```yaml
Manifest-Version: 1.0
Implementation-Title: setup-spring-boot
Implementation-Version: 1.0-SNAPSHOT
Built-By: jianglexing
Implementation-Vendor-Id: com.sqlpyl
Created-By: Apache Maven 3.5.4
Build-Jdk: 1.8.0_282
Implementation-URL: https://www.sqlpy.com
```

---

