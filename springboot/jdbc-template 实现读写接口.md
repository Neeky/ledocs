## 背景
最开始搞 spring-boot 的时候操作 MySQL 都是走的 jdbc ，用 jdbc 操作数据库的话比较原始，代码量也比较多，并且多出来的代码还主要是样板代码，这点真让人受不了。 最近学到了一个 jdbc-template 的技术，这家伙可以把 jdbc 的样板代码都去掉。

下面看一下用 jdbc-template 怎么实现一个读写接口。

![](static/2021-02/sonia-dauer-w4YEnlcYhZ0-unsplash.jpg)

---

## 配置 spring-boot 
1、在 pom.mxl 中添加 jdbc-template 相关的依赖
```xml
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.26</version>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
    </dependency>
```
2、在配置文件中添加相关的连接配置
```ini
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://neekystudio.com:3306/tempdb
spring.datasource.username=appuser
spring.datasource.password=dbma@0352
```
---

## 定义领域对象
假设我们的领域内只有一个用户对象，其它对应的属性如下。
```java
import lombok.Data;
import java.util.Date;

@Data
public class User {
    private Long id;
    private Date createAt;
    private String name;
    private String passwd;

    public User(String name,String passwd) {
        this.name = name;
        this.passwd = passwd;
    }
}
```

---

## 约定 http 接口
在这里我们只实现一个 insert 接口，两个查询接口。
|**接口**|**请求方法**|**意义**|
|----|----|----|
|http://127.0.0.1:8080/users | get |查询所有的 user 信息|
|http://127.0.0.1:8080/users/{id} | get |查询指定 id 对应的用户信息|
|http://127.0.0.1:8080/users | post | 添加一个新的用户|

---

## 定义数据接口
我们在这里只实现简单的插入和查询功能。
```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCreator;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.stereotype.Repository;

import java.sql.*;

public interface IUserRepository {
    Iterable<User> findAll();
    User findOne(Long id);
    User save(User user);
}

@Repository
@Slf4j
public class UserRepository implements IUserRepository{
    private JdbcTemplate jdbcTemplate;

    /**ORM*/
    private User mapRowToUser(ResultSet rs,int rowNum) throws SQLException {
        User user = new User(
            rs.getString("name"),
            rs.getString("passwd"));
        user.setId(rs.getLong("id"));
        user.setCreateAt(rs.getDate("createAt"));
        return user;
    }

    @Autowired
    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Iterable<User> findAll() {
        return this.jdbcTemplate.query("select id,createAt,name,passwd from tempdb.user",this::mapRowToUser);
    }

    @Override
    public User findOne(Long id) {
        return this.jdbcTemplate.queryForObject("select id,createAt,name,passwd from tempdb.user where id = ? ",this::mapRowToUser,id);
    }

    @Override
    public User save(User user) {
        // 使用数据库的时间，而不是使用应用服务器的时间，可以防止多台应用服务器时间不一致的问题。
        //int effected_rows = this.jdbcTemplate.update("insert into tempdb.user(createAt,name,passwd) values(now(),?,?)",user.getName(),user.getPasswd());

        // 如果要得到新插入行的 id 值可以这样做。
        KeyHolder keyHolder = new GeneratedKeyHolder();
        this.jdbcTemplate.update(
            new PreparedStatementCreator() {
                @Override
                public PreparedStatement createPreparedStatement(Connection connection) throws SQLException {
                    // Statement.RETURN_GENERATED_KEYS 让语句执行完成之后把最后得到的主键也带出来
                    PreparedStatement ps = jdbcTemplate.getDataSource().getConnection()
                        .prepareStatement("insert into tempdb.user(createAt,name,passwd) values(now(),?,?)", Statement.RETURN_GENERATED_KEYS);
                    ps.setString(1,user.getName());
                    ps.setString(2,user.getPasswd());
                    return ps;
                }
            },keyHolder
        );
        Long primaryKey = keyHolder.getKey().longValue();
        user.setId(primaryKey);

        log.info("user用户的 id = " + user.getId());
        return user;
    }

}

```

`mapRowToUser` 函数可以把一行数据转换成一个 `User` 对象，也就是说在使用 jdbc-template 这个技术解决方案的场景下，ORM 要程序员自己做。

---

## 实现控制器
我们在控制器中实例业务逻辑。
```java
import com.example.demo.common.JsonResult;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@Slf4j
public class UserController {
    @Autowired
    private UserRepository userRepository;

    @GetMapping(value="/users")
    public JsonResult<Object> queryAllUser() {
        log.info("查询所有的 User 对象 .");
        JsonResult<Object> jsonResult = new JsonResult<>();
        try {
            jsonResult.setData(userRepository.findAll());
        } catch (Exception ex) {
            log.error("查询所有用户的时候遇到异常 " + ex.getMessage());
            jsonResult.setErrorCode(2);
            jsonResult.setErrorMessage(ex.getMessage());
        }
        log.info("查询所有的 User 对象成功 .");
        return jsonResult;
    }

    @GetMapping(value="/users/{id}")
    public JsonResult<Object> queryUser(@PathVariable Long id) {
        log.info("进入查询用户的逻辑.");
        JsonResult<Object> jr = new JsonResult<>();
        try {
            User user = this.userRepository.findOne(id);
            jr.setData(user);
        } catch (Exception ex) {
            log.info("检查用户的时候出现了异常.");
            jr.setData(null);
            jr.setErrorCode(1);
            jr.setErrorMessage(ex.getMessage());
            ex.printStackTrace();
        }
        log.info("将要退出查询用户的逻辑.");
        return jr;
    }

    @PostMapping(value = "/users")
    public JsonResult<Object> createUser(@RequestBody User user) {
        JsonResult<Object> jsonResult = null;
        log.info("准备创建新用户 " + user.toString() );
        try {
            this.userRepository.save(user);
            jsonResult = new JsonResult<>();
            jsonResult.setData(user);
            log.info("创建完成");
        } catch (Exception ex) {
            log.info("创建用户失败 \n" + ex.getMessage());
        }

        return jsonResult;
    }
}

```

`@PathVariable` 用于从 url 路径中提取数据。

`@RequestBody` 用于从提交上来的 json 对象中提取数据。

---