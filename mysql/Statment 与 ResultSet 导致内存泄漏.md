## 背景
最近看 MySQL 官方驱动程序 `MySQL Connector/J` 的文档，发现一处比较奇怪的代码。
```java
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("select id,name from tempdb.t;");

// ... 

// 
rs.close();
stmt.close();

```
之前的主力语言是 Python3 ，在那里我们只对 Connection 对象做资源的回收，看 java 代码他还要对 `Statement ResultSet` 类型的对象做资源的回收。难不成不回收的话还真会有内存泄漏？

![](static/2021-02/casey-horner-ygmZUdAzeBw-unsplash.jpg)

---

## 第一版 不回收--泄漏内存
```java
public class ResourceTest {
    /**
     * 测试不 close 语句对象和结果集对象会不会有内存泄漏的问题
     */
    public static void Qury(Connection conn) {
        try {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select id,name from tempdb.t;");
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                System.out.println("line [id=" + id + " name=" + name + "]");
                // rs.close();
                // stmt.close();
            }
        } catch (SQLException ex) {

        }
    }

    public static void main(String[] args) {
        System.out.println("test starting ");
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/tempdb?user=appuser");
            while(true) {
                Qury(conn);
            }
        } catch (SQLException ex) {

        } catch (Exception ex) {

        }
        System.out.println("test ending ");
    }
}
```

这版本下我的机器内存使用量从 12.02G 不一下就上涨到了 14G 看样子还在一直涨 ；应该是内存泄漏了。

---

## 第二版 回收--不泄漏内存 
```java
public class ResourceTest {
    /**
     * 测试不 close 语句对象和结果集对象会不会有内存泄漏的问题
     */
    public static void Qury(Connection conn) {
        try {
            Statement stmt = conn.createStatement();
            ResultSet rs = stmt.executeQuery("select id,name from tempdb.t;");
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                System.out.println("line [id=" + id + " name=" + name + "]");
                // close 进行内存回收
                rs.close();
                stmt.close();
            }
        } catch (SQLException ex) {

        }
    }

    public static void main(String[] args) {
        System.out.println("test starting ");
        Connection conn = null;
        try {
            conn = DriverManager.getConnection("jdbc:mysql://127.0.0.1:3306/tempdb?user=appuser");
            while(true) {
                Qury(conn);
            }
        } catch (SQLException ex) {

        } catch (Exception ex) {

        }
        System.out.println("test ending ");
    }
}
```
这个本版本的代码内存使用量基本不变。

---