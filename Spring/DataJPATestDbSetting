# Using H2 DB in Test Environment

I'm using SQL in my side project but I was wondering to use H2 DB for the test codes. 




Add these syntax in `application.yml`
```yml
# H2 Database setting for test
spring:
  config:
    activate:
      on-profile: test
  h2:
    console:
      enabled: true
  jpa:
    hibernate:
      ddl-auto: create-drop
    database: h2
  datasource:
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb;MODE=MySQL
    username: SA
    password:
```

I set `on-profile: test` in `application.yml`. It named "test" and add `@ActiveProfiles("test")` at test code like this.  
```
@DataJpaTest
@ActiveProfiles("test")
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
```

`@DataJpaTest` loads only JPA related test setting. It is automatically using in-memory embedded database(h2). It is replaced automatically, so if you don't want to replace database, you have to change `@AutoConfigureTestDatabase` like this. `@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)`. 
>There is 3 options for `@AutoConfigureTestDatabase`.  
>
>**ANY** : Replace the DataSource whether it was auto-configured or manually defined.  
>**AUTO_CONFIGURED** : Only replace the Datasource if it was auto-configured.  
>**NONE** : Don't replace the application default DataSource  
