# DB - Using Data Source (CRUD)

Previously, I posted about [connecting to DB and read and write data using JDBC](https://github.com/Taehoon92/TIL/blob/main/Spring/DB-crud-jdbc.md).  In this post, let's study about connecting to DB using Connection Pool and Data Source.

## Connection Pool

Obtaining a DB connection involves the following complex process. 
1. Application logic search connection using DB driver.
2. DB driver connects DB and TCP/IP Connection.
3. After connect TCP/IP Connection, DB driver send additional information such as ID and PW.
4. DB progress internal authentication using ID and PW. Then, generate DB session.
5. DB sends a response that connection creation is complete.
6. DB driver generates object of connection and returns it.

If server runs these steps every time, it's not only complicated but also takes lots of time. In the end, it affect response time. To resolve this problem, a method called **Connection Pool** is used to generate connections in advance.

You can set how many connections to keep in the pool. At application startup, the connection pool get connections. When application logic requests a connection, the connection pool returns one of the connections (by reference) it has. After using the connection, it must be closed.  

There are several open source connection pool. Representative open source connection pools include `HikariCP`, `tomcat-jdbc pool` and `commons-dbcp2`. Recently, `hikariCP` is mainly used. Also, `hikariCP` is provided as a default connection pool from Spring Boot 2.0

## DataSource
When changing the connection pool, there is a problem that the code must always be changed. To solve these problem, Java provides the `javax.sql.DataSource` interface. `DataSource` is the interface that abstract the method to get connection. The core function of this interface is `getConnection` 

Most of connection pools implement the `DataSource` interface. However, `DriverManager` doesn't use the `DataSource` interface. But Spring provide `DriverManagerDataSource` class that implement `DataSource`.

### DataSource Example - DriverManager
Here is an example of `DriverManager` posted in [the previous post](https://github.com/Taehoon92/TIL/blob/main/Spring/DB-crud-jdbc.md).

```java
public class ConnectionTest {
    @Test
    void driverManager() throws SQL Exception {
        Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connextion={}, class={}", con2, con2.getClass());
    }
}
```

RESULT (log)
```
connection=conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class org.h2.jdbc.JdbcConnection
connection=conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class org.h2.jdbc.JdbcConnection
```

### DataSource Example - DriverManagerDataSource
Here is an example of `DriverManagerDataSource`. It's similar with the code using `DriverManager`, but it's available to get connection through `DataSource`.

```java
public class ConnectionTest {
    @Test
    void dataSourceDriverManager() throws SQLException {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
    
        Connection con1 = dataSource.getConnection();
        Connection con2 = dataSource.getConnection();
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connextion={}, class={}", con2, con2.getClass());
    }
}
```

RESULT (log)
```
connection=conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class org.h2.jdbc.JdbcConnection
connection=conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class org.h2.jdbc.JdbcConnection
```

When using the `DriverManager`, parameters such as URL, USERNAME and PASSWORD must always be passed whenever a connection is acquired. On the other hand, with `DataSource`, parameters need only be passed when the object is created for the first time. To obtain a connection, you only need to call `dataSource.getConnection()`.

### DataSource Example - Connection Pool (Hikari)

```java
public class ConnectionTest {
    @Test
    void dataSourceConnectionPool() throws SQLException, InterruptedException {
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);
        dataSource.setMaximumPoolSize(10);
        dataSource.setPoolName("TestPool");

        Connection con1 = dataSource.getConnection();
        Connection con2 = dataSource.getConnection();
        log.info("connection={}, class={}", con1, con1.getClass());
        log.info("connextion={}, class={}", con2, con2.getClass());
    }
}
```

RESULT (log)
```
TestPool - Starting...
TestPool - Added connection conn0: url=jdbc:h2:tcp://localhost/~/test user=SA
TestPool - Start completed.
connection=HikariProxyConnection@1431530910 wrapping conn0: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection
connection=HikariProxyConnection@1607792885 wrapping conn1: url=jdbc:h2:tcp://localhost/~/test user=SA, class=class com.zaxxer.hikari.pool.HikariProxyConnection
```

### Applying DataSource

```java
public class MemberRepository {
    
    private final DataSource dataSource;

    public MemberRepository(DataSource dataSource) {
        this.dataSource = dataSource;
    }
    
    //Same as previous codes
    // save() ...
    // findById() ...
    // update() ...
    // delete() ...

    private void close(Connection con, Statement stmt, ResultSet rs) {
        JdbcUtils.closeResultSet(rs);
        JdbcUtils.closeStatement(stmt);
        JdbcUtils.closeConnection(con);
    }

    private Connection getConnection() throws SQLException {
        Connection con = dataSource.getConnection();
        return con;
    }
}   
```

```java
class MemberRepositoryTest {
    MemberRepository repository;

    @BeforeEach
    void beforeEach() throws Exception {
        // DriverManager - obtaining new connection every time.
        //DriverManagerDataSource dataSoruce = new DriverManagerDataSource(URL, USERNAME, PASSWORD);

        // Connection Pooling : HikariProxyConnection -> JdbcConnection
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);

        repository = new MemberRepository(dataSource);
    }

    @Test
    void crud()
    ...
}
```