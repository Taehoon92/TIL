# DB CRUD - using JDBC

**JDBC** (**J**ava **D**ata**B**ase **C**onnectivity) is a java api that makes  access to databases from java.  
These three functions are standard interface.  
* java.sql.Connection - Connect
* java.sql.Statement - Contents of SQL
* java.sql.ResultSet - Response of SQL request

JDBC is old and complicated to use. Recently, there are various technologies that conveniently use JDBC rather than using JDBC directly. Representatively, there are **SQL Mapper** and **ORM** technologies. These technologies also use JDBC internally. Therefore, even if we don't have to use JDBC directly, we should know the basic principle of how it works.

## Connect to Database
I used H2 DB for these examples. 

Basic information for DB connection.
```java
public abstract class ConnectionConst {
    public static final String URL = "jdbc:h2:tcp://localhost/~/test";
    public static final String USERNAME = "sa";
    public static final String PASSWORD = "";
}
```

Connect DB using JDBC
```java
public class DBConnectionUtil {

    public static Connection getConnection() {
        try {
            //static import - ConnectionConst
            Connection connection = DriverManager.getConncection(URL, USERNAME, PASSWORD);
            return connection;
        } catch(SQLException e) {
            throw new IllegalStateException(e);
        }
    }
}
```

## Manage member data in database using JDBC

### Member class
```java
@Data
public class Member {

    private String memberId;
    private int money;

    public Member() {
    }

    public Member(String memberId, int money) {
        this.memberId = memberId;
        this.moeny = money;
    }
}
```
### Member Register
#### MemberRepository
```java
public class MemberRepository  {
    public Member save(Member member) throws SQLException {
        String sql = "insert into member(member_id, money) values(?, ?)";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            //set value(memberId) in first "?" 
            pstmt.setString(1, member.getMemberId);
            //set value(money) in second "?"
            pstmt.setInt(2, member.getMoney());
            //The SQL prepared through the statement is delivered to the DB through the connection
            pstmt.executeUpdate();
            return member;
        } catch (SQLException e) {
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }

    private void close(Connection con, Statement stmt, ResultSet rs) {
        if(rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }

        if(stmt != null) {
            try {
                stmt.close();
            } catch(SQLException e) {
                log.info("error", e);
            }
        }

        if(con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                log.info("error", e);
            }
        }
    }

    private Connection getConnection() {
        return DBConnectionUtil.getConnection();
    }
}
```
#### Test
```java
//Test
class MemberRepositoryTest {
    MemberRepository repository = new MemberRepository();

    @Test
    void crud() throws SQLException {
        //save
        Member member = new Member("member1", 10000);
        repository.save(member);
    }
}
```

### Member Find
#### MemberRepository
```java
public class MemberRepository  {
    ...
    public Member findById(String memberId) throws SQLException {
        String sql = "select * from member where member_id";

        Conncetion con = null;
        PreparedStatement pstmt = null;
        ResultSet rs = null;

        try {
            con = getConnection();
            pstmt = con.prepareStatement(sql);
            pstmt.setString(1, memberId);
            //Use .executeQuery() to read data
            rs = pstmt.executeQuery();

            if(rs.next()) {
                Member member = new Member();
                member.setMemberId(rs.getString("member_id"));
                member.setMoney(rs.getInt("money"));
                return member;
            } else {
                throw new NoSuchElementException("member not found memberId = " + memberId);
            }
        } catch (SQLException e) {
            log.error("db error",e);
            throw e;
        } finally {
            close(con, pstmt, rs);
        }
    }
}
```

Results of query are stored in ResultSet in order. For example, if the query is `select member_id, money ...`, the data will be saved as `member_id` and `money`. (cf. `select *` means select all of the column)  
There is a cursor inside the ResultSet. At first, cursor doesn't point data, so you have to call `rs.next()` at least once to read data. Cursor moves next data when `rs.next()` is called. If the return value of `rs.next()` is true, it means the data is exist where cursor is pointed now.  
* `rs.getString("member_id")` : return the `member_id` data currently pointed at by the cursor as `String` type.
* `rs.getInt("money")` : return the `money` data currently pointed at by the cursor as `int` type.

#### Test
```java
class MemberRepositoryTest {
    
    @Test
    void crud() throws SQLException {
        ...
        
        //findById
        Member findMember = repository.findById(member.getMemberId())
        Assertions.assertThat(findMember).isEqualTo(member);
    }
}
```

### Member Update
#### MemberRepository
```java
public class MemberRepository  {
    ...

    public void update(String memberId, int money) throws SQLException {
        String sql = "update member set money=? where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.preparedStatement(sql);
            pstmt.setInt(1, money);
            pstmt.setString(2, memberId);
            int resultSize = pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
}
```
`executeUpdate()` return number of effected row after execute query. In this code `resultSize` is 1. If there are 100 members and update data for all of them, `executeUpdate()` will return 100.

#### Test
```java
class MemberRepositoryTest {
    
    @Test
    void crud() throws SQLException {
        ...
        //update - money : 10000 -> 20000
        repository.update(member.getMemberId(), 20000);
        Member updatedMember = repository.findById(member.getMemberId());
        Assertions.assertThat(updatedMember.getMoney()).isEqualTo(20000);
    }
}
```

### Member Delete
#### MemberRepository
```java
public class MemberRepository  {
    ...

    public void delete(String memberId) throws SQLException {
        String sql = "delete from member where member_id=?";

        Connection con = null;
        PreparedStatement pstmt = null;

        try {
            con = getConnection();
            pstmt = con.preparedStatement(sql);
            pstmt.setString(1, memberId);
            pstmt.executeUpdate();
        } catch (SQLException e) {
            log.error("db error", e);
            throw e;
        } finally {
            close(con, pstmt, null);
        }
    }
}
```

#### Test 
```java
class MemberRepositoryTest {
    
    @Test
    void crud() throws SQLException {
        ...
        //delete
        repository.delete(member.getMemberId());
        Assertions.assertThatThrownBy(() -> repository.findById(member.getMemberId()))
            .isInstanceof(NoSuchElementException.class);
    }
}
```
