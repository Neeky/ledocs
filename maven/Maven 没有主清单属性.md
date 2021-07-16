## 背景
默认情况下 maven 项目不会帮我们创建任何类，如果不指定我们类中的 main 方法作为入口点，那么在执行时会报如下错误。
```bash
#打包
mvn clean package

#运行的时候会报错
java -jar target/letcode01-1.0-SNAPSHOT.jar 
target/letcode01-1.0-SNAPSHOT.jar中没有主清单属性

```
![](static/2021-02/tim-umphreys-SA_Fq_qPy7U-unsplash.jpg)


## 解决办法
解决办法也比较直接，那就是告诉 maven 去哪个类中找 main 方法并执行就行了；默认生成的配置文件是没有这个信息的。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>letcode01</artifactId>
    <version>1.0-SNAPSHOT</version>


</project>
```
添加 build 结点配置 maven 编译时的用到的信息，完整的 pom.xml 文件内容如下。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>letcode01</artifactId>
    <version>1.0-SNAPSHOT</version>

    <build>
        <finalName>${project.artifactId}-${project.version}</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.2.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                            <mainClass>com.letcode.Solution</mainClass> <!-- 指定程序入口点所在类 -->
                        </manifest>
                    </archive>
                </configuration>

            </plugin>
        </plugins>
    </build>
</project>
```

---
