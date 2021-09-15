## 背景

最近在搞 spring-boot 的 web 开发，发现这个框架有一个非常有意思的点就是 IOC 容器，在这里 spring 不建议我们创建领域内的对象，而是推荐由 IOC 容器创建，我们直接用就是了。一开始我还为自己不到直接取得这些实例而感到不自在，一个 demo 下来之后发现，我还是真没有必要什么都自己做。

现在记录一下这个典型的流程。

![](static/2021-02/tobias-rademacher-305t95tcAgk-unsplash.jpg)


---

## 第一步 定义领域对象
假设我只有一个 User 类。
```java
package com.example.demo.pojos;

import lombok.Data;

@Data
public class User {
    private String name;
    private Integer age;

    public User(String name,Integer age) {
        this.name = name;
        this.age = age;
    }
}

```

---

## 第二步 配置 IOC 容器
配置好我们的 IOC 容器。
```java
package com.example.demo.configs;

import com.example.demo.pojos.User;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {

    @Bean
    public User user() {
        return new User("tom",16);
    }
}
```
---

## 第三步 在控制器中使用
在控制器中使用自动装配。
```java
package com.example.demo.controllers;

import com.example.demo.pojos.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/hello")
public class HelloController {

    @Autowired
    private User user; // user 与 AppConfig 中的 user 函数对应

    @GetMapping
    public String hello() {
        return "你好 我叫 " + this.user.getName() + " 。";
    }
}

```
这里的 user 会自动从我们的 IOC 容器中取得，对象默认是单例的。

---

## 第四小 启动服务
入口代码动都不用动。
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}

```

```bash
mvn spring-boot:run

2021-09-15 17:09:15.498  INFO 97648 --- [           main] com.example.demo.DemoApplication         : Starting DemoApplication using Java 1.8.0_282 on NEEKYJIANG-MB2 with PID 97648 (/Users/jianglexing/repos/java-exsamples/springboots/java-base-bean/target/classes started by jianglexing in /Users/jianglexing/repos/java-exsamples/springboots/java-base-bean)
2021-09-15 17:09:15.499  INFO 97648 --- [           main] com.example.demo.DemoApplication         : No active profile set, falling back to default profiles: default
2021-09-15 17:09:16.051  INFO 97648 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-09-15 17:09:16.059  INFO 97648 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-09-15 17:09:16.059  INFO 97648 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.52]
2021-09-15 17:09:16.094  INFO 97648 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-09-15 17:09:16.095  INFO 97648 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 559 ms
2021-09-15 17:09:16.287  INFO 97648 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-09-15 17:09:16.293  INFO 97648 --- [           main] com.example.demo.DemoApplication         : Started DemoApplication in 1.06 seconds (JVM running for 1.29)


```

---

## 检查效果
直接用 curl 访问对应的接口。
```bash
curl http://127.0.0.1:8080/hello
你好 我叫 tom 
```

---