## SpringBoot rest-service 项目

rest-service 作为 SpringBoot 官方的第一个项目，还是有不少东西的；从技术上来讲项目要实现一个简单的功能，服务端接收一个 GET 请求，并返回一段 json 。
请求的 url 格式如下 `http://localhost:8080/greeting?name=User` ，返回的 Json 格式如下。
```json
{"id":1,"content":"Hello, User!"}
```
![](static/2021-01/bakd-raw-by-karolin-baitinger--w2JJjlcXcw-unsplash.jpg)

---

## 第一步 创建 rest-service 项目
1、创建有 Spring Web 依赖的 SpringBoot 项目
![](static/2021-01/rest-server-01.jpg)

---

## 第二步 创建资源表现类
SpringBoot 会自动帮我们把对应转化为 Json ，我们要做的就是定义好要导出属性的 Get 访问器就行了。

`src/main/java/com/example/restservice/Greeting.java`

```java
package com.example.restservice;

public class Greeting {
    private final String content;
    private final long id;

    public Greeting(long id,String content) {
        this.id = id;
        this.content = content;
    }

    public long getId() {
        return id;
    }

    public String getContent(){
        return content;
    }
}

```
---

## 第三步 创建资源控制器

所有的 Http 请求都交由控制器来处理，第二步我们已经定义好了要返回的数据，这里我们只要实现处理的逻辑就行了。
`src/main/java/com/example/restservice/GreetingController.java`
```java
package com.example.restservice;

import java.util.concurrent.atomic.AtomicLong;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class GreetingController {

	private static final String template = "Hello, %s!";
	private final AtomicLong counter = new AtomicLong();

	@GetMapping("/greeting")
	public Greeting greeting(@RequestParam(value = "name", defaultValue = "World") String name) {
		return new Greeting(counter.incrementAndGet(), String.format(template, name));
	}
}
```

---

## 第四步 编译
SpringBoot 把 tomcat 也内嵌了，这就使得我们编译出来的 jar 包直接有 web 服务，所以运行起来网站后台就完成了。
```bash
mvn package

...
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

2021-07-13 20:17:34.718  INFO 45597 --- [           main] c.e.r.RestServiceApplicationTests        : Starting RestServiceApplicationTests using Java 1.8.0_282 on NEEKYJIANG-MB2 with PID 45597 (started by jianglexing in /Users/jianglexing/repos/java-projects/rest-service)
2021-07-13 20:17:34.720  INFO 45597 --- [           main] c.e.r.RestServiceApplicationTests        : No active profile set, falling back to default profiles: default
2021-07-13 20:17:35.627  INFO 45597 --- [           main] c.e.r.RestServiceApplicationTests        : Started RestServiceApplicationTests in 1.054 seconds (JVM running for 1.646)
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.428 s - in com.example.restservice.RestServiceApplicationTests
[INFO] 
[INFO] Results:
[INFO] 
[INFO] Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
[INFO] 
[INFO] 
[INFO] --- maven-jar-plugin:3.2.0:jar (default-jar) @ rest-service ---
[INFO] 
[INFO] --- spring-boot-maven-plugin:2.5.2:repackage (repackage) @ rest-service ---
[INFO] Replacing main artifact with repackaged archive
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 3.509 s
[INFO] Finished at: 2021-07-13T20:17:36+08:00
[INFO] ------------------------------------------------------------------------
```

---

## 第五步 起服务
这一步最简单了直接运行 jar 文件就行了
```bash
java -jar target/rest-service-0.0.1-SNAPSHOT.jar

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.2)

2021-07-13 20:19:04.292  INFO 45710 --- [           main] c.e.restservice.RestServiceApplication   : Starting RestServiceApplication v0.0.1-SNAPSHOT using Java 1.8.0_282 on NEEKYJIANG-MB2 with PID 45710 (/Users/jianglexing/repos/java-projects/rest-service/target/rest-service-0.0.1-SNAPSHOT.jar started by jianglexing in /Users/jianglexing/repos/java-projects/rest-service)
2021-07-13 20:19:04.294  INFO 45710 --- [           main] c.e.restservice.RestServiceApplication   : No active profile set, falling back to default profiles: default
2021-07-13 20:19:05.239  INFO 45710 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2021-07-13 20:19:05.251  INFO 45710 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2021-07-13 20:19:05.251  INFO 45710 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.48]
2021-07-13 20:19:05.307  INFO 45710 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2021-07-13 20:19:05.307  INFO 45710 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 968 ms
2021-07-13 20:19:05.615  INFO 45710 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2021-07-13 20:19:05.623  INFO 45710 --- [           main] c.e.restservice.RestServiceApplication   : Started RestServiceApplication in 1.706 seconds (JVM running for 2.124)
```

---

## 第六步 检查 WEB 服务是否正常

![](static/2021-01/rest-server-02.jpg)


---

## 例子中用到的关键注解
1、`@SpringBootApplication` 会去扫项目中所有的控制器，并且会去做一些自动配置。

2、`@RestController` 会把类标记为“控制器”。

3、`@GetMapping("/greeting")` 映射对应路径的 GET 请求到方法。

4、`@RequestParam(value = "name", defaultValue = "World")` 请求的值从哪个请求的参数中取，并且指出默认值是多少。

---