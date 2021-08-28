## 背景
按 Spring 官方文档的路子，是先学习 xml 配置 bean ，学会之后感觉自己没有必要再搞其它的配置方式了，今天顺着文档看下去，发现可以通过注解配置 bean ；我只想说真香啊！

![](static/2021-02/claudio-schwarz-CpLtolBlZmQ-unsplash.jpg)

---

## 第一步添加实例类
搞一个 Person 的实体类。
```java
package com.example.usejavabens;

public class Person {
    private String name;

    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Person{" +
            "name='" + name + '\'' +
            ", age=" + age +
            '}';
    }
}

```

---

## 第二步添加配置类
新的解决方案把类对应到了 beans 标签，把方法对应到了 bean 标签。
```java
package com.example.usejavabens;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration // 这个注定类似说这个类就是一个 bean 的配置文件
public class AppConfig {


    @Bean // 告诉 Spring 这是一个 bean ，到时候要把它放到 IOC 容器中去
    public Person person() {
        return new Person();
    }
}
```
---

## 第三步使用IOC容器
用起来也是非常的爽。
```java
package com.example.usejavabens;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

@SpringBootApplication
public class UseJavaBensApplication {

    public static void main(String[] args) {
        SpringApplication.run(UseJavaBensApplication.class, args);

        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        Person person = context.getBean(Person.class);
        System.out.println(person.toString());
    }

}
```

---


