# Auto Increment problem in Unit Test

There was problem in my project using Spring Data JPA when I executed test code.


### PROBLEM
In more than 2 test methods, the value of id in first execution method is 1L. So, the first test is passed without any problem. 

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class UserRepositoryTest {

    @Autowired UserRepository userRepository;
    @PersistenceContext EntityManager em;

    @Test
    void createAndReadTest() {
        User user = UserFactory.createUser();
        userRepository.save(user);

        User foundUser = userRepository.findById(1L);
        assertThat(foundUser.getId()).isEqualTo(user.getId());
    }

    @Test
    void memberDateTest() {
        User user = UserFactory.createUser();
        userRepository.save(user);

        User foundUser = userRepository.findById(1L).orElse(null);
        assertThat(foundUser.getCreatedDate()).isNotNull();
    }

    @Test
    void updateTest() {
        User user = UserFactory.createUser();
        userRepository.save(user);

        User foundUser = userRepository.findById(1L).orElse(null);
        foundUser.modify("Hello", user.getPassword());
        assertThat(foundUser.getName()).isEqualTo("Hello");
    }

```

However, other tests failed. Even if `userRepository.deleteAll()` used, record of id is not initialized. For example, it will be 2L, 3L, .... So, `findById(1L)` will throw an error.  
When these tests are executed one by one, it works fine. But we can't execute all the tests one by one.

--- 
      
### SOLUTION
The problem is auto increment value. Therefore, the solution is to reassign the auto-increment value according to the database used for testing.
```java
@AfterEach
    void teardown() {
        this.userRepository.deleteAll();
        this.em.createNativeQuery("alter table `user` auto_increment = 1")
                .executeUpdate();
    }
```
The method with `@AfterEach` annotation is executed after every single test. ``alter table `user` auto_increment = 1`` this query set auto increment value to 1.

---    

### RESTRICTION

This solution is only available at Service Class that is applied `@Transactional` annotation (such as `@DataJpaTest`).