# Spring Data JPA – Complete Developer Notes

> 🧑‍💻 *A single, comprehensive reference for learning, revision, and real-world backend development.*  
> Covers beginner → advanced. Clean code. Interview-ready.

---

## 📋 Table of Contents

1. [Fundamentals](#1-fundamentals)
2. [Project Setup](#2-project-setup)
3. [Entity Mapping](#3-entity-mapping)
4. [Repository Layer](#4-repository-layer)
5. [Query Mechanisms](#5-query-mechanisms)
6. [Advanced Topics](#6-advanced-topics)
7. [Performance Optimization](#7-performance-optimization)
8. [Auditing](#8-auditing)
9. [Specifications & Dynamic Queries](#9-specifications--dynamic-queries)
10. [Real-World Best Practices](#10-real-world-best-practices)
11. [Quick Revision Cheat Sheet](#11-quick-revision-cheat-sheet)
12. [Interview Questions & Answers](#12-interview-questions--answers)

---

## 1. Fundamentals

### 1.1 What is JPA?

**JPA (Java Persistence API)** is a **specification** (a set of rules/interfaces), not actual code. Think of it like a contract or blueprint that says: *"Here's how Java objects should be saved to a database."*

> 🧠 **Analogy:** JPA is like a building code for construction. It defines the rules, but you still need actual builders (Hibernate) to do the work.

JPA lets you:
- Map Java classes to database tables
- Perform CRUD (Create, Read, Update, Delete) operations using Java objects
- Write queries in **JPQL** (Java Persistence Query Language) instead of raw SQL

**JPA is just an interface. It needs an implementation to work.**

---

### 1.2 What is Hibernate?

**Hibernate** is the most popular **implementation of JPA**. It's actual code that follows the JPA rules and translates your Java operations into SQL queries.

> 🧠 **Analogy:** If JPA is the building code, Hibernate is the construction company that follows those codes and builds the actual house.

Hibernate adds extra features on top of JPA:
- Caching (1st and 2nd level)
- Lazy loading
- HQL (Hibernate Query Language) — similar to JPQL
- Schema generation

---

### 1.3 What is Spring Data JPA?

**Spring Data JPA** is a **Spring abstraction layer on top of JPA/Hibernate** that eliminates boilerplate code.

> 🧠 **Analogy:** If JPA is the rules, Hibernate is the builder, then Spring Data JPA is a smart project manager that handles most of the routine work for you — you just say "I need a house with 3 rooms" and it handles the rest.

Without Spring Data JPA (pure JPA/Hibernate):
```java
// You had to write this every time
EntityManager em = entityManagerFactory.createEntityManager();
em.getTransaction().begin();
em.persist(user);
em.getTransaction().commit();
em.close();
```

With Spring Data JPA:
```java
userRepository.save(user); // That's it!
```

---

### 1.4 Differences & Relationships

```
┌─────────────────────────────────────────────────────┐
│                  Spring Data JPA                     │
│   (Abstraction layer - reduces boilerplate code)     │
├─────────────────────────────────────────────────────┤
│                      JPA                             │
│         (Specification / Interface / Rules)          │
├─────────────────────────────────────────────────────┤
│                   Hibernate                          │
│    (JPA Implementation - does the actual work)       │
├─────────────────────────────────────────────────────┤
│                    JDBC                              │
│             (Raw database connection)                │
├─────────────────────────────────────────────────────┤
│              Database (MySQL/PostgreSQL)              │
└─────────────────────────────────────────────────────┘
```

| Feature | JPA | Hibernate | Spring Data JPA |
|---|---|---|---|
| Type | Specification | Implementation | Abstraction |
| Who defines it? | Jakarta EE | Red Hat | Spring Team |
| Boilerplate | High | Medium | Very Low |
| Extra features | No | Yes (caching, HQL) | Yes (Repositories, Pageable) |
| Can work standalone? | No | Yes | No (needs JPA impl) |

---

### 1.5 Why Use Spring Data JPA?

- ✅ **Zero boilerplate** — No EntityManager, no transactions manually
- ✅ **Auto-generated queries** from method names (`findByEmailAndStatus(...)`)
- ✅ **Pagination and sorting** built-in
- ✅ **Works with any JPA provider** (Hibernate, EclipseLink)
- ✅ **Integrates perfectly** with Spring Boot, Spring Security, etc.
- ✅ **Reduces time to production** significantly

---

## 2. Project Setup

### 2.1 Maven Dependencies

```xml
<!-- pom.xml -->
<dependencies>

    <!-- Spring Data JPA (includes Hibernate as default JPA provider) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- MySQL Driver -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- PostgreSQL Driver (use instead of MySQL if needed) -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- H2 for in-memory testing -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- Lombok to reduce boilerplate -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

</dependencies>
```

### 2.2 Gradle Dependencies

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.mysql:mysql-connector-j'
    // runtimeOnly 'org.postgresql:postgresql'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testRuntimeOnly 'com.h2database:h2'
}
```

---

### 2.3 application.properties — MySQL

```properties
# ─── Database Connection ───────────────────────────────
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ─── JPA / Hibernate Settings ─────────────────────────
# validate: check schema, create: create fresh, update: auto-alter, none: do nothing
spring.jpa.hibernate.ddl-auto=update

# Show SQL in console (useful for debugging, disable in production)
spring.jpa.show-sql=true

# Format the SQL nicely in logs
spring.jpa.properties.hibernate.format_sql=true

# Hibernate dialect (Spring Boot usually auto-detects, but explicit is safer)
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# ─── Connection Pool (HikariCP is the default) ─────────
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
```

### 2.4 application.yml — PostgreSQL

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: yourpassword
    driver-class-name: org.postgresql.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5

  jpa:
    hibernate:
      ddl-auto: update         # Options: create, create-drop, validate, update, none
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect
        # Batch insert optimization
        jdbc:
          batch_size: 50
        order_inserts: true
        order_updates: true
```

### 2.5 DDL Auto Strategies Explained

| Value | What Happens | When to Use |
|---|---|---|
| `create` | Drops and creates schema on startup | Local dev only |
| `create-drop` | Creates on startup, drops on shutdown | Testing |
| `update` | Alters schema to match entities | Dev (careful!) |
| `validate` | Validates schema matches entities, fails if not | Staging/Prod |
| `none` | Does nothing | Production (use Flyway/Liquibase) |

> ⚠️ **Common Mistake:** Never use `create` or `update` in production. Use **Flyway** or **Liquibase** for database migrations in production.

---

## 3. Entity Mapping

### 3.1 Basic Entity — The Building Block

> 🧠 **Analogy:** An **Entity** is like a row in an Excel spreadsheet. The class is the spreadsheet template (table), and each object is one row.

```java
// src/main/java/com/example/entity/User.java

package com.example.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;

@Entity                         // Marks this class as a JPA entity (maps to a DB table)
@Table(
    name = "users",             // Explicit table name (default would be "user" - reserved word!)
    indexes = {
        @Index(name = "idx_email", columnList = "email"),       // DB index for faster lookups
        @Index(name = "idx_status", columnList = "status")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uc_email", columnNames = {"email"})
    }
)
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {

    @Id                                     // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-increment (best for MySQL)
    private Long id;

    @Column(
        name = "first_name",                // Column name in DB
        nullable = false,                   // NOT NULL constraint
        length = 50                         // VARCHAR(50)
    )
    private String firstName;

    @Column(nullable = false, unique = true, length = 100)
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)            // Store enum as string "ACTIVE", not 0/1
    @Column(nullable = false, length = 20)
    private UserStatus status;

    @Column(columnDefinition = "TEXT")      // Use TEXT type for large strings
    private String bio;

    @Column(name = "is_verified")
    private boolean verified = false;

    @Column(name = "created_at", updatable = false) // Can't be updated after insert
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

```java
// Enum for user status
public enum UserStatus {
    ACTIVE, INACTIVE, BANNED, PENDING_VERIFICATION
}
```

---

### 3.2 @GeneratedValue Strategies

```java
// Strategy 1: IDENTITY — DB auto-increment (MySQL, PostgreSQL SERIAL)
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// Strategy 2: SEQUENCE — DB sequence (best for PostgreSQL, Oracle)
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 50)
private Long id;
// allocationSize=50 means Hibernate fetches 50 IDs at once → better performance

// Strategy 3: UUID — Best for distributed systems / microservices
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;

// Strategy 4: TABLE — Uses a separate table to track IDs (slowest, avoid)
@Id
@GeneratedValue(strategy = GenerationType.TABLE)
private Long id;
```

| Strategy | Database Support | Performance | Use When |
|---|---|---|---|
| `IDENTITY` | MySQL, PostgreSQL, SQL Server | Good | Simple apps, MySQL |
| `SEQUENCE` | PostgreSQL, Oracle | Best | PostgreSQL, batch inserts |
| `UUID` | All | Good | Microservices, distributed systems |
| `TABLE` | All | Worst | Avoid in production |

---

### 3.3 Relationships

> 🧠 **Analogy for relationships:**
> - **@OneToOne** → A person has one passport
> - **@OneToMany** → A department has many employees
> - **@ManyToOne** → Many employees belong to one department
> - **@ManyToMany** → Students can enroll in many courses, courses have many students

#### @OneToOne

```java
// User.java — Owner side (has the foreign key)
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    // owns the relationship — the FK lives in this table
    @OneToOne(
        cascade = CascadeType.ALL,      // All operations propagate to UserProfile
        fetch = FetchType.LAZY,          // Load profile only when accessed
        orphanRemoval = true             // Delete profile if user is deleted
    )
    @JoinColumn(name = "profile_id")    // Column in 'users' table pointing to 'user_profiles'
    private UserProfile profile;
}
```

```java
// UserProfile.java — Inverse side (mappedBy means "I don't own this")
@Entity
@Table(name = "user_profiles")
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;
    private String avatarUrl;

    @OneToOne(mappedBy = "profile")     // "profile" = field name in User class
    private User user;
}
```

---

#### @OneToMany and @ManyToOne (Most Common Relationship)

```java
// Department.java — "One" side
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    // One department → many employees
    @OneToMany(
        mappedBy = "department",        // "department" = field name in Employee class
        cascade = CascadeType.ALL,
        fetch = FetchType.LAZY,          // ALWAYS use LAZY for collections!
        orphanRemoval = true
    )
    private List<Employee> employees = new ArrayList<>();

    // Convenience methods to keep both sides in sync
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);
    }
}
```

```java
// Employee.java — "Many" side (owns the relationship — FK is here)
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Many employees → one department
    @ManyToOne(fetch = FetchType.LAZY)  // LAZY is better here too
    @JoinColumn(
        name = "department_id",         // FK column in 'employees' table
        nullable = false
    )
    private Department department;
}
```

---

#### @ManyToMany

```java
// Student.java
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(
        cascade = {CascadeType.PERSIST, CascadeType.MERGE}, // Don't cascade REMOVE!
        fetch = FetchType.LAZY
    )
    @JoinTable(
        name = "student_courses",           // Join table name
        joinColumns = @JoinColumn(name = "student_id"),     // FK to Student
        inverseJoinColumns = @JoinColumn(name = "course_id") // FK to Course
    )
    private Set<Course> courses = new HashSet<>();   // Use Set, not List (avoids duplicates)

    public void enrollIn(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}
```

```java
// Course.java
@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)
    private Set<Student> students = new HashSet<>();
}
```

> 📌 **Key Point:** For `@ManyToMany`, use a **join table** with `@JoinTable`. The owner side defines it; the inverse side uses `mappedBy`.

---

### 3.4 Fetch Types

> 🧠 **Analogy:** 
> - **EAGER** = Ordering food and automatically getting dessert with it (even if you don't want it)
> - **LAZY** = Ordering food, and only ordering dessert when you actually want it

```java
// EAGER: Loads related data IMMEDIATELY with the parent query (one big SQL JOIN)
@ManyToOne(fetch = FetchType.EAGER)
private Department department; // Department is loaded whenever Employee is loaded

// LAZY: Loads related data only when you ACCESS the field (separate SQL query on demand)
@OneToMany(fetch = FetchType.LAZY)
private List<Employee> employees; // Employees loaded only when you call getEmployees()
```

**Default Fetch Types:**

| Annotation | Default Fetch Type |
|---|---|
| `@ManyToOne` | `EAGER` ⚠️ |
| `@OneToOne` | `EAGER` ⚠️ |
| `@OneToMany` | `LAZY` ✅ |
| `@ManyToMany` | `LAZY` ✅ |

> ⚠️ **Common Mistake:** `@ManyToOne` defaults to EAGER — always override it to LAZY! EAGER loading on many relationships causes massive performance issues.

> ✅ **Best Practice:** **Always use `FetchType.LAZY`** and load eagerly only when needed using `JOIN FETCH` in queries.

---

### 3.5 Cascade Types

> 🧠 **Analogy:** Cascading is like "when you delete a parent record, what happens to the children?"

| CascadeType | What it does |
|---|---|
| `PERSIST` | Saving parent also saves children |
| `MERGE` | Updating parent also updates children |
| `REMOVE` | Deleting parent also deletes children |
| `REFRESH` | Refreshing parent also refreshes children |
| `DETACH` | Detaching parent also detaches children |
| `ALL` | All of the above |

```java
// Example: When you save/delete an Order, its OrderItems are also saved/deleted
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();

// For ManyToMany: NEVER use CascadeType.REMOVE or ALL
// Deleting a Student should NOT delete all Courses they enrolled in!
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private Set<Course> courses = new HashSet<>();
```

> ⚠️ **Common Mistake:** Using `CascadeType.ALL` on `@ManyToMany` — this would delete shared entities when you remove one side!

---

## 4. Repository Layer

### 4.1 Repository Hierarchy

```
Repository (marker interface)
    └── CrudRepository<T, ID>       → Basic CRUD
            └── PagingAndSortingRepository<T, ID>  → + Pagination & Sorting
                    └── JpaRepository<T, ID>       → + flush(), batch ops, JPA-specific
```

### 4.2 Creating a Repository

```java
// src/main/java/com/example/repository/UserRepository.java

package com.example.repository;

import com.example.entity.User;
import com.example.entity.UserStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;
import java.util.Optional;

@Repository // Optional — Spring auto-detects it, but good for clarity
public interface UserRepository extends JpaRepository<User, Long> {
    // JpaRepository<EntityType, PrimaryKeyType>

    // Spring Data JPA auto-generates the implementation!
    // You just declare the method signatures.

    // Built-in methods you get for FREE:
    // save(S entity)
    // findById(Long id) → Optional<User>
    // findAll() → List<User>
    // findAll(Pageable pageable) → Page<User>
    // deleteById(Long id)
    // count()
    // existsById(Long id)
    // saveAll(Iterable<User> users)
    // deleteAll()
}
```

---

### 4.3 Query Methods by Naming Convention

Spring Data JPA **reads your method name** and generates the SQL automatically!

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ─── Find by single field ──────────────────────────────
    Optional<User> findByEmail(String email);
    // SQL: SELECT * FROM users WHERE email = ?

    List<User> findByStatus(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ?

    // ─── Find by multiple fields ───────────────────────────
    Optional<User> findByEmailAndStatus(String email, UserStatus status);
    // SQL: SELECT * FROM users WHERE email = ? AND status = ?

    List<User> findByFirstNameOrLastName(String firstName, String lastName);
    // SQL: SELECT * FROM users WHERE first_name = ? OR last_name = ?

    // ─── Comparison operators ──────────────────────────────
    List<User> findByAgeLessThan(int age);              // WHERE age < ?
    List<User> findByAgeGreaterThanEqual(int age);      // WHERE age >= ?
    List<User> findByAgeBetween(int min, int max);      // WHERE age BETWEEN ? AND ?
    List<User> findByCreatedAtAfter(LocalDateTime date); // WHERE created_at > ?

    // ─── String operations ─────────────────────────────────
    List<User> findByFirstNameContaining(String keyword);     // WHERE first_name LIKE %keyword%
    List<User> findByEmailStartingWith(String prefix);        // WHERE email LIKE prefix%
    List<User> findByLastNameEndingWith(String suffix);       // WHERE last_name LIKE %suffix
    List<User> findByFirstNameIgnoreCase(String name);        // WHERE LOWER(first_name) = LOWER(?)

    // ─── Null checks ───────────────────────────────────────
    List<User> findByBioIsNull();                       // WHERE bio IS NULL
    List<User> findByBioIsNotNull();                    // WHERE bio IS NOT NULL

    // ─── Ordering ──────────────────────────────────────────
    List<User> findByStatusOrderByCreatedAtDesc(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ? ORDER BY created_at DESC

    // ─── Limiting ──────────────────────────────────────────
    User findFirstByOrderByCreatedAtDesc();             // Most recent user
    List<User> findTop5ByStatusOrderByCreatedAtDesc(UserStatus status); // Latest 5 active

    // ─── Count and Existence ───────────────────────────────
    long countByStatus(UserStatus status);
    boolean existsByEmail(String email);

    // ─── Delete ────────────────────────────────────────────
    void deleteByStatus(UserStatus status);
    long deleteByEmail(String email); // Returns count of deleted rows

    // ─── Nested property access ────────────────────────────
    // If User has a Department, you can do:
    List<User> findByDepartmentName(String departmentName);
    // SQL: JOIN department WHERE department.name = ?
}
```

**Keyword Reference Table:**

| Keyword | SQL Equivalent | Example |
|---|---|---|
| `And` | `AND` | `findByFirstNameAndLastName` |
| `Or` | `OR` | `findByStatusOrEmail` |
| `Is`, `Equals` | `=` | `findByStatusIs` |
| `Between` | `BETWEEN` | `findByAgeBetween` |
| `LessThan` | `<` | `findByAgeLessThan` |
| `GreaterThan` | `>` | `findByAgeGreaterThan` |
| `After` | `>` (dates) | `findByCreatedAtAfter` |
| `Before` | `<` (dates) | `findByCreatedAtBefore` |
| `IsNull` | `IS NULL` | `findByEmailIsNull` |
| `IsNotNull` | `IS NOT NULL` | `findByEmailIsNotNull` |
| `Like` | `LIKE` | `findByNameLike` |
| `Containing` | `LIKE %x%` | `findByNameContaining` |
| `StartingWith` | `LIKE x%` | `findByNameStartingWith` |
| `EndingWith` | `LIKE %x` | `findByNameEndingWith` |
| `Not` | `!=` | `findByStatusNot` |
| `In` | `IN (...)` | `findByStatusIn` |
| `NotIn` | `NOT IN` | `findByStatusNotIn` |
| `True`/`False` | `= true/false` | `findByVerifiedTrue` |
| `IgnoreCase` | `LOWER()` | `findByEmailIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByStatusOrderByCreatedAtDesc` |
| `Top`/`First` | `LIMIT` | `findTop10ByStatus` |

---

### 4.4 Pagination & Sorting

> 🧠 **Analogy:** Pagination is like reading a book — you don't read all pages at once, you go page by page.

```java
// Repository — add Pageable parameter
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(UserStatus status, Pageable pageable);
    Slice<User> findByDepartmentId(Long deptId, Pageable pageable); // Slice = lighter (no count query)
}
```

```java
// Service — using Pageable
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public Page<User> getActiveUsers(int page, int size) {
        // page = 0-indexed page number, size = records per page
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }

    public Page<User> getActiveUsersSorted(int page, int size) {
        // Sort by createdAt descending, then by name ascending
        Sort sort = Sort.by(Sort.Direction.DESC, "createdAt")
                        .and(Sort.by(Sort.Direction.ASC, "firstName"));

        Pageable pageable = PageRequest.of(page, size, sort);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }
}
```

```java
// Controller — exposing pagination as REST API
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    @GetMapping
    public ResponseEntity<Page<UserDto>> getUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "10") int size,
            @RequestParam(defaultValue = "createdAt") String sortBy,
            @RequestParam(defaultValue = "desc") String direction) {

        Sort sort = direction.equalsIgnoreCase("desc")
                ? Sort.by(sortBy).descending()
                : Sort.by(sortBy).ascending();

        Pageable pageable = PageRequest.of(page, size, sort);
        Page<User> users = userRepository.findAll(pageable);

        // Map to DTO
        Page<UserDto> dtoPage = users.map(UserMapper::toDto);
        return ResponseEntity.ok(dtoPage);
    }
}
```

**Page Object — What you get:**
```json
{
  "content": [...],          // The actual data
  "totalElements": 150,       // Total records in DB
  "totalPages": 15,           // Total pages
  "number": 0,                // Current page (0-indexed)
  "size": 10,                 // Page size
  "first": true,              // Is this the first page?
  "last": false,              // Is this the last page?
  "numberOfElements": 10      // Elements in current page
}
```

---

## 5. Query Mechanisms

### 5.1 JPQL — Java Persistence Query Language

> 🧠 **Analogy:** JPQL is like SQL, but you write it using **class names and field names** (Java), not **table names and column names** (Database).

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // @Query uses JPQL by default
    // "User" = Java class name (not "users" table!)
    // "u.email" = Java field name (not "email" column directly)
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.status = :status")
    Optional<User> findActiveByEmail(
            @Param("email") String email,
            @Param("status") UserStatus status
    );

    // JOIN — using JPQL join on relationship
    @Query("SELECT u FROM User u JOIN u.department d WHERE d.name = :deptName")
    List<User> findByDepartmentName(@Param("deptName") String deptName);

    // JOIN FETCH — solves N+1 problem (loads relationship in one query)
    @Query("SELECT u FROM User u JOIN FETCH u.department WHERE u.status = :status")
    List<User> findWithDepartmentByStatus(@Param("status") UserStatus status);

    // WHERE clause with LIKE
    @Query("SELECT u FROM User u WHERE LOWER(u.firstName) LIKE LOWER(CONCAT('%', :name, '%'))")
    List<User> searchByName(@Param("name") String name);

    // Aggregate functions
    @Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
    long countByStatusJpql(@Param("status") UserStatus status);

    // Custom DTO/projection via constructor expression
    @Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.email) FROM User u WHERE u.status = 'ACTIVE'")
    List<UserSummaryDto> findUserSummaries();

    // Modifying query (UPDATE / DELETE)
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
    int updateStatus(@Param("oldStatus") UserStatus oldStatus, @Param("newStatus") UserStatus newStatus);

    @Modifying
    @Transactional
    @Query("DELETE FROM User u WHERE u.status = :status AND u.createdAt < :cutoff")
    int deleteInactiveUsers(@Param("status") UserStatus status, @Param("cutoff") LocalDateTime cutoff);
}
```

---

### 5.2 Native SQL Queries

> Use native queries when JPQL can't express something, or when you need DB-specific features (window functions, CTEs, etc.)

```java
@Query(
    value = "SELECT * FROM users WHERE email = :email",   // Raw SQL
    nativeQuery = true                                     // Tells Spring: use raw SQL
)
Optional<User> findByEmailNative(@Param("email") String email);

// Native query with pagination
@Query(
    value = "SELECT u.*, d.name as dept_name FROM users u JOIN departments d ON u.department_id = d.id WHERE u.status = :status",
    countQuery = "SELECT COUNT(*) FROM users WHERE status = :status", // Required for pagination
    nativeQuery = true
)
Page<Object[]> findUsersWithDept(@Param("status") String status, Pageable pageable);

// Complex native query using DB-specific features
@Query(
    value = """
        WITH ranked_users AS (
            SELECT u.*, ROW_NUMBER() OVER (PARTITION BY u.department_id ORDER BY u.created_at DESC) as rn
            FROM users u
            WHERE u.status = 'ACTIVE'
        )
        SELECT * FROM ranked_users WHERE rn = 1
        """,
    nativeQuery = true
)
List<Object[]> findLatestUserPerDepartment();
```

---

### 5.3 Named Queries

```java
// Define on the entity class
@Entity
@Table(name = "users")
@NamedQuery(
    name = "User.findByEmailAndStatus",
    query = "SELECT u FROM User u WHERE u.email = :email AND u.status = :status"
)
@NamedQueries({
    @NamedQuery(name = "User.findActive", query = "SELECT u FROM User u WHERE u.status = 'ACTIVE'"),
    @NamedQuery(name = "User.countByDept", query = "SELECT COUNT(u) FROM User u WHERE u.department.id = :deptId")
})
public class User { ... }
```

```java
// Repository — method name must match named query name (after the entity name)
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring auto-maps this to "User.findByEmailAndStatus"
    Optional<User> findByEmailAndStatus(String email, UserStatus status);
}
```

> 📌 **Key Point:** Named queries are validated at application startup — typos fail fast! But they're rarely used now since `@Query` is cleaner and co-located with the repository.

---

### 5.4 Projections — Getting Only What You Need

> 🧠 **Analogy:** Instead of fetching a whole user object (30 fields), fetch only the name and email you actually need. Less data = faster.

#### Interface-Based Projection (Lightweight, Recommended)

```java
// Define projection interface — Spring creates a proxy at runtime
public interface UserSummary {
    Long getId();
    String getFirstName();
    String getEmail();

    // SpEL expression — computed field
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();
}
```

```java
// Repository — return the projection type
public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByStatus(UserStatus status); // Returns only id, firstName, email
    Optional<UserSummary> findProjectedById(Long id);
}
```

#### DTO-Based (Class) Projection — More Type-Safe

```java
// DTO record (Java 16+)
public record UserSummaryDto(Long id, String firstName, String email) {}

// OR traditional DTO
@Value // Lombok immutable DTO
public class UserSummaryDto {
    Long id;
    String firstName;
    String email;
}
```

```java
// Repository with JPQL constructor expression
@Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.email) " +
       "FROM User u WHERE u.status = :status")
List<UserSummaryDto> findSummariesByStatus(@Param("status") UserStatus status);
```

#### Dynamic Projections — Choose at Runtime

```java
// Generic method — caller decides projection type
public interface UserRepository extends JpaRepository<User, Long> {
    <T> List<T> findByStatus(UserStatus status, Class<T> type);
}

// Usage:
List<UserSummary> summaries = repo.findByStatus(ACTIVE, UserSummary.class);
List<User>        full      = repo.findByStatus(ACTIVE, User.class);
```

---

## 6. Advanced Topics

### 6.1 The N+1 Problem (⚠️ CRITICAL — Most Common Performance Issue)

> 🧠 **Analogy:** Imagine you have 100 orders and want to print each order with its customer name. 
> - **N+1 way (bad):** Go to the filing cabinet once to get the 100 orders. Then make 100 separate trips to get each customer file. Total: **101 trips**.  
> - **Join fetch way (good):** One trip to get all orders AND customer info together. Total: **1 trip**.

```java
// ❌ BAD — N+1 Problem
// 1 query: SELECT * FROM employees
List<Employee> employees = employeeRepository.findAll();

// N queries: SELECT * FROM departments WHERE id = ? (once per employee!)
employees.forEach(e -> System.out.println(e.getDepartment().getName()));

// If you have 1000 employees = 1001 SQL queries!
```

```java
// ✅ SOLUTION 1: JOIN FETCH in JPQL
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartment();
// 1 query: SELECT e.*, d.* FROM employees e JOIN departments d ON e.department_id = d.id
```

```java
// ✅ SOLUTION 2: @EntityGraph — declarative approach
@EntityGraph(attributePaths = {"department", "address"}) // Load these eagerly
List<Employee> findByStatus(EmployeeStatus status);

// Or define a named entity graph on the entity:
@Entity
@NamedEntityGraph(
    name = "Employee.withDepartment",
    attributeNodes = @NamedAttributeNode("department")
)
public class Employee { ... }

// Use it in repository:
@EntityGraph("Employee.withDepartment")
List<Employee> findAll();
```

```java
// ✅ SOLUTION 3: Hibernate @BatchSize (good for collections)
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@BatchSize(size = 30) // Load 30 departments' employees at a time
private List<Employee> employees;
// Instead of N queries, loads in batches: N/30 queries
```

```java
// ✅ SOLUTION 4: Fetch subselect (Hibernate-specific)
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT) // Loads all children in one IN subselect
private List<Employee> employees;
```

> 📌 **Key Point:** Always check your query count with `show-sql=true` or tools like **p6spy** or **datasource-proxy** in tests.

---

### 6.2 Entity Lifecycle

Every JPA entity goes through these states:

```
  New/Transient → [persist()] → Managed → [remove()] → Removed
                                    ↑                      |
                                [merge()]                   ↓
                   Detached ←─────────────── [detach()/clear()/close()]
```

| State | Description | Example |
|---|---|---|
| **Transient** | Object created, not known to JPA | `new User()` |
| **Managed** | JPA is tracking it — changes auto-synced | After `save()` or `findById()` |
| **Detached** | Was managed, now outside JPA context | After transaction ends |
| **Removed** | Scheduled for deletion | After `delete()` |

```java
// Transient — not tracked
User user = new User(); // JPA doesn't know about this

// Managed — JPA is watching this!
User savedUser = userRepository.save(user); // Now it's managed

// Detached — after transaction ends, the object is detached
// Accessing lazy-loaded fields on detached entity = LazyInitializationException!

// Re-attach a detached entity
User merged = userRepository.save(detachedUser); // merge = re-attach
```

---

### 6.3 Dirty Checking (Auto-Flush)

> 🧠 **Analogy:** Dirty checking is like a photographer who takes a snapshot of your room (entity state) when you enter. When you leave (end transaction), they compare the current state to the snapshot. If anything changed, they automatically update the record.

```java
@Transactional
public void updateUserEmail(Long userId, String newEmail) {
    User user = userRepository.findById(userId).orElseThrow();
    // user is now MANAGED — JPA is watching it

    user.setEmail(newEmail); // Just change the field!
    // NO need to call save() — Hibernate auto-detects the change
    // and issues UPDATE SQL when the transaction commits!
}
```

```java
// ⚠️ Dirty checking only works within a @Transactional method
// Outside a transaction, the entity is detached — changes won't be saved
public void badExample(Long userId) {
    User user = userRepository.findById(userId).orElseThrow();
    // Transaction ends when findById returns (no @Transactional here)
    user.setEmail("new@email.com"); // This change is LOST!
}
```

> ✅ **Best Practice:** Always use `@Transactional` on service methods that modify entities.

---

### 6.4 Transactions (@Transactional)

> 🧠 **Analogy:** A transaction is like a bank transfer — either all steps succeed (debit + credit), or none do. No partial success.

```java
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;

@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    // Basic transaction — rolls back on ANY RuntimeException
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Order order = new Order();
        // ... setup order ...

        inventoryService.reserve(request.getProductId(), request.getQuantity());
        paymentService.charge(request.getPaymentDetails());

        return orderRepository.save(order);
        // If any step throws RuntimeException → everything rolls back!
    }

    // Custom rollback rules
    @Transactional(rollbackFor = Exception.class)          // Rollback on checked exceptions too
    public void sensitiveOperation() { ... }

    @Transactional(noRollbackFor = EmailException.class)   // Don't rollback for this
    public void sendWithEmail() { ... }

    // Read-only transaction — performance optimization for reads
    @Transactional(readOnly = true)
    public List<Order> getAllOrders() {
        return orderRepository.findAll();
        // Hibernate skips dirty checking → faster
    }

    // Timeout — rollback if takes too long
    @Transactional(timeout = 30) // 30 seconds
    public void longRunningOperation() { ... }
}
```

**Propagation Types:**

```java
// REQUIRED (default): Join existing transaction, or create new one
@Transactional(propagation = Propagation.REQUIRED)

// REQUIRES_NEW: Always create a new transaction (suspends existing)
@Transactional(propagation = Propagation.REQUIRES_NEW)
// Use case: Audit logging — must be saved even if main tx rolls back

// NESTED: Execute in a savepoint within the existing transaction
@Transactional(propagation = Propagation.NESTED)

// SUPPORTS: Use transaction if exists, otherwise non-transactional
@Transactional(propagation = Propagation.SUPPORTS)

// NOT_SUPPORTED: Suspend any existing transaction, run non-transactional
@Transactional(propagation = Propagation.NOT_SUPPORTED)

// NEVER: Must not have a transaction; throw exception if one exists
@Transactional(propagation = Propagation.NEVER)

// MANDATORY: Must have an existing transaction; throw exception if none
@Transactional(propagation = Propagation.MANDATORY)
```

**Isolation Levels:**

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use When |
|---|---|---|---|---|
| `READ_UNCOMMITTED` | ✅ possible | ✅ possible | ✅ possible | Never in production |
| `READ_COMMITTED` | ❌ prevented | ✅ possible | ✅ possible | Default in most DBs |
| `REPEATABLE_READ` | ❌ prevented | ❌ prevented | ✅ possible | Default in MySQL |
| `SERIALIZABLE` | ❌ prevented | ❌ prevented | ❌ prevented | High-consistency use cases |

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void transferMoney(Long fromId, Long toId, BigDecimal amount) { ... }
```

> ⚠️ **Common Mistake:** `@Transactional` on a `private` method — it won't work! Spring uses a proxy, so only `public` methods work. Also, calling a `@Transactional` method from within the same class bypasses the proxy — use self-injection or restructure.

---

### 6.5 Optimistic vs Pessimistic Locking

> 🧠 **Analogy:**
> - **Optimistic Locking:** "I'll assume no one else edited this, and I'll check when I save. If they did, I'll retry."
> - **Pessimistic Locking:** "I'm going to lock this row while I work on it — nobody else touches it."

#### Optimistic Locking

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int stock;

    @Version  // Magic annotation! Hibernate manages this version number
    private int version;
    // On every UPDATE, Hibernate checks: WHERE id=? AND version=?
    // If version changed (someone else updated) → OptimisticLockException!
}
```

```java
// When two users try to update the same product simultaneously:
// User A reads product (version=1), User B reads product (version=1)
// User A saves → version becomes 2
// User B tries to save → WHERE id=? AND version=1 → 0 rows updated → EXCEPTION!

@Transactional
public Product updateStock(Long productId, int delta) {
    try {
        Product product = productRepository.findById(productId).orElseThrow();
        product.setStock(product.getStock() + delta);
        return productRepository.save(product);
    } catch (OptimisticLockingFailureException ex) {
        // Retry logic or return conflict response
        throw new ConflictException("Product was modified by another user. Please retry.");
    }
}
```

#### Pessimistic Locking

```java
// Repository method with explicit lock
public interface ProductRepository extends JpaRepository<Product, Long> {

    // Lock the row for update (other transactions wait)
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithLock(@Param("id") Long id);

    // PESSIMISTIC_READ — others can read, but not write
    @Lock(LockModeType.PESSIMISTIC_READ)
    Optional<Product> findById(Long id);
}
```

| Locking Type | Best For | Risk |
|---|---|---|
| Optimistic | Low contention, many reads | Retry overhead on conflict |
| Pessimistic Write | High contention, critical sections | Deadlocks, reduced throughput |
| Pessimistic Read | Shared reads, prevent writes | Lower throughput |

---

### 6.6 Caching

#### First-Level Cache (Always On — Session Cache)

```java
// First-level cache is per-session (per-EntityManager / per-transaction)
@Transactional
public void demo() {
    User u1 = userRepository.findById(1L).get(); // SQL query executed
    User u2 = userRepository.findById(1L).get(); // NO SQL! Returns same object from cache
    System.out.println(u1 == u2); // true — same instance!
}
// Cache is cleared when transaction ends
```

#### Second-Level Cache (Cross-Session — Needs Config)

```java
// 1. Add dependency (e.g., Caffeine or Ehcache)
// <dependency>
//   <groupId>org.hibernate.orm</groupId>
//   <artifactId>hibernate-jcache</artifactId>
// </dependency>
// <dependency>
//   <groupId>com.github.ben-manes.caffeine</groupId>
//   <artifactId>caffeine</artifactId>
// </dependency>

// 2. Enable in application.properties
// spring.jpa.properties.hibernate.cache.use_second_level_cache=true
// spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.internal.JCacheRegionFactory
// spring.jpa.properties.javax.cache.provider=com.github.benmanes.caffeine.jcache.spi.CaffeineCachingProvider

// 3. Annotate entity
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // Hibernate cache strategy
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

```java
// 4. Query cache (cache query results)
@Query("SELECT c FROM Category c WHERE c.active = true")
@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
List<Category> findActiveCategories();
```

| Cache Strategy | Use When |
|---|---|
| `READ_ONLY` | Data never changes (reference data) |
| `NONSTRICT_READ_WRITE` | Rarely changes, eventual consistency OK |
| `READ_WRITE` | Moderate updates (most common) |
| `TRANSACTIONAL` | Full transactional consistency needed |

---

## 7. Performance Optimization

### 7.1 Fetch Joins

```java
// ❌ Without fetch join — causes N+1 for department
@Query("SELECT e FROM Employee e WHERE e.status = :status")
List<Employee> findByStatus(@Param("status") String status);

// ✅ With fetch join — loads department in same query
@Query("SELECT e FROM Employee e JOIN FETCH e.department WHERE e.status = :status")
List<Employee> findByStatusWithDept(@Param("status") String status);

// ✅ Multiple joins
@Query("""
    SELECT DISTINCT o FROM Order o
    JOIN FETCH o.customer
    JOIN FETCH o.items i
    JOIN FETCH i.product
    WHERE o.status = :status
    """)
List<Order> findOrdersWithDetails(@Param("status") String status);
```

> ⚠️ **Common Mistake:** Using `JOIN FETCH` with `Pageable` — Hibernate fetches all rows in memory then paginates (HHH90003004 warning!). Use `@EntityGraph` with `countQuery` or separate queries.

```java
// ✅ Correct approach: use two queries
@Query(
    value = "SELECT o FROM Order o JOIN FETCH o.customer WHERE o.status = :status",
    countQuery = "SELECT COUNT(o) FROM Order o WHERE o.status = :status"
)
Page<Order> findWithCustomer(@Param("status") String status, Pageable pageable);
```

---

### 7.2 Batch Operations

```yaml
# application.yml — enable batching
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50        # Group 50 inserts into one batch
        order_inserts: true     # Group same entity inserts together
        order_updates: true     # Group same entity updates together
```

```java
@Service
@RequiredArgsConstructor
public class BulkImportService {

    private final UserRepository userRepository;
    private final EntityManager entityManager;

    @Transactional
    public void bulkInsert(List<UserCreateDto> dtos) {
        int batchSize = 50;

        for (int i = 0; i < dtos.size(); i++) {
            User user = mapToUser(dtos.get(i));
            entityManager.persist(user);

            if (i % batchSize == 0 && i > 0) {
                entityManager.flush();  // Send batch to DB
                entityManager.clear();  // Clear 1st-level cache (prevent OOM)
            }
        }
    }

    // Alternative: saveAll with batch size configured
    @Transactional
    public void bulkSave(List<User> users) {
        userRepository.saveAll(users); // Uses batch_size from config
    }
}
```

> ⚠️ **Note:** For batch inserts to work with `SEQUENCE` strategy, use `allocationSize` >= `batch_size`. `IDENTITY` strategy **disables batching** — use `SEQUENCE` for bulk operations!

---

### 7.3 Read Performance — Projections vs Entities

```java
// ❌ Slow — loads all 30 columns even if you need 3
List<User> users = userRepository.findByStatus(ACTIVE);
users.stream().map(u -> u.getEmail()).collect(toList());

// ✅ Fast — loads only the columns you need
List<String> emails = userRepository.findEmailsByStatus(ACTIVE);

// Repository method returning just a field
@Query("SELECT u.email FROM User u WHERE u.status = :status")
List<String> findEmailsByStatus(@Param("status") UserStatus status);

// Or use projection interface
interface EmailOnly { String getEmail(); }
List<EmailOnly> findByStatus(UserStatus status, Class<EmailOnly> type);
```

---

### 7.4 Indexing Strategy

```java
// 1. Index frequently queried columns
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email"),           // Equality lookup
    @Index(name = "idx_status_created", columnList = "status, created_at DESC"), // Composite
})

// 2. Use @Column to signal importance
@Column(unique = true, nullable = false, length = 100) // Adds unique index
private String email;

// 3. Covering indexes — include all SELECT columns in the index
// Do this via migration script (Flyway/Liquibase):
// CREATE INDEX idx_cover ON users(status) INCLUDE (id, email, first_name);
```

---

### 7.5 Performance Best Practices Summary

```
✅ Always use FetchType.LAZY on relationships
✅ Use JOIN FETCH or @EntityGraph only when you need the data
✅ Use projections (interface or DTO) for read-only endpoints
✅ Enable batch inserts for bulk operations
✅ Use @Transactional(readOnly = true) for queries
✅ Index frequently queried and JOIN columns
✅ Use SEQUENCE strategy for batch inserts (not IDENTITY)
✅ Monitor with datasource-proxy or p6spy in tests
✅ Use connection pooling (HikariCP — default in Spring Boot)
✅ Limit result sets with pagination
```

---

## 8. Auditing

### 8.1 Auto-populate Created/Updated Timestamps

```java
// Step 1: Enable auditing in a config class
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    // Optional: provide current auditor (for @CreatedBy/@LastModifiedBy)
    @Bean
    public AuditorAware<String> auditorAware() {
        return () -> Optional.ofNullable(SecurityContextHolder.getContext())
                .map(SecurityContext::getAuthentication)
                .filter(Authentication::isAuthenticated)
                .map(Authentication::getName);
    }
}
```

```java
// Step 2: Create a base auditable entity (reusable across all entities)
@MappedSuperclass  // Not an entity itself — just shares fields with subclasses
@EntityListeners(AuditingEntityListener.class) // Hooks into JPA lifecycle events
@Getter
public abstract class Auditable {

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(name = "created_by", updatable = false)
    private String createdBy;

    @LastModifiedBy
    @Column(name = "updated_by")
    private String updatedBy;
}
```

```java
// Step 3: Extend it in your entities
@Entity
@Table(name = "products")
@Getter
@Setter
public class Product extends Auditable {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private BigDecimal price;

    // createdAt, updatedAt, createdBy, updatedBy — inherited from Auditable!
}
```

### 8.2 JPA Entity Lifecycle Callbacks (Alternative)

```java
@Entity
public class Order {

    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist  // Called before INSERT
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate   // Called before UPDATE
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    @PostPersist  // Called after INSERT
    protected void afterCreate() {
        System.out.println("Order created: " + id);
    }

    @PostRemove   // Called after DELETE
    protected void afterDelete() {
        System.out.println("Order deleted: " + id);
    }
}
```

**Lifecycle Callback Annotations:**

| Annotation | When Called |
|---|---|
| `@PrePersist` | Before INSERT |
| `@PostPersist` | After INSERT |
| `@PreUpdate` | Before UPDATE |
| `@PostUpdate` | After UPDATE |
| `@PreRemove` | Before DELETE |
| `@PostRemove` | After DELETE |
| `@PostLoad` | After SELECT (entity loaded) |

---

## 9. Specifications & Dynamic Queries

### 9.1 The Problem — Dynamic Filtering

```java
// ❌ Bad approach — combinatorial explosion
public List<User> findUsers(String name, String email, UserStatus status) {
    if (name != null && email != null && status != null) return repo.findByFirstNameAndEmailAndStatus(...);
    if (name != null && email != null) return repo.findByFirstNameAndEmail(...);
    if (name != null && status != null) return repo.findByFirstNameAndStatus(...);
    // ... 8 combinations for 3 filters, exponential for more!
}
```

### 9.2 Specification Pattern

```java
// Step 1: Extend JpaSpecificationExecutor
public interface UserRepository extends JpaRepository<User, Long>,
        JpaSpecificationExecutor<User> {
    // JpaSpecificationExecutor adds:
    // findAll(Specification<T> spec)
    // findAll(Specification<T> spec, Pageable pageable)
    // findOne(Specification<T> spec)
    // count(Specification<T> spec)
}
```

```java
// Step 2: Build specifications — each is a reusable query predicate
public class UserSpecifications {

    public static Specification<User> hasFirstName(String firstName) {
        return (root, query, cb) -> {
            if (firstName == null) return null; // No filter applied
            return cb.like(cb.lower(root.get("firstName")), "%" + firstName.toLowerCase() + "%");
        };
    }

    public static Specification<User> hasStatus(UserStatus status) {
        return (root, query, cb) ->
                status == null ? null : cb.equal(root.get("status"), status);
    }

    public static Specification<User> hasEmail(String email) {
        return (root, query, cb) ->
                email == null ? null : cb.equal(root.get("email"), email);
    }

    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) ->
                date == null ? null : cb.greaterThan(root.get("createdAt"), date);
    }

    public static Specification<User> inDepartment(Long deptId) {
        return (root, query, cb) -> {
            if (deptId == null) return null;
            Join<User, Department> dept = root.join("department", JoinType.INNER);
            return cb.equal(dept.get("id"), deptId);
        };
    }
}
```

```java
// Step 3: Compose specifications in service
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public Page<User> searchUsers(UserSearchRequest request, Pageable pageable) {
        Specification<User> spec = Specification.where(null); // Start with "no filter"

        spec = spec.and(UserSpecifications.hasFirstName(request.getFirstName()));
        spec = spec.and(UserSpecifications.hasStatus(request.getStatus()));
        spec = spec.and(UserSpecifications.hasEmail(request.getEmail()));
        spec = spec.and(UserSpecifications.createdAfter(request.getCreatedAfter()));
        spec = spec.and(UserSpecifications.inDepartment(request.getDepartmentId()));

        return userRepository.findAll(spec, pageable);
        // Only non-null predicates are applied — clean and flexible!
    }
}
```

```java
// Request DTO
@Data
public class UserSearchRequest {
    private String firstName;
    private String email;
    private UserStatus status;
    private LocalDateTime createdAfter;
    private Long departmentId;
}
```

```java
// Controller
@GetMapping("/search")
public ResponseEntity<Page<UserDto>> searchUsers(
        @ModelAttribute UserSearchRequest request,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("createdAt").descending());
    Page<User> users = userService.searchUsers(request, pageable);
    return ResponseEntity.ok(users.map(UserMapper::toDto));
}
```

### 9.3 Criteria API (Lower Level — Behind Specifications)

```java
@Repository
@RequiredArgsConstructor
public class UserCriteriaRepository {

    private final EntityManager em;

    public List<User> findWithCriteria(String name, UserStatus status) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> user = query.from(User.class);

        List<Predicate> predicates = new ArrayList<>();

        if (name != null) {
            predicates.add(cb.like(cb.lower(user.get("firstName")), "%" + name.toLowerCase() + "%"));
        }
        if (status != null) {
            predicates.add(cb.equal(user.get("status"), status));
        }

        query.where(predicates.toArray(new Predicate[0]));
        query.orderBy(cb.desc(user.get("createdAt")));

        return em.createQuery(query).getResultList();
    }
}
```

> 📌 **Key Point:** The `Specification` pattern is just a cleaner wrapper around the Criteria API. Prefer Specifications for their composability and readability.

---

## 10. Real-World Best Practices

### 10.1 Layered Architecture

```
┌─────────────────────────────────────────┐
│            Controller Layer              │  ← HTTP, request/response, validation
│         (@RestController)               │
├─────────────────────────────────────────┤
│             Service Layer               │  ← Business logic, transactions
│            (@Service)                   │
├─────────────────────────────────────────┤
│           Repository Layer              │  ← Data access, queries
│  (@Repository / JpaRepository)          │
├─────────────────────────────────────────┤
│             Entity Layer                │  ← DB table mapping
│              (@Entity)                  │
└─────────────────────────────────────────┘
```

### 10.2 DTO vs Entity Separation

> ⚠️ **Never expose entities directly in API responses!**

```java
// ❌ WRONG — Exposing entity directly
@GetMapping("/{id}")
public User getUser(@PathVariable Long id) {
    return userRepository.findById(id).orElseThrow();
    // Problems: Exposes passwordHash, lazy-load exceptions, tight coupling to DB schema
}

// ✅ CORRECT — Use a DTO
@GetMapping("/{id}")
public ResponseEntity<UserResponseDto> getUser(@PathVariable Long id) {
    User user = userService.findById(id);
    return ResponseEntity.ok(UserMapper.toDto(user));
}
```

```java
// User Response DTO
@Data
@Builder
public class UserResponseDto {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    private String status;
    private LocalDateTime createdAt;
    // Note: NO passwordHash, NO internal fields!
}

// User Create/Update DTO
@Data
public class UserCreateDto {
    @NotBlank
    @Size(min = 2, max = 50)
    private String firstName;

    @NotBlank
    @Email
    private String email;

    @NotBlank
    @Size(min = 8)
    private String password;
}
```

```java
// Mapper (use MapStruct in production for large projects)
@Component
public class UserMapper {

    public UserResponseDto toDto(User user) {
        return UserResponseDto.builder()
                .id(user.getId())
                .firstName(user.getFirstName())
                .lastName(user.getLastName())
                .email(user.getEmail())
                .status(user.getStatus().name())
                .createdAt(user.getCreatedAt())
                .build();
    }

    public User toEntity(UserCreateDto dto) {
        return User.builder()
                .firstName(dto.getFirstName())
                .email(dto.getEmail())
                .status(UserStatus.PENDING_VERIFICATION)
                .build();
    }
}
```

---

### 10.3 Complete Service Example

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class UserService {

    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    private final UserMapper userMapper;

    @Transactional(readOnly = true)
    public UserResponseDto findById(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
        return userMapper.toDto(user);
    }

    @Transactional(readOnly = true)
    public Page<UserResponseDto> findAll(Pageable pageable) {
        return userRepository.findAll(pageable)
                .map(userMapper::toDto);
    }

    @Transactional
    public UserResponseDto create(UserCreateDto dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new ConflictException("Email already in use: " + dto.getEmail());
        }

        User user = userMapper.toEntity(dto);
        user.setPasswordHash(passwordEncoder.encode(dto.getPassword()));

        User savedUser = userRepository.save(user);
        log.info("Created user with id={}", savedUser.getId());

        return userMapper.toDto(savedUser);
    }

    @Transactional
    public UserResponseDto update(Long id, UserUpdateDto dto) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));

        // Only update non-null fields (partial update / PATCH pattern)
        if (dto.getFirstName() != null) user.setFirstName(dto.getFirstName());
        if (dto.getLastName() != null) user.setLastName(dto.getLastName());

        // No need to call save() — dirty checking handles it!
        return userMapper.toDto(user);
    }

    @Transactional
    public void delete(Long id) {
        User user = userRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
        userRepository.delete(user); // Triggers cascades on related entities
        log.info("Deleted user with id={}", id);
    }
}
```

### 10.4 Common Mistakes to Avoid

| Mistake | Problem | Fix |
|---|---|---|
| `@Transactional` on private methods | Ignored by proxy | Use public methods |
| Calling `@Transactional` from same class | Bypasses proxy | Self-inject or restructure |
| EAGER fetch on collections | Loads all data always | Use LAZY + JOIN FETCH |
| No `@Transactional(readOnly=true)` on reads | Slower, dirty checking runs | Add `readOnly=true` |
| Returning entities from API | Exposes internals | Use DTOs |
| Using `create` DDL in production | Drops data on restart | Use Flyway/Liquibase |
| Not handling `Optional` from `findById` | NPE | Use `.orElseThrow()` |
| Missing indexes on FK columns | Slow joins | Add `@Index` |
| Bulk operations without batch config | Many round-trips to DB | Configure `batch_size` |
| Not using `orphanRemoval` | Orphan records in DB | Add `orphanRemoval=true` |
| LazyInitializationException | Accessing lazy data outside tx | Open tx or eagerly fetch |

---

## 11. Quick Revision Cheat Sheet

```
┌──────────────────────────────────────────────────────────────────────┐
│               SPRING DATA JPA — QUICK REFERENCE                      │
├──────────────────────────────────────────────────────────────────────┤
│  STACK:  DB ← JDBC ← Hibernate (JPA impl) ← Spring Data JPA         │
├──────────────────────────────────────────────────────────────────────┤
│  ENTITY ANNOTATIONS                                                   │
│  @Entity           → marks class as DB table                         │
│  @Table(name="x")  → custom table name                               │
│  @Id               → primary key                                     │
│  @GeneratedValue   → auto-generate PK (IDENTITY/SEQUENCE/UUID)       │
│  @Column           → column options (nullable, unique, length)       │
│  @Enumerated       → store enum as STRING (never ORDINAL)            │
│  @Version          → optimistic locking                              │
│  @Transient        → field not mapped to DB                          │
├──────────────────────────────────────────────────────────────────────┤
│  RELATIONSHIPS                                                        │
│  @OneToOne         → Default: EAGER ⚠️  Override → LAZY             │
│  @OneToMany        → Default: LAZY ✅   Always: mappedBy              │
│  @ManyToOne        → Default: EAGER ⚠️  Override → LAZY             │
│  @ManyToMany       → Default: LAZY ✅   Use Set, not List            │
│  @JoinColumn       → specifies FK column (owner side)                │
│  mappedBy          → non-owner side (no FK here)                     │
├──────────────────────────────────────────────────────────────────────┤
│  REPOSITORIES                                                         │
│  JpaRepository<T,ID>           → extends CrudRepository             │
│  JpaSpecificationExecutor<T>   → dynamic queries                     │
│  findBy, countBy, deleteBy     → method naming magic                 │
│  @Query("JPQL")                → custom JPQL                         │
│  @Query(nativeQuery=true)      → raw SQL                             │
│  @Modifying + @Transactional   → for UPDATE/DELETE @Query            │
├──────────────────────────────────────────────────────────────────────┤
│  PERFORMANCE                                                          │
│  JOIN FETCH        → solve N+1 for to-one                            │
│  @EntityGraph      → declarative eager loading                       │
│  @BatchSize        → solve N+1 for collections                       │
│  readOnly=true     → faster reads (skip dirty checking)              │
│  Projections       → load only needed columns                        │
│  batch_size config → efficient bulk inserts                          │
├──────────────────────────────────────────────────────────────────────┤
│  TRANSACTIONS                                                         │
│  @Transactional            → default REQUIRED, rollback RuntimeEx    │
│  readOnly = true           → for read queries                         │
│  propagation               → REQUIRED (default), REQUIRES_NEW, etc.  │
│  rollbackFor               → include checked exceptions               │
│  Dirty checking            → no need to call save() in @Transactional│
├──────────────────────────────────────────────────────────────────────┤
│  AUDITING                                                             │
│  @EnableJpaAuditing        → in @Configuration class                 │
│  @EntityListeners(AuditingEntityListener.class)                      │
│  @CreatedDate / @LastModifiedDate                                    │
│  @CreatedBy / @LastModifiedBy + AuditorAware bean                    │
├──────────────────────────────────────────────────────────────────────┤
│  QUERY TYPES (PRIORITY ORDER)                                         │
│  1. Method names          → simplest                                  │
│  2. @Query (JPQL)         → flexible, type-safe                       │
│  3. Specification         → dynamic multi-filter queries              │
│  4. @Query (native)       → DB-specific, complex SQL                 │
│  5. EntityManager/Criteria → maximum control                         │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 12. Interview Questions & Answers

### Fundamentals

---

**Q1: What is the difference between JPA, Hibernate, and Spring Data JPA?**

**A:** JPA is a **specification** (a set of interfaces and rules defined by Jakarta EE). Hibernate is an **implementation** of that specification — it's the actual code that translates Java operations to SQL. Spring Data JPA is an **abstraction layer on top of JPA** that eliminates boilerplate code by auto-generating repository implementations, supporting method-name-based query generation, and providing built-in pagination. The relationship is:  
`Spring Data JPA → JPA (interfaces) → Hibernate (implementation) → JDBC → Database`

---

**Q2: What is an EntityManager and what is its role?**

**A:** The `EntityManager` is the core JPA interface for interacting with the **persistence context** (first-level cache). It manages entity lifecycle operations: `persist()` (insert), `merge()` (update detached), `remove()` (delete), `find()` (select by PK), and `createQuery()` (JPQL queries). Spring Data JPA hides this behind repositories, but it's used internally and can be injected with `@PersistenceContext` for custom repository implementations.

---

**Q3: What is the persistence context?**

**A:** The persistence context is a **cache that holds managed entities** during a unit of work (usually one transaction). It's Hibernate's first-level cache — every entity loaded or saved within a transaction is stored here. This enables dirty checking (auto-detecting and flushing changes) and ensures identity guarantee (same DB row = same Java object instance within one session). The persistence context is flushed (SQL sent to DB) before commits or explicit `flush()` calls.

---

### Entity Mapping

---

**Q4: What is the difference between `@JoinColumn` and `mappedBy`?**

**A:** `@JoinColumn` is placed on the **owner side** of the relationship (the entity whose table contains the foreign key column). `mappedBy` is placed on the **inverse/non-owner side** and tells JPA "don't create a foreign key here — the relationship is owned by the field named X in the other entity." If you forget `mappedBy`, JPA creates a join table or extra columns.

---

**Q5: Why should you use `@Enumerated(EnumType.STRING)` instead of the default `ORDINAL`?**

**A:** `ORDINAL` stores the enum's position (0, 1, 2...). If you reorder enum constants or add new ones in the middle, all stored data becomes invalid/corrupted. `STRING` stores the enum name ("ACTIVE", "INACTIVE"), which is stable — reordering or adding constants doesn't affect stored data. Always use `EnumType.STRING`.

---

**Q6: What is the difference between `CascadeType.ALL` and `orphanRemoval = true`?**

**A:** `CascadeType.ALL` means all JPA operations (persist, merge, remove, refresh, detach) on the parent are cascaded to children. `orphanRemoval = true` specifically handles the case where you **remove a child from the parent's collection** — the child is automatically deleted from the database. `CascadeType.REMOVE` only deletes children when the parent itself is deleted. `orphanRemoval` handles collection removals. Both are often used together on `@OneToMany` relationships.

---

### N+1 & Performance

---

**Q7: What is the N+1 problem? How do you detect and fix it?**

**A:** The N+1 problem occurs when fetching N entities triggers N additional queries to load their related entities (e.g., fetching 100 orders then 100 separate queries for each order's customer = 101 queries total).

**Detection:** Enable `show-sql=true` and count queries, or use `datasource-proxy` / `p6spy` in tests. Hibernate also logs warnings.

**Fixes:**
1. **JOIN FETCH** in JPQL: `SELECT e FROM Employee e JOIN FETCH e.department`
2. **@EntityGraph**: `@EntityGraph(attributePaths = {"department"})`
3. **@BatchSize**: `@BatchSize(size = 30)` on the collection — loads in batches
4. **FetchMode.SUBSELECT**: Loads all children in one subquery

---

**Q8: When would you use `PESSIMISTIC_WRITE` vs `@Version` (optimistic locking)?**

**A:** Use **optimistic locking (`@Version`)** when conflicts are rare — it allows concurrent reads, and only fails at commit if a concurrent update happened. It's lighter weight and scales better. Use **`PESSIMISTIC_WRITE`** when conflicts are frequent or the cost of retrying is high — it locks the row immediately (other transactions wait or fail). For example, inventory reduction in flash sales should use pessimistic locking to prevent overselling; a user profile update can use optimistic locking.

---

### Transactions

---

**Q9: `@Transactional` on a private method — does it work?**

**A:** No. Spring's transaction management uses **AOP proxies**. The proxy wraps the bean and intercepts method calls to apply transactional behavior. Private methods are not visible to the proxy (and are not overridable), so `@Transactional` is completely ignored. The fix: make the method `public`, or refactor the logic into a separate Spring bean where the proxy can intercept it.

---

**Q10: What is the difference between transaction propagation `REQUIRED` and `REQUIRES_NEW`?**

**A:**
- `REQUIRED` (default): If a transaction already exists, join it. If not, create a new one. All operations share one transaction — one rollback affects everything.
- `REQUIRES_NEW`: Always start a brand new transaction, suspending any existing one. The new transaction has its own commit/rollback lifecycle. Use case: **audit logging** — you want the audit record saved even if the main business transaction rolls back.

---

**Q11: What is dirty checking?**

**A:** Hibernate takes a **snapshot** of managed entities when they're loaded. At flush time (before commit), Hibernate compares the current state with the snapshot. If anything changed, it automatically generates and executes the `UPDATE` SQL — without you calling `save()`. This only works for **managed entities** (inside a `@Transactional` method). After the transaction ends, entities are detached and changes are no longer tracked.

---

### Queries

---

**Q12: What's the difference between JPQL and native SQL queries in Spring Data JPA?**

**A:**
| | JPQL | Native SQL |
|---|---|---|
| Syntax | Entity/field names | Table/column names |
| Portability | Works on any DB | DB-specific |
| JPA features | Full (caching, 2nd level) | Limited |
| Complex queries | Limited | Full SQL power |
| Use for | Standard queries | Window functions, CTEs, specific DB features |

---

**Q13: How do projections improve performance?**

**A:** When you return a full entity, Hibernate loads all columns from the table. If you only need 3 out of 30 columns, you're wasting 90% of the data transfer. **Interface projections** and **DTO projections** let you define exactly which fields to fetch. Spring Data JPA generates `SELECT id, first_name, email FROM users` instead of `SELECT *`. This reduces: data transfer, memory usage, and deserialization time. For read-heavy endpoints, this can be a 3-10x performance improvement.

---

**Q14: What is the difference between `Page` and `Slice` in Spring Data JPA?**

**A:**
- `Page<T>`: Executes TWO queries — one for the data and one for `COUNT(*)` to know the total records and pages. More expensive but gives you `totalElements` and `totalPages`.
- `Slice<T>`: Executes ONE query — just fetches page+1 records to know if there's a next page. No total count. Use `Slice` for infinite scroll UIs where you don't need total count. Use `Page` for traditional pagination with page numbers.

---

### Advanced

---

**Q15: Explain the first-level and second-level cache in Hibernate.**

**A:**
- **First-level cache** (Session cache): Always on, scoped to a single `Session`/transaction. Every entity you load or save is stored here. Within the same transaction, fetching the same entity twice returns the same object without an extra SQL query. Cleared when the session closes.
- **Second-level cache**: Shared across sessions/transactions. Stores entity data by ID. Needs explicit configuration (Ehcache, Caffeine, Redis). Annotate entities with `@Cache`. Great for frequently read, rarely changed data (countries, categories, etc.). Query cache is an extension that caches query results.

---

**Q16: How do you handle `LazyInitializationException`?**

**A:** This happens when you access a lazy-loaded relationship outside an open transaction (the session is closed). Solutions:
1. **Best:** Load the data within the transaction using `JOIN FETCH` or `@EntityGraph`
2. **Use DTO projection**: Don't pass entities outside the service layer
3. **`@Transactional` on service method**: Keeps session open during the call
4. **`spring.jpa.open-in-view=false`** (disable OSIV): Forces you to solve the problem properly at the design level
5. **Avoid:** `FetchType.EAGER` — solves LazyInitException but creates N+1

---

**Q17: How does the `Specification` pattern help with dynamic queries?**

**A:** When you have multiple optional filters (name, status, date range, department), combining them with method-name queries leads to combinatorial explosion (2^n methods). The `Specification` pattern wraps each filter as a predicate and lets you compose them with `.and()`, `.or()`, `.not()`. You extend `JpaSpecificationExecutor` in the repository and build `Specification<T>` objects in the service. Only non-null predicates are applied, giving you clean, dynamic, composable SQL without if-else chains.

---

**Q18: What is `@MappedSuperclass` and when do you use it?**

**A:** `@MappedSuperclass` marks a class whose **mapped fields are inherited by entity subclasses** — but the class itself is not an entity and has no table. Used for sharing common fields across multiple entities (e.g., `id`, `createdAt`, `updatedAt`, `createdBy`). Unlike `@Inheritance`, there's no hierarchy table — each entity gets its own table with the inherited columns. The base auditing class pattern uses `@MappedSuperclass` with `@EntityListeners(AuditingEntityListener.class)`.

---

**Q19: What are the `GenerationType` strategies and which should you use?**

**A:**
- `IDENTITY`: DB auto-increment. Simple, works everywhere. **Disables Hibernate batch inserts** (DB generates ID after insert, so Hibernate can't batch them). Use for simple apps.
- `SEQUENCE`: DB sequence object. Hibernate pre-allocates IDs in batches (`allocationSize`). **Best for performance and batch inserts**. Best for PostgreSQL/Oracle.
- `UUID`: Generates random UUID. Good for microservices and distributed systems. Slightly larger index size.
- `TABLE`: Uses a separate table for ID tracking. Slowest, avoid in production.

**Recommendation:** `SEQUENCE` for PostgreSQL apps needing performance; `IDENTITY` for simple MySQL apps; `UUID` for distributed systems.

---

**Q20: How do you implement soft deletes with Spring Data JPA?**

**A:** Instead of physically deleting records, add a `deletedAt` or `deleted` flag:

```java
@Entity
@Where(clause = "deleted = false")          // Hibernate filter — always adds WHERE deleted=false
@SQLDelete(sql = "UPDATE users SET deleted = true WHERE id = ?") // Override DELETE SQL
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private boolean deleted = false;
    private LocalDateTime deletedAt;
}

// Now repository.delete(user) executes UPDATE instead of DELETE
// And all findBy methods automatically exclude deleted records
```

---

> 📝 **Final Note:** This document covers Spring Data JPA from foundation to production patterns. Keep it bookmarked for quick reference during development and interview preparation. The most important concepts to deeply understand are: Entity relationships and fetch strategies, the N+1 problem, transaction management, and the Specification pattern for dynamic queries.

---

*Generated for developers building production-grade Spring Boot applications.*  
*Spring Boot 3.x | Spring Data JPA 3.x | Hibernate 6.x | Java 17+*