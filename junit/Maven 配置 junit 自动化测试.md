## 背景

最近写了一些 java 代码，但是没有给他们配套上相应的测试用例，心里总有点不踏实。看 java 大家都在用 JUnit 我也就来学习一下怎么用。

![](static/2021-02/pavel-marianov-8nAb6Rt4CEs-unsplash.jpg)

---

## 第一步 添加依赖
把 JUnit 相关的依赖声明添加到 pom.xml 中去。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sqlpy</groupId>
    <artifactId>ujtest</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

---

## 第二步 添加业务代码

对于 TDD 来说应该是先写测试用例的，在这里还是按传统的来，先添加业务代码；我在这里先做一个简单的加法计算器。文件所在位置如下 `src/main/java/com/sqlpy/Calclulator.java` 。
```java
package com.sqlpy;

public class Calculator {
    public double add(double x,double y) {
        return x + y;
    }
}
```

---

## 第三步 添加测试代码
添加业务逻辑的测试代码。
```java
package com.sqlpy;

import static org.junit.Assert.*;
import org.junit.Test;

public class CalculatorTest {
    @Test
    public void testAdd() {
        Calculator cal = new Calculator();
        double result = cal.add(100,100);
        assertEquals(200,result,0);
    }
}
```

---

## 第四步 通过 maven 运行用例
```bash
mvn test

[INFO] Scanning for projects...
[INFO] 
[INFO] --------------------------< com.sqlpy:ujtest >--------------------------
[INFO] Building ujtest 1.0-SNAPSHOT
[INFO] --------------------------------[ jar ]---------------------------------
[INFO] 
[INFO] --- maven-resources-plugin:2.6:resources (default-resources) @ ujtest ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] Copying 0 resource
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:compile (default-compile) @ ujtest ---
[INFO] Nothing to compile - all classes are up to date
[INFO] 
[INFO] --- maven-resources-plugin:2.6:testResources (default-testResources) @ ujtest ---
[WARNING] Using platform encoding (UTF-8 actually) to copy filtered resources, i.e. build is platform dependent!
[INFO] skip non existing resourceDirectory /Users/jianglexing/temps/java-learing/letcodes/ujtest/src/test/resources
[INFO] 
[INFO] --- maven-compiler-plugin:3.1:testCompile (default-testCompile) @ ujtest ---
[INFO] Changes detected - recompiling the module!
[WARNING] File encoding has not been set, using platform encoding UTF-8, i.e. build is platform dependent!
[INFO] Compiling 1 source file to /Users/jianglexing/temps/java-learing/letcodes/ujtest/target/test-classes
[INFO] 
[INFO] --- maven-surefire-plugin:2.12.4:test (default-test) @ ujtest ---
[INFO] Surefire report directory: /Users/jianglexing/temps/java-learing/letcodes/ujtest/target/surefire-reports

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.sqlpy.CalculatorTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.05 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 1.481 s
[INFO] Finished at: 2021-07-26T19:50:23+08:00
[INFO] ------------------------------------------------------------------------
```

---
