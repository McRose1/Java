# JDBC
Java Database Connectivity

是一种用于执行 SQL 语句的 Java API，可以为多种关系数据库提供统一访问，它由一组用 Java 语言编写的类和接口组成。

JDBC 提供了一种基准，据此可以构建更高级的工具和接口，使数据库开发人员能够编写数据库应用程序。

JDBC 需要连接驱动，驱动是两个设备要进行通信，满足一定通信数据格式，数据格式由设备提供商规定，设备提供商为设备提供驱动软件，通过软件可以与该设备进行通信。

## 原理
JDBC 是接口，驱动是接口的实现，没有驱动将无法完成数据库连接。

每个数据库厂商都需要提供自己的驱动，用来连接自己公司的数据库。

## 开发步骤
### 注册驱动
用来告诉 JVM 使用哪个生产厂商的驱动（MySQL、Oracle）

```java
class.forName("com.mysql.jdbc.Driver");
```

### 获得链接，链接数据库
使用 JDBC 中的类，完成对 MySQL 数据库的连接

获得连接需要方法 **DriverManager.getConnection(url, username, password)**
- url：需要连接数据库的位置（网址），由三部分组成，每个部分中间使用冒号分隔
  - jdbc：固定的
  - 数据库名称：MySQL
  - 数据库服务器的 IP 地址（localhost）、端口号（3306）、数据库名称：数据库厂商规定的
- username：用户名
- password：密码

```java
Connection conn = DriverManager.getConnection("jdbc:Mysql://localhost:3306/mydb,'root',''123456'");
```

### 获得 SQL 语句执行平台
使用 PreparedStatement 预处理对象，每条 SQL 语句所有的实际参数都使用逗号分隔

```java
String sql = "insert into sort(sid, sname) values(?,?)";
PreparedStatement ps = conn.preparedStatement(sql);
```

### 处理结果集
ResultSet 实际上就是一张二维的表格，我们可以调用其 boolean next() 方法指向某行记录，当第一次调用 next() 方法时，便指向第一行记录的位置，这时就可以使用 ResultSet 提供的 getXXX(int col) 方法来来获取指定列的数据：

有了 PreparedStatement 以后，我们可以采用：
- executeQeury(String)：查询
- executeUpdate(String sql)：更新数据
- execte(String sql)：不知道是查询还是修改的时候

### 释放资源
先得到的后关闭，后得到的先关闭

```java
rs.close()
con.close
```


