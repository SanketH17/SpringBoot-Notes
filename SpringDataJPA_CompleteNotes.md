# Spring Data JPA – Complete Developer Notes

> 🧑‍💻 *A single, comprehensive reference for learning, revision, and real-world backend development.*  
> Covers beginner → advanced. Clean code. Interview-ready.

---

## 📋 Table of Contents

1. [Fundamentals](#1-fundamentals)
   - [1.1 What is JPA?](#11-what-is-jpa)
   - [1.2 What is Hibernate?](#12-what-is-hibernate)
   - [1.3 What is Spring Data JPA?](#13-what-is-spring-data-jpa)
   - [1.4 How They All Fit Together](#14-how-they-all-fit-together)
   - [1.5 Why Use Spring Data JPA?](#15-why-use-spring-data-jpa)
2. [Project Setup](#2-project-setup)
   - [2.1 Maven Dependencies](#21-maven-dependencies)
   - [2.2 Gradle Dependencies](#22-gradle-dependencies)
   - [2.3 application.properties — MySQL Configuration](#23-applicationproperties--mysql-configuration)
   - [2.4 application.yml — PostgreSQL Configuration](#24-applicationyml--postgresql-configuration)
   - [2.5 DDL Auto Strategies Explained](#25-ddl-auto-strategies-explained)
3. [Entity Mapping](#3-entity-mapping)
   - [3.1 Basic Entity — The Building Block](#31-basic-entity--the-building-block)
   - [3.2 @GeneratedValue Strategies](#32-generatedvalue-strategies)
   - [3.3 Relationships](#33-relationships)
   - [3.4 Fetch Types](#34-fetch-types)
   - [3.5 Cascade Types](#35-cascade-types)
4. [Repository Layer](#4-repository-layer)
   - [4.1 Repository Hierarchy — Understanding the Family Tree](#41-repository-hierarchy--understanding-the-family-tree)
   - [4.2 Creating a Repository](#42-creating-a-repository)
   - [4.3 Query Methods by Naming Convention (Derived Queries)](#43-query-methods-by-naming-convention-derived-queries)
   - [4.4 Pagination & Sorting](#44-pagination--sorting)
5. [Query Mechanisms](#5-query-mechanisms)
   - [5.1 JPQL — Java Persistence Query Language](#51-jpql--java-persistence-query-language)
   - [5.2 Native SQL Queries](#52-native-sql-queries)
   - [5.3 Named Queries](#53-named-queries)
   - [5.4 Projections — Getting Only What You Need](#54-projections--getting-only-what-you-need)
6. [Advanced Topics](#6-advanced-topics)
   - [6.1 The N+1 Problem](#61-the-n1-problem--critical--most-common-performance-issue)
   - [6.2 Entity Lifecycle](#62-entity-lifecycle)
   - [6.3 Dirty Checking (Auto-Flush)](#63-dirty-checking-auto-flush)
   - [6.4 Transactions (@Transactional)](#64-transactions-transactional)
   - [6.5 Optimistic vs Pessimistic Locking](#65-optimistic-vs-pessimistic-locking)
   - [6.6 Caching](#66-caching)
     - [First-Level Cache (Always On — Per-Transaction)](#first-level-cache-always-on--per-transaction)
     - [Second-Level Cache (Cross-Transaction — Needs Configuration)](#second-level-cache-cross-transaction--needs-configuration)
     - [Cache Concurrency Strategy — Which One to Choose?](#cache-concurrency-strategy--which-one-to-choose)
     - [Spring Boot Application-Level Caching (@Cacheable, @CachePut, @CacheEvict)](#spring-boot-application-level-caching-cacheable-cacheput-cacheevict)
7. [Quick Revision Cheat Sheet](#7-quick-revision-cheat-sheet)
8. [Interview Questions & Answers](#8-interview-questions--answers)

---

## 1. Fundamentals

### 1.1 What is JPA?

**JPA (Java Persistence API)** is a **specification** — a set of rules and interfaces that tells Java developers: *"Here's the standard way to save, read, update, and delete Java objects in a relational database."*

JPA itself is **not code you can run**. It's a blueprint. It defines *what* should happen, but not *how* it happens.

> 🧠 **Analogy — The Restaurant Menu:**  
> Think of JPA as a **restaurant menu**. The menu lists every dish you can order (save, find, delete...), describes what each dish should look like, and sets expectations. But the menu **doesn't cook anything** — you need a kitchen (Hibernate) to actually prepare the food.

#### What does JPA give you?

- **Object-Relational Mapping (ORM):** Map your Java classes directly to database tables. A `User` class becomes a `users` table. A field `String email` becomes an `email` column. You work with Java objects — JPA handles the SQL.
- **CRUD operations:** Create, Read, Update, and Delete data using Java methods instead of writing SQL by hand.
- **JPQL (Java Persistence Query Language):** A query language that looks like SQL but uses your **Java class and field names** instead of table and column names.

> 📝 **Beginner Note:** JPA is *just an interface*. By itself, it does nothing. You always need a JPA **implementation** (like Hibernate) to do the actual database work. This is the same idea as `List` being an interface and `ArrayList` being the implementation.

#### What problem does JPA solve?

Before JPA, every developer wrote raw JDBC code to talk to the database — opening connections, writing SQL strings, mapping result sets to objects manually, and closing resources. This was **repetitive**, **error-prone**, and **tightly coupled** to a specific database. JPA standardized all of this so you write Java, and the framework handles SQL.

---

### 1.2 What is Hibernate?

**Hibernate** is the most popular **implementation of JPA**. It is the actual engine that takes your JPA instructions and converts them into real SQL queries that run against your database.

> 🧠 **Analogy — The Kitchen:**  
> If JPA is the restaurant menu, **Hibernate is the kitchen and the chef**. When you order "save this User" from the menu, Hibernate is what actually fires up the stove, writes the `INSERT INTO users ...` SQL, sends it to the database, and gives you back the result.

#### What does Hibernate do behind the scenes?

Here's what happens step-by-step when you save an object:

1. You call `save(user)` (a JPA operation)
2. Hibernate reads the `@Entity` annotations on your `User` class
3. It figures out which table and columns to use
4. It generates the correct SQL: `INSERT INTO users (first_name, email, ...) VALUES (?, ?, ...)`
5. It sends the SQL to the database via JDBC
6. It returns the saved object with the generated ID

You never see steps 2–6. Hibernate does it all for you.

#### Hibernate's bonus features (beyond what JPA requires):

- **Caching** — Stores recently accessed data in memory so repeated reads don't hit the database (1st-level and 2nd-level cache)
- **Lazy Loading** — Loads related data only when you actually access it, not upfront
- **HQL (Hibernate Query Language)** — Hibernate's own query language, very similar to JPQL
- **Automatic Schema Generation** — Can create or update your database tables based on your Java classes

> 📝 **Beginner Note:** You rarely interact with Hibernate directly in a Spring Boot project. Spring Data JPA sits on top and calls Hibernate for you. But understanding that Hibernate is the engine under the hood helps you debug issues and understand logs.

---

### 1.3 What is Spring Data JPA?

**Spring Data JPA** is a **Spring framework module** that sits on top of JPA and Hibernate. Its job is to **eliminate boilerplate code** so you can accomplish database operations with minimal effort.

> 🧠 **Analogy — The Waiter:**  
> You're sitting in a restaurant (your application). JPA is the menu. Hibernate is the kitchen. **Spring Data JPA is your waiter.** You just tell the waiter "I'd like the user with email john@example.com" and the waiter handles everything — walks to the kitchen, places the order, brings back the result. You never step into the kitchen yourself.

#### The before and after

**Without Spring Data JPA** (using raw JPA/Hibernate), you had to write all this code every single time:

```java
// Pure JPA — lots of manual work
EntityManager em = entityManagerFactory.createEntityManager();  // Get a connection
em.getTransaction().begin();                                     // Start a transaction
em.persist(user);                                                // Insert the user
em.getTransaction().commit();                                    // Commit the transaction
em.close();                                                      // Close the connection
```

That's 5 lines of plumbing just to save one object. Now imagine doing this for every CRUD operation across every entity in your app.

**With Spring Data JPA**, the same thing becomes:

```java
userRepository.save(user); // That's it. One line. Done.
```

Spring Data JPA auto-generates the repository implementation at runtime. You just declare an interface, and Spring writes the code for you.

#### What problem does Spring Data JPA solve?

Even with Hibernate doing the heavy SQL work, developers still had to write **repetitive boilerplate**: creating `EntityManager` instances, managing transactions, writing basic CRUD methods over and over for every entity. Spring Data JPA removes all of that by:

- Auto-implementing repository interfaces (you write zero implementation code)
- Generating SQL queries from method names (e.g., `findByEmail(String email)` automatically becomes `SELECT * FROM users WHERE email = ?`)
- Providing built-in pagination, sorting, and batch operations

> 📝 **Beginner Note:** Spring Data JPA doesn't replace Hibernate — it **uses** Hibernate internally. Think of it as a convenience layer. The stack is:  
> **Your Code → Spring Data JPA → JPA (interface) → Hibernate (engine) → JDBC → Database**

---

### 1.4 How They All Fit Together

Here's the full picture of how these technologies are layered:

```
┌─────────────────────────────────────────────────────┐
│              YOUR APPLICATION CODE                   │
│         (Service classes, Controllers)                │
├─────────────────────────────────────────────────────┤
│                  Spring Data JPA                     │
│   (Convenience layer — auto-generates repositories)  │
├─────────────────────────────────────────────────────┤
│                      JPA                             │
│     (Specification — defines the rules/interfaces)   │
├─────────────────────────────────────────────────────┤
│                   Hibernate                          │
│  (Implementation — converts Java operations to SQL)  │
├─────────────────────────────────────────────────────┤
│                    JDBC                              │
│        (Low-level Java database connection API)      │
├─────────────────────────────────────────────────────┤
│              Database (MySQL/PostgreSQL)              │
└─────────────────────────────────────────────────────┘
```

> 🧠 **Analogy — Ordering Online:**  
> Think of ordering food through a delivery app:
> - **You** = Your application code (the customer placing the order)
> - **Delivery App (Zomato/Swiggy)** = Spring Data JPA (takes your request, simplifies everything)
> - **Restaurant Menu** = JPA (standardized set of things you can order)
> - **Kitchen & Chef** = Hibernate (actually cooks the food / generates SQL)
> - **Delivery Driver** = JDBC (physically carries data between kitchen and your door)
> - **Restaurant** = Database (where the food/data lives)

#### Comparison Table

| Feature | JPA | Hibernate | Spring Data JPA |
|---|---|---|---|
| **What is it?** | Specification (rules) | Implementation (engine) | Abstraction (convenience) |
| **Who creates it?** | Jakarta EE (Java standard) | Red Hat / Community | Spring Team (Pivotal/VMware) |
| **Amount of boilerplate** | High | Medium | Very Low |
| **Extra features** | No (just the standard) | Yes (caching, HQL, schema gen) | Yes (Repositories, Pageable, query methods) |
| **Can work alone?** | No (needs an implementation) | Yes (standalone Hibernate works) | No (needs JPA + Hibernate underneath) |

> 💡 **Important to Remember:** In a Spring Boot project, when you add the `spring-boot-starter-data-jpa` dependency, you get **all three** automatically — Spring Data JPA, JPA interfaces, and Hibernate as the default implementation. You don't need to configure them separately.

---

### 1.5 Why Use Spring Data JPA?

Here's why Spring Data JPA is the go-to choice for database access in modern Spring Boot applications:

- ✅ **Zero boilerplate** — No manual `EntityManager`, no hand-written transactions. Just declare interfaces.
- ✅ **Magic query methods** — Write `findByEmailAndStatus(String email, UserStatus status)` and Spring auto-generates the SQL. No query writing needed for simple cases.
- ✅ **Built-in pagination and sorting** — Paginate results with `PageRequest.of(page, size)`. No manual `LIMIT`/`OFFSET` SQL.
- ✅ **Database-agnostic** — Switch from MySQL to PostgreSQL by changing a config property. Your Java code stays the same.
- ✅ **Works with any JPA provider** — Hibernate is the default, but you can swap in EclipseLink or others without changing your repository code.
- ✅ **Seamless Spring ecosystem integration** — Works beautifully with Spring Boot auto-configuration, Spring Security, Spring Transactions, etc.
- ✅ **Production-ready features** — Auditing (`@CreatedDate`), specifications (dynamic queries), entity graphs, and more — all built in.

> 📝 **Beginner Note — When NOT to use Spring Data JPA:**  
> - For extremely simple apps with 1–2 tables, `spring-boot-starter-jdbc` with `JdbcTemplate` might be simpler.  
> - For read-heavy analytics or complex reporting queries, consider using a lighter tool like **jOOQ** or **MyBatis** alongside JPA.  
> - JPA adds a learning curve — it's worth it for most CRUD-heavy business apps, but not for every use case.

---

## 2. Project Setup

> This section walks you through everything you need to set up Spring Data JPA in a Spring Boot project — from adding dependencies to configuring your database connection.

### 2.1 Maven Dependencies

> 🧠 **Analogy — Buying Ingredients Before Cooking:**  
> Before you can cook a meal, you need to buy the right ingredients. Dependencies are the ingredients for your project. Without the right ones, your app won't know how to talk to a database.

Add these dependencies to your `pom.xml`. Each one has a specific role:

```xml
<!-- pom.xml -->
<dependencies>

    <!-- 
        Spring Data JPA Starter
        This single dependency pulls in EVERYTHING you need:
        - Spring Data JPA (the convenience layer)
        - JPA API (the specification interfaces)
        - Hibernate (the default JPA implementation / engine)
        - HikariCP (the default connection pool)
        - Spring Transaction Management
    -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- 
        MySQL Database Driver
        This is the "translator" between Java and MySQL.
        Without it, your app can't communicate with a MySQL database.
        scope=runtime means it's only needed when the app runs, not during compilation.
    -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- 
        PostgreSQL Database Driver (use INSTEAD of MySQL if your DB is PostgreSQL)
        Uncomment this and remove MySQL driver if using PostgreSQL.
    -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- 
        H2 Database — Lightweight in-memory database for testing
        Your tests can run without needing a real MySQL/PostgreSQL server.
        scope=test means it's only available during testing, not in production.
    -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>

    <!-- 
        Lombok — Reduces Java boilerplate (getters, setters, constructors, builders)
        optional=true means projects that depend on YOUR project won't get Lombok automatically.
    -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>

</dependencies>
```

> 📝 **Beginner Note:** You only need **one** database driver — pick MySQL *or* PostgreSQL based on your database. Don't include both in a real project.

> 💡 **Important to Remember:** The `spring-boot-starter-data-jpa` dependency is the key. It automatically brings in Hibernate, JPA, and connection pooling. You don't need to add them separately.

### 2.2 Gradle Dependencies

If you use Gradle instead of Maven, add these to your `build.gradle`:

```groovy
// build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.mysql:mysql-connector-j'         // MySQL driver
    // runtimeOnly 'org.postgresql:postgresql'          // Uncomment for PostgreSQL
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'     // Lombok needs annotation processing
    testRuntimeOnly 'com.h2database:h2'                // H2 for tests only
}
```

> 📝 **Beginner Note — Maven vs Gradle:** Both do the same thing (manage dependencies). Maven uses XML (`pom.xml`), Gradle uses Groovy/Kotlin (`build.gradle`). Pick whichever your team uses. Most Spring Boot tutorials use Maven.

---

### 2.3 application.properties — MySQL Configuration

This is where you tell Spring Boot **how to connect to your database**. This file lives at `src/main/resources/application.properties`.

> 🧠 **Analogy — Filling Out a Delivery Address Form:**  
> Imagine you're ordering something online. You need to tell the delivery service: where to deliver (URL), who you are (username), and your credentials (password). That's exactly what these properties do — they tell Spring Boot where the database lives and how to log in.

```properties
# ─── Database Connection ───────────────────────────────
# URL Format: jdbc:mysql://HOST:PORT/DATABASE_NAME?options
# localhost  = database is on your machine
# 3306       = default MySQL port
# mydb       = your database name (create it first in MySQL!)
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=yourpassword
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# ─── JPA / Hibernate Settings ─────────────────────────
# ddl-auto controls what Hibernate does with your database schema on startup
# See section 2.5 below for a detailed explanation of each option
spring.jpa.hibernate.ddl-auto=update

# Show the SQL that Hibernate generates in the console
# Extremely useful for learning and debugging — you can see every query!
spring.jpa.show-sql=true

# Format the SQL nicely (instead of one long unreadable line)
spring.jpa.properties.hibernate.format_sql=true

# Tell Hibernate which SQL dialect to use for MySQL 8
# Spring Boot usually auto-detects this, but being explicit avoids surprises
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# ─── Connection Pool (HikariCP) ───────────────────────
# A connection pool keeps a set of database connections ready to use,
# so your app doesn't create a new connection for every single query (expensive!).
# HikariCP is Spring Boot's default pool — fast and reliable.
spring.datasource.hikari.maximum-pool-size=10       # Max 10 connections open at once
spring.datasource.hikari.minimum-idle=5             # Keep at least 5 idle connections ready
spring.datasource.hikari.connection-timeout=30000   # Wait max 30 seconds for a connection
```

#### What happens step-by-step when your app starts:

1. Spring Boot reads `application.properties`
2. It creates a **HikariCP connection pool** with the database URL, username, and password
3. Hibernate starts up and scans for classes annotated with `@Entity`
4. Based on `ddl-auto`, it creates/updates/validates the database tables
5. Spring Data JPA auto-generates implementations for your repository interfaces
6. Your app is ready to handle database operations!

> ⚠️ **Common Mistake:** Forgetting to create the database first! `spring.datasource.url` points to `mydb`, but if that database doesn't exist in MySQL, the app will fail on startup. Run `CREATE DATABASE mydb;` in MySQL first.

> 🧠 **Analogy — Connection Pool:**  
> Think of a connection pool like a **taxi stand** at a hotel. Instead of calling a new taxi every time a guest needs a ride (slow and expensive), the hotel keeps 10 taxis parked and ready. When a guest needs one, they grab an available taxi. When done, the taxi returns to the stand for the next guest. That's exactly what HikariCP does with database connections.

---

### 2.4 application.yml — PostgreSQL Configuration

Spring Boot also supports YAML format (`application.yml`). YAML uses indentation instead of repeated prefixes, making it cleaner for nested properties:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/mydb   # PostgreSQL default port is 5432
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
        # Batch insert optimization — groups multiple INSERTs into one round-trip
        jdbc:
          batch_size: 50
        order_inserts: true    # Reorder INSERT statements to enable batching
        order_updates: true    # Reorder UPDATE statements to enable batching
```

> 📝 **Beginner Note — .properties vs .yml:** Both do the same thing. Use whichever your team prefers. YAML looks cleaner for deeply nested config, but indentation mistakes can break it. Properties files are simpler and more forgiving. **Never use both files for the same properties** — Spring Boot loads both, and conflicts can cause confusing bugs.

---

### 2.5 DDL Auto Strategies Explained

The `spring.jpa.hibernate.ddl-auto` property tells Hibernate what to do with your database tables when the application starts up. This is one of the most important settings to understand.

> 🧠 **Analogy — Setting Up a Room:**  
> Imagine you're assigned a hotel room (database) every day:
> - **`create`** = Throw away everything in the room and set it up fresh every morning (destructive!)
> - **`create-drop`** = Set up the room when you arrive, destroy everything when you leave
> - **`update`** = Walk in and rearrange the room to match your needs, but keep existing stuff
> - **`validate`** = Walk in and just check if the room matches your expectations. If not, refuse to stay (app crashes)
> - **`none`** = Don't touch the room at all. Assume someone else set it up perfectly.

| Value | What Happens on App Startup | When to Use |
|---|---|---|
| `create` | **Drops all tables** and recreates them from scratch. **All data is lost!** | Local development only (when you don't care about data) |
| `create-drop` | Creates tables on startup, **drops them when the app shuts down** | Automated testing (clean state every test run) |
| `update` | Compares your entities to existing tables. Adds new columns/tables but **never deletes** existing ones. | Development (convenient, but see warning below) |
| `validate` | Checks that your entities match the existing database schema. If they don't match, the **app refuses to start** with an error. | Staging / Pre-production |
| `none` | Hibernate does nothing to the schema. You manage it yourself. | **Production** (use Flyway or Liquibase for migrations) |

> ⚠️ **Common Mistake — NEVER use `create` or `update` in production!**
> - `create` destroys all your data on every restart.
> - `update` seems safe, but it can silently make schema changes that are hard to track or reverse. It also **never drops columns** — if you remove a field from your entity, the column stays in the database forever.
> - In production, use a **database migration tool** like **Flyway** or **Liquibase**. These tools track every schema change as versioned scripts, making changes reviewable, repeatable, and reversible.

> 📝 **Beginner Note — Recommended setup for learning:**  
> Use `ddl-auto=update` while learning. It automatically creates and modifies tables as you change your entity classes, so you can focus on learning JPA without writing SQL schema scripts. Switch to `validate` + Flyway when building real projects.

---

## 3. Entity Mapping

> Entity mapping is the heart of JPA. It's how you tell Java: "this class represents a database table, and these fields represent its columns." Once you master this section, you'll be able to model any database schema using pure Java code.

### 3.1 Basic Entity — The Building Block

#### What is an Entity?

An **entity** is a plain Java class that JPA maps to a database table. Each **instance** of the class represents one **row** in that table. Each **field** in the class represents one **column**.

> 🧠 **Analogy — A Spreadsheet:**  
> Think of a database table as a **Google Sheet**:
> - The **class** (`User`) is the sheet template — it defines the column headers (firstName, email, status...)
> - Each **object** (`new User("John", "john@example.com")`) is one **row** of data in that sheet
> - The **annotations** (`@Column`, `@Id`) are like formatting rules you set on each column (required, unique, max length...)

#### A Complete Entity Example

Here's a real-world `User` entity with every common annotation explained:

```java
// File: src/main/java/com/example/entity/User.java

package com.example.entity;

import jakarta.persistence.*;
import lombok.*;
import java.time.LocalDateTime;

@Entity                         // ① Tells JPA: "This class maps to a database table"
@Table(
    name = "users",             // ② Explicitly set table name. Default would be "User",
                                //    but "user" is a reserved word in many databases!
    indexes = {
        @Index(name = "idx_email", columnList = "email"),       // ③ Database index = faster lookups
        @Index(name = "idx_status", columnList = "status")
    },
    uniqueConstraints = {
        @UniqueConstraint(name = "uc_email", columnNames = {"email"}) // ④ No duplicate emails
    }
)
@Getter                         // Lombok: auto-generates all getter methods
@Setter                         // Lombok: auto-generates all setter methods
@NoArgsConstructor              // Lombok: generates empty constructor (JPA requires this!)
@AllArgsConstructor             // Lombok: generates constructor with all fields
@Builder                        // Lombok: enables builder pattern — User.builder().name("John").build()
public class User {

    @Id                                     // ⑤ This field is the PRIMARY KEY
    @GeneratedValue(strategy = GenerationType.IDENTITY) // ⑥ Database auto-generates this value (1, 2, 3...)
    private Long id;

    @Column(
        name = "first_name",                // ⑦ Column name in DB (default would be "firstName")
        nullable = false,                   // NOT NULL — this field is required
        length = 50                         // VARCHAR(50) — max 50 characters
    )
    private String firstName;

    @Column(nullable = false, unique = true, length = 100)  // Required + must be unique
    private String email;

    @Column(name = "password_hash", nullable = false)
    private String passwordHash;

    @Enumerated(EnumType.STRING)            // ⑧ Store enum as text "ACTIVE", not number 0
    @Column(nullable = false, length = 20)
    private UserStatus status;

    @Column(columnDefinition = "TEXT")      // ⑨ TEXT type for long strings (no length limit)
    private String bio;

    @Column(name = "is_verified")
    private boolean verified = false;       // Default value

    @Column(name = "created_at", updatable = false) // ⑩ Cannot be changed after initial insert
    private LocalDateTime createdAt;

    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

```java
// Enum for user status — used by the @Enumerated field above
public enum UserStatus {
    ACTIVE, INACTIVE, BANNED, PENDING_VERIFICATION
}
```

#### Annotation-by-Annotation Breakdown

Let's walk through every annotation used above so there's zero confusion:

| # | Annotation | What It Does | Why It Matters |
|---|---|---|---|
| ① | `@Entity` | Marks this class as a JPA entity (a DB table) | Without it, JPA completely ignores this class |
| ② | `@Table(name = "users")` | Sets the table name explicitly | Avoids conflicts with reserved words like "user" |
| ③ | `@Index` | Creates a database index on specified columns | Makes `WHERE email = ?` queries much faster |
| ④ | `@UniqueConstraint` | Ensures no duplicate values in specified columns | Prevents two users from having the same email |
| ⑤ | `@Id` | Marks the primary key field | Every entity MUST have exactly one `@Id` |
| ⑥ | `@GeneratedValue` | Auto-generates the ID value | You don't set the ID — the database assigns it |
| ⑦ | `@Column(name = ...)` | Customizes column name, nullability, length | Controls how the field maps to the actual column |
| ⑧ | `@Enumerated(STRING)` | Stores enum as readable text, not number | See warning below about ORDINAL |
| ⑨ | `columnDefinition = "TEXT"` | Uses DB-specific column type | For large text fields that exceed `VARCHAR` limits |
| ⑩ | `updatable = false` | Prevents this column from being changed after insert | Perfect for `createdAt` timestamps |

> ⚠️ **Common Mistake — `@Enumerated(EnumType.ORDINAL)` (the default!):**  
> If you use `ORDINAL`, the enum is stored as a number (ACTIVE=0, INACTIVE=1, BANNED=2). Now imagine you add a new status `SUSPENDED` between ACTIVE and INACTIVE — suddenly all your database values shift and mean the wrong thing! **Always use `EnumType.STRING`** to store the actual name like `"ACTIVE"`, `"BANNED"`.

> 📝 **Beginner Note — Why `@NoArgsConstructor`?**  
> JPA creates entity objects using reflection, which requires an **empty constructor** (no arguments). If you don't have Lombok, you must write `public User() {}` yourself. Without it, your app crashes at startup with a confusing error.

> 📝 **Beginner Note — `@Table` naming tips:**
> - Always set `name` explicitly. Don't rely on defaults.
> - Use **lowercase, plural, snake_case** names: `users`, `order_items`, `user_profiles`
> - Avoid reserved SQL words: `user`, `order`, `group`, `table`, `index` — these cause cryptic SQL errors if used as table names

---

### 3.2 @GeneratedValue Strategies

When you use `@Id` to mark a primary key, you usually want the database (or Hibernate) to **auto-generate** the ID for you. The `@GeneratedValue` annotation controls *how* that ID is generated.

> 🧠 **Analogy — Getting a Token Number:**  
> When you walk into a bank, you get a token number. There are different ways the bank can assign it:
> - **Counter machine (IDENTITY):** You take the next number from the counter display — simple and sequential
> - **Pre-printed ticket book (SEQUENCE):** The bank has a book of pre-printed numbers and tears out the next one — faster because they can prepare batches
> - **Random lottery number (UUID):** You get a unique random number — guaranteed unique even across branches
> - **Handwritten ledger (TABLE):** Someone manually writes the next number in a notebook — slow and unreliable

```java
// ── Strategy 1: IDENTITY ──────────────────────────────────────────────
// The database auto-increments: 1, 2, 3, 4...
// Simplest and most common for MySQL
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
// Behind the scenes: INSERT INTO users (name, email) VALUES (?, ?)
// The database assigns the next available ID automatically

// ── Strategy 2: SEQUENCE ──────────────────────────────────────────────
// Uses a database "sequence" object to generate IDs
// Best performance for PostgreSQL and Oracle
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 50)
private Long id;
// allocationSize=50 → Hibernate fetches 50 IDs at once from the database,
// then assigns them in memory without extra DB calls. Much faster for bulk inserts!

// ── Strategy 3: UUID ──────────────────────────────────────────────────
// Generates a universally unique identifier like "550e8400-e29b-41d4-a716-446655440000"
// Best for microservices where multiple services create records independently
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;
// Advantage: No risk of ID collisions across different databases/services
// Disadvantage: Larger column size, slightly slower indexing

// ── Strategy 4: TABLE ─────────────────────────────────────────────────
// Uses a separate database table to track the next ID value
// SLOWEST strategy — requires a separate table query and lock for each ID
@Id
@GeneratedValue(strategy = GenerationType.TABLE)
private Long id;
// Avoid this in production. It exists for compatibility, not performance.
```

#### Which strategy should you use?

| Strategy | Best For | Database | Performance | Trade-off |
|---|---|---|---|---|
| `IDENTITY` | Simple apps, getting started | MySQL, PostgreSQL, SQL Server | Good | Disables Hibernate batch inserts |
| `SEQUENCE` | High-performance apps, bulk operations | PostgreSQL, Oracle | **Best** | Not supported by all databases |
| `UUID` | Microservices, distributed systems | All databases | Good | Larger IDs, bigger indexes |
| `TABLE` | Legacy compatibility only | All databases | **Worst** | Avoid in new projects |

> 💡 **Important to Remember:**  
> - **MySQL users:** Use `IDENTITY` — it's the most natural fit.
> - **PostgreSQL users:** Use `SEQUENCE` — it gives the best performance, especially for batch inserts.
> - **Building microservices?** Use `UUID` — no two services will ever generate the same ID.
> - **Need batch inserts?** Avoid `IDENTITY` — it forces Hibernate to insert one row at a time (because it needs the DB-generated ID back immediately). Use `SEQUENCE` with a high `allocationSize`.

---

### 3.3 Relationships

In real-world applications, tables don't exist in isolation — they're connected. A user has orders. An order has items. An item belongs to a product category. JPA lets you model these connections using relationship annotations.

> 🧠 **Analogy — A Family Tree:**  
> Think of database relationships like family relationships:
> - **@OneToOne** → A person has one passport (and that passport belongs to one person)
> - **@OneToMany** → A mother has many children
> - **@ManyToOne** → Many children have one mother
> - **@ManyToMany** → Many students attend many classes, and each class has many students

#### Key Concepts Before We Start

Before looking at the code, understand these three critical terms:

- **Owner side:** The entity whose table has the **foreign key column**. This is the side that "owns" the relationship in the database.
- **Inverse side:** The other entity. It uses `mappedBy` to say "the relationship is already defined on the other side; don't create another foreign key here."
- **`@JoinColumn`:** Specifies the name of the foreign key column on the owner's table.

> 🧠 **Analogy — Owner vs Inverse:**  
> Think of a **landlord and tenant** relationship. The **lease agreement** (foreign key) is stored in the **tenant's file** (owner side). The landlord (inverse side) knows about the relationship but doesn't keep the lease in their own file — they just reference the tenant's file (`mappedBy`).

---

#### @OneToOne

**Scenario:** Each `User` has exactly one `UserProfile`, and each `UserProfile` belongs to exactly one `User`.

```java
// User.java — OWNER SIDE (the foreign key column lives in the 'users' table)
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    @OneToOne(
        cascade = CascadeType.ALL,      // When you save/delete a User, do the same to its Profile
        fetch = FetchType.LAZY,          // Don't load the profile until you actually access it
        orphanRemoval = true             // If you set profile = null, delete the old Profile from DB
    )
    @JoinColumn(name = "profile_id")    // This creates a "profile_id" column in the 'users' table
    private UserProfile profile;
}
```

```java
// UserProfile.java — INVERSE SIDE (no foreign key here — it points back via mappedBy)
@Entity
@Table(name = "user_profiles")
public class UserProfile {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String bio;
    private String avatarUrl;

    @OneToOne(mappedBy = "profile")     // "profile" = the field name in User class that owns this
    private User user;                  // This does NOT create a column — it's just a Java reference
}
```

**What happens in the database:**
```
users table:           user_profiles table:
+----+-------+------------+    +----+------+
| id | email | profile_id |    | id | bio  |
+----+-------+------------+    +----+------+
|  1 | a@b.c |          1 |    |  1 | Hi!  |
+----+-------+------------+    +----+------+
```
The `profile_id` foreign key lives in the `users` table because `User` is the owner side.

---

#### @OneToMany and @ManyToOne (Most Common Relationship)

**Scenario:** One `Department` has many `Employees`. Each `Employee` belongs to one `Department`.

> 🧠 **Analogy — A Manager and Their Team:**  
> A department manager (Department) manages a team of employees (List<Employee>). The employee's badge (Employee table) has a "department_id" printed on it — that's the foreign key. The manager doesn't carry a list of employee IDs; instead, they know to look for everyone whose badge says their department.

```java
// Department.java — the "ONE" side (one department has many employees)
@Entity
@Table(name = "departments")
public class Department {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(
        mappedBy = "department",        // "department" = the field name in Employee that points back
        cascade = CascadeType.ALL,      // Operations cascade to employees
        fetch = FetchType.LAZY,          // ✅ ALWAYS use LAZY for collections! (see section 3.4)
        orphanRemoval = true             // Removing an employee from the list deletes it from DB
    )
    private List<Employee> employees = new ArrayList<>();  // Initialize to avoid NullPointerException

    // ── Convenience methods ──────────────────────────────────
    // These keep BOTH sides of the relationship in sync.
    // Without them, you'd need to manually set department on the employee AND
    // add the employee to the list — easy to forget and causes bugs!
    public void addEmployee(Employee employee) {
        employees.add(employee);
        employee.setDepartment(this);    // Sync the other side
    }

    public void removeEmployee(Employee employee) {
        employees.remove(employee);
        employee.setDepartment(null);    // Sync the other side
    }
}
```

```java
// Employee.java — the "MANY" side (many employees belong to one department)
// This is the OWNER SIDE — the foreign key column lives here
@Entity
@Table(name = "employees")
public class Employee {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    @ManyToOne(fetch = FetchType.LAZY)  // ✅ Override default EAGER to LAZY!
    @JoinColumn(
        name = "department_id",         // Foreign key column in the 'employees' table
        nullable = false                // Every employee MUST belong to a department
    )
    private Department department;
}
```

**What happens in the database:**
```
departments table:         employees table:
+----+-------------+      +----+-------+---------------+
| id | name        |      | id | name  | department_id |
+----+-------------+      +----+-------+---------------+
|  1 | Engineering |      |  1 | Alice |             1 |
|  2 | Marketing   |      |  2 | Bob   |             1 |
+----+-------------+      |  3 | Carol |             2 |
                           +----+-------+---------------+
```

> 📝 **Beginner Note — Who is the owner in @OneToMany/@ManyToOne?**  
> The **@ManyToOne side is always the owner**. That's because the foreign key naturally lives in the "many" table. Think about it: it makes sense for each employee row to store which department it belongs to — not for the department row to store a list of all employee IDs.

> ⚠️ **Common Mistake — Forgetting sync methods:**  
> If you do `department.getEmployees().add(employee)` but forget `employee.setDepartment(department)`, the relationship might not be saved correctly. Always sync both sides, or use the convenience methods shown above.

---

#### @ManyToMany

**Scenario:** `Students` can enroll in many `Courses`, and each `Course` has many `Students`.

> 🧠 **Analogy — A Library Membership:**  
> Many members can borrow from many libraries, and each library serves many members. Neither the member's card nor the library's system can store this cleanly — you need a **separate sign-up registry** (join table) that records "Member X is registered at Library Y."

A `@ManyToMany` relationship requires a **join table** — a third table that connects the two entities:

```java
// Student.java — OWNER SIDE (defines the join table)
@Entity
@Table(name = "students")
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToMany(
        cascade = {CascadeType.PERSIST, CascadeType.MERGE}, // Save/update cascade — but NOT delete!
        fetch = FetchType.LAZY
    )
    @JoinTable(
        name = "student_courses",                               // Name of the join table
        joinColumns = @JoinColumn(name = "student_id"),         // FK pointing to Student
        inverseJoinColumns = @JoinColumn(name = "course_id")    // FK pointing to Course
    )
    private Set<Course> courses = new HashSet<>();  // Use Set (not List) to avoid duplicates

    // Convenience method — keeps both sides in sync
    public void enrollIn(Course course) {
        courses.add(course);
        course.getStudents().add(this);
    }
}
```

```java
// Course.java — INVERSE SIDE (uses mappedBy, no @JoinTable needed)
@Entity
@Table(name = "courses")
public class Course {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany(mappedBy = "courses", fetch = FetchType.LAZY)  // Points to Student.courses field
    private Set<Student> students = new HashSet<>();
}
```

**What happens in the database:**
```
students table:        student_courses (JOIN TABLE):     courses table:
+----+-------+        +------------+-----------+        +----+---------+
| id | name  |        | student_id | course_id |        | id | title   |
+----+-------+        +------------+-----------+        +----+---------+
|  1 | Alice |        |          1 |         1 |        |  1 | Math    |
|  2 | Bob   |        |          1 |         2 |        |  2 | Science |
+----+-------+        |          2 |         1 |        +----+---------+
                       +------------+-----------+
```
Alice is enrolled in Math and Science. Bob is enrolled in Math. The join table stores these many-to-many connections.

> ⚠️ **Common Mistake — `CascadeType.ALL` on @ManyToMany:**  
> Never use `CascadeType.REMOVE` or `CascadeType.ALL` on a `@ManyToMany` relationship. Why? If you delete Student "Alice", cascade would try to delete Course "Math" — but "Bob" is still enrolled in Math! You'd be deleting shared data. Only use `PERSIST` and `MERGE`.

> 📝 **Beginner Note — Why `Set` instead of `List`?**  
> `Set` prevents duplicate entries (a student can't enroll in the same course twice). With `List`, Hibernate can also generate less efficient SQL for many-to-many deletions. Use `HashSet` as the default implementation.

> 💡 **Important to Remember — When @ManyToMany isn't enough:**  
> If your join table needs extra columns (e.g., `enrollment_date`, `grade`), you can't use `@ManyToMany`. Instead, create a separate `Enrollment` entity with `@ManyToOne` to both `Student` and `Course`. This is common in real projects.

---

### 3.4 Fetch Types

Fetch type controls **when** JPA loads related data from the database. This is one of the most important performance concepts in JPA.

> 🧠 **Analogy — Online Shopping Cart:**  
> Imagine you're browsing an online store:
> - **EAGER loading** = Every time you open a product page, the website also loads all reviews, related products, seller info, warranty details, and shipping options — even if you just wanted to see the price. Slow and wasteful.
> - **LAZY loading** = The product page loads just the basic info (name, price, image). Reviews only load when you scroll down and click "Show Reviews." Fast and efficient.

#### How It Works

```java
// EAGER: Related data is loaded IMMEDIATELY in the same SQL query
// When you load an Employee, their Department is fetched RIGHT AWAY (via SQL JOIN)
@ManyToOne(fetch = FetchType.EAGER)
private Department department;
// SQL: SELECT e.*, d.* FROM employees e JOIN departments d ON e.department_id = d.id

// LAZY: Related data is loaded ONLY when you explicitly access it
// When you load a Department, its employees are NOT fetched until you call getEmployees()
@OneToMany(fetch = FetchType.LAZY)
private List<Employee> employees;
// SQL: SELECT * FROM departments WHERE id = ?   ← only this runs initially
// SQL: SELECT * FROM employees WHERE department_id = ?  ← runs only when you access employees
```

#### Step-by-step: What happens with LAZY loading

1. You call `departmentRepository.findById(1L)` → Hibernate runs `SELECT * FROM departments WHERE id = 1`
2. You get back a `Department` object. The `employees` list is **not loaded yet** — it's a special Hibernate **proxy** (placeholder)
3. You call `department.getEmployees()` → **Now** Hibernate runs `SELECT * FROM employees WHERE department_id = 1`
4. The employees are loaded and returned

This means: if you never call `getEmployees()`, that second SQL query **never runs** — saving time and memory.

#### Default Fetch Types (memorize this!)

| Annotation | Default | Why | Your Action |
|---|---|---|---|
| `@ManyToOne` | `EAGER` ⚠️ | JPA spec assumes loading one parent is cheap | **Override to LAZY!** |
| `@OneToOne` | `EAGER` ⚠️ | JPA spec assumes loading one related item is cheap | **Override to LAZY!** |
| `@OneToMany` | `LAZY` ✅ | Loading a collection could be huge | Keep as LAZY |
| `@ManyToMany` | `LAZY` ✅ | Loading a collection could be huge | Keep as LAZY |

> ⚠️ **Common Mistake — Leaving @ManyToOne as default EAGER:**  
> If an `Employee` has an EAGER `Department`, and a `Department` has an EAGER `Company`, and a `Company` has EAGER `Addresses`... loading one Employee triggers a chain reaction that loads half your database! This is why EAGER loading is the #1 cause of unexpected performance problems.

> ✅ **Best Practice: Always use `FetchType.LAZY` on every relationship.** When you actually need the related data, use `JOIN FETCH` in your JPQL query (covered in Section 5) to load it efficiently in one query.

> 📝 **Beginner Note — LazyInitializationException:**  
> If you load an entity with LAZY relationships, then try to access the lazy field **after the database session is closed** (e.g., outside a `@Transactional` method), you'll get a `LazyInitializationException`. This is one of the most common JPA errors. The fix: load the data you need inside the transaction, either by accessing the field or using `JOIN FETCH`.

---

### 3.5 Cascade Types

Cascade types define what happens to **child entities** when you perform an operation on the **parent entity**.

> 🧠 **Analogy — A Filing Cabinet:**  
> Think of a `Department` as a folder in a filing cabinet. Each `Employee` document is inside that folder.
> - **CascadeType.PERSIST** = When you put the folder in the cabinet (save department), all documents inside are also filed (employees are saved)
> - **CascadeType.REMOVE** = When you throw away the folder (delete department), all documents inside go in the trash too (employees are deleted)
> - **CascadeType.ALL** = Every action you do to the folder happens to all documents inside

#### Cascade Type Reference

| CascadeType | What Happens | Real-World Example |
|---|---|---|
| `PERSIST` | Saving parent also saves new children | Saving an Order also saves its OrderItems |
| `MERGE` | Updating parent also updates changed children | Updating a blog Post also updates edited Comments |
| `REMOVE` | Deleting parent also deletes children | Deleting a Department deletes its Employees |
| `REFRESH` | Reloading parent from DB also reloads children | Refreshing an Order also refreshes its Items |
| `DETACH` | Detaching parent from JPA also detaches children | (Rarely used directly) |
| `ALL` | All of the above combined | Use when parent fully "owns" the children |

#### When to Use Each

```java
// ── @OneToMany: Parent fully owns children ────────────────────────────
// An Order owns its OrderItems. If the order is deleted, the items should go too.
// Using CascadeType.ALL + orphanRemoval = full lifecycle control
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items = new ArrayList<>();
// ✅ Save order → items are saved
// ✅ Delete order → items are deleted
// ✅ Remove item from list → item is deleted from DB (orphanRemoval)

// ── @ManyToMany: Shared entities — BE CAREFUL ────────────────────────
// A Student enrolls in Courses. Deleting a Student should NOT delete Courses!
// Only cascade PERSIST and MERGE — NEVER REMOVE
@ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
private Set<Course> courses = new HashSet<>();
// ✅ Save student with new courses → courses are saved
// ✅ Update student → course changes are merged
// ❌ Delete student → courses are NOT deleted (correct behavior!)
```

#### What is `orphanRemoval`?

`orphanRemoval = true` means: if a child entity is **removed from the parent's collection**, delete it from the database. This is different from `CascadeType.REMOVE`, which only deletes children when the **parent itself** is deleted.

```java
// With orphanRemoval = true:
department.getEmployees().remove(employee);  // Remove from list
// → Hibernate automatically runs: DELETE FROM employees WHERE id = ?

// Without orphanRemoval (only cascade REMOVE):
department.getEmployees().remove(employee);  // Remove from list
// → The employee still exists in the database! It's just no longer in the Java list.
// → This creates an "orphan" — a record with no parent.
```

> ⚠️ **Common Mistake — `CascadeType.ALL` on @ManyToMany:**  
> This is a **dangerous mistake** beginners often make. If Student A and Student B are both enrolled in Course "Math", and you delete Student A with `CascadeType.ALL`, Hibernate will try to **delete the Math course** — which breaks Student B's enrollment! Only use `PERSIST` and `MERGE` on `@ManyToMany`.

> 📝 **Beginner Note — When to use `CascadeType.ALL`:**  
> Use it only when the parent **completely owns** the children and the children **can't exist without the parent**. Examples: Order → OrderItems, Invoice → InvoiceLines, Blog Post → Comments. Don't use it when entities are shared (like Courses that many Students use).

---

## 4. Repository Layer

> This section covers how Spring Data JPA lets you talk to the database without writing any implementation code. You just declare an interface, and Spring generates all the CRUD operations, queries, pagination, and sorting for you at runtime.

### 4.1 Repository Hierarchy — Understanding the Family Tree

#### What is a Repository?

A **repository** is a Java **interface** that acts as a bridge between your service/business logic and the database. Instead of writing SQL or managing `EntityManager` yourself, you declare a repository interface and Spring Data JPA **auto-generates the implementation** at startup.

> 🧠 **Analogy — A Bank Teller Window:**  
> Imagine a bank. You (your service class) don't walk into the vault to get money yourself. You go to a **teller window** (the repository). You say "I need the account with ID 42" and the teller handles everything — finding the record, returning the result. The repository is that teller window for your database.

Spring Data JPA provides a **hierarchy** of repository interfaces, each one adding more features on top of the previous one:

```
Repository (marker interface — empty, does nothing by itself)
    └── CrudRepository<T, ID>                          → Basic CRUD (save, find, delete, count)
            └── ListCrudRepository<T, ID>              → Same as CrudRepository but returns List instead of Iterable
            └── PagingAndSortingRepository<T, ID>      → + Pagination & Sorting
                    └── JpaRepository<T, ID>           → + flush(), batch ops, JPA-specific extras
```

#### What does each level give you?

| Interface | What It Adds | When to Use |
|---|---|---|
| `CrudRepository` | `save()`, `findById()`, `findAll()`, `delete()`, `count()`, `existsById()` | When you only need basic CRUD — no pagination, no sorting |
| `PagingAndSortingRepository` | `findAll(Pageable)`, `findAll(Sort)` | When you need paginated or sorted results |
| `JpaRepository` | `flush()`, `saveAllAndFlush()`, `deleteInBatch()`, `getReferenceById()` | **Most common choice** — gives you everything above + JPA-specific optimizations |

> 📝 **Beginner Note:** In almost every real-world project, you'll extend `JpaRepository`. It includes everything from `CrudRepository` and `PagingAndSortingRepository`, plus extra JPA features. There's rarely a reason to use the others directly.

> 💡 **Important to Remember:** You **never write a class that implements** these interfaces. Spring Data JPA creates the implementation **automatically at runtime** using proxies. You just declare the interface — that's it!

---

### 4.2 Creating a Repository

#### What is this?
A repository interface is a plain Java interface that extends `JpaRepository`. By doing this, you instantly get dozens of ready-to-use database methods — without writing a single line of implementation code.

#### Why does it exist?
Without repositories, you'd have to write repetitive `EntityManager` code for every entity — `persist()`, `find()`, `remove()`, manual transaction handling, etc. Repositories eliminate all of that boilerplate.

> 🧠 **Analogy — A TV Remote:**  
> Think of `JpaRepository` as a universal TV remote that comes with **all the standard buttons pre-programmed** — power, volume, channel up/down, mute. You don't have to build these buttons yourself. You just pick up the remote (extend the interface) and start pressing buttons (calling methods). If you need a special button (a custom query), you can add it — but the basics are already there.

```java
// File: src/main/java/com/example/repository/UserRepository.java

package com.example.repository;

import com.example.entity.User;
import com.example.entity.UserStatus;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;
import java.util.List;
import java.util.Optional;

@Repository // Optional — Spring auto-detects JpaRepository interfaces, but this makes it explicit
public interface UserRepository extends JpaRepository<User, Long> {
    //                                                  ^^^^  ^^^^
    //                                               Entity  Primary Key
    //                                               Type    Type
    
    // That's it! No implementation class needed.
    // Spring auto-generates all the code behind the scenes.
}
```

#### What happens behind the scenes when Spring starts up?

1. Spring Boot scans your project for interfaces that extend `JpaRepository`
2. For each one, it **dynamically creates a class** (using a Java proxy) that implements all the methods
3. It registers that class as a Spring Bean in the application context
4. When your service class injects `UserRepository`, it gets the auto-generated implementation

You never see this generated class — it happens at runtime. But it's a real, fully functional implementation.

#### Built-in Methods You Get for FREE

Once you extend `JpaRepository<User, Long>`, you can immediately use all of these methods without writing any code:

```java
// ─── CREATE / UPDATE ──────────────────────────────────────────────────
userRepository.save(user);                    // Insert new OR update existing user
userRepository.saveAll(listOfUsers);          // Save multiple users at once
userRepository.saveAndFlush(user);            // Save and immediately push to DB (not wait for transaction end)

// ─── READ ─────────────────────────────────────────────────────────────
userRepository.findById(1L);                  // Returns Optional<User> — might be empty!
userRepository.findAll();                     // Returns List<User> — all users in the table
userRepository.findAllById(List.of(1L, 2L));  // Find multiple users by their IDs
userRepository.getReferenceById(1L);          // Returns a lazy proxy (doesn't hit DB until you access a field)

// ─── DELETE ───────────────────────────────────────────────────────────
userRepository.deleteById(1L);                // Delete user with ID 1
userRepository.delete(user);                  // Delete this specific user object
userRepository.deleteAll();                   // Delete ALL users (be careful!)
userRepository.deleteAllInBatch();            // Faster bulk delete (single DELETE SQL, no entity loading)

// ─── UTILITY ──────────────────────────────────────────────────────────
userRepository.count();                       // Total number of users in the table
userRepository.existsById(1L);                // Does a user with ID 1 exist? (returns boolean)

// ─── PAGINATION & SORTING ─────────────────────────────────────────────
userRepository.findAll(PageRequest.of(0, 10));            // First page, 10 records per page
userRepository.findAll(Sort.by("firstName").ascending()); // All users sorted by name A→Z
```

> 📝 **Beginner Note — `save()` does double duty!**  
> `save()` is smart. If the entity's `@Id` field is `null` (new entity), it runs an `INSERT`. If the `@Id` already has a value (existing entity), it runs an `UPDATE`. You don't need separate "create" and "update" methods!

> ⚠️ **Common Mistake — `findById()` returns `Optional`, not the entity directly!**  
> You can't do `User user = userRepository.findById(1L);` — that won't compile. You need to handle the possibility that no user exists:
> ```java
> // ✅ Correct — handle the Optional
> User user = userRepository.findById(1L)
>     .orElseThrow(() -> new RuntimeException("User not found with ID: 1"));
> ```

> 💡 **Important to Remember — `deleteById()` vs `deleteAllInBatch()`:**  
> - `deleteById()` first loads the entity, then deletes it (2 SQL queries). This is needed if you have cascade/lifecycle callbacks.
> - `deleteAllInBatch()` runs a single `DELETE FROM users` SQL — much faster for bulk operations, but skips cascade and lifecycle callbacks.

---

### 4.3 Query Methods by Naming Convention (Derived Queries)

#### What is this?

Spring Data JPA can **read your method name** and automatically generate the SQL query from it. You just write a method signature following a naming convention, and Spring figures out the WHERE clause, ORDER BY, LIMIT, etc.

#### Why does it exist?

For simple queries like "find users by email" or "find active users sorted by name," writing full `@Query` annotations or SQL is overkill. Derived query methods let you express these in one line — the method name IS the query.

#### What problem does it solve?

Without this feature, you'd need to write a `@Query` annotation (or native SQL) for every single query, even trivial ones. Derived queries handle 70–80% of simple queries with zero SQL.

> 🧠 **Analogy — Voice Commands:**  
> Think of it like a smart assistant. You don't say "Execute SQL: SELECT star FROM users WHERE email equals john@gmail.com." You just say **"Find by email: john@gmail.com"** and the assistant knows exactly what to do. Spring Data JPA works the same way — your method name is the voice command, and Spring translates it into the right SQL.

#### How does Spring parse the method name?

Spring breaks your method name into parts and maps each part to SQL:

```
findByFirstNameAndStatusOrderByCreatedAtDesc
│     │         │  │      │       │         │
│     │         │  │      │       │         └── DESC = descending order
│     │         │  │      │       └── CreatedAt = ORDER BY created_at
│     │         │  │      └── OrderBy = ORDER BY clause
│     │         │  └── Status = AND status = ?
│     │         └── And = AND keyword
│     └── FirstName = WHERE first_name = ?
└── findBy = SELECT ... WHERE ...

Generated SQL: SELECT * FROM users WHERE first_name = ? AND status = ? ORDER BY created_at DESC
```

#### Step-by-Step: What happens when you call a derived query method

1. At **application startup**, Spring reads every method in your repository interface
2. For methods that follow the naming convention, it **parses the method name** into a query tree
3. It validates that the field names in the method name actually exist on your entity class
4. At **runtime**, when your code calls the method, Spring generates the SQL, sets the parameters, executes it, and maps the results back to Java objects

> 📝 **Beginner Note — Validation at startup:**  
> If you misspell a field name (e.g., `findByEmali` instead of `findByEmail`), your application **won't start**. Spring checks at startup and throws an error like: *"No property 'emali' found for type 'User'."* This is a good thing — it catches bugs early!

#### Complete Example with All Common Patterns

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ═══════════════════════════════════════════════════════════
    // FIND BY SINGLE FIELD
    // ═══════════════════════════════════════════════════════════
    
    // Find one user by their email address
    // Returns Optional because there might be no user with that email
    Optional<User> findByEmail(String email);
    // SQL: SELECT * FROM users WHERE email = ?

    // Find all users with a specific status
    // Returns a List because multiple users can have the same status
    List<User> findByStatus(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ?

    // ═══════════════════════════════════════════════════════════
    // FIND BY MULTIPLE FIELDS (AND / OR)
    // ═══════════════════════════════════════════════════════════
    
    // Both conditions must be true (AND)
    Optional<User> findByEmailAndStatus(String email, UserStatus status);
    // SQL: SELECT * FROM users WHERE email = ? AND status = ?

    // Either condition can be true (OR)
    List<User> findByFirstNameOrLastName(String firstName, String lastName);
    // SQL: SELECT * FROM users WHERE first_name = ? OR last_name = ?

    // ═══════════════════════════════════════════════════════════
    // COMPARISON OPERATORS — Numbers and Dates
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByAgeLessThan(int age);                   // WHERE age < ?
    List<User> findByAgeGreaterThanEqual(int age);           // WHERE age >= ?
    List<User> findByAgeBetween(int min, int max);           // WHERE age BETWEEN ? AND ?
    List<User> findByCreatedAtAfter(LocalDateTime date);     // WHERE created_at > ?
    List<User> findByCreatedAtBefore(LocalDateTime date);    // WHERE created_at < ?

    // ═══════════════════════════════════════════════════════════
    // STRING MATCHING — Searching Text
    // ═══════════════════════════════════════════════════════════
    
    // Contains — matches anywhere in the string (most common for search)
    List<User> findByFirstNameContaining(String keyword);
    // SQL: WHERE first_name LIKE '%keyword%'

    // Starts with — matches beginning of string
    List<User> findByEmailStartingWith(String prefix);
    // SQL: WHERE email LIKE 'prefix%'

    // Ends with — matches end of string
    List<User> findByLastNameEndingWith(String suffix);
    // SQL: WHERE last_name LIKE '%suffix'

    // Case-insensitive — "JOHN", "john", "John" all match
    List<User> findByFirstNameIgnoreCase(String name);
    // SQL: WHERE LOWER(first_name) = LOWER(?)

    // ═══════════════════════════════════════════════════════════
    // NULL CHECKS
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByBioIsNull();          // WHERE bio IS NULL     (users who haven't written a bio)
    List<User> findByBioIsNotNull();       // WHERE bio IS NOT NULL (users who have written a bio)

    // ═══════════════════════════════════════════════════════════
    // BOOLEAN CHECKS
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByVerifiedTrue();       // WHERE verified = true
    List<User> findByVerifiedFalse();      // WHERE verified = false

    // ═══════════════════════════════════════════════════════════
    // COLLECTION CHECKS — IN
    // ═══════════════════════════════════════════════════════════
    
    // Find users whose status is in a given list
    List<User> findByStatusIn(List<UserStatus> statuses);
    // SQL: WHERE status IN ('ACTIVE', 'PENDING_VERIFICATION')

    List<User> findByStatusNotIn(List<UserStatus> statuses);
    // SQL: WHERE status NOT IN (...)

    // ═══════════════════════════════════════════════════════════
    // ORDERING — Sort Results
    // ═══════════════════════════════════════════════════════════
    
    List<User> findByStatusOrderByCreatedAtDesc(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ? ORDER BY created_at DESC
    // "Desc" = newest first. Use "Asc" for oldest first.

    List<User> findByStatusOrderByLastNameAscFirstNameAsc(UserStatus status);
    // SQL: ... ORDER BY last_name ASC, first_name ASC (multi-column sort)

    // ═══════════════════════════════════════════════════════════
    // LIMITING RESULTS — Top / First
    // ═══════════════════════════════════════════════════════════
    
    // Get only the most recent user (single result)
    User findFirstByOrderByCreatedAtDesc();
    // SQL: SELECT * FROM users ORDER BY created_at DESC LIMIT 1

    // Get the 5 most recently created active users
    List<User> findTop5ByStatusOrderByCreatedAtDesc(UserStatus status);
    // SQL: SELECT * FROM users WHERE status = ? ORDER BY created_at DESC LIMIT 5

    // ═══════════════════════════════════════════════════════════
    // COUNT AND EXISTENCE — Quick Checks
    // ═══════════════════════════════════════════════════════════
    
    // How many active users are there?
    long countByStatus(UserStatus status);
    // SQL: SELECT COUNT(*) FROM users WHERE status = ?

    // Does a user with this email exist? (returns true/false — very fast)
    boolean existsByEmail(String email);
    // SQL: SELECT CASE WHEN COUNT(*) > 0 THEN true ELSE false END FROM users WHERE email = ?

    // ═══════════════════════════════════════════════════════════
    // DELETE — Remove by Condition
    // ═══════════════════════════════════════════════════════════
    
    void deleteByStatus(UserStatus status);
    // SQL: DELETE FROM users WHERE status = ?

    long deleteByEmail(String email);
    // Returns count of deleted rows — useful for confirming deletion

    // ═══════════════════════════════════════════════════════════
    // NESTED PROPERTY ACCESS — Query Through Relationships
    // ═══════════════════════════════════════════════════════════
    
    // If User has a @ManyToOne Department, you can query through it!
    // Spring automatically joins the tables for you.
    List<User> findByDepartmentName(String departmentName);
    // SQL: SELECT u.* FROM users u JOIN departments d ON u.department_id = d.id WHERE d.name = ?
}
```

> ⚠️ **Common Mistake — Ambiguous property paths:**  
> If your `User` entity has both a field `departmentName` AND a relationship `department` with a `name` field, Spring gets confused. To be explicit, use an underscore: `findByDepartment_Name` (means `department.name`) vs `findByDepartmentName` (could be either). In practice, avoid naming fields that collide with relationship paths.

> 📝 **Beginner Note — When do derived queries become impractical?**  
> Once your method name gets very long (e.g., `findByStatusAndDepartmentNameAndCreatedAtAfterOrderByLastNameAsc`), it becomes hard to read. At that point, switch to `@Query` with JPQL (covered in Section 5). A good rule of thumb: if the method name has more than 3–4 keywords, use `@Query` instead.

#### Keyword Quick-Reference Table

| Keyword | SQL Equivalent | Example Method |
|---|---|---|
| `And` | `AND` | `findByFirstNameAndLastName` |
| `Or` | `OR` | `findByStatusOrEmail` |
| `Is`, `Equals` | `=` | `findByStatusIs` / `findByStatusEquals` |
| `Not` | `!=` | `findByStatusNot` |
| `Between` | `BETWEEN` | `findByAgeBetween` |
| `LessThan` | `<` | `findByAgeLessThan` |
| `LessThanEqual` | `<=` | `findByAgeLessThanEqual` |
| `GreaterThan` | `>` | `findByAgeGreaterThan` |
| `GreaterThanEqual` | `>=` | `findByAgeGreaterThanEqual` |
| `After` | `>` (for dates) | `findByCreatedAtAfter` |
| `Before` | `<` (for dates) | `findByCreatedAtBefore` |
| `IsNull` | `IS NULL` | `findByEmailIsNull` |
| `IsNotNull` / `NotNull` | `IS NOT NULL` | `findByEmailIsNotNull` |
| `Like` | `LIKE` (you provide the `%`) | `findByNameLike("%john%")` |
| `Containing` | `LIKE %x%` (auto-wraps with `%`) | `findByNameContaining("john")` |
| `StartingWith` | `LIKE x%` | `findByNameStartingWith("J")` |
| `EndingWith` | `LIKE %x` | `findByNameEndingWith("son")` |
| `In` | `IN (...)` | `findByStatusIn(List.of(ACTIVE, PENDING))` |
| `NotIn` | `NOT IN (...)` | `findByStatusNotIn(...)` |
| `True` / `False` | `= true` / `= false` | `findByVerifiedTrue` |
| `IgnoreCase` | `LOWER()` comparison | `findByEmailIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByStatusOrderByCreatedAtDesc` |
| `Top` / `First` | `LIMIT` | `findTop10ByStatus` / `findFirstByOrderByIdAsc` |

---

### 4.4 Pagination & Sorting

#### What is Pagination?

When your database has thousands (or millions) of records, loading all of them at once is **slow**, **memory-intensive**, and **bad UX**. Pagination lets you load data in small chunks — one "page" at a time.

#### What is Sorting?

Sorting lets you control the **order** in which results are returned — newest first, alphabetical, highest price, etc.

> 🧠 **Analogy — Google Search Results:**  
> When you search on Google, it doesn't dump all 10 million results on your screen. It shows you **10 results per page** (that's pagination), sorted by **relevance** (that's sorting). You click "Next" to see the next page. That's exactly what `Pageable` does for your database queries.

#### Key Concepts

| Concept | What It Is |
|---|---|
| `Pageable` | An object that holds the **page number**, **page size**, and **sort order** — you pass it to repository methods |
| `PageRequest` | The most common way to **create** a `Pageable` instance |
| `Page<T>` | The result object — contains the data **plus** metadata (total pages, total elements, etc.) |
| `Slice<T>` | A lighter result object — contains the data and knows if there's a **next page**, but does NOT count total records (faster!) |
| `Sort` | Defines how to order results — by which field(s), ascending or descending |

> 📝 **Beginner Note — Page vs Slice:**  
> - `Page` runs **two SQL queries**: one for the data, one for `COUNT(*)` to calculate total pages/elements. Use this when you need to show "Page 3 of 15" in the UI.
> - `Slice` runs only **one SQL query** (fetches one extra record to check if there's a next page). Use this for "Load More" buttons or infinite scroll — faster because it skips the count.

#### Step-by-Step Example: Repository → Service → Controller

**Step 1: Repository — Accept `Pageable` as a parameter**

Just add a `Pageable` parameter to any query method. Spring Data JPA handles the rest:

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Paginated query — returns a Page with metadata (total count, total pages, etc.)
    Page<User> findByStatus(UserStatus status, Pageable pageable);

    // Slice — lighter alternative, no count query (good for "Load More" UIs)
    Slice<User> findByDepartmentId(Long deptId, Pageable pageable);

    // You can also paginate custom @Query methods:
    @Query("SELECT u FROM User u WHERE u.firstName LIKE %:name%")
    Page<User> searchByName(@Param("name") String name, Pageable pageable);
}
```

**Step 2: Service — Build the `Pageable` object**

The service layer creates a `Pageable` using `PageRequest.of()`:

```java
import org.springframework.data.domain.Page;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    // Simple pagination — no sorting
    public Page<User> getActiveUsers(int page, int size) {
        // page = 0-indexed! Page 0 = first page, Page 1 = second page, etc.
        // size = number of records per page
        Pageable pageable = PageRequest.of(page, size);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }

    // Pagination WITH sorting
    public Page<User> getActiveUsersSorted(int page, int size) {
        // Sort by createdAt descending (newest first), then by firstName ascending (A→Z)
        Sort sort = Sort.by(Sort.Direction.DESC, "createdAt")
                        .and(Sort.by(Sort.Direction.ASC, "firstName"));

        Pageable pageable = PageRequest.of(page, size, sort);
        return userRepository.findByStatus(UserStatus.ACTIVE, pageable);
    }
}
```

> ⚠️ **Common Mistake — Page numbers are 0-indexed!**  
> `PageRequest.of(0, 10)` = first page (records 1–10). `PageRequest.of(1, 10)` = second page (records 11–20). If your frontend sends `page=1` meaning "first page," you'll need to subtract 1!

**Step 3: Controller — Expose pagination via REST API**

The controller receives pagination parameters from the client (query params) and passes them to the service:

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {

    private final UserService userService;

    // GET /api/users?page=0&size=10&sortBy=createdAt&direction=desc
    @GetMapping
    public ResponseEntity<Page<UserDto>> getUsers(
            @RequestParam(defaultValue = "0") int page,       // Default: first page
            @RequestParam(defaultValue = "10") int size,      // Default: 10 per page
            @RequestParam(defaultValue = "createdAt") String sortBy,   // Default: sort by creation date
            @RequestParam(defaultValue = "desc") String direction) {   // Default: newest first

        // Build Sort object from query params
        Sort sort = direction.equalsIgnoreCase("desc")
                ? Sort.by(sortBy).descending()
                : Sort.by(sortBy).ascending();

        Pageable pageable = PageRequest.of(page, size, sort);
        Page<User> users = userService.findAll(pageable);

        // Convert entities to DTOs (never expose entities directly in API!)
        Page<UserDto> dtoPage = users.map(UserMapper::toDto);
        return ResponseEntity.ok(dtoPage);
    }
}
```

#### What does the `Page` response look like?

When your REST API returns a `Page`, Spring automatically serializes it to JSON with this structure:

```json
{
  "content": [                   // ← The actual data (list of users on this page)
    { "id": 1, "firstName": "Alice", "email": "alice@example.com" },
    { "id": 2, "firstName": "Bob", "email": "bob@example.com" }
  ],
  "totalElements": 150,          // ← Total records matching the query in the entire DB
  "totalPages": 15,              // ← Total number of pages (totalElements ÷ size)
  "number": 0,                   // ← Current page number (0-indexed)
  "size": 10,                    // ← Page size (records per page)
  "first": true,                 // ← Is this the first page?
  "last": false,                 // ← Is this the last page?
  "numberOfElements": 10,        // ← How many elements are on THIS page
  "empty": false                 // ← Is this page empty?
}
```

> 📝 **Beginner Note — Using Sort with multiple fields:**  
> You can sort by multiple fields. For example, to sort by status first, then by name:
> ```java
> Sort sort = Sort.by("status").ascending()
>                 .and(Sort.by("firstName").ascending());
> ```
> This produces: `ORDER BY status ASC, first_name ASC`

> 💡 **Important to Remember:** You can add `Pageable` as a parameter to **any** repository method — built-in methods, derived query methods, and even `@Query` methods. Spring Data JPA handles the `LIMIT` and `OFFSET` SQL for you.

---

## 5. Query Mechanisms

> When derived query methods (Section 4.3) aren't enough — maybe the query is too complex, requires joins, or needs database-specific SQL — Spring Data JPA gives you multiple ways to write custom queries. This section covers all of them, from simplest to most advanced.

### 5.1 JPQL — Java Persistence Query Language

#### What is JPQL?

**JPQL** is a query language that looks like SQL, but instead of using **table names and column names** (database world), it uses **Java class names and field names** (Java world). Hibernate then translates your JPQL into the correct SQL for your database.

#### Why does JPQL exist?

Regular SQL is tied to a specific database structure — you reference tables like `users` and columns like `first_name`. But what if you rename a table? Or switch databases? JPQL uses your **Java class and field names**, so your queries stay in sync with your code. If you rename a field in your entity, you know exactly which queries to update.

#### What problem does it solve?

Derived query methods (Section 4.3) work great for simple queries, but they can't handle:
- Queries with JOINs across multiple entities
- Aggregate functions (COUNT, SUM, AVG)
- Subqueries or complex WHERE clauses
- UPDATE or DELETE operations on multiple rows at once
- Queries that return partial data (DTOs/projections)

JPQL handles all of these while staying database-independent.

> 🧠 **Analogy — Ordering Food in Your Language:**  
> Imagine you're at a restaurant in a foreign country. You could try to read the menu in the local language (raw SQL — tables, columns). Or you could use a **translator** who lets you order in your own language (JPQL — classes, fields). You say "I want the User where email is X" and the translator (Hibernate) converts it to the local language (SQL) that the kitchen (database) understands.

#### How JPQL Differs from SQL — Side by Side

```
JPQL:  SELECT u FROM User u WHERE u.email = :email
                    ^^^^      ^^^^^^^
                    Java       Java
                    class      field
                    name       name

SQL:   SELECT * FROM users WHERE email = ?
                     ^^^^^       ^^^^^
                     table       column
                     name        name
```

Key differences:
- `User` (capital U) = the Java class, NOT the `users` table
- `u.email` = the Java field, NOT the `email` column (though they often match)
- `:email` = a **named parameter** (safer than string concatenation — prevents SQL injection!)

#### Step-by-Step: How a JPQL Query Runs

1. You write `@Query("SELECT u FROM User u WHERE u.email = :email")` on a repository method
2. At **startup**, Spring validates the JPQL syntax and checks that `User` class and `email` field exist
3. At **runtime**, when you call the method, Hibernate:
   - Reads the JPQL string
   - Looks at the `User` entity's `@Table` and `@Column` annotations
   - Translates JPQL → SQL: `SELECT u.id, u.first_name, u.email, ... FROM users u WHERE u.email = ?`
   - Binds the parameter values safely (preventing SQL injection)
   - Executes the SQL and maps the results back to `User` objects

#### Complete JPQL Examples — From Basic to Advanced

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ═══════════════════════════════════════════════════════════
    // BASIC SELECT WITH PARAMETERS
    // ═══════════════════════════════════════════════════════════

    // Find a user by email AND status using named parameters
    // :email and :status are placeholders — Spring fills them with the method arguments
    @Query("SELECT u FROM User u WHERE u.email = :email AND u.status = :status")
    Optional<User> findActiveByEmail(
            @Param("email") String email,         // @Param binds this argument to :email
            @Param("status") UserStatus status    // @Param binds this argument to :status
    );
    // Generated SQL: SELECT * FROM users WHERE email = ? AND status = ?

    // ═══════════════════════════════════════════════════════════
    // JOIN — Query Across Related Entities
    // ═══════════════════════════════════════════════════════════

    // Find all users in a specific department
    // "JOIN u.department d" = join using the @ManyToOne relationship defined in User class
    @Query("SELECT u FROM User u JOIN u.department d WHERE d.name = :deptName")
    List<User> findByDepartmentName(@Param("deptName") String deptName);
    // Generated SQL: SELECT u.* FROM users u 
    //                JOIN departments d ON u.department_id = d.id 
    //                WHERE d.name = ?

    // ═══════════════════════════════════════════════════════════
    // JOIN FETCH — Solve the N+1 Problem
    // ═══════════════════════════════════════════════════════════

    // Load users AND their departments in ONE query (not N+1 separate queries)
    // JOIN FETCH tells Hibernate: "Load the department data NOW, not later"
    @Query("SELECT u FROM User u JOIN FETCH u.department WHERE u.status = :status")
    List<User> findWithDepartmentByStatus(@Param("status") UserStatus status);
    // Generated SQL: SELECT u.*, d.* FROM users u 
    //                JOIN departments d ON u.department_id = d.id 
    //                WHERE u.status = ?
    // ✅ One query loads BOTH users and departments — no N+1!

    // ═══════════════════════════════════════════════════════════
    // LIKE — Case-Insensitive Search
    // ═══════════════════════════════════════════════════════════

    // Search for users by name (case-insensitive, partial match)
    // LOWER() converts both sides to lowercase for case-insensitive comparison
    // CONCAT('%', :name, '%') wraps the search term in wildcards
    @Query("SELECT u FROM User u WHERE LOWER(u.firstName) LIKE LOWER(CONCAT('%', :name, '%'))")
    List<User> searchByName(@Param("name") String name);
    // If name = "john", this matches: "John", "JOHN", "johnny", "Johnson"

    // ═══════════════════════════════════════════════════════════
    // AGGREGATE FUNCTIONS — Count, Sum, Avg, Min, Max
    // ═══════════════════════════════════════════════════════════

    // Count how many users have a specific status
    @Query("SELECT COUNT(u) FROM User u WHERE u.status = :status")
    long countByStatusJpql(@Param("status") UserStatus status);
    // Generated SQL: SELECT COUNT(*) FROM users WHERE status = ?

    // ═══════════════════════════════════════════════════════════
    // DTO PROJECTION — Return Only Specific Fields
    // ═══════════════════════════════════════════════════════════

    // Instead of loading all 20+ fields of User, load only id, firstName, and email
    // "SELECT new com.example.dto.UserSummaryDto(...)" calls the DTO's constructor
    @Query("SELECT new com.example.dto.UserSummaryDto(u.id, u.firstName, u.email) " +
           "FROM User u WHERE u.status = 'ACTIVE'")
    List<UserSummaryDto> findUserSummaries();
    // Generated SQL: SELECT u.id, u.first_name, u.email FROM users u WHERE u.status = 'ACTIVE'
    // ✅ Much faster than loading full User entities when you only need a few fields!

    // ═══════════════════════════════════════════════════════════
    // MODIFYING QUERIES — UPDATE and DELETE Multiple Rows
    // ═══════════════════════════════════════════════════════════

    // Bulk update — change status of all users matching a condition
    // @Modifying tells Spring this is NOT a SELECT — it changes data
    // @Transactional is required for any data modification
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.status = :newStatus WHERE u.status = :oldStatus")
    int updateStatus(
            @Param("oldStatus") UserStatus oldStatus,
            @Param("newStatus") UserStatus newStatus
    );
    // Returns int = number of rows affected
    // Generated SQL: UPDATE users SET status = ? WHERE status = ?

    // Bulk delete — remove old inactive users
    @Modifying
    @Transactional
    @Query("DELETE FROM User u WHERE u.status = :status AND u.createdAt < :cutoff")
    int deleteInactiveUsers(
            @Param("status") UserStatus status,
            @Param("cutoff") LocalDateTime cutoff
    );
    // Returns int = number of rows deleted
}
```

> ⚠️ **Common Mistake — Forgetting `@Modifying` on UPDATE/DELETE queries:**  
> If you write an UPDATE or DELETE `@Query` without `@Modifying`, Spring will try to run it as a SELECT and you'll get a confusing error. Always pair `@Modifying` + `@Transactional` with any query that changes data.

> ⚠️ **Common Mistake — `@Modifying` and the persistence context:**  
> After a `@Modifying` query runs, the entities already loaded in your current transaction are **stale** — they still have the old values in memory. Add `@Modifying(clearAutomatically = true)` to clear the persistence context after the query, so the next read fetches fresh data from the database.

> 📝 **Beginner Note — Named parameters (`:email`) vs Positional parameters (`?1`):**  
> You can use either:
> ```java
> // Named parameter — recommended (more readable, order doesn't matter)
> @Query("SELECT u FROM User u WHERE u.email = :email")
> User findByEmail(@Param("email") String email);
> 
> // Positional parameter — shorter but fragile (order matters!)
> @Query("SELECT u FROM User u WHERE u.email = ?1")
> User findByEmail(String email);   // ?1 = first method parameter
> ```
> Named parameters are recommended because they're self-documenting and don't break if you reorder method parameters.

> 💡 **Important to Remember — JPQL vs SQL quick comparison:**
> | | JPQL | SQL |
> |---|---|---|
> | References | Java class/field names | Table/column names |
> | Database-independent? | ✅ Yes | ❌ No |
> | Validated at startup? | ✅ Yes | ❌ No |
> | Supports DB-specific features? | ❌ No | ✅ Yes |
> | Use when | Most queries | DB-specific features needed |

---

### 5.2 Native SQL Queries

#### What are Native Queries?

Native queries let you write **raw SQL** — the exact same SQL you'd run directly in your database console. Spring Data JPA passes it straight to the database without any translation.

#### Why would you use native SQL instead of JPQL?

JPQL is great, but it can't do everything. Some database features are only available in native SQL:

- **Window functions:** `ROW_NUMBER()`, `RANK()`, `LAG()`, `LEAD()`
- **Common Table Expressions (CTEs):** `WITH ... AS (...)`
- **Database-specific functions:** MySQL's `GROUP_CONCAT()`, PostgreSQL's `JSONB` operators
- **Complex subqueries** that JPQL doesn't support
- **Performance-critical queries** where you need exact control over the SQL

> 🧠 **Analogy — Speaking Directly to the Chef:**  
> JPQL is like telling the waiter your order in English (the waiter translates it for the chef). Native SQL is like **walking into the kitchen and telling the chef exactly what to cook, in their language**. You get full control, but you need to know the kitchen's language (SQL dialect), and your instructions won't work in a different kitchen (different database) without changes.

#### When to Use Native SQL vs JPQL

| Scenario | Use |
|---|---|
| Simple CRUD, basic JOINs, standard filtering | JPQL (or derived methods) |
| Window functions, CTEs, database-specific syntax | **Native SQL** |
| Need to be database-independent | JPQL |
| Migrating existing SQL queries into Spring Data | **Native SQL** |
| Complex reporting queries | **Native SQL** |

#### Examples

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ═══════════════════════════════════════════════════════════
    // BASIC NATIVE QUERY
    // ═══════════════════════════════════════════════════════════

    // nativeQuery = true tells Spring: "This is raw SQL, don't treat it as JPQL"
    // Notice: we use the TABLE name "users" and COLUMN name "email" (not Java class/field names)
    @Query(
        value = "SELECT * FROM users WHERE email = :email",
        nativeQuery = true
    )
    Optional<User> findByEmailNative(@Param("email") String email);

    // ═══════════════════════════════════════════════════════════
    // NATIVE QUERY WITH PAGINATION
    // ═══════════════════════════════════════════════════════════

    // For pagination with native queries, you MUST provide a separate countQuery
    // Why? Spring needs to know the total record count to build the Page object,
    // but it can't automatically derive a count query from complex native SQL
    @Query(
        value = "SELECT u.*, d.name as dept_name " +
                "FROM users u JOIN departments d ON u.department_id = d.id " +
                "WHERE u.status = :status",
        countQuery = "SELECT COUNT(*) FROM users WHERE status = :status",
        nativeQuery = true
    )
    Page<Object[]> findUsersWithDept(@Param("status") String status, Pageable pageable);

    // ═══════════════════════════════════════════════════════════
    // COMPLEX NATIVE QUERY — Window Functions (impossible in JPQL)
    // ═══════════════════════════════════════════════════════════

    // Find the most recently created active user in each department
    // This uses ROW_NUMBER() — a window function that JPQL doesn't support
    @Query(
        value = """
            WITH ranked_users AS (
                SELECT u.*, 
                       ROW_NUMBER() OVER (
                           PARTITION BY u.department_id 
                           ORDER BY u.created_at DESC
                       ) as rn
                FROM users u
                WHERE u.status = 'ACTIVE'
            )
            SELECT * FROM ranked_users WHERE rn = 1
            """,
        nativeQuery = true
    )
    List<Object[]> findLatestUserPerDepartment();
    // This query:
    // 1. Groups users by department (PARTITION BY)
    // 2. Ranks them by creation date within each group (ORDER BY)
    // 3. Picks only the #1 ranked user from each group (WHERE rn = 1)
}
```

> ⚠️ **Common Mistake — Native queries return `Object[]` by default:**  
> Unlike JPQL, native queries don't automatically map to your entity class when doing JOINs or selecting specific columns. You get raw `Object[]` arrays and have to extract values by index (`result[0]`, `result[1]`...). For cleaner results, use `@SqlResultSetMapping` or interface-based projections (Section 5.4).

> ⚠️ **Common Mistake — Forgetting `countQuery` for pagination:**  
> If you use a native query with `Pageable` but don't provide `countQuery`, Spring may try to wrap your complex SQL in `SELECT COUNT(*) FROM (your_query)` — which can fail for complex queries or perform poorly. Always provide an explicit `countQuery`.

> 📝 **Beginner Note — Native queries are NOT validated at startup:**  
> Unlike JPQL queries, native SQL is NOT checked when your application starts. If you have a typo in your SQL, you'll only discover it at runtime when the query actually runs. This makes native queries riskier — always test them!

> 💡 **Important to Remember — Portability trade-off:**  
> Native queries tie you to a specific database. If you switch from MySQL to PostgreSQL, your MySQL-specific native queries might break. JPQL queries would work on both without changes. Use native queries only when JPQL genuinely can't express what you need.

---

### 5.3 Named Queries

#### What are Named Queries?

Named queries are JPQL (or SQL) queries defined **once on the entity class** and referenced by **name** in the repository. Think of them as pre-registered queries that live alongside your entity definition.

#### Why do they exist?

Named queries are **validated at application startup**. If you make a typo in the query, your app won't start — you catch the bug before it reaches production. They were popular before `@Query` existed in Spring Data JPA.

#### When should you use them?

**Honestly? Rarely in modern projects.** The `@Query` annotation (Section 5.1) is more convenient because:
- It keeps the query **right next to the repository method** that uses it (easier to find and maintain)
- Named queries clutter the entity class with query definitions that don't belong there

Named queries are still useful if:
- You want to centralize queries on the entity for organizational reasons
- You're working with a legacy codebase that already uses them

> 🧠 **Analogy — Speed Dial vs Contact Search:**  
> Named queries are like **speed dial** on an old phone — you set them up once (on the entity), assign them a name, and later you just press the name to use them. `@Query` is like searching your contacts directly — you write the query right where you need it. Both get you to the same place, but most people prefer searching contacts (it's quicker to find and update).

#### How They Work

**Step 1: Define the query on the entity class using `@NamedQuery`**

```java
@Entity
@Table(name = "users")
// Single named query
@NamedQuery(
    name = "User.findByEmailAndStatus",    // Name format: EntityName.methodName
    query = "SELECT u FROM User u WHERE u.email = :email AND u.status = :status"
)
// Multiple named queries — use @NamedQueries
@NamedQueries({
    @NamedQuery(
        name = "User.findActive",
        query = "SELECT u FROM User u WHERE u.status = 'ACTIVE'"
    ),
    @NamedQuery(
        name = "User.countByDept",
        query = "SELECT COUNT(u) FROM User u WHERE u.department.id = :deptId"
    )
})
public class User { ... }
```

**Step 2: Use them in your repository — the method name must match!**

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // Spring looks for a named query called "User.findByEmailAndStatus"
    // (Entity name + "." + method name)
    // If it finds one, it uses that query. If not, it falls back to derived query parsing.
    Optional<User> findByEmailAndStatus(String email, UserStatus status);
}
```

#### How Spring resolves which query to use (priority order)

When you declare a method in your repository, Spring checks in this order:
1. **Named query** — Is there a `@NamedQuery` with name `User.methodName`? → Use it
2. **`@Query` annotation** — Is there a `@Query` on the method? → Use it
3. **Derived query** — Parse the method name and auto-generate the query

> 📝 **Beginner Note:** For new projects, skip named queries entirely and use `@Query` annotations on your repository methods. They're more readable, more maintainable, and just as powerful.

---

### 5.4 Projections — Getting Only What You Need

#### What are Projections?

A **projection** lets you fetch only **specific columns** from the database, instead of loading the entire entity with all its fields.

#### Why do projections exist?

Loading a full entity means fetching **every column** — even ones you don't need. If your `User` entity has 20 fields but your API endpoint only needs `id`, `firstName`, and `email`, you're wasting memory and network bandwidth by loading the other 17 fields. Projections solve this by telling the database: "Only give me these 3 columns."

#### What problem do they solve?

- **Performance** — Less data transferred from database = faster queries
- **Memory** — Smaller objects in memory = lower memory usage
- **Security** — You don't accidentally expose sensitive fields (like `passwordHash`) in API responses
- **Clarity** — Your code makes it explicit exactly which data it needs

> 🧠 **Analogy — Ordering at a Restaurant:**  
> Loading a full entity is like ordering the **entire menu** just because you wanted a burger. A projection is like ordering **only the burger** — you get exactly what you need, and the kitchen (database) does less work preparing it.

#### Three Types of Projections

Spring Data JPA supports three approaches. Here's when to use each:

| Projection Type | How It Works | Best For |
|---|---|---|
| **Interface-based** | You define a Java interface with getter methods. Spring auto-implements it. | Simple, read-only views — most common |
| **DTO-based (class)** | You define a DTO class/record. JPQL calls its constructor. | When you need logic, validation, or immutability |
| **Dynamic** | One method, multiple projection types — caller chooses at runtime. | Flexible APIs that serve different clients |

---

#### Interface-Based Projection (Recommended for Most Cases)

This is the **simplest and most common** projection type. You define a plain Java interface with getter methods that match your entity's field names. Spring automatically creates a lightweight implementation at runtime.

**Step 1: Define the projection interface**

```java
// This is NOT a class — it's an interface. Spring creates the implementation for you.
// Each getter method corresponds to a field you want to fetch from the entity.
public interface UserSummary {

    Long getId();           // Fetches the 'id' field
    String getFirstName();  // Fetches the 'firstName' field
    String getEmail();      // Fetches the 'email' field

    // Computed field using SpEL (Spring Expression Language)
    // This doesn't map to a database column — it's calculated from other fields
    @Value("#{target.firstName + ' ' + target.lastName}")
    String getFullName();   // Returns "John Doe" by combining firstName and lastName
}
```

**Step 2: Use it as the return type in your repository**

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Instead of returning List<User> (all fields), return List<UserSummary> (only 3 fields)
    List<UserSummary> findByStatus(UserStatus status);
    // Generated SQL: SELECT u.id, u.first_name, u.email, u.last_name FROM users u WHERE u.status = ?
    // ✅ Only fetches the columns needed by UserSummary — NOT all 20+ columns!

    Optional<UserSummary> findProjectedById(Long id);
}
```

> 📝 **Beginner Note — How does Spring know which columns to fetch?**  
> Spring reads the getter methods in your projection interface (`getId()` → `id` column, `getFirstName()` → `first_name` column). It then generates SQL that selects **only those columns**. The method name must follow the standard getter naming: `get` + FieldName.

> 💡 **Important to Remember — Projections summary:**
> - ✅ **Use projections** for read-only endpoints where you don't need all entity fields
> - ✅ Projections are especially valuable when your entity has many fields (10+) or contains large text/blob columns
> - ❌ **Don't use projections** when you need to modify and save the entity — you can't save a projection back to the database
> - ✅ Interface projections are the **simplest** and should be your default choice

---

## 6. Advanced Topics

> This section covers the concepts that separate a beginner JPA developer from a confident, production-ready one. These topics — the N+1 problem, entity lifecycle, dirty checking, transactions, locking, and caching — are the areas where real-world bugs and performance issues live. Master these, and you'll debug JPA problems faster than most senior developers.

### 6.1 The N+1 Problem (⚠️ CRITICAL — Most Common Performance Issue)

#### What is the N+1 Problem?

The N+1 problem is the **#1 performance killer** in JPA applications. It happens when your code triggers **1 query** to load a list of entities, and then **N additional queries** — one for each entity — to load a related object. If you have 1,000 employees, that's 1,001 database round-trips instead of 1.

#### Why does it happen?

It happens because of **lazy loading** (Section 3.4). When you load a list of entities, their related objects aren't fetched immediately — they're loaded one-by-one as you access them in a loop. Each access triggers a separate SQL query.

#### What problem does solving it fix?

A page that takes 10 seconds to load (because of 1,001 queries) can load in 50 milliseconds (with 1 query) once you fix the N+1 problem. This is often the single biggest performance improvement you can make in a JPA application.

> 🧠 **Analogy — Grocery Shopping:**  
> Imagine you need 20 items from the grocery store.
> - **N+1 way (terrible):** Drive to the store, buy 1 item, drive home. Then drive to the store again, buy the 2nd item, drive home. Repeat 20 times. That's **21 trips** (1 to make the list + 20 to buy each item).
> - **Join fetch way (smart):** Drive to the store once, put all 20 items in your cart, drive home. **1 trip.**
>
> Every SQL query is a "trip" to the database. The network round-trip cost adds up fast — even if each individual query is fast, 1,000 of them are slow.

#### Seeing the Problem in Action

Here's exactly how the N+1 problem looks in code and what SQL is generated:

```java
// ❌ BAD — N+1 Problem in action
// Step 1: Hibernate runs 1 query to get all employees
List<Employee> employees = employeeRepository.findAll();
// SQL: SELECT * FROM employees                          ← 1 query

// Step 2: For EACH employee, you access the department (which is LAZY)
// Hibernate runs a SEPARATE query for each employee's department!
for (Employee e : employees) {
    System.out.println(e.getDepartment().getName());
    // SQL: SELECT * FROM departments WHERE id = 1       ← query 2
    // SQL: SELECT * FROM departments WHERE id = 2       ← query 3
    // SQL: SELECT * FROM departments WHERE id = 3       ← query 4
    // ... and so on for EVERY employee!
}

// If you have 1000 employees → 1 + 1000 = 1001 SQL queries!
// Even if each query takes just 5ms, that's 5 SECONDS of just waiting for the database.
```

> 📝 **Beginner Note — How to detect the N+1 problem:**  
> Turn on `spring.jpa.show-sql=true` in your `application.properties` and watch the console. If you see the same `SELECT * FROM departments WHERE id = ?` repeated dozens (or hundreds) of times, you have an N+1 problem. In production, tools like **p6spy** or **datasource-proxy** can count queries automatically in tests.

#### Solution 1: JOIN FETCH in JPQL (Most Common Fix)

`JOIN FETCH` tells Hibernate: *"Load the related data in the SAME query — don't wait for me to access it."*

```java
// ✅ SOLUTION 1: JOIN FETCH — load employees AND departments in ONE query
@Query("SELECT e FROM Employee e JOIN FETCH e.department")
List<Employee> findAllWithDepartment();
// Generated SQL: SELECT e.*, d.* FROM employees e 
//                JOIN departments d ON e.department_id = d.id
// ✅ Just 1 query — no matter how many employees!
```

**Step-by-step: What happens behind the scenes:**
1. Hibernate generates a single SQL `JOIN` query that fetches employees AND departments together
2. For each row in the result, Hibernate builds an `Employee` object AND its associated `Department` object
3. Since the department is already loaded, accessing `employee.getDepartment()` later does NOT trigger another query
4. Result: 1 query instead of N+1

> 📝 **Beginner Note — `JOIN FETCH` vs regular `JOIN`:**  
> A regular `JOIN` only uses the joined table for filtering (WHERE clause). `JOIN FETCH` goes further — it **loads the joined data into the entity objects**. Always use `JOIN FETCH` when you want to avoid lazy loading queries.

#### Solution 2: @EntityGraph (Declarative Approach — No JPQL Needed)

`@EntityGraph` is a cleaner, annotation-based way to say *"when you run this query, also load these relationships."* You don't need to write any JPQL.

```java
// ✅ SOLUTION 2a: @EntityGraph directly on the repository method
// Just list the relationships you want loaded eagerly for THIS specific query
@EntityGraph(attributePaths = {"department", "address"})
List<Employee> findByStatus(EmployeeStatus status);
// Spring Data JPA generates: SELECT e.*, d.*, a.* FROM employees e
//   LEFT JOIN departments d ON ... LEFT JOIN addresses a ON ...
//   WHERE e.status = ?
// ✅ One query loads employees + departments + addresses
```

You can also define a **named entity graph** on the entity class and reuse it across multiple repository methods:

```java
// Define the graph on the entity — reusable across multiple queries
@Entity
@NamedEntityGraph(
    name = "Employee.withDepartment",
    attributeNodes = @NamedAttributeNode("department")
)
public class Employee { ... }

// Use the named graph in any repository method
@EntityGraph("Employee.withDepartment")
List<Employee> findAll();
// ✅ Uses the pre-defined graph to load department data
```

> 💡 **Important to Remember — When to use `@EntityGraph` vs `JOIN FETCH`:**
> - Use `@EntityGraph` when you want to **add eager loading to an existing derived query method** without rewriting it as JPQL
> - Use `JOIN FETCH` in JPQL when you're already writing a custom `@Query` and need full control over the join conditions
> - Both solve the N+1 problem equally well — choose based on readability and convenience

#### Solution 3: @BatchSize (Reduce N+1 to N/batch+1)

`@BatchSize` doesn't eliminate extra queries entirely, but it **dramatically reduces them** by loading related entities in batches instead of one-by-one.

```java
// ✅ SOLUTION 3: @BatchSize — load related entities in batches
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@BatchSize(size = 30)  // Load up to 30 departments' employees at a time
private List<Employee> employees;
// Instead of 100 separate queries, Hibernate runs:
// SELECT * FROM employees WHERE department_id IN (1, 2, 3, ..., 30)  ← batch 1
// SELECT * FROM employees WHERE department_id IN (31, 32, ..., 60)   ← batch 2
// ... etc.
// 100 queries → ~4 queries (100 / 30 ≈ 4 batches)
```

> 🧠 **Analogy — Batch Loading = Carpooling:**  
> Without `@BatchSize`, each department sends its own car to pick up its employees (one query each = many cars on the road). With `@BatchSize(size = 30)`, you send a bus that picks up employees from 30 departments at once. Fewer trips, same result.

> 📝 **Beginner Note — When to use `@BatchSize`:**  
> Use it when `JOIN FETCH` isn't practical — for example, when you have multiple collections that can't all be fetched in one join (Hibernate limits multi-collection fetch joins to avoid Cartesian products). `@BatchSize` is a quick, low-effort improvement you can add to any `@OneToMany` or `@ManyToMany` field.

#### Solution 4: @Fetch(FetchMode.SUBSELECT) (Hibernate-Specific)

This loads **all** related children in a single subselect query — regardless of how many parents there are.

```java
// ✅ SOLUTION 4: Fetch subselect — one query loads ALL children
@OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<Employee> employees;
// When you access employees on ANY department, Hibernate runs:
// SELECT * FROM employees WHERE department_id IN 
//   (SELECT id FROM departments)   ← one subselect gets them ALL
// ✅ Exactly 2 queries total: 1 for departments + 1 for ALL employees
```

> 📝 **Beginner Note — Choosing the right N+1 solution:**

| Solution | Queries | Effort | Best For |
|---|---|---|---|
| `JOIN FETCH` | 1 | Write JPQL | Most cases — your default go-to |
| `@EntityGraph` | 1 | Add annotation | When you want to keep derived query methods |
| `@BatchSize` | N/batch + 1 | Add annotation | Multiple collections, quick fixes |
| `@Fetch(SUBSELECT)` | 2 | Add annotation | When you always load all parents |

> ⚠️ **Common Mistake — Not detecting N+1 in development:**  
> The N+1 problem is silent — your app works correctly, it's just slow. Always enable `spring.jpa.show-sql=true` during development and **count your queries**. If a single page load generates 50+ queries, you almost certainly have an N+1 problem.

---

### 6.2 Entity Lifecycle

#### What is the Entity Lifecycle?

Every JPA entity (a Java object mapped to a database row) goes through a series of **states** during its life. Understanding these states is essential because JPA behaves very differently depending on which state your entity is in — some changes are auto-saved, some are ignored, and some cause exceptions.

#### Why does it matter?

If you don't understand entity states, you'll hit confusing bugs:
- "I called `setEmail()` but the database didn't update!" → Your entity was **detached**.
- "I got a `LazyInitializationException`!" → You tried to access a lazy field on a **detached** entity.
- "My `save()` created a duplicate instead of updating!" → You passed a **transient** entity instead of a **managed** one.

> 🧠 **Analogy — A Student and Their School:**  
> Think of JPA as a **school**, and each entity as a **student**:
> - **Transient (not enrolled):** The student exists as a person, but the school doesn't know about them. Their grades aren't tracked.
> - **Managed (currently enrolled):** The student is registered at the school. Every quiz they take, every grade they earn — the school records it automatically. The school is "watching" them.
> - **Detached (graduated/transferred):** The student used to be enrolled, but they left. The school still has their old records, but if they take a new quiz now, no one records it.
> - **Removed (expelled):** The student is being removed from the school's records. Once the paperwork is processed (transaction commits), they're gone.

#### The Four States — Visual Diagram

```
                        ┌──────────────────────────────────────────────────┐
                        │                                                  │
  ┌──────────────┐      │      ┌───────────┐     remove()     ┌─────────┐ │
  │              │  persist()   │           │ ────────────────→ │         │ │
  │  TRANSIENT   │ ──────────→ │  MANAGED  │                   │ REMOVED │ │
  │  (new object)│             │ (tracked) │ ←── merge() ──── │         │ │
  │              │             │           │                   └─────────┘ │
  └──────────────┘             └───────────┘                               │
                                    │                                      │
                            detach() / clear()                             │
                            / transaction ends                             │
                                    │                                      │
                                    ▼                                      │
                              ┌───────────┐                                │
                              │           │       merge()                  │
                              │ DETACHED  │ ───────────────────────────────┘
                              │ (orphaned)│       (re-attaches to Managed)
                              │           │
                              └───────────┘
```

#### Each State Explained in Detail

| State | What It Means | How You Get Here | What JPA Does |
|---|---|---|---|
| **Transient** | A regular Java object. JPA has no idea it exists. | `new User()` | **Nothing.** It's just a Java object in memory. |
| **Managed** | JPA is actively watching this object. Any change you make to its fields will be **automatically saved** to the database when the transaction commits. | After `save()`, `findById()`, `persist()` | **Tracks all changes.** Runs UPDATE SQL automatically at commit time. |
| **Detached** | The object **was** managed, but the JPA session/transaction has ended. It still holds data, but JPA is no longer watching it. | Transaction ends, `EntityManager.detach()`, `clear()`, `close()` | **Ignores changes.** You can modify the object, but nothing is saved. |
| **Removed** | The object is scheduled for deletion. When the transaction commits, the row will be deleted from the database. | After `delete()`, `remove()` | **Deletes on commit.** Runs DELETE SQL when the transaction completes. |

#### Code Examples — Walking Through Each State

```java
// ── STATE 1: TRANSIENT ──────────────────────────────────────
User user = new User();                // ← TRANSIENT
user.setFirstName("Alice");            // Just a Java object in memory
user.setEmail("alice@example.com");    // JPA doesn't know or care about this object
// No database operation happens. If you lose the reference, this object is gone.

// ── STATE 2: MANAGED ────────────────────────────────────────
User savedUser = userRepository.save(user);  // ← Now MANAGED
// Hibernate ran: INSERT INTO users (first_name, email, ...) VALUES ('Alice', 'alice@example.com', ...)
// The returned 'savedUser' has a generated ID and is being tracked by JPA.
// ANY change to 'savedUser' will be auto-saved when the transaction commits (see 6.3 Dirty Checking).

// ── STATE 3: DETACHED ───────────────────────────────────────
// After the @Transactional method returns, the transaction ends.
// 'savedUser' is now DETACHED — JPA is no longer watching it.
// savedUser.setEmail("newemail@example.com"); ← This change is LOST! JPA won't save it.

// ⚠️ Accessing lazy-loaded fields on a detached entity = LazyInitializationException!
// savedUser.getDepartment().getName(); ← CRASH if department wasn't loaded during the transaction

// ── RE-ATTACHING: Detached → Managed ───────────────────────
// You can re-attach a detached entity by calling save() again inside a new transaction
@Transactional
public User reattachUser(User detachedUser) {
    return userRepository.save(detachedUser); // ← merge() happens → MANAGED again
    // Hibernate runs: UPDATE users SET ... WHERE id = ?
}

// ── STATE 4: REMOVED ────────────────────────────────────────
@Transactional
public void deleteUser(Long userId) {
    User user = userRepository.findById(userId).orElseThrow(); // ← MANAGED
    userRepository.delete(user);  // ← Now REMOVED (scheduled for deletion)
    // Hibernate will run: DELETE FROM users WHERE id = ?
    // The DELETE SQL runs when the transaction commits.
}
```

> ⚠️ **Common Mistake — Modifying a detached entity and expecting it to save:**  
> After a transaction ends, the entity is detached. Calling `user.setEmail("new@email.com")` on a detached entity has **zero effect** on the database. You must re-attach it by calling `save()` inside a new `@Transactional` method.

> ⚠️ **Common Mistake — `LazyInitializationException`:**  
> This is one of the most common JPA errors. It happens when you try to access a lazy-loaded relationship (like `user.getDepartment()`) **outside** of a transaction. The entity is detached, and JPA can't go back to the database to fetch the lazy data. **Fix:** Load everything you need *inside* the `@Transactional` method — either by accessing the field or using `JOIN FETCH`.

> 💡 **Important to Remember:**  
> - **Managed** = JPA is watching. Changes are auto-saved.
> - **Detached** = JPA is NOT watching. Changes are ignored.
> - Most bugs happen because developers don't realize their entity is detached. When in doubt, check if you're inside a `@Transactional` method.

---

### 6.3 Dirty Checking (Auto-Flush)

#### What is Dirty Checking?

Dirty checking is Hibernate's ability to **automatically detect changes** you make to managed entities and **save them to the database** — without you calling `save()` or writing any UPDATE SQL. If an entity is "dirty" (modified since it was loaded), Hibernate flushes the change at commit time.

#### Why does it exist?

Without dirty checking, you'd have to explicitly call `save()` or `update()` every time you change a field. With dirty checking, you just modify the Java object, and Hibernate handles the rest. This makes your code cleaner and less error-prone.

#### What problem does it solve?

It prevents a common bug: forgetting to call `save()` after modifying an entity. With dirty checking, as long as the entity is managed (inside a `@Transactional` method), your changes are guaranteed to be persisted.

> 🧠 **Analogy — A Google Doc with Auto-Save:**  
> Think of a managed entity as a **Google Doc**. When you edit a Google Doc, you never click "Save" — Google automatically detects every keystroke and saves it. That's exactly what dirty checking does. The moment you change a field (`user.setEmail("new@email.com")`), Hibernate notes the change and "auto-saves" it when the transaction finishes.
>
> But if you **download the Google Doc as a PDF** and edit the PDF locally, Google has no idea — your changes aren't saved. That's what happens when you edit a **detached** entity: Hibernate isn't watching anymore, so nothing is auto-saved.

#### Step-by-Step: How Dirty Checking Works Behind the Scenes

1. You call `userRepository.findById(1L)` inside a `@Transactional` method
2. Hibernate loads the `User` from the database and **takes a snapshot** of all its field values (like a "before" photo)
3. The entity is now **managed** — Hibernate stores both the entity and the snapshot
4. You call `user.setEmail("newemail@example.com")` — you just changed a field
5. When the transaction commits, Hibernate **compares** the current entity state to the original snapshot
6. It detects that `email` changed from `"old@email.com"` to `"newemail@example.com"`
7. Hibernate generates and executes: `UPDATE users SET email = 'newemail@example.com' WHERE id = 1`

You never called `save()`. Hibernate figured it out on its own.

#### Example: Dirty Checking in Action

```java
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    // ✅ CORRECT — Dirty checking works inside @Transactional
    @Transactional
    public void updateUserEmail(Long userId, String newEmail) {
        User user = userRepository.findById(userId).orElseThrow();
        // 'user' is now MANAGED — Hibernate is watching it

        user.setEmail(newEmail);  // Just change the field — that's all you need to do!

        // NO save() call needed!
        // When this method returns, the transaction commits.
        // Hibernate compares the current state to the original snapshot,
        // sees that 'email' changed, and runs:
        // UPDATE users SET email = ? WHERE id = ?
    }
}
```

Here's what goes wrong if you're NOT inside a `@Transactional` method:

```java
// ❌ BAD — Dirty checking does NOT work outside a transaction
public void badExample(Long userId) {
    User user = userRepository.findById(userId).orElseThrow();
    // findById() opens its own mini-transaction, loads the user, and closes the transaction immediately.
    // The user is now DETACHED — Hibernate is no longer watching it.

    user.setEmail("new@email.com");
    // This change is LOST! The entity is detached.
    // Hibernate doesn't know about this change and won't save it.
    // No UPDATE SQL is ever generated. The database still has the old email.
}
```

> ⚠️ **Common Mistake — Calling `save()` unnecessarily on managed entities:**  
> If you're inside a `@Transactional` method and the entity was loaded in that same transaction, calling `save()` is redundant — dirty checking will handle it. However, calling `save()` doesn't hurt; it just triggers an unnecessary merge check. Some teams prefer to always call `save()` for readability — that's fine, but know that it's not required.

> ⚠️ **Common Mistake — Dirty checking on ALL fields:**  
> By default, Hibernate generates an UPDATE that includes **every column** of the entity, even if only one field changed. For entities with many columns, this can be wasteful. You can optimize this with `@DynamicUpdate` on the entity class, which makes Hibernate generate UPDATE statements that include only the changed columns:
> ```java
> @Entity
> @DynamicUpdate  // UPDATE only includes changed columns
> public class User { ... }
> ```

> 📝 **Beginner Note — Summary of dirty checking rules:**
> - ✅ Works on **managed** entities (loaded inside a `@Transactional` method)
> - ❌ Does NOT work on **detached** entities (outside a transaction)
> - ❌ Does NOT work on **transient** entities (new objects that were never saved)
> - ✅ You do NOT need to call `save()` for changes to be persisted — just modify the fields

> 💡 **Important to Remember:** Dirty checking is the reason why `@Transactional` on service methods is so important. Without it, your entities are detached, and no changes are auto-saved. Always annotate your service methods that modify data with `@Transactional`.

---

### 6.4 Transactions (@Transactional)

#### What is a Transaction?

A **transaction** is a group of database operations that must either **all succeed together** or **all fail together**. There's no in-between. If any operation fails, everything is rolled back as if nothing happened.

#### Why do transactions exist?

Without transactions, a failure halfway through a multi-step operation could leave your database in an inconsistent state. Imagine transferring money from Account A to Account B: you debit A, but the app crashes before crediting B. Without a transaction, A lost money and B didn't receive it. With a transaction, the debit is rolled back and both accounts stay consistent.

#### What problem does `@Transactional` solve?

In raw Java, managing transactions requires a lot of boilerplate code — begin transaction, try-catch, commit on success, rollback on failure, close resources. Spring's `@Transactional` annotation handles **all of that** with a single annotation. You focus on business logic; Spring handles the transaction plumbing.

> 🧠 **Analogy — An All-or-Nothing Wedding Ceremony:**  
> Think of a transaction as a **wedding ceremony**:
> - The bride says "I do" ✅
> - The groom says "I do" ✅
> - The officiant says "You are now married" ✅ → **COMMIT** (everything is saved)
>
> But if the groom says "I don't" ❌ → **ROLLBACK** — the wedding is cancelled, the bride's "I do" is also undone. Nobody is married. It's all or nothing.
>
> In database terms: either ALL your saves, updates, and deletes within the transaction are committed to the database, or NONE of them are.

#### Step-by-Step: What Happens When You Use @Transactional

1. You call a `@Transactional` method
2. **Spring's proxy** intercepts the call and **begins a database transaction**
3. Your method runs — any `save()`, `findById()`, `delete()` calls are part of this transaction
4. If your method **completes normally** → Spring **commits** the transaction (all changes are saved)
5. If your method **throws a RuntimeException** → Spring **rolls back** the transaction (all changes are undone)

```java
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
public class OrderService {

    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    private final PaymentService paymentService;

    // ═══════════════════════════════════════════════════════════
    // BASIC TRANSACTION — All-or-Nothing
    // ═══════════════════════════════════════════════════════════

    @Transactional  // ← This one annotation gives you full transaction safety!
    public Order createOrder(CreateOrderRequest request) {
        // Step 1: Create the order
        Order order = new Order();
        // ... setup order details ...

        // Step 2: Reserve inventory (deduct stock)
        inventoryService.reserve(request.getProductId(), request.getQuantity());

        // Step 3: Charge payment
        paymentService.charge(request.getPaymentDetails());

        // Step 4: Save the order
        return orderRepository.save(order);

        // ✅ If ALL steps succeed → transaction COMMITS → order saved, inventory updated, payment charged
        // ❌ If ANY step throws RuntimeException → transaction ROLLS BACK →
        //    order NOT saved, inventory NOT deducted, payment NOT charged
        //    The database is left exactly as it was before this method was called!
    }
}
```

#### Common @Transactional Configuration Options

```java
@Service
@RequiredArgsConstructor
public class UserService {

    // ── Read-Only Transaction (Performance Optimization) ─────────
    // Use this for methods that ONLY read data — never modify it.
    // Hibernate skips dirty checking (no snapshot comparison), making reads faster.
    @Transactional(readOnly = true)
    public List<User> getAllUsers() {
        return userRepository.findAll();
        // Hibernate knows not to check for changes → faster!
    }

    // ── Custom Rollback Rules ────────────────────────────────────
    // By default, @Transactional only rolls back on RuntimeException (unchecked exceptions).
    // Checked exceptions (like IOException) do NOT trigger rollback by default!
    // Use rollbackFor to change this:
    @Transactional(rollbackFor = Exception.class)   // Rollback on ALL exceptions (checked + unchecked)
    public void importData() throws IOException { ... }

    @Transactional(noRollbackFor = EmailException.class)  // Don't rollback if email fails
    public void createUserAndSendWelcomeEmail() { ... }
    // Even if the welcome email fails, the user is still saved.
    // This is useful when a secondary action (email) shouldn't affect the primary action (user creation).

    // ── Timeout ──────────────────────────────────────────────────
    // If the transaction takes longer than this, it's rolled back.
    // Protects against long-running queries holding database connections.
    @Transactional(timeout = 30) // 30 seconds max
    public void longRunningReport() { ... }
}
```

> 📝 **Beginner Note — `readOnly = true` explained:**  
> When you set `readOnly = true`, you're telling Spring and Hibernate: *"This method only reads data. I promise I won't modify anything."* This lets Hibernate skip dirty checking (the snapshot comparison), which makes reads noticeably faster, especially when loading many entities. **Always use `readOnly = true` on methods that don't modify data.**

#### Propagation Types — How Transactions Interact

When one `@Transactional` method calls another `@Transactional` method, **propagation** decides what happens. Should they share the same transaction? Should each get its own?

> 🧠 **Analogy — Sharing a Taxi:**  
> - **REQUIRED (default):** "If there's already a taxi (transaction), I'll share it. If not, I'll call a new one."
> - **REQUIRES_NEW:** "I always call my own taxi, even if there's already one. The other taxi waits for me."
> - **MANDATORY:** "I refuse to walk — there MUST be a taxi already. If not, I'll throw a tantrum (exception)."
> - **NEVER:** "I refuse to take a taxi. If someone tries to put me in one, I'll throw a tantrum."
> - **SUPPORTS:** "If there's a taxi, I'll ride along. If not, I'll walk — no problem either way."

```java
// REQUIRED (default) — Most common. Join existing transaction, or start a new one.
// 99% of the time, this is what you want.
@Transactional(propagation = Propagation.REQUIRED)
public void doWork() { ... }

// REQUIRES_NEW — Always start a fresh transaction.
// The outer transaction is PAUSED until this one finishes.
// Use case: Audit logging that MUST be saved even if the main operation fails.
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void writeAuditLog(String action) {
    auditRepository.save(new AuditLog(action));
    // This save is in its OWN transaction.
    // Even if the calling method rolls back, this audit log is KEPT.
}

// NESTED — Run within a "savepoint" inside the existing transaction.
// If this nested part fails, only the nested part rolls back — not the whole transaction.
@Transactional(propagation = Propagation.NESTED)
public void nestedWork() { ... }

// SUPPORTS — Run in a transaction if one exists, otherwise run without one.
@Transactional(propagation = Propagation.SUPPORTS)
public void flexibleRead() { ... }

// NOT_SUPPORTED — Pause any existing transaction and run without one.
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void nonTransactionalWork() { ... }

// NEVER — Must NOT be called within a transaction. Throws exception if one exists.
@Transactional(propagation = Propagation.NEVER)
public void mustBeNonTransactional() { ... }

// MANDATORY — Must be called within an existing transaction. Throws exception if none exists.
@Transactional(propagation = Propagation.MANDATORY)
public void requiresExistingTransaction() { ... }
```

> 📝 **Beginner Note — Which propagation should I use?**  
> For 99% of cases, stick with the default `REQUIRED`. Use `REQUIRES_NEW` only when you need an operation to succeed independently (like audit logs or notification records). The other types are rarely needed in typical applications.

> ⚠️ **Common Mistake — `@Transactional` on a `private` method doesn't work!**  
> Spring implements `@Transactional` using a **proxy pattern**. When Spring creates a proxy, it wraps your bean and intercepts method calls. But it can only intercept **public** methods called from **outside** the class. Two things break:
> 1. **Private methods:** The proxy can't override them → `@Transactional` is silently ignored.
> 2. **Self-invocation (calling a `@Transactional` method from within the same class):** The call goes directly to `this`, bypassing the proxy → `@Transactional` is silently ignored.
>
> ```java
> @Service
> public class UserService {
>
>     // ❌ BROKEN — private method, proxy can't intercept
>     @Transactional
>     private void doWork() { ... }
>
>     // ❌ BROKEN — self-invocation bypasses the proxy
>     public void outerMethod() {
>         this.innerMethod(); // Direct call → proxy is bypassed → no transaction!
>     }
>
>     @Transactional
>     public void innerMethod() { ... }
>
>     // ✅ FIX — inject the service into itself (self-injection)
>     @Autowired
>     private UserService self; // Spring injects the PROXY, not 'this'
>
>     public void outerMethodFixed() {
>         self.innerMethod(); // Goes through the proxy → transaction works!
>     }
> }
> ```

> 💡 **Important to Remember — @Transactional Checklist:**
> - ✅ Always on `public` methods
> - ✅ Always on the **service layer** (not controllers, not repositories)
> - ✅ Use `readOnly = true` for read-only methods
> - ✅ Use `rollbackFor = Exception.class` if you throw checked exceptions
> - ❌ Never on `private` methods
> - ❌ Never rely on self-invocation to trigger transactions

---

### 6.5 Optimistic vs Pessimistic Locking

#### What is Locking?

When two users try to modify the **same database record** at the **same time**, you have a **concurrency conflict**. Without any protection, one user's changes silently overwrite the other's — this is called the **lost update problem**. Locking prevents this.

#### Why do you need it?

Imagine two customer support agents both open the same customer profile. Agent A changes the phone number. Agent B changes the email. They both click "Save" at the same time. Without locking, whoever saves last wins — and the other agent's change is silently lost. Locking detects or prevents this conflict.

> 🧠 **Analogy — Editing a Shared Document:**  
> - **Optimistic Locking = Google Docs with conflict detection:** Everyone can edit freely. If two people edit the same paragraph, the system shows a warning: *"This paragraph was changed by someone else. Please review."* Most of the time, there's no conflict, so this approach is fast and non-blocking.
> - **Pessimistic Locking = Locking a file on a shared drive:** When you open the file, it's locked — nobody else can open it until you close it. Guarantees no conflicts, but other people have to wait. If you forget to close it (or your computer crashes), the file stays locked.

---

#### Optimistic Locking (Recommended for Most Applications)

**How it works:** Each entity has a **version number**. When you read the entity, you read its version. When you save, Hibernate checks: *"Is the version in the database still the same as when I read it?"* If yes, the save succeeds and the version increments. If no (someone else changed it), Hibernate throws an exception.

**Step-by-step: What happens behind the scenes:**

1. User A reads Product (id=1, stock=100, **version=1**)
2. User B reads Product (id=1, stock=100, **version=1**)
3. User A changes stock to 90 and saves
   - Hibernate runs: `UPDATE products SET stock=90, version=2 WHERE id=1 AND version=1`
   - 1 row updated ✅ → version is now **2** in the database
4. User B changes stock to 80 and saves
   - Hibernate runs: `UPDATE products SET stock=80, version=2 WHERE id=1 AND version=1`
   - 0 rows updated ❌ → version is no longer 1 → **`OptimisticLockException`!**
5. User B's code catches the exception and can retry with fresh data

```java
@Entity
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private int stock;

    @Version  // ← This is the magic annotation!
    private int version;
    // Hibernate manages this field automatically:
    // - Starts at 0 when the entity is first saved
    // - Increments by 1 on every successful UPDATE
    // - Every UPDATE includes "AND version = ?" in its WHERE clause
    // - If the version doesn't match, the update fails → exception
    // You should NEVER set this field manually!
}
```

Here's how to handle the conflict in your service:

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    @Transactional
    public Product updateStock(Long productId, int delta) {
        try {
            Product product = productRepository.findById(productId).orElseThrow();
            product.setStock(product.getStock() + delta);
            return productRepository.save(product);
            // If another user modified the product between our findById and save,
            // the version won't match → OptimisticLockingFailureException!
        } catch (OptimisticLockingFailureException ex) {
            // Option 1: Retry the operation with fresh data
            // Option 2: Return a 409 Conflict response to the client
            throw new ConflictException("Product was modified by another user. Please refresh and try again.");
        }
    }
}
```

> 📝 **Beginner Note — When to use Optimistic Locking:**  
> Use it when **conflicts are rare** — which is most applications. If 100 users read a product page but only 1 updates it, optimistic locking is perfect: 99 reads have zero overhead, and the rare conflict is handled gracefully. No database rows are locked, so there's no waiting.

---

#### Pessimistic Locking (For High-Contention Scenarios)

**How it works:** When you read a row, you **lock it in the database**. Other transactions that try to read or write that row are **blocked** (forced to wait) until your transaction finishes and releases the lock.

> 🧠 **Analogy — A Fitting Room:**  
> Think of a database row as a **fitting room** in a clothing store:
> - **PESSIMISTIC_WRITE** = You go into the fitting room and **lock the door**. Nobody else can enter or even peek. They wait in line until you're done.
> - **PESSIMISTIC_READ** = The fitting room has a **window**. Others can look (read), but nobody can enter (write) until you leave.

```java
public interface ProductRepository extends JpaRepository<Product, Long> {

    // PESSIMISTIC_WRITE — Lock the row. Nobody else can read or write until this transaction ends.
    // Use for: critical updates where you MUST prevent any concurrent access.
    @Lock(LockModeType.PESSIMISTIC_WRITE)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdForUpdate(@Param("id") Long id);
    // Generated SQL: SELECT * FROM products WHERE id = ? FOR UPDATE
    // The "FOR UPDATE" clause tells the database to lock this row.

    // PESSIMISTIC_READ — Others can read, but cannot write until this transaction ends.
    // Use for: scenarios where you need consistent reads but don't want to block other readers.
    @Lock(LockModeType.PESSIMISTIC_READ)
    @Query("SELECT p FROM Product p WHERE p.id = :id")
    Optional<Product> findByIdWithReadLock(@Param("id") Long id);
    // Generated SQL: SELECT * FROM products WHERE id = ? FOR SHARE (or LOCK IN SHARE MODE)
}
```

**Using pessimistic locking in a service:**

```java
@Service
@RequiredArgsConstructor
public class InventoryService {

    private final ProductRepository productRepository;

    @Transactional
    public void decreaseStock(Long productId, int quantity) {
        // This query LOCKS the row — no other transaction can modify it until we commit
        Product product = productRepository.findByIdForUpdate(productId)
                .orElseThrow(() -> new NotFoundException("Product not found"));

        if (product.getStock() < quantity) {
            throw new InsufficientStockException("Not enough stock");
        }

        product.setStock(product.getStock() - quantity);
        // Transaction commits → lock is released → other transactions can proceed
    }
}
```

> ⚠️ **Common Mistake — Deadlocks with pessimistic locking:**  
> If Transaction A locks Row 1 and waits for Row 2, while Transaction B locks Row 2 and waits for Row 1, **neither can proceed** — this is a **deadlock**. The database detects it and kills one of the transactions. To prevent deadlocks, always lock rows **in the same order** across your application (e.g., always lock the lower ID first).

---

#### Optimistic vs Pessimistic — Which Should You Use?

| | Optimistic Locking | Pessimistic Locking |
|---|---|---|
| **How it works** | Check version on save | Lock row on read |
| **Blocking?** | No — other users aren't blocked | Yes — other users wait |
| **Performance** | Better (no lock overhead) | Worse (locks reduce throughput) |
| **Conflict handling** | Exception on conflict → retry | No conflicts (others wait) |
| **Risk** | Retry overhead if conflicts are frequent | Deadlocks, reduced throughput |
| **Best for** | **Most applications** — read-heavy, low contention | Financial systems, inventory — high contention on specific rows |
| **Setup** | Add `@Version` field | Use `@Lock` on queries |

> 💡 **Important to Remember:**  
> - **Start with Optimistic Locking.** It's simpler, faster, and sufficient for 90% of applications.
> - Switch to **Pessimistic Locking** only for specific high-contention operations (like deducting inventory, processing payments) — and only on the specific queries that need it, not everywhere.
> - You can use **both** in the same application: optimistic for most entities, pessimistic for critical sections.

---

### 6.6 Caching

#### What is Caching?

Caching stores frequently accessed data **in memory** so that repeated requests don't hit the database every time. Instead of running a SQL query for every request, your app checks the cache first — if the data is there, it returns instantly.

#### Why does caching matter?

Database queries involve network round-trips, SQL parsing, disk I/O, and result transmission. Even a fast query takes 1–10ms. A cache hit takes microseconds. For data that doesn't change often (product categories, country lists, user profiles), caching can make your API **10–100x faster** and dramatically reduce database load.

#### What problem does it solve?

If your app serves 1,000 requests/second and each one queries the same "list of categories" from the database, that's 1,000 identical queries per second — wasteful. With caching, the first request queries the database and stores the result. The next 999 requests get the result from the cache instantly.

> 🧠 **Analogy — A Library's Front Desk:**  
> Imagine a library where every book request requires walking to the back warehouse (database) and finding the book.
> - **No cache:** Every visitor waits while the librarian walks to the warehouse. Even if 50 people ask for the same bestseller, the librarian makes 50 trips.
> - **First-Level Cache (desk drawer):** The librarian keeps the last few requested books in a desk drawer. If you ask for the same book again *during the same visit*, it's right there. When you leave (transaction ends), the drawer is cleared.
> - **Second-Level Cache (display shelf):** The library puts popular books on a shelf near the entrance. ANY visitor can grab them — even across different visits. The shelf is restocked periodically or when books are updated.

---

#### First-Level Cache (Always On — Per-Transaction)

The first-level cache (also called the **session cache** or **persistence context cache**) is **built into Hibernate** and is always active. You don't configure it — it just works.

**How it works:**
- Each `@Transactional` method gets its own first-level cache
- When you load an entity by ID, Hibernate stores it in this cache
- If you load the **same entity by ID again** within the same transaction, Hibernate returns the cached instance — no SQL query
- The cache is cleared when the transaction ends

```java
@Transactional
public void demonstrateFirstLevelCache() {
    // Query 1: Hits the database
    User u1 = userRepository.findById(1L).get();
    // SQL executed: SELECT * FROM users WHERE id = 1

    // Query 2: Does NOT hit the database — returns cached instance!
    User u2 = userRepository.findById(1L).get();
    // No SQL executed! u2 comes from the first-level cache.

    // They are the EXACT SAME Java object in memory:
    System.out.println(u1 == u2); // true — same reference!

    // This means any changes you make to u1 are also visible through u2
    // (because they are literally the same object)
}
// When this method returns, the first-level cache is cleared.
// A new @Transactional method starts with an empty cache.
```

> 📝 **Beginner Note — First-level cache only works with `findById()`:**  
> If you run the same JPQL query twice (e.g., `findByEmail("alice@example.com")`), Hibernate runs the SQL **both times**. The first-level cache only caches by **entity ID** — it doesn't cache query results. For query-level caching, you need the second-level cache (below).

> 📝 **Beginner Note — First-level cache scope:**  
> The first-level cache lives only within a single `@Transactional` method (or more precisely, within a single Hibernate `Session`). Two different transactions have completely separate caches. If User A loads entity #1 and User B loads entity #1, each gets their own database query — they don't share.

---

#### Second-Level Cache (Cross-Transaction — Needs Configuration)

The second-level cache (L2 cache) is a **shared, application-wide cache** that survives across transactions. If User A loads a `Category` entity, and 10 seconds later User B loads the same `Category`, User B gets it from the L2 cache — no database query.

**Step-by-step setup:**

**Step 1: Add cache dependencies to your `pom.xml`**

```xml
<!-- The Hibernate JCache integration -->
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jcache</artifactId>
</dependency>

<!-- Caffeine — a fast, in-memory cache library by Google -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Step 2: Enable the second-level cache in `application.properties`**

```properties
# Turn on the second-level cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true

# Tell Hibernate which cache provider to use (JCache + Caffeine)
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.internal.JCacheRegionFactory
spring.jpa.properties.javax.cache.provider=com.github.benmanes.caffeine.jcache.spi.CaffeineCachingProvider
```

**Step 3: Mark which entities should be cached**

Not every entity should be cached — only entities that are **read frequently** and **change rarely**.

```java
@Entity
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE) // ← Enable L2 caching for this entity
public class Category {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private boolean active;
}
// Now, when any transaction loads a Category by ID, it's stored in the L2 cache.
// The next transaction that loads the same Category gets it from cache — no SQL!
```

**Step 4 (Optional): Enable query caching for specific queries**

By default, the L2 cache only caches entities loaded by ID. To cache the results of specific queries, enable the query cache:

```properties
# Enable the query cache (in addition to entity cache)
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

```java
// Mark specific queries as cacheable
@Query("SELECT c FROM Category c WHERE c.active = true")
@QueryHints(@QueryHint(name = "org.hibernate.cacheable", value = "true"))
List<Category> findActiveCategories();
// First call: runs SQL, stores results in query cache.
// Second call: returns results from cache — no SQL!
// Cache is invalidated when any Category is inserted, updated, or deleted.
```

#### Cache Concurrency Strategy — Which One to Choose?

The `CacheConcurrencyStrategy` controls how the cache handles concurrent reads and writes:

| Strategy | Description | Use When | Example |
|---|---|---|---|
| `READ_ONLY` | Cache never expects data to change. Fastest. | Reference data that **never** changes after creation | Countries, currencies, languages |
| `NONSTRICT_READ_WRITE` | Cache may briefly show stale data after an update. Good enough for most cases. | Data that changes **rarely**, and brief staleness is OK | Product categories, user preferences |
| `READ_WRITE` | Cache is updated synchronously with the database. Consistent but slightly slower. | Data that changes **moderately** and needs to be accurate | Product details, user profiles |
| `TRANSACTIONAL` | Cache participates in the database transaction (uses JTA). Strongest consistency but slowest. | Financial data, pricing — absolutely **must** be consistent | Account balances (rarely used in practice) |

> 📝 **Beginner Note — When NOT to use the second-level cache:**  
> - Don't cache entities that change **very frequently** (like real-time counters, chat messages) — the cache will be invalidated constantly, making it useless.
> - Don't cache entities with **huge data** (like entities with large TEXT or BLOB fields) — they'll eat up memory.
> - Don't enable caching everywhere "just in case" — it adds complexity and can cause **stale data bugs** that are hard to debug.
> - **Start without caching.** Only add it when you've identified a performance bottleneck through actual measurement.

> ⚠️ **Common Mistake — Forgetting to invalidate the cache:**  
> If you update data directly with native SQL or a `@Modifying` query (bypassing Hibernate), the L2 cache won't know about the change — it will keep serving **stale (outdated) data**. Always invalidate or evict the cache after direct SQL updates.

> 💡 **Important to Remember — Cache Summary:**
> | Cache | Scope | Lifetime | Config Needed | Caches |
> |---|---|---|---|---|
> | **First-Level** | Single transaction | Transaction duration | None (always on) | Entities by ID |
> | **Second-Level** | Entire application | Until evicted/expired | Yes (dependency + config + annotations) | Entities by ID + query results |

---

#### Spring Boot Application-Level Caching (@Cacheable, @CachePut, @CacheEvict)

Everything above (first-level and second-level cache) is **Hibernate's** caching — it works at the JPA/entity level. But Spring Boot also provides its own **application-level caching framework** that works on **any method** in your application — not just database queries. This is what most developers mean when they say "caching in Spring Boot."

#### What is Spring Boot Caching?

Spring Boot caching uses annotations to **automatically store and retrieve method return values** in a cache. The caller never knows whether the data came from the actual method execution, the database, or the cache — it's completely transparent.

> 🧠 **Analogy — A Sticky Note on Your Monitor:**  
> Imagine you keep asking your colleague the same question every hour: "What's the WiFi password?" Instead of walking to their desk every time, you write the answer on a sticky note on your monitor. Next time you need it, you just look at the sticky note. That sticky note is your cache. Spring Boot does this automatically for your methods — the first call executes the method and "writes the sticky note," and all subsequent calls with the same input just "read the sticky note."

#### Why Do We Need Application-Level Caching?

- **Improve Performance** — Avoid repeated database or API calls; get faster response times
- **Reduce Load on Backend** — Fewer DB queries; less stress on microservices
- **Better User Experience** — Pages load faster; APIs respond quicker

#### Step 1: Enable Caching in Spring Boot

First, add the caching starter dependency and enable caching in your main application class:

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```java
@EnableCaching          // ← This annotation turns on Spring's caching infrastructure
@SpringBootApplication
public class MyApplication { ... }
```

> 📝 **Beginner Note:** Without `@EnableCaching`, all caching annotations (`@Cacheable`, `@CachePut`, `@CacheEvict`) are **silently ignored**. Your app still works, but nothing is cached. This is a common "why isn't my cache working?" moment.

#### Step 2: The Three Core Caching Annotations

Spring provides three annotations that cover all caching operations. Think of them as **read**, **update**, and **delete** for your cache:

| Annotation | Purpose | When to Use |
|---|---|---|
| `@Cacheable` | **Read from cache** if present, else execute the method and cache the result for future calls | Read-only or lookup methods (`findById`, `findByName`) |
| `@CachePut` | **Always execute** the method AND **update the cache** with the new result | Save/update methods — keeps cache in sync with DB |
| `@CacheEvict` | **Remove** an entry from the cache (or clear the entire cache) | Delete methods — removes stale data from cache |

> 🧠 **Analogy — A Phone Book:**  
> - `@Cacheable` = Look up a contact in your phone book. If it's there, use it. If not, call the person, get the number, and write it in the phone book for next time.
> - `@CachePut` = Someone changed their phone number. Update the entry in your phone book.
> - `@CacheEvict` = Someone moved away. Remove their entry from the phone book so you don't call a wrong number.

---

#### @Cacheable — Cache Method Results on First Call

`@Cacheable` is the most commonly used caching annotation. It works like this:
1. **First call** with a given input → method executes normally, result is stored in cache
2. **Next call** with the **same input** → method is **skipped entirely**, cached result is returned instantly

**Example 1: Cache an expensive calculation**

```java
@Service
public class MathService {

    @Cacheable("piDecimals")  // Cache name = "piDecimals"
    public int computePiDecimal(int precision) {
        // This is an expensive calculation that takes several seconds...
        // On the first call with precision=100, the result is computed and cached.
        // On the second call with precision=100, the cached result is returned instantly!
        // A call with precision=200 is a DIFFERENT key → computed fresh and cached separately.
        return expensiveComputation(precision);
    }
}
```

**Example 2: Cache static database lookups (most common real-world use case)**

Reference tables like roles, countries, and categories rarely change. Instead of hitting the DB every time, cache them:

```java
public interface RoleRepository extends JpaRepository<Role, Long> {

    // ✅ Caches the role by name on first fetch
    // Cache name = "roles", cache key = the 'name' parameter value
    @Cacheable("roles")
    Optional<Role> findRoleByName(String name);
    // First call: roleRepository.findRoleByName("ROLE_JOB_SEEKER") → SQL query → result cached
    // Second call: roleRepository.findRoleByName("ROLE_JOB_SEEKER") → NO SQL! Returns from cache
    // Third call: roleRepository.findRoleByName("ROLE_ADMIN") → different key → SQL query → cached
}
```

---

#### @CachePut — Always Execute and Update Cache

Unlike `@Cacheable` (which skips the method if cache has data), `@CachePut` **always executes the method** and **updates the cache** with the fresh result. Use it when you're saving or updating data and want the cache to reflect the change.

```java
public interface RoleRepository extends JpaRepository<Role, Long> {

    // 🔄 Updates cache when a role is saved/updated
    // The method always runs (INSERT/UPDATE SQL), AND the cache is updated with the result
    @CachePut(value = "roles", key = "#role.name")
    Role save(Role role);
    // key = "#role.name" means: use the role's name field as the cache key
    // So saving a role with name="ROLE_ADMIN" updates the "roles" cache entry for "ROLE_ADMIN"
}
```

> 📝 **Beginner Note — `@Cacheable` vs `@CachePut`:**
> - `@Cacheable` = "Don't run the method if the cache already has the answer." → For **reads**
> - `@CachePut` = "Always run the method, then update the cache with the new answer." → For **writes**
> - **Never use `@Cacheable` on a save/update method!** It would skip the save operation if the cache already has data — meaning your data wouldn't actually be saved to the database!

---

#### @CacheEvict — Remove Stale Data from Cache

When you delete data from the database, you should also remove it from the cache. Otherwise, the cache serves **stale (outdated) data** that no longer exists in the DB.

```java
public interface RoleRepository extends JpaRepository<Role, Long> {

    // ✅ Caches the role by name on first fetch
    @Cacheable("roles")
    Optional<Role> findRoleByName(String name);

    // 🔄 Updates cache when role is saved/updated
    @CachePut(value = "roles", key = "#role.name")
    Role save(Role role);

    // ❌ Evicts (removes) a specific role from cache when deleted by name
    @CacheEvict(value = "roles", key = "#name")
    void deleteRoleByName(String name);
    // After this runs, the cache entry for that role name is removed.
    // Next call to findRoleByName() for this name will hit the DB again.

    // 🚫 Optional: Clear ALL entries from the "roles" cache
    @CacheEvict(value = "roles", allEntries = true)
    void deleteAll();  // Only if appropriate — this wipes the entire "roles" cache
}
```

> 💡 **Important to Remember — Tips for Effective Caching:**
> - Use `@Cacheable` on **read-only or static lookup methods**
> - Use `@CachePut` when **updating records** and you want to sync the cache
> - Use `@CacheEvict` when **deleting or modifying** cached data
> - **Never use `@Cacheable` on write/update operations** — it would skip the actual write!

---

#### How Does Spring Decide the Cache Key?

When you use `@Cacheable("users")`, Spring needs to know: *"Under what key should I store and retrieve this data?"* Spring follows these rules automatically:

**Case 1: Method has one parameter → parameter value IS the key**

```java
@Cacheable("users")
public User getUserById(Long id) {
    // ...
}
// getUserById(1) → cache key = 1
// getUserById(2) → cache key = 2
// Each unique parameter value creates a separate cache entry
```

**Case 2: Method has multiple parameters → combination of all parameters is the key**

```java
@Cacheable("users")
public User getUser(String email, String role) {
    // ...
}
// getUser("a@x.com", "ADMIN") → cache key = ("a@x.com", "ADMIN")
// getUser("a@x.com", "USER")  → cache key = ("a@x.com", "USER") ← DIFFERENT entry!
// Same email but different role = different cache entry
```

**Case 3: Method has NO parameters → Spring uses `SimpleKey.EMPTY`**

```java
@Cacheable("roles")
public List<Role> getAllRoles() {
    // ...
}
// There's only ONE cache entry for this method (since there are no parameters).
// Method executes only once. All subsequent calls return the cached list.
```

> 📝 **Beginner Note — How Spring creates the key internally:**  
> Spring uses `SimpleKeyGenerator` behind the scenes:
> - **0 params** → `SimpleKey.EMPTY` (single cache entry for the method)
> - **1 param** → the parameter itself (e.g., `Long id = 1` → key is `1`)
> - **Multiple params** → `SimpleKey(param1, param2, ...)` (composite key)
>
> You normally don't see this, but it's good to know when debugging cache misses.

---

#### Custom Cache Keys — Using the `key` Attribute and SpEL

Sometimes the default key (method parameters) isn't what you want. You can control the cache key manually using **SpEL (Spring Expression Language)**:

```java
// Explicit key using a method parameter
@Cacheable(value = "users", key = "#id")
public User getUserById(Long id) {
    // ...
}

// Key using a field of an object parameter
@Cacheable(value = "users", key = "#user.email")
public User getUser(User user) {
    // ...
}
// Uses user.getEmail() as the cache key — much more reliable than the full User object
```

**Common SpEL expressions for cache keys:**

| Expression | Meaning |
|---|---|
| `#id` | Method parameter named `id` |
| `#email` | Method parameter named `email` |
| `#user.email` | The `email` field of the `user` parameter |
| `#user.id` | The `id` field of the `user` parameter |
| `#root.methodName` | Name of the method being called |
| `#root.args[0]` | First argument passed to the method |

> ⚠️ **Common Mistake — Bad key choices:**  
> ```java
> // ❌ BAD — Using a full object as cache key
> @Cacheable("users")
> public User getUser(User user) { ... }
> // The User object's identity in memory may differ between calls,
> // causing cache misses even when the data is the same!
> 
> // ✅ GOOD — Use a stable, unique field as the key
> @Cacheable(value = "users", key = "#user.id")
> public User getUser(User user) { ... }
> ```

> ⚠️ **Common Mistakes with Caching:**
> - **Caching entities with mutable fields** — if cached objects are modified, the cache serves wrong data
> - **Using full objects as cache keys** — use IDs or unique fields instead
> - **Forgetting that same method + different params = different cache entries** — each unique parameter combo gets its own entry

---

#### Spring Boot Caching with TTL (Time-To-Live) Configuration

By default, Spring Boot uses `ConcurrentMapCacheManager` — a simple in-memory cache that **never expires entries**. This means cached data stays forever until the app restarts or you explicitly evict it. For production use, you need a real cache provider that supports **TTL (Time-To-Live)** — automatic expiration of cache entries after a set time.

#### What is TTL?

TTL (Time-To-Live) is the maximum amount of time a cache entry is valid. After the TTL expires, the entry is automatically removed, and the next request fetches fresh data from the database.

> 🧠 **Analogy — Expiration Dates on Food:**  
> Think of cache entries like items in your fridge. Milk expires after 7 days (`expireAfterWrite = 7 days`). After that, even though the milk is still in the fridge (cache), you throw it away and buy fresh milk (fetch from DB). Without expiration dates (no TTL), you might drink month-old milk (serve stale data).

#### Setting Up Caffeine Cache with TTL

**Caffeine** is a high-performance, in-memory caching library by the makers of Google Guava. It's the **recommended cache provider** for Spring Boot applications that run on a single server.

**Step 1: Add the Caffeine dependency**

```xml
<!-- pom.xml -->
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**Step 2: Configure cache names, TTL, and max size**

You can configure Caffeine via a Java `@Configuration` class for fine-grained control over each cache:

```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.concurrent.TimeUnit;

@Configuration
public class CaffeineCacheConfig {

    @Bean
    public CacheManager caffeineCacheManager() {

        // Each CaffeineCache has its own TTL and size limit
        // This gives you fine-grained control per cache region

        CaffeineCache companiesCache = new CaffeineCache("companies", Caffeine.newBuilder()
                .expireAfterWrite(10, TimeUnit.MINUTES)  // Entries expire 10 mins after being written
                .maximumSize(500)                         // Hold max 500 entries in memory
                .build());

        CaffeineCache jobsCache = new CaffeineCache("jobs", Caffeine.newBuilder()
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .maximumSize(5000)
                .build());

        CaffeineCache rolesCache = new CaffeineCache("roles", Caffeine.newBuilder()
                .expireAfterWrite(1, TimeUnit.DAYS)      // Roles rarely change — cache for 1 day
                .maximumSize(100)
                .build());

        SimpleCacheManager manager = new SimpleCacheManager();
        manager.setCaches(Arrays.asList(companiesCache, jobsCache, rolesCache));
        return manager;
    }
}
```

**What this configuration does:**

| Cache Name | TTL Duration | Max Entries | Use Case |
|---|---|---|---|
| `companies` | 10 Minutes | 500 | Company data changes moderately |
| `jobs` | 10 Minutes | 5,000 | Job listings are updated frequently |
| `roles` | 1 Day | 100 | Roles are nearly static — long TTL is fine |

**How TTL and max size work together:**
- **`expireAfterWrite`** — Entries expire after this duration from their **last write**. After expiry, the next request fetches fresh data from the DB and re-caches it.
- **`maximumSize`** — Keeps memory usage under control. When the limit is reached, the **least recently used** (LRU) entries are evicted to make room for new ones.

> 📝 **Beginner Note — Simpler alternative using `application.properties`:**  
> If you don't need different TTLs per cache, you can configure Caffeine globally via properties — much simpler:
> ```properties
> # application.properties
> spring.cache.type=caffeine
> spring.cache.cache-names=jobs,roles
> spring.cache.caffeine.spec=maximumSize=5000,expireAfterAccess=600s
> ```
> This sets **all caches** to the same spec: max 5,000 entries, expire 600 seconds (10 minutes) after last access. Use the Java `@Configuration` approach (above) when you need **different settings per cache**.

> ⚠️ **Common Mistake — Using `expireAfterWrite` vs `expireAfterAccess`:**  
> - `expireAfterWrite` — Entry expires X time after it was **written/created**. Timer resets only when data is written again. Best for keeping data fresh.
> - `expireAfterAccess` — Entry expires X time after it was **last read or written**. Frequently accessed data stays alive longer. Best for reducing eviction of popular entries.
> - Choose based on your use case. For most DB caching, `expireAfterWrite` is safer because it guarantees freshness.

> 💡 **Important to Remember — Spring Boot Caching vs Hibernate L2 Cache:**
> | | Spring Boot Caching (`@Cacheable`) | Hibernate L2 Cache (`@Cache`) |
> |---|---|---|
> | **Level** | Application/service layer | JPA/entity layer |
> | **Works on** | Any Spring Bean method | Entity loading by ID + JPQL queries |
> | **Annotations** | `@Cacheable`, `@CachePut`, `@CacheEvict` | `@Cache`, `@QueryHints` |
> | **Key** | Method parameters | Entity ID |
> | **Provider** | Caffeine, Redis, EhCache, etc. | Caffeine (via JCache), EhCache, etc. |
> | **When to use** | Service-layer caching for API responses, computations | Entity-level caching for repeated `findById` calls |
> | **Can use both?** | ✅ Yes — they complement each other! |

---

## 7. Quick Revision Cheat Sheet

### 7.1 The Stack

```
Your Code → Spring Data JPA → JPA (spec) → Hibernate (engine) → JDBC → Database
```

- **JPA** = specification (the menu) — defines rules, can't run alone
- **Hibernate** = implementation (the kitchen) — generates & executes SQL
- **Spring Data JPA** = convenience layer (the waiter) — auto-generates repositories, eliminates boilerplate

---

### 7.2 Entity Essentials

| Annotation | Purpose |
|---|---|
| `@Entity` | Marks class as a DB table (required) |
| `@Table(name = "users")` | Custom table name — use lowercase, plural, snake_case |
| `@Id` | Primary key (every entity must have one) |
| `@GeneratedValue` | Auto-generate PK — see strategy table below |
| `@Column(nullable, unique, length, columnDefinition)` | Customize column mapping |
| `@Enumerated(EnumType.STRING)` | Store enum as text — **never use ORDINAL** |
| `@Transient` | Field not mapped to DB |
| `@Version` | Optimistic locking version field |
| `@DynamicUpdate` | UPDATE only changed columns (not all) |
| `@NoArgsConstructor` | **Required** by JPA — empty constructor for reflection |

**@GeneratedValue Strategies**

| Strategy | Best For | Notes |
|---|---|---|
| `IDENTITY` | MySQL — simplest | Disables Hibernate batch inserts |
| `SEQUENCE` | PostgreSQL/Oracle — best perf | Use `allocationSize` for bulk inserts |
| `UUID` | Microservices / distributed | No ID collisions; larger index size |
| `TABLE` | Legacy only | Slowest — avoid in new projects |

---

### 7.3 Relationships

| Annotation | Default Fetch | Your Action | Owner Side? |
|---|---|---|---|
| `@OneToOne` | EAGER ⚠️ | **Override → LAZY** | Side with `@JoinColumn` |
| `@ManyToOne` | EAGER ⚠️ | **Override → LAZY** | **Always the owner** (has FK) |
| `@OneToMany` | LAZY ✅ | Keep LAZY, use `mappedBy` | Never — use `mappedBy` |
| `@ManyToMany` | LAZY ✅ | Keep LAZY, use `Set` not `List` | Side with `@JoinTable` |

> **Rule:** `@ManyToOne` side is always the owner (FK lives in the "many" table).

**Key terms:**
- **Owner side** → has `@JoinColumn`, holds the FK column
- **Inverse side** → uses `mappedBy`, no FK created
- Always sync both sides in bidirectional relationships

---

### 7.4 Cascade Types

| CascadeType | Effect | Use When |
|---|---|---|
| `PERSIST` | Save parent → saves children | Parent creates children (Order → Items) |
| `MERGE` | Update parent → updates children | Parent modifies children |
| `REMOVE` | Delete parent → deletes children | Children can't exist without parent |
| `ALL` | All of the above | Parent **fully owns** children |

- `orphanRemoval = true` → removing child from collection deletes it from DB
- **@ManyToMany** → only use `PERSIST` + `MERGE`, **never** `REMOVE` or `ALL`

---

### 7.5 DDL-Auto Strategies

| Value | Behavior | Environment |
|---|---|---|
| `create` | Drop & recreate all tables — **data lost** | Local dev (throwaway) |
| `create-drop` | Create on start, drop on shutdown | Automated tests |
| `update` | Add new columns/tables, never drops | Development |
| `validate` | Check schema match — crash if mismatch | Staging / pre-prod |
| `none` | Do nothing — you manage schema | **Production** (use Flyway/Liquibase) |

---

### 7.6 Repositories

**Hierarchy:** `CrudRepository` → `PagingAndSortingRepository` → `JpaRepository` (use this one)

```java
public interface UserRepository extends JpaRepository<User, Long> { }
```

**Built-in methods (free):**

| Operation | Methods |
|---|---|
| Create/Update | `save()`, `saveAll()`, `saveAndFlush()` |
| Read | `findById()` → `Optional<T>`, `findAll()`, `existsById()`, `count()` |
| Delete | `deleteById()`, `delete()`, `deleteAllInBatch()` (faster bulk) |
| Page/Sort | `findAll(Pageable)`, `findAll(Sort)` |

- `save()` = INSERT if ID is null, UPDATE if ID exists
- `findById()` returns `Optional` — use `.orElseThrow()`

---

### 7.7 Derived Query Keywords (Top 15)

| Keyword | SQL | Example |
|---|---|---|
| `And` / `Or` | `AND` / `OR` | `findByNameAndStatus` |
| `Is` / `Equals` | `=` | `findByStatusIs` |
| `Not` | `!=` | `findByStatusNot` |
| `Between` | `BETWEEN` | `findByAgeBetween` |
| `LessThan` / `GreaterThan` | `<` / `>` | `findByAgeLessThan` |
| `Before` / `After` | `<` / `>` (dates) | `findByCreatedAtAfter` |
| `IsNull` / `IsNotNull` | `IS NULL` / `IS NOT NULL` | `findByEmailIsNull` |
| `Containing` | `LIKE %x%` | `findByNameContaining` |
| `StartingWith` / `EndingWith` | `LIKE x%` / `LIKE %x` | `findByNameStartingWith` |
| `In` | `IN (...)` | `findByStatusIn(List)` |
| `True` / `False` | `= true` / `= false` | `findByVerifiedTrue` |
| `IgnoreCase` | `LOWER()` | `findByEmailIgnoreCase` |
| `OrderBy` | `ORDER BY` | `findByStatusOrderByNameAsc` |
| `Top` / `First` | `LIMIT` | `findTop5ByStatus` |

> If method name > 3–4 keywords long → switch to `@Query` with JPQL.

---

### 7.8 Query Resolution Priority

When Spring finds a repository method, it checks in this order:

1. **@NamedQuery** (`User.methodName` on entity) → oldest, rarely used now
2. **@Query annotation** (JPQL or native SQL on the method) → most flexible
3. **Derived query** (parsed from method name) → simplest, auto-generated

**JPQL vs Native SQL:**

| | JPQL | Native SQL |
|---|---|---|
| References | Java class/field names | Table/column names |
| DB-independent? | ✅ Yes | ❌ No |
| Validated at startup? | ✅ Yes | ❌ No (runtime only) |
| Supports window functions, CTEs? | ❌ No | ✅ Yes |

- `@Modifying` + `@Transactional` required for UPDATE/DELETE `@Query`
- Add `@Modifying(clearAutomatically = true)` to avoid stale persistence context

**Projections** — fetch only needed columns:
- **Interface-based** (recommended): define interface with getters → Spring auto-implements
- **DTO-based**: JPQL `new` constructor expression
- **Dynamic**: pass projection type as parameter

---

### 7.9 Pagination & Sorting

```java
PageRequest.of(page, size, Sort.by("field").descending())  // page is 0-indexed!
```

| Concept | Purpose |
|---|---|
| `Pageable` / `PageRequest` | Holds page number, size, sort |
| `Page<T>` | Data + metadata (totalPages, totalElements) — runs COUNT query |
| `Slice<T>` | Data + hasNext — **no COUNT** (faster for infinite scroll) |
| `Sort` | `Sort.by("field").ascending().and(Sort.by("other"))` |

> Add `Pageable` to **any** repository method — built-in, derived, or `@Query`.

---

### 7.10 N+1 Problem — Solutions

**Problem:** 1 query loads N entities + N extra queries for each entity's lazy relationship.

| Solution | Queries | How | Best For |
|---|---|---|---|
| `JOIN FETCH` (JPQL) | **1** | `SELECT e FROM Employee e JOIN FETCH e.department` | Default go-to |
| `@EntityGraph` | **1** | `@EntityGraph(attributePaths = {"department"})` on method | Keep derived methods clean |
| `@BatchSize(size=N)` | N/batch + 1 | Loads children in `IN (...)` batches | Multiple collections |
| `@Fetch(SUBSELECT)` | **2** | Loads all children via subselect | Always loading all parents |

> **Detection:** Enable `spring.jpa.show-sql=true` — if same SELECT repeats dozens of times, it's N+1.

---

### 7.11 Entity Lifecycle States

```
new Object()          save()/persist()          transaction ends
     │                      │                         │
 TRANSIENT ──────────→ MANAGED ──────────────→ DETACHED
                         │    ↑                       │
                  delete()/   save()/merge()    save() inside
                  remove()    (re-attach)       new @Transactional
                         ↓                            │
                      REMOVED                    MANAGED again
```

| State | JPA Watching? | Changes Auto-Saved? | How You Get Here |
|---|---|---|---|
| **Transient** | ❌ No | ❌ No | `new Entity()` |
| **Managed** | ✅ Yes | ✅ Yes (dirty checking) | After `save()`, `findById()` |
| **Detached** | ❌ No | ❌ No | Transaction ends, `detach()`, `clear()` |
| **Removed** | ✅ Scheduled | DELETE on commit | After `delete()` / `remove()` |

---

### 7.12 Dirty Checking

- Hibernate **snapshots** every managed entity when loaded
- At commit, compares current state vs snapshot → auto-generates UPDATE for changed fields
- **No `save()` needed** inside `@Transactional` for managed entities
- Does **NOT** work on detached or transient entities
- Use `@DynamicUpdate` to UPDATE only changed columns (not all)

---

### 7.13 @Transactional

| Attribute | Default | Purpose |
|---|---|---|
| `propagation` | `REQUIRED` | Join existing tx, or start new |
| `readOnly` | `false` | Set `true` for reads → skips dirty checking |
| `rollbackFor` | `RuntimeException` only | Add `Exception.class` for checked exceptions |
| `timeout` | none | Max seconds before rollback |

**Propagation quick reference:**

| Type | Behavior |
|---|---|
| `REQUIRED` | Join existing tx or create new (99% of cases) |
| `REQUIRES_NEW` | Always create new tx; outer tx pauses |
| `MANDATORY` | Must be called within existing tx, else exception |
| `NEVER` | Must NOT be within a tx, else exception |

**Critical rules:**
- ✅ Only on **public** methods (proxy can't intercept private)
- ✅ Place on **service layer** (not controllers or repositories)
- ❌ **Self-invocation** bypasses proxy → `@Transactional` ignored (fix: inject `self`)
- ✅ Use `readOnly = true` on all read-only methods

---

### 7.14 Locking

**Optimistic (recommended for most apps):**
- Add `@Version` field → Hibernate checks version on every UPDATE
- Conflict → `OptimisticLockException` → retry or return 409
- No DB locks, no blocking, no deadlocks
- Best when: conflicts are **rare** (read-heavy apps)

**Pessimistic (high-contention scenarios):**
- `@Lock(LockModeType.PESSIMISTIC_WRITE)` → `SELECT ... FOR UPDATE`
- Row is locked — other transactions **wait**
- Risk: deadlocks (always lock rows in consistent order)
- Best when: concurrent writes to **same row** are frequent (inventory, payments)

---

### 7.15 Caching

**First-Level Cache (always on):**
- Per-transaction, per-session — automatic
- Same `findById()` twice in one `@Transactional` → 1 SQL query
- Cleared when transaction ends

**Second-Level Cache (L2 — needs config):**
- Shared across transactions, application-wide
- Add `@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)` on entity
- Requires: dependency (Caffeine/EhCache) + `hibernate.cache.use_second_level_cache=true`

| Strategy | Staleness | Use For |
|---|---|---|
| `READ_ONLY` | None (never changes) | Countries, currencies |
| `NONSTRICT_READ_WRITE` | Brief staleness OK | Categories, preferences |
| `READ_WRITE` | Consistent | User profiles, products |

**Spring Boot Caching (`@Cacheable`):**
- Works on **any Spring Bean method**, not just DB queries
- Requires `@EnableCaching` on config class

| Annotation | Purpose |
|---|---|
| `@Cacheable("name")` | Return from cache if present; else execute & cache |
| `@CachePut("name")` | Always execute, then update cache |
| `@CacheEvict("name")` | Remove entry from cache |

- **Never** use `@Cacheable` on write methods (would skip the actual save!)
- Use Caffeine for TTL: `expireAfterWrite` (freshness) vs `expireAfterAccess` (popularity)

---

### 7.16 Common Gotchas

| Gotcha | Fix |
|---|---|
| `LazyInitializationException` | Load data inside `@Transactional`, or use `JOIN FETCH` |
| `@Transactional` on private method | Make it `public` — proxy can't intercept private |
| Self-invocation skips `@Transactional` | Inject `self` via `@Autowired` and call through proxy |
| `@Enumerated` default is ORDINAL | Always use `@Enumerated(EnumType.STRING)` |
| `findById()` returns `Optional` | Use `.orElseThrow()`, not direct assignment |
| Editing detached entity — changes lost | Re-attach via `save()` inside new `@Transactional` |
| `@Modifying` query → stale context | Add `clearAutomatically = true` |
| `CascadeType.ALL` on `@ManyToMany` | Use only `PERSIST` + `MERGE` — never `REMOVE` |
| N+1 silently slow | Enable `show-sql=true`, count repeated queries |
| `ddl-auto=update` in production | Use `none` + Flyway/Liquibase for migrations |
| Native query typo not caught at startup | Test all native queries — only validated at runtime |
| Page numbers are 0-indexed | `PageRequest.of(0, 10)` = first page |

---

## 8. Interview Questions & Answers

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