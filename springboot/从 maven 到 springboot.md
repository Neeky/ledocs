## 背景
之前做 spring-boot 的项目开发，总是用的 IDEA 中的项目模板；虽然来的快不用去关注细节，高屋建瓴的来讲它就是一个 maven 项目，但是对于怎么从一个空的 maven 项目一步步建设成一个 spring-boot 项目还是不了解。最近刚好知道了一些，在这里记录一下。

![](static/2021-02/nicolas-lafargue-jdpJqWO9Sx8-unsplash.jpg)

---

## 第一步 创建一个 maven 项目
1、创建一个空的 spring-boot 项目。
```bash
mvn archetype:generate -DgroupId=com.sqlpy -DartifactId=setup-spring-boot -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false

tree setup-spring-boot 
setup-spring-boot
├── pom.xml
└── src
    ├── main
    │   └── java
    │       └── com
    │           └── sqlpy
    │               └── App.java
    └── test
        └── java
            └── com
                └── sqlpy
                    └── AppTest.java

```
2、pom 文件的内容如下。
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

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
  </properties>

  <dependencies>
    
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>


</project>
```

---

## 第二步 添加依赖
1、添加 spring-boot 的依赖。
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


</project>
```
2、这个依赖被添加之后，它会自动加载它的子依赖。

![](static/2021-02/spring-boot-dependences.jpeg)

---

## 第三步 添加代码
```java
package com.sqlpy;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Hello world!
 *
 */
@RestController
@SpringBootApplication
public class App 
{
    public static void main( String[] args )
    {
        SpringApplication.run(App.class);
    }

    @RequestMapping("/")
    public String index() {
        return "hello this is sqlpy.com";
    }
}

```

---

## 第四步 在 IDEA 中运行
1、在 IDEA 中运行 App.main 函数
```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.2.RELEASE)

2021-09-04 11:11:14.871  INFO 20886 --- [           main] com.sqlpy.App                            : Starting App on NEEKYJIANG-MB2 with PID 20886 (/Users/jianglexing/temps/setup-spring-boot/target/classes started by jianglexing in /Users/jianglexing/temps/setup-spring-boot)
2021-09-04 11:11:14.874  INFO 20886 --- [           main] com.sqlpy.App                            : No active profile set, falling back to default profiles: default
2021-09-04 11:11:14.907  INFO 20886 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@60611244: startup date [Sat Sep 04 11:11:14 CST 2021]; root of context hierarchy
2021-09-04 11:11:15.543  INFO 20886 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-09-04 11:11:15.557  INFO 20886 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-09-04 11:11:15.557  INFO 20886 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.31
2021-09-04 11:11:15.560  INFO 20886 --- [ost-startStop-1] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/Users/jianglexing/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.]
2021-09-04 11:11:15.609  INFO 20886 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-09-04 11:11:15.609  INFO 20886 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 704 ms
2021-09-04 11:11:15.685  INFO 20886 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2021-09-04 11:11:15.688  INFO 20886 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2021-09-04 11:11:15.689  INFO 20886 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2021-09-04 11:11:15.689  INFO 20886 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2021-09-04 11:11:15.689  INFO 20886 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2021-09-04 11:11:15.760  INFO 20886 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:11:15.890  INFO 20886 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@60611244: startup date [Sat Sep 04 11:11:14 CST 2021]; root of context hierarchy
2021-09-04 11:11:15.923  INFO 20886 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/]}" onto public java.lang.String com.sqlpy.App.index()
2021-09-04 11:11:15.927  INFO 20886 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2021-09-04 11:11:15.927  INFO 20886 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2021-09-04 11:11:15.940  INFO 20886 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:11:15.940  INFO 20886 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:11:16.022  INFO 20886 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2021-09-04 11:11:16.054  INFO 20886 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-09-04 11:11:16.058  INFO 20886 --- [           main] com.sqlpy.App                            : Started App in 1.496 seconds (JVM running for 2.069)
```
2、这个时候在浏览器中可以看到如下内容。

![](static/2021-02/manul-spring-boot-page.jpeg)

---

## 解决手工执行时的报错问题
1、刚才我们通过 IDEA 执行是没有问题的，但是有人选择手工执行，那么会遇到这样的一个错误。
```bash
mvn spring-boot:run

[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.479 s
[INFO] Finished at: 2021-09-04T11:20:30+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] No plugin found for prefix 'spring-boot' in the current project and in the plugin groups [org.apache.maven.plugins, org.codehaus.mojo] available from the repositories [local (/Users/jianglexing/.m2/repository), central-mirror (http://maven.sqlpy.com:8080/nexus/content/groups/public/)] -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/NoPluginFoundForPrefixException

```
2、从报错信息来看是插件在本地仓库中找不到，这里有一个要记住的地方是，这个插件声明了在 spring-boot-starter-parent 这个 pom 中，所以我们还要加一个 parent。
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


</project>

```
3、现在的工执行也没有问题了。
```bash
mvn spring-boot:run

[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------< com.sqlpy:setup-spring-boot >---------------------
[INFO] Building setup-spring-boot 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] >>> spring-boot-maven-plugin:2.0.2.RELEASE:run (default-cli) > test-compile @ setup-spring-boot >>>
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
[INFO] <<< spring-boot-maven-plugin:2.0.2.RELEASE:run (default-cli) < test-compile @ setup-spring-boot <<<
[INFO] 
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.0.2.RELEASE:run (default-cli) @ setup-spring-boot ---

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.0.2.RELEASE)

2021-09-04 11:28:57.520  INFO 22279 --- [           main] com.sqlpy.App                            : Starting App on NEEKYJIANG-MB2 with PID 22279 (/Users/jianglexing/temps/setup-spring-boot/target/classes started by jianglexing in /Users/jianglexing/temps/setup-spring-boot)
2021-09-04 11:28:57.522  INFO 22279 --- [           main] com.sqlpy.App                            : No active profile set, falling back to default profiles: default
2021-09-04 11:28:57.559  INFO 22279 --- [           main] ConfigServletWebServerApplicationContext : Refreshing org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@272de720: startup date [Sat Sep 04 11:28:57 CST 2021]; root of context hierarchy
2021-09-04 11:28:58.277  INFO 22279 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-09-04 11:28:58.297  INFO 22279 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-09-04 11:28:58.297  INFO 22279 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet Engine: Apache Tomcat/8.5.31
2021-09-04 11:28:58.307  INFO 22279 --- [ost-startStop-1] o.a.catalina.core.AprLifecycleListener   : The APR based Apache Tomcat Native library which allows optimal performance in production environments was not found on the java.library.path: [/Users/jianglexing/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.]
2021-09-04 11:28:58.369  INFO 22279 --- [ost-startStop-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-09-04 11:28:58.369  INFO 22279 --- [ost-startStop-1] o.s.web.context.ContextLoader            : Root WebApplicationContext: initialization completed in 813 ms
2021-09-04 11:28:58.455  INFO 22279 --- [ost-startStop-1] o.s.b.w.servlet.ServletRegistrationBean  : Servlet dispatcherServlet mapped to [/]
2021-09-04 11:28:58.459  INFO 22279 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'characterEncodingFilter' to: [/*]
2021-09-04 11:28:58.459  INFO 22279 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'hiddenHttpMethodFilter' to: [/*]
2021-09-04 11:28:58.459  INFO 22279 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'httpPutFormContentFilter' to: [/*]
2021-09-04 11:28:58.459  INFO 22279 --- [ost-startStop-1] o.s.b.w.servlet.FilterRegistrationBean   : Mapping filter: 'requestContextFilter' to: [/*]
2021-09-04 11:28:58.536  INFO 22279 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**/favicon.ico] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:28:58.668  INFO 22279 --- [           main] s.w.s.m.m.a.RequestMappingHandlerAdapter : Looking for @ControllerAdvice: org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@272de720: startup date [Sat Sep 04 11:28:57 CST 2021]; root of context hierarchy
2021-09-04 11:28:58.721  INFO 22279 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/]}" onto public java.lang.String com.sqlpy.App.index()
2021-09-04 11:28:58.724  INFO 22279 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2021-09-04 11:28:58.725  INFO 22279 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2021-09-04 11:28:58.741  INFO 22279 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:28:58.742  INFO 22279 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2021-09-04 11:28:58.856  INFO 22279 --- [           main] o.s.j.e.a.AnnotationMBeanExporter        : Registering beans for JMX exposure on startup
2021-09-04 11:28:58.891  INFO 22279 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-09-04 11:28:58.895  INFO 22279 --- [           main] com.sqlpy.App     
```

---