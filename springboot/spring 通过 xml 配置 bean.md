## 背景
最近在读 spring 的官方文档，IOC 容器一上来就是通过 xml 来配置 bean 并通过 `ApplicationContext` 来获取 xml 中配置的实体；如果不动手搞一搞的话会非常容易眼高手低。所以这里也写一个例子实践一下。

![](static/2021-02/alex-loup-hMTB568ur0g-unsplash.jpg)

---

## 第一步 创建 maven 项目
创建 maven 项目并添加必要的依赖。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>use-xml-beans</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.8</version>
        </dependency>
    </dependencies>

</project>
```
这里直接引用 spring-webmvc 主要是为了方便，主要是因为它会自动帮我们引用其它的 spring 包。

---

## 第二步 添加实例类
先定义一个 Student 实例类，为后面写 xml 文件做准备。
```java
package org.example;

public class Student {
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                '}';
    }

    private String name;
}
```

---

## 第三步 添加 application.xml 文件
按约定这个文件要保存在 `resources` 目录下，application.xml 的内容如下。
```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="student" class="org.example.Student">
        <property name="name" value="tom"></property>
    </bean>

</beans>
```

---

## 第四步 添加入口类
```java
package org.example;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {

    }
}
```

---

## 第五步 使用 IOC 容器
```java
package org.example;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class App {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");
        Student student = context.getBean("student",Student.class);
        System.out.println(student);
    }
}
```

---

## 第六步 执行查看效果
```bash
/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/bin/java -agentlib:jdwp=transport=dt_socket,address=127.0.0.1:64770,suspend=y,server=n -javaagent:/Users/jianglexing/Library/Caches/JetBrains/IntelliJIdea2020.2/captureAgent/debugger-agent.jar -Dfile.encoding=UTF-8 -classpath /Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/charsets.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/cldrdata.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/dnsns.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/jaccess.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/localedata.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/nashorn.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/sunec.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/sunjce_provider.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/sunpkcs11.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/ext/zipfs.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/jce.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/jfr.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/jsse.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/management-agent.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/resources.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/jre/lib/rt.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/lib/jconsole.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/lib/sa-jdi.jar:/Library/Java/JavaVirtualMachines/openjdk-8.jdk/Contents/Home/lib/tools.jar:/Users/jianglexing/repos/java-exsamples/springs/use-xml-beans/target/classes:/Users/jianglexing/.m2/repository/org/springframework/spring-webmvc/5.3.8/spring-webmvc-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-aop/5.3.8/spring-aop-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-beans/5.3.8/spring-beans-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-context/5.3.8/spring-context-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-core/5.3.8/spring-core-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-jcl/5.3.8/spring-jcl-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-expression/5.3.8/spring-expression-5.3.8.jar:/Users/jianglexing/.m2/repository/org/springframework/spring-web/5.3.8/spring-web-5.3.8.jar:/Applications/IntelliJ IDEA.app/Contents/lib/idea_rt.jar org.example.App


Student{name='tom'}

```