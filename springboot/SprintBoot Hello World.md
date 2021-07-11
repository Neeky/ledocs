## 背景
之前一直是用 `django` 来做 web ,最近因为工作上的一些原因，要用 `java` 来做 web 了，当然啦不可能从 socket 开始写，上面已经把框架定好了 `SpringtBoot`。

初步试了一下感觉 SpringtBoot 还是比较智能的，记一下今天写的 hello world 方面一次直接下手。


---

## 第一步 通过 idea 创建项目
当然可以通过 SpringBoot 的官网来创建项目，由于目前团队用的开发工具为 IDEA ，而它已经帮我们把这个一步“代理”了一下，所以可以直接通过它来创建项目。

1、新建项目
![static](static/2021-01/springboot-01.jpg)

2、选择创建一个 springboot 项目
![static](static/2021-01/springboot-02.jpg)

3、填写项目信息(这里直接使用默认值了)
![static](static/2021-01/springboot-03.jpg)

4、添加 spring-web 这个依赖
![static](static/2021-01/springboot-04.jpg)

5、指定项目的保存路径
![static](static/2021-01/springboot-05.jpg)

---

## 第二步 检查项目是否可以成功运行
通过 SpringBoot 创建的项目是一个最小化的可运行的 web 站点,我们可以直接通过 IDEA 运行它。

![static](static/2021-01/springboot-06.jpg)

```bash
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

2021-07-11 23:17:49.120  INFO 11823 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 15.0.2 on LexingdeMacBook-Pro.local with PID 11823 (/Users/jianglexing/java-projects/demo/target/classes started by jianglexing in /Users/jianglexing/java-projects/demo)
2021-07-11 23:17:49.122  INFO 11823 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2021-07-11 23:17:50.018  INFO 11823 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-07-11 23:17:50.033  INFO 11823 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-07-11 23:17:50.034  INFO 11823 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.48]
2021-07-11 23:17:50.103  INFO 11823 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-07-11 23:17:50.103  INFO 11823 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 934 ms
2021-07-11 23:17:50.398  INFO 11823 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-07-11 23:17:50.406  INFO 11823 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.924 seconds (JVM running for 2.501)

Process finished with exit code 130 (interrupted by signal 2: SIGINT)

```

## 第三步 加一个自己的控制器
1、在 DemoApplication 同级目录下创建一个叫 `controller` 的包，再在包里面加一个 `HelloWorldController` 的类。

![static](static/2021-01/springboot-07.jpg)

2、控制器完整的代码如下
```java
package com.example.demo.controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloWorldController {

    @RequestMapping("/hello")
    public String hello() {
        return "Hello world";
    }
}


```

---

## 第四步 访问我们的 HelloWorld 程序
现在可以通过浏览器访问到我们的页面了。

![static](static/2021-01/springboot-08.jpg)

---