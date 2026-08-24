# Spring Boot
- Spring Boot is an application development framework mostly used for REST APIs, microservices, enterprise apps
- It removes much of Spring's complexity by providing
  - Autoconfiguration
  - Started dependencies
  - Embedded web servers
  - Production-ready monitoring features
  - Flow diagram
    - Spring Boot => Spring Data JPA => JPA specifications => Hibernate(implementations) => JDBC => Database

```
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

# Hibernate
- Hibernate is an ORM (Object Relational Mapping) framework for database access.
- It works together with sprint boot
- Hibernate automatically
  - Maps objects to tables
  - Generates SQL
  - Handles relationships
  - Support caching and lazy loading

```
@Entity
public class User {
    @Id
    private Long id;
    private String name;
}
```

# JPA
- JPA (Java Persistence API) is a specification/standard, while Hibernate is an implementation of that specification.
- Hibernate is the most common JPA implementation.
- Annotations like @Entity, @Id, @Table come from JPA
- EntityManager is the core JPA API used for persistence operations
```
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    private String name;
}
```
- JPA vs JDBC
  - JDBC
```
Connection conn = DriverManager.getConnection(...);
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users");
```
You write SQL manually.
  - JPA
```
User user = entityManager.find(User.class, 1L);
```
JPA/Hibernate generates SQL for you.

 - What is Spring Data JPA?
   - Spring Data JPA sits on top of JPA and removes even more boilerplate. 
