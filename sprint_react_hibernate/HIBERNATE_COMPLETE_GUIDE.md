# Hibernate - Complete Guide from Basic to Advanced 🎯

> Comprehensive Hibernate guide for staff engineer interview preparation - Covers fundamentals, advanced topics, performance optimization, and real-world patterns

---

## Table of Contents
1. [Introduction to Hibernate](#introduction-to-hibernate)
2. [Hibernate Architecture](#hibernate-architecture)
3. [Configuration and Setup](#configuration-and-setup)
4. [Entity Mapping](#entity-mapping)
5. [Relationships](#relationships)
6. [HQL and JPQL](#hql-and-jpql)
7. [Criteria API](#criteria-api)
8. [Caching](#caching)
9. [Performance Optimization](#performance-optimization)
10. [Advanced Topics](#advanced-topics)
11. [Best Practices](#best-practices)
12. [Common Pitfalls](#common-pitfalls)
13. [Interview Questions](#interview-questions)

---

Hibernate Senior Guide
# Hibernate – Senior Engineer Guide

This document is a **production-oriented Hibernate/JPA guide** for senior engineers. It focuses on **entity lifecycle, transactions, performance, and common pitfalls**.

---

## 1. Hibernate Entity States

Hibernate manages entities through well-defined lifecycle states:

### 1.1 Transient
- Plain Java object
- Not associated with a session
- No database row

```java
Employee e = new Employee(); // transient
```

### 1.2 Persistent (Managed)
- Associated with an active session / EntityManager
- Changes are tracked automatically (dirty checking)

```java
Employee e = session.get(Employee.class, 1L);
e.setSalary(5000); // auto-update on commit
```

### 1.3 Detached
- Previously persistent
- Session closed
- Changes are NOT tracked

```java
session.close();
e.setSalary(6000); // not saved
```

### 1.4 Removed
- Marked for deletion
- Deleted at flush/commit

```java
session.delete(e);
```

### State Transition Diagram

```
Transient → Persistent → Detached → Persistent → Removed
```

---

## 2. save(), persist(), update(), merge()

| Method | Purpose | Notes |
|------|--------|------|
| save | Insert | Hibernate-specific |
| persist | Insert | JPA standard |
| update | Reattach | Risky for detached entities |
| merge | Copy state | Safe, recommended |

```java
Employee managed = entityManager.merge(detachedEmployee);
```

---

## 3. Dirty Checking

Hibernate automatically detects changes to managed entities:

```java
Employee e = em.find(Employee.class, 1L);
e.setSalary(8000); // no save()
```

- Snapshot taken when entity becomes managed
- Compared at flush/commit
- UPDATE generated if changed

⚠️ Works **only** for persistent entities inside a transaction.

---

## 4. Transactions (Critical)

Hibernate **must run inside transactions** for correctness.

```java
@Transactional
public void updateSalary() {
    Employee e = repo.findById(1L).get();
    e.setSalary(9000);
}
```

Without transactions:
- Lazy loading fails
- Dirty checking unreliable
- Flush timing unpredictable

---

## 5. Lazy Loading

```java
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
```

### LazyInitializationException
Occurs when accessing lazy data outside an active session.

```java
employee.getOrders().size(); // ❌ outside transaction
```

### Fixes
- Fetch joins
- DTO projections
- Proper transaction boundaries

```java
@Query("select e from Employee e join fetch e.orders where e.id = :id")
Employee findWithOrders(Long id);
```

---

## 6. N+1 Query Problem

```java
List<Employee> emps = repo.findAll();
for (Employee e : emps) {
    e.getOrders().size(); // ❌ N+1
}
```

### Solutions
- Fetch join
- EntityGraph
- Batch fetching

---

## 7. Caching

### First-Level Cache (L1)
- Session scoped
- Always enabled
- Guarantees identity

### Second-Level Cache (L2)
- Shared across sessions
- Optional
- Use for read-heavy reference data

```java
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@Entity
class Product {}
```

---

## 8. Flush vs Commit

| Flush | Commit |
|-----|-------|
| Sends SQL | Ends transaction |
| Not permanent | Permanent |

```java
em.flush();
em.getTransaction().commit();
```

---

## 9. Locking

### Optimistic Locking (Preferred)

```java
@Version
private Long version;
```

### Pessimistic Locking

```java
em.find(Employee.class, id, LockModeType.PESSIMISTIC_WRITE);
```

---

## 10. Open Session In View (OSIV)

- Keeps session open till response
- Hides lazy issues
- ❌ Avoid in microservices

---

## 11. Production Best Practices

- Short-lived persistence context
- Always use @Transactional
- Avoid FetchType.EAGER
- Prefer DTOs for APIs
- Monitor SQL and query counts

---

## 12. Senior Interview One-Liners

- "Persistent entities are tracked via dirty checking."
- "Detached entities are common in web apps; merge is safer."
- "Lazy loading failures indicate transaction issues."
- "Hibernate improves productivity, not raw performance."

---

## TL;DR
Hibernate is powerful but unforgiving. Correct transaction boundaries, fetch strategies, and lifecycle management matter more than annotations.


## Introduction to Hibernate

### What is Hibernate?

**Hibernate** is an open-source Object-Relational Mapping (ORM) framework for Java. It provides a framework for mapping an object-oriented domain model to a relational database.

### Why Hibernate?

**Problems with JDBC:**
- ❌ Boilerplate code (connection management, SQL, result set handling)
- ❌ Manual object-relational mapping
- ❌ No automatic relationship handling
- ❌ No caching
- ❌ SQL vendor-specific

**Benefits of Hibernate:**
- ✅ Reduces boilerplate code by 70-80%
- ✅ Automatic object-relational mapping
- ✅ Database independence (HQL/JPQL)
- ✅ Built-in caching (first-level, second-level)
- ✅ Lazy loading
- ✅ Automatic relationship handling
- ✅ Transaction management
- ✅ Connection pooling

### Hibernate vs JPA

| Aspect | Hibernate | JPA (Java Persistence API) |
|--------|-----------|---------------------------|
| **Type** | Implementation | Specification |
| **Relationship** | Hibernate implements JPA | JPA is the standard |
| **Features** | More features | Standard features |
| **Vendor Lock-in** | Yes (if using Hibernate-specific) | No (portable) |
| **Usage** | Can use Hibernate API or JPA API | Use JPA API |

**Best Practice:** Use JPA annotations and API, but leverage Hibernate-specific features when needed.

---

## Hibernate Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    HIBERNATE ARCHITECTURE                │
└─────────────────────────────────────────────────────────┘

Application Layer
    │
    ▼
SessionFactory (Singleton)
    │
    ▼
Session (Per Transaction)
    │
    ├─── Persistent Context (Identity Map)
    ├─── First-Level Cache
    ├─── Transaction Management
    └─── Connection Management
    │
    ▼
JDBC Layer
    │
    ▼
Database
```

### Core Components

#### 1. SessionFactory
- **Purpose**: Factory for creating Session instances
- **Lifecycle**: Created once, shared across application
- **Thread-safe**: Yes
- **Cost**: Expensive to create

```java
Configuration configuration = new Configuration();
configuration.configure("hibernate.cfg.xml");
SessionFactory sessionFactory = configuration.buildSessionFactory();
```

#### 2. Session
- **Purpose**: Main interface for persistence operations
- **Lifecycle**: One per transaction/request
- **Thread-safe**: No (not thread-safe)
- **Cost**: Lightweight

```java
Session session = sessionFactory.openSession();
try {
    // Perform operations
    Transaction transaction = session.beginTransaction();
    // ... operations
    transaction.commit();
} finally {
    session.close();
}
```

#### 3. Persistent Context (Identity Map)
- **Purpose**: Cache of persistent objects within a session
- **Key Concept**: Same object instance for same database row
- **Scope**: Per session

```java
// First call - loads from database
User user1 = session.get(User.class, 1L);

// Second call - returns from persistent context (no DB query)
User user2 = session.get(User.class, 1L);

// Same instance
assert user1 == user2; // true
```

#### 4. Transaction
- **Purpose**: Atomic unit of work
- **ACID Properties**: Ensured by Hibernate

```java
Transaction transaction = session.beginTransaction();
try {
    // Operations
    session.save(user);
    session.update(order);
    transaction.commit();
} catch (Exception e) {
    transaction.rollback();
    throw e;
}
```

---

## Configuration and Setup

### Maven Dependencies

```xml
<dependencies>
    <!-- Hibernate Core -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
        <version>5.6.15.Final</version>
    </dependency>
    
    <!-- Hibernate Entity Manager (JPA) -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.6.15.Final</version>
    </dependency>
    
    <!-- Database Driver -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- Connection Pooling -->
    <dependency>
        <groupId>com.zaxxer</groupId>
        <artifactId>HikariCP</artifactId>
        <version>5.0.1</version>
    </dependency>
</dependencies>
```

### Configuration File: hibernate.cfg.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- Database Connection Settings -->
        <property name="hibernate.connection.driver_class">com.mysql.cj.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/ecommerce</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">password</property>
        
        <!-- Connection Pool Settings -->
        <property name="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.timeout">300</property>
        <property name="hibernate.c3p0.max_statements">50</property>
        
        <!-- SQL Dialect -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL8Dialect</property>
        
        <!-- Show SQL -->
        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>
        
        <!-- Schema Management -->
        <property name="hibernate.hbm2ddl.auto">update</property>
        <!-- Options: validate, update, create, create-drop -->
        
        <!-- Second-Level Cache -->
        <property name="hibernate.cache.use_second_level_cache">true</property>
        <property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
        <property name="hibernate.cache.use_query_cache">true</property>
        
        <!-- Batch Processing -->
        <property name="hibernate.jdbc.batch_size">20</property>
        <property name="hibernate.order_inserts">true</property>
        <property name="hibernate.order_updates">true</property>
        
        <!-- Entity Classes -->
        <mapping class="com.example.entity.User"/>
        <mapping class="com.example.entity.Order"/>
        <mapping class="com.example.entity.Product"/>
    </session-factory>
</hibernate-configuration>
```

### Programmatic Configuration

```java
public class HibernateUtil {
    private static SessionFactory sessionFactory;
    
    static {
        try {
            Configuration configuration = new Configuration();
            
            // Database properties
            Properties properties = new Properties();
            properties.put("hibernate.connection.driver_class", "com.mysql.cj.jdbc.Driver");
            properties.put("hibernate.connection.url", "jdbc:mysql://localhost:3306/ecommerce");
            properties.put("hibernate.connection.username", "root");
            properties.put("hibernate.connection.password", "password");
            properties.put("hibernate.dialect", "org.hibernate.dialect.MySQL8Dialect");
            properties.put("hibernate.show_sql", "true");
            properties.put("hibernate.format_sql", "true");
            properties.put("hibernate.hbm2ddl.auto", "update");
            
            configuration.setProperties(properties);
            configuration.addAnnotatedClass(User.class);
            configuration.addAnnotatedClass(Order.class);
            
            sessionFactory = configuration.buildSessionFactory();
        } catch (Exception e) {
            throw new ExceptionInInitializerError(e);
        }
    }
    
    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
    
    public static void shutdown() {
        if (sessionFactory != null) {
            sessionFactory.close();
        }
    }
}
```

### Spring Boot Configuration (application.properties)

```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/ecommerce
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Hibernate Configuration
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect

# Connection Pool (HikariCP)
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.maximum-pool-size=20
spring.datasource.hikari.idle-timeout=300000
spring.datasource.hikari.max-lifetime=600000

# Batch Processing
spring.jpa.properties.hibernate.jdbc.batch_size=20
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Second-Level Cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

---

## Entity Mapping

### Basic Entity

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "username", nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(name = "email", nullable = false, unique = true)
    private String email;
    
    @Column(name = "password", nullable = false)
    private String password;
    
    @Column(name = "created_at")
    @Temporal(TemporalType.TIMESTAMP)
    private Date createdAt;
    
    @Column(name = "updated_at")
    @Temporal(TemporalType.TIMESTAMP)
    private Date updatedAt;
    
    // Constructors
    public User() {}
    
    public User(String username, String email, String password) {
        this.username = username;
        this.email = email;
        this.password = password;
    }
    
    // Lifecycle Callbacks
    @PrePersist
    protected void onCreate() {
        createdAt = new Date();
        updatedAt = new Date();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = new Date();
    }
    
    // Getters and Setters
    // ...
}
```

### Primary Key Generation Strategies

#### 1. IDENTITY (Auto-increment)

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

**Characteristics:**
- Database generates ID
- Insert happens immediately (no batching)
- Supported by: MySQL, PostgreSQL, SQL Server

#### 2. SEQUENCE

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
private Long id;
```

**Characteristics:**
- Uses database sequence
- Better for batch inserts
- Supported by: Oracle, PostgreSQL

#### 3. TABLE

```java
@Id
@GeneratedValue(strategy = GenerationType.TABLE, generator = "user_gen")
@TableGenerator(name = "user_gen", table = "id_generator", 
    pkColumnName = "gen_name", valueColumnName = "gen_value",
    pkColumnValue = "user_id", allocationSize = 1)
private Long id;
```

**Characteristics:**
- Uses separate table for ID generation
- Portable across databases
- Slower than SEQUENCE

#### 4. AUTO

```java
@Id
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

**Characteristics:**
- Hibernate chooses strategy
- Default: IDENTITY or SEQUENCE based on database

### Column Mapping

```java
@Entity
@Table(name = "products")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    // Basic column
    @Column(name = "product_name")
    private String name;
    
    // With constraints
    @Column(name = "sku", nullable = false, unique = true, length = 50)
    private String sku;
    
    // Precision and scale for decimal
    @Column(name = "price", precision = 10, scale = 2)
    private BigDecimal price;
    
    // Large text
    @Column(name = "description", columnDefinition = "TEXT")
    private String description;
    
    // Enum
    @Enumerated(EnumType.STRING)
    @Column(name = "status")
    private ProductStatus status;
    
    // Boolean
    @Column(name = "is_active")
    private Boolean isActive;
    
    // Temporal types
    @Temporal(TemporalType.DATE)
    @Column(name = "release_date")
    private Date releaseDate;
    
    @Temporal(TemporalType.TIMESTAMP)
    @Column(name = "created_at")
    private Date createdAt;
    
    // Transient (not persisted)
    @Transient
    private String displayName;
    
    // Lob (Large Object)
    @Lob
    @Column(name = "image")
    private byte[] image;
}
```

### Embedded Objects

```java
@Embeddable
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    
    // Getters and setters
}

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Embedded
    private Address address;
    
    // Or with column prefix
    @Embedded
    @AttributeOverrides({
        @AttributeOverride(name = "street", column = @Column(name = "home_street")),
        @AttributeOverride(name = "city", column = @Column(name = "home_city"))
    })
    private Address homeAddress;
}
```

### Inheritance Strategies

#### 1. Single Table (Default)

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "vehicle_type", discriminatorType = DiscriminatorType.STRING)
@Table(name = "vehicles")
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String brand;
    private String model;
}

@Entity
@DiscriminatorValue("CAR")
public class Car extends Vehicle {
    private Integer numberOfDoors;
}

@Entity
@DiscriminatorValue("BIKE")
public class Bike extends Vehicle {
    private Integer numberOfWheels;
}
```

**Table Structure:**
```
vehicles
├── id
├── vehicle_type (discriminator)
├── brand
├── model
├── number_of_doors (null for Bike)
└── number_of_wheels (null for Car)
```

#### 2. Table Per Class

```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;
    
    private String brand;
    private String model;
}

@Entity
@Table(name = "cars")
public class Car extends Vehicle {
    private Integer numberOfDoors;
}

@Entity
@Table(name = "bikes")
public class Bike extends Vehicle {
    private Integer numberOfWheels;
}
```

**Table Structure:**
```
vehicles (abstract - no table)
cars
├── id
├── brand
├── model
└── number_of_doors

bikes
├── id
├── brand
├── model
└── number_of_wheels
```

#### 3. Joined Table

```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
@Table(name = "vehicles")
public abstract class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String brand;
    private String model;
}

@Entity
@Table(name = "cars")
@PrimaryKeyJoinColumn(name = "vehicle_id")
public class Car extends Vehicle {
    private Integer numberOfDoors;
}

@Entity
@Table(name = "bikes")
@PrimaryKeyJoinColumn(name = "vehicle_id")
public class Bike extends Vehicle {
    private Integer numberOfWheels;
}
```

**Table Structure:**
```
vehicles
├── id (PK)
├── brand
└── model

cars
├── vehicle_id (PK, FK to vehicles.id)
└── number_of_doors

bikes
├── vehicle_id (PK, FK to vehicles.id)
└── number_of_wheels
```

---

## Relationships

### One-to-One

#### Unidirectional

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}

@Entity
@Table(name = "user_profiles")
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String bio;
    private String avatar;
}
```

#### Bidirectional

```java
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne(mappedBy = "user", cascade = CascadeType.ALL)
    private UserProfile profile;
}

@Entity
@Table(name = "user_profiles")
public class UserProfile {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @OneToOne
    @JoinColumn(name = "user_id")
    private User user;
    
    private String bio;
}
```

### One-to-Many / Many-to-One

```java
@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id")
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
    
    private BigDecimal totalAmount;
}

@Entity
@Table(name = "order_items")
public class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id")
    private Product product;
    
    private Integer quantity;
    private BigDecimal price;
}
```

### Many-to-Many

```java
@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @ManyToMany(mappedBy = "courses")
    private Set<Student> students = new HashSet<>();
}
```

### Fetch Types

#### EAGER Loading

```java
@ManyToOne(fetch = FetchType.EAGER)
@JoinColumn(name = "user_id")
private User user;
```

**Characteristics:**
- Loads immediately
- Can cause N+1 problem
- Use sparingly

#### LAZY Loading (Default for Collections)

```java
@OneToMany(fetch = FetchType.LAZY)
@JoinColumn(name = "order_id")
private List<OrderItem> items;
```

**Characteristics:**
- Loads on demand
- Better performance
- Can cause LazyInitializationException if accessed outside session

### Cascade Types

```java
@OneToMany(cascade = CascadeType.ALL)
private List<OrderItem> items;

// Available cascade types:
// - CascadeType.PERSIST: Save parent saves children
// - CascadeType.MERGE: Update parent updates children
// - CascadeType.REMOVE: Delete parent deletes children
// - CascadeType.REFRESH: Refresh parent refreshes children
// - CascadeType.DETACH: Detach parent detaches children
// - CascadeType.ALL: All operations cascade
```

### Orphan Removal

```java
@OneToMany(cascade = CascadeType.ALL, orphanRemoval = true)
private List<OrderItem> items;

// When item is removed from collection, it's deleted from database
order.getItems().remove(item); // Deletes item from database
```

---

## HQL and JPQL

### HQL (Hibernate Query Language)

HQL is object-oriented SQL that works with entities instead of tables.

#### Basic Queries

```java
// Select all
String hql = "FROM User";
Query query = session.createQuery(hql);
List<User> users = query.list();

// Select with WHERE
String hql = "FROM User WHERE email = :email";
Query query = session.createQuery(hql);
query.setParameter("email", "user@example.com");
User user = (User) query.uniqueResult();

// Select specific fields
String hql = "SELECT u.username, u.email FROM User u WHERE u.id = :id";
Query query = session.createQuery(hql);
query.setParameter("id", 1L);
Object[] result = (Object[]) query.uniqueResult();
```

#### Named Queries

```java
@Entity
@NamedQueries({
    @NamedQuery(
        name = "User.findByEmail",
        query = "SELECT u FROM User u WHERE u.email = :email"
    ),
    @NamedQuery(
        name = "User.findActiveUsers",
        query = "SELECT u FROM User u WHERE u.isActive = true"
    )
})
public class User {
    // ...
}

// Usage
Query query = session.getNamedQuery("User.findByEmail");
query.setParameter("email", "user@example.com");
User user = (User) query.uniqueResult();
```

#### Joins

```java
// Inner Join
String hql = "SELECT o FROM Order o JOIN o.user u WHERE u.email = :email";
Query query = session.createQuery(hql);
query.setParameter("email", "user@example.com");
List<Order> orders = query.list();

// Left Join
String hql = "SELECT u FROM User u LEFT JOIN u.orders o WHERE o.totalAmount > :amount";
Query query = session.createQuery(hql);
query.setParameter("amount", new BigDecimal("1000"));
List<User> users = query.list();

// Fetch Join (avoids N+1 problem)
String hql = "SELECT o FROM Order o JOIN FETCH o.items WHERE o.id = :id";
Query query = session.createQuery(hql);
query.setParameter("id", 1L);
Order order = (Order) query.uniqueResult();
// Items are loaded in same query
```

#### Aggregations

```java
// Count
String hql = "SELECT COUNT(u) FROM User u";
Query query = session.createQuery(hql);
Long count = (Long) query.uniqueResult();

// Group By
String hql = "SELECT u.status, COUNT(u) FROM User u GROUP BY u.status";
Query query = session.createQuery(hql);
List<Object[]> results = query.list();

// Having
String hql = "SELECT u.status, COUNT(u) FROM User u GROUP BY u.status HAVING COUNT(u) > 10";
Query query = session.createQuery(hql);
List<Object[]> results = query.list();
```

#### Subqueries

```java
// EXISTS
String hql = "SELECT u FROM User u WHERE EXISTS (SELECT o FROM Order o WHERE o.user = u)";
Query query = session.createQuery(hql);
List<User> users = query.list();

// IN
String hql = "SELECT o FROM Order o WHERE o.user.id IN (SELECT u.id FROM User u WHERE u.isActive = true)";
Query query = session.createQuery(hql);
List<Order> orders = query.list();
```

#### Pagination

```java
String hql = "FROM User ORDER BY createdAt DESC";
Query query = session.createQuery(hql);
query.setFirstResult(0);  // Offset
query.setMaxResults(10); // Limit
List<User> users = query.list();
```

### JPQL (Java Persistence Query Language)

JPQL is similar to HQL but is part of JPA specification.

```java
// Using EntityManager
EntityManager em = entityManagerFactory.createEntityManager();
String jpql = "SELECT u FROM User u WHERE u.email = :email";
TypedQuery<User> query = em.createQuery(jpql, User.class);
query.setParameter("email", "user@example.com");
User user = query.getSingleResult();
```

### Native SQL Queries

```java
// Native SQL
String sql = "SELECT * FROM users WHERE email = :email";
Query query = session.createSQLQuery(sql)
    .addEntity(User.class)
    .setParameter("email", "user@example.com");
User user = (User) query.uniqueResult();

// Scalar result
String sql = "SELECT COUNT(*) FROM users";
Query query = session.createSQLQuery(sql);
Long count = ((Number) query.uniqueResult()).longValue();
```

---

## Criteria API

### Type-Safe Queries

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> query = cb.createQuery(User.class);
Root<User> root = query.from(User.class);

// WHERE clause
Predicate emailPredicate = cb.equal(root.get("email"), "user@example.com");
query.where(emailPredicate);

// Execute
TypedQuery<User> typedQuery = session.createQuery(query);
List<User> users = typedQuery.getResultList();
```

### Complex Queries

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Order> query = cb.createQuery(Order.class);
Root<Order> root = query.from(Order.class);

// Join
Join<Order, User> userJoin = root.join("user");

// Multiple conditions
Predicate emailPredicate = cb.equal(userJoin.get("email"), "user@example.com");
Predicate amountPredicate = cb.greaterThan(root.get("totalAmount"), new BigDecimal("1000"));
Predicate combined = cb.and(emailPredicate, amountPredicate);

query.where(combined);

// Order By
query.orderBy(cb.desc(root.get("createdAt")));

// Execute
List<Order> orders = session.createQuery(query).getResultList();
```

### Aggregations

```java
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<Object[]> query = cb.createQuery(Object[].class);
Root<Order> root = query.from(Order.class);

// Select
query.multiselect(
    root.get("user").get("id"),
    cb.sum(root.get("totalAmount")),
    cb.count(root)
);

// Group By
query.groupBy(root.get("user").get("id"));

// Having
query.having(cb.greaterThan(cb.sum(root.get("totalAmount")), new BigDecimal("10000")));

List<Object[]> results = session.createQuery(query).getResultList();
```

---

## Caching

### First-Level Cache (Session Cache)

**Scope:** Per session (transaction)

```java
Session session = sessionFactory.openSession();

// First call - hits database
User user1 = session.get(User.class, 1L);

// Second call - returns from first-level cache (no DB query)
User user2 = session.get(User.class, 1L);

// Same instance
assert user1 == user2; // true

session.close();
```

**Characteristics:**
- Automatic
- Per session
- Cannot be disabled
- Cleared when session closes

### Second-Level Cache

**Scope:** Application-wide (shared across sessions)

#### Configuration

```xml
<!-- Enable second-level cache -->
<property name="hibernate.cache.use_second_level_cache">true</property>
<property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
```

#### Entity-Level Cache

```java
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@Table(name = "products")
public class Product {
    // ...
}
```

**Cache Concurrency Strategies:**

1. **READ_ONLY**: Read-only entities
```java
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
```

2. **READ_WRITE**: Read-write entities
```java
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
```

3. **NONSTRICT_READ_WRITE**: Non-strict read-write
```java
@Cache(usage = CacheConcurrencyStrategy.NONSTRICT_READ_WRITE)
```

4. **TRANSACTIONAL**: Full transactional support
```java
@Cache(usage = CacheConcurrencyStrategy.TRANSACTIONAL)
```

#### Query Cache

```java
// Enable query cache
query.setCacheable(true);
query.setCacheRegion("productQueries");

// Usage
String hql = "FROM Product WHERE category = :category";
Query query = session.createQuery(hql);
query.setParameter("category", "Electronics");
query.setCacheable(true);
List<Product> products = query.list();
```

### Cache Regions

```java
// Evict specific region
sessionFactory.getCache().evictEntityRegion(Product.class);

// Evict all regions
sessionFactory.getCache().evictAllRegions();

// Evict specific entity
sessionFactory.getCache().evictEntity(Product.class, productId);
```

---

## Performance Optimization

### N+1 Problem

**Problem:**

```java
// ❌ BAD: N+1 queries
List<Order> orders = session.createQuery("FROM Order").list();
for (Order order : orders) {
    // Each access triggers a query
    List<OrderItem> items = order.getItems(); // Query for each order
}
// Total: 1 + N queries
```

**Solutions:**

#### 1. Fetch Join

```java
// ✅ GOOD: Single query with JOIN
String hql = "SELECT DISTINCT o FROM Order o JOIN FETCH o.items";
List<Order> orders = session.createQuery(hql).list();
// Items loaded in same query
```

#### 2. Batch Fetching

```java
@Entity
@BatchSize(size = 10)
@Table(name = "orders")
public class Order {
    @OneToMany(fetch = FetchType.LAZY)
    @BatchSize(size = 10)
    private List<OrderItem> items;
}
```

#### 3. Subselect Fetching

```java
@OneToMany(fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<OrderItem> items;
```

### Batch Processing

```java
// Enable batch processing
session.setJdbcBatchSize(20);

// Batch inserts
for (int i = 0; i < 1000; i++) {
    User user = new User("user" + i, "user" + i + "@example.com", "password");
    session.save(user);
    
    if (i % 20 == 0) {
        session.flush();
        session.clear();
    }
}
```

### Lazy Loading Best Practices

```java
// ✅ GOOD: Access lazy associations within session
Session session = sessionFactory.openSession();
try {
    Order order = session.get(Order.class, 1L);
    // Access within session
    List<OrderItem> items = order.getItems(); // OK
} finally {
    session.close();
}

// ❌ BAD: Access lazy associations outside session
Session session = sessionFactory.openSession();
Order order = session.get(Order.class, 1L);
session.close();
List<OrderItem> items = order.getItems(); // LazyInitializationException!
```

### Projections

```java
// ✅ GOOD: Select only needed fields
String hql = "SELECT u.id, u.username, u.email FROM User u";
Query query = session.createQuery(hql);
List<Object[]> results = query.list();

// ❌ BAD: Select entire entity when only need few fields
String hql = "FROM User";
Query query = session.createQuery(hql);
List<User> users = query.list(); // Loads all fields
```

### Pagination

```java
// ✅ GOOD: Use pagination
String hql = "FROM User ORDER BY createdAt DESC";
Query query = session.createQuery(hql);
query.setFirstResult(0);
query.setMaxResults(20);
List<User> users = query.list();

// ❌ BAD: Load all records
String hql = "FROM User";
Query query = session.createQuery(hql);
List<User> users = query.list(); // Loads all users
```

### StatelessSession

```java
// For bulk operations
StatelessSession statelessSession = sessionFactory.openStatelessSession();
try {
    for (int i = 0; i < 10000; i++) {
        User user = new User("user" + i, "user" + i + "@example.com", "password");
        statelessSession.insert(user);
    }
} finally {
    statelessSession.close();
}
```

**Characteristics:**
- No first-level cache
- No dirty checking
- No lazy loading
- Better for bulk operations

---

## Advanced Topics

### Interceptors

```java
public class AuditInterceptor extends EmptyInterceptor {
    @Override
    public boolean onSave(Object entity, Serializable id, Object[] state, 
                          String[] propertyNames, Type[] types) {
        if (entity instanceof Auditable) {
            for (int i = 0; i < propertyNames.length; i++) {
                if ("createdAt".equals(propertyNames[i])) {
                    state[i] = new Date();
                    return true;
                }
            }
        }
        return false;
    }
    
    @Override
    public boolean onFlushDirty(Object entity, Serializable id, Object[] currentState,
                                Object[] previousState, String[] propertyNames, Type[] types) {
        if (entity instanceof Auditable) {
            for (int i = 0; i < propertyNames.length; i++) {
                if ("updatedAt".equals(propertyNames[i])) {
                    currentState[i] = new Date();
                    return true;
                }
            }
        }
        return false;
    }
}

// Usage
Session session = sessionFactory.withOptions()
    .interceptor(new AuditInterceptor())
    .openSession();
```

### Event Listeners

```java
public class AuditEventListener implements PreInsertEventListener, PreUpdateEventListener {
    @Override
    public boolean onPreInsert(PreInsertEvent event) {
        Object entity = event.getEntity();
        if (entity instanceof Auditable) {
            event.getState()[getPropertyIndex(event, "createdAt")] = new Date();
        }
        return false;
    }
    
    @Override
    public boolean onPreUpdate(PreUpdateEvent event) {
        Object entity = event.getEntity();
        if (entity instanceof Auditable) {
            event.getState()[getPropertyIndex(event, "updatedAt")] = new Date();
        }
        return false;
    }
    
    private int getPropertyIndex(AbstractPreDatabaseOperationEvent event, String propertyName) {
        String[] propertyNames = event.getPersister().getPropertyNames();
        for (int i = 0; i < propertyNames.length; i++) {
            if (propertyName.equals(propertyNames[i])) {
                return i;
            }
        }
        return -1;
    }
}
```

### Custom Types

```java
public class MoneyType implements UserType {
    @Override
    public int[] sqlTypes() {
        return new int[]{Types.DECIMAL};
    }
    
    @Override
    public Class returnedClass() {
        return Money.class;
    }
    
    @Override
    public boolean equals(Object x, Object y) throws HibernateException {
        return Objects.equals(x, y);
    }
    
    @Override
    public int hashCode(Object x) throws HibernateException {
        return x.hashCode();
    }
    
    @Override
    public Object nullSafeGet(ResultSet rs, String[] names, SessionImplementor session, Object owner)
            throws HibernateException, SQLException {
        BigDecimal amount = rs.getBigDecimal(names[0]);
        return amount != null ? new Money(amount) : null;
    }
    
    @Override
    public void nullSafeSet(PreparedStatement st, Object value, int index, SessionImplementor session)
            throws HibernateException, SQLException {
        if (value == null) {
            st.setNull(index, Types.DECIMAL);
        } else {
            Money money = (Money) value;
            st.setBigDecimal(index, money.getAmount());
        }
    }
    
    // ... other methods
}

// Usage
@Entity
public class Product {
    @Type(type = "com.example.MoneyType")
    @Column(name = "price")
    private Money price;
}
```

### Filters

```java
// Define filter
@FilterDef(name = "activeProducts", parameters = @ParamDef(name = "active", type = "boolean"))
@Filter(name = "activeProducts", condition = "is_active = :active")
@Entity
@Table(name = "products")
public class Product {
    // ...
}

// Enable filter
session.enableFilter("activeProducts").setParameter("active", true);
List<Product> products = session.createQuery("FROM Product").list();
```

### Multi-Tenancy

```java
// Configure multi-tenancy
Properties properties = new Properties();
properties.put("hibernate.multiTenancy", "SCHEMA");
properties.put("hibernate.multi_tenant_connection_provider", 
    "com.example.MultiTenantConnectionProvider");
properties.put("hibernate.tenant_identifier_resolver", 
    "com.example.TenantIdentifierResolver");

// Tenant resolver
public class TenantIdentifierResolver implements CurrentTenantIdentifierResolver {
    @Override
    public String resolveCurrentTenantIdentifier() {
        return TenantContext.getCurrentTenant();
    }
    
    @Override
    public boolean validateExistingCurrentSessions() {
        return true;
    }
}
```

### Envers (Auditing)

```java
// Add dependency
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-envers</artifactId>
    <version>5.6.15.Final</version>
</dependency>

// Enable auditing
@Entity
@Audited
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private BigDecimal price;
}

// Query audit history
AuditReader reader = AuditReaderFactory.get(session);
List<Number> revisions = reader.getRevisions(Product.class, productId);
Product product = reader.find(Product.class, productId, revisionNumber);
```

---

## Best Practices

### 1. Always Close Sessions

```java
// ✅ GOOD
Session session = sessionFactory.openSession();
try {
    // Operations
} finally {
    session.close();
}

// Or use try-with-resources (Java 7+)
try (Session session = sessionFactory.openSession()) {
    // Operations
}
```

### 2. Use Transactions

```java
// ✅ GOOD
Session session = sessionFactory.openSession();
Transaction transaction = session.beginTransaction();
try {
    // Operations
    transaction.commit();
} catch (Exception e) {
    transaction.rollback();
    throw e;
} finally {
    session.close();
}
```

### 3. Avoid N+1 Problem

```java
// ✅ GOOD: Use fetch join
String hql = "SELECT DISTINCT o FROM Order o JOIN FETCH o.items";
List<Order> orders = session.createQuery(hql).list();

// ❌ BAD: Causes N+1
List<Order> orders = session.createQuery("FROM Order").list();
for (Order order : orders) {
    order.getItems(); // N queries
}
```

### 4. Use Lazy Loading Wisely

```java
// ✅ GOOD: LAZY for collections
@OneToMany(fetch = FetchType.LAZY)
private List<OrderItem> items;

// ✅ GOOD: EAGER for single associations (if needed)
@ManyToOne(fetch = FetchType.EAGER)
private User user;
```

### 5. Use Projections

```java
// ✅ GOOD: Select only needed fields
String hql = "SELECT u.id, u.username FROM User u";

// ❌ BAD: Select entire entity
String hql = "FROM User";
```

### 6. Use Batch Processing

```java
// ✅ GOOD: Batch inserts
session.setJdbcBatchSize(20);
for (int i = 0; i < 1000; i++) {
    session.save(new User(...));
    if (i % 20 == 0) {
        session.flush();
        session.clear();
    }
}
```

### 7. Use Second-Level Cache

```java
// ✅ GOOD: Cache read-only entities
@Entity
@Cacheable
@Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Product {
    // ...
}
```

### 8. Use Named Queries

```java
// ✅ GOOD: Named queries are pre-compiled
@NamedQuery(name = "User.findByEmail", 
    query = "SELECT u FROM User u WHERE u.email = :email")
```

---

## Common Pitfalls

### 1. LazyInitializationException

```java
// ❌ BAD
Session session = sessionFactory.openSession();
Order order = session.get(Order.class, 1L);
session.close();
List<OrderItem> items = order.getItems(); // LazyInitializationException!

// ✅ GOOD
Session session = sessionFactory.openSession();
try {
    Order order = session.get(Order.class, 1L);
    List<OrderItem> items = order.getItems(); // Access within session
} finally {
    session.close();
}
```

### 2. Session Management

```java
// ❌ BAD: Session per request
public User getUser(Long id) {
    Session session = sessionFactory.openSession();
    User user = session.get(User.class, id);
    session.close();
    return user; // Detached entity
}

// ✅ GOOD: Session per transaction
@Transactional
public User getUser(Long id) {
    return session.get(User.class, id);
}
```

### 3. N+1 Problem

```java
// ❌ BAD
List<Order> orders = session.createQuery("FROM Order").list();
for (Order order : orders) {
    order.getItems(); // N queries
}

// ✅ GOOD
String hql = "SELECT DISTINCT o FROM Order o JOIN FETCH o.items";
List<Order> orders = session.createQuery(hql).list();
```

### 4. Memory Leaks

```java
// ❌ BAD: Not clearing session
for (int i = 0; i < 10000; i++) {
    User user = new User(...);
    session.save(user); // Accumulates in session
}

// ✅ GOOD: Clear session periodically
for (int i = 0; i < 10000; i++) {
    User user = new User(...);
    session.save(user);
    if (i % 100 == 0) {
        session.flush();
        session.clear();
    }
}
```

---

## Interview Questions

### Basic Questions

#### Q1: What is Hibernate and why use it?

**Answer:**
"Hibernate is an ORM framework that maps Java objects to database tables. It eliminates boilerplate JDBC code, provides database independence through HQL/JPQL, offers built-in caching, lazy loading, and automatic relationship handling. It reduces development time by 70-80% and provides better maintainability."

#### Q2: Explain Hibernate architecture.

**Answer:**
"Hibernate has a layered architecture:
1. **SessionFactory**: Singleton factory for creating sessions, expensive to create, thread-safe
2. **Session**: Per-transaction interface, lightweight, not thread-safe, contains persistent context
3. **Persistent Context**: Identity map caching objects within a session
4. **Transaction**: Ensures ACID properties
5. **Connection Pool**: Manages database connections efficiently

The flow: Application → SessionFactory → Session → JDBC → Database"

#### Q3: What is the difference between get() and load()?

**Answer:**
"`get()` immediately hits the database and returns null if not found. `load()` returns a proxy and throws ObjectNotFoundException if accessed when not found. Use `get()` when you need to check existence, `load()` when you're sure the entity exists."

### Intermediate Questions

#### Q4: Explain Hibernate caching levels.

**Answer:**
"Hibernate has two cache levels:
1. **First-Level Cache (Session Cache)**: Per-session, automatic, cannot be disabled, cleared when session closes
2. **Second-Level Cache**: Application-wide, shared across sessions, optional, requires configuration

First-level cache ensures same object instance for same database row within a session. Second-level cache improves performance by caching frequently accessed entities across sessions."

#### Q5: What is the N+1 problem and how do you solve it?

**Answer:**
"The N+1 problem occurs when loading a collection triggers N additional queries. For example, loading 100 orders and accessing items for each triggers 100 additional queries.

Solutions:
1. **Fetch Join**: `SELECT o FROM Order o JOIN FETCH o.items`
2. **Batch Fetching**: `@BatchSize(size = 10)`
3. **Subselect Fetching**: `@Fetch(FetchMode.SUBSELECT)`

I prefer fetch joins for specific queries and batch fetching for general use."

#### Q6: Explain Hibernate entity states.

**Answer:**
"Hibernate entities have four states:
1. **Transient**: New object, not associated with session, not in database
2. **Persistent**: Associated with session, tracked by Hibernate, synchronized with database
3. **Detached**: Previously persistent, session closed, changes not tracked
4. **Removed**: Marked for deletion, will be deleted on flush

Transitions: Transient → Persistent (save/persist), Persistent → Detached (evict/close), Persistent → Removed (delete)"

### Advanced Questions

#### Q7: How does Hibernate handle lazy loading?

**Answer:**
"Hibernate uses proxies for lazy loading. When you access a lazy association, Hibernate:
1. Checks if session is open
2. If open, executes query to load association
3. If closed, throws LazyInitializationException

To avoid LazyInitializationException:
- Access lazy associations within session
- Use fetch joins for specific queries
- Use `@Transactional` at service layer
- Use DTOs/projections instead of entities"

#### Q8: Explain Hibernate transaction management.

**Answer:**
"Hibernate provides transaction management through `Transaction` interface. It ensures ACID properties:
- **Atomicity**: All or nothing
- **Consistency**: Database remains consistent
- **Isolation**: Configurable isolation levels
- **Durability**: Committed changes persist

Best practices:
- Always use transactions for write operations
- Use `@Transactional` at service layer
- Handle exceptions and rollback appropriately
- Use appropriate isolation levels"

#### Q9: How do you optimize Hibernate performance?

**Answer:**
"Performance optimization strategies:
1. **Avoid N+1**: Use fetch joins, batch fetching
2. **Use Projections**: Select only needed fields
3. **Batch Processing**: Enable batch inserts/updates
4. **Second-Level Cache**: Cache read-only entities
5. **Lazy Loading**: Use LAZY for collections
6. **Pagination**: Don't load all records
7. **StatelessSession**: For bulk operations
8. **Connection Pooling**: Proper pool configuration
9. **Query Optimization**: Use indexes, analyze queries
10. **Monitor**: Use statistics, log slow queries"

#### Q10: Explain Hibernate inheritance strategies.

**Answer:**
"Hibernate supports three inheritance strategies:
1. **SINGLE_TABLE**: One table with discriminator column, fastest queries, but null columns
2. **TABLE_PER_CLASS**: Separate table per class, no joins, but unions for queries
3. **JOINED**: Base table + subclass tables, normalized, but joins required

Choose based on:
- Query performance needs
- Data normalization requirements
- Null column tolerance
- Polymorphic query frequency"

#### Q11: How does Hibernate handle concurrency?

**Answer:**
"Hibernate handles concurrency through:
1. **Optimistic Locking**: Version/timestamp columns, detects conflicts
2. **Pessimistic Locking**: Database-level locks, prevents conflicts
3. **Isolation Levels**: Configurable transaction isolation
4. **Cache Concurrency**: Second-level cache strategies (READ_ONLY, READ_WRITE, etc.)

For high concurrency, use optimistic locking with version columns. For critical data, use pessimistic locking."

#### Q12: Explain Hibernate query execution plan.

**Answer:**
"When executing a query:
1. **Parse**: HQL/JPQL parsed to AST
2. **Validate**: Entity and property validation
3. **Compile**: Converted to SQL
4. **Execute**: SQL executed via JDBC
5. **Result Mapping**: Results mapped to entities
6. **Cache**: Results cached if query cache enabled

Named queries are pre-compiled, improving performance. Use named queries for frequently executed queries."

#### Q13: How do you handle Hibernate in a multi-threaded environment?

**Answer:**
"Key points:
1. **Session is NOT thread-safe**: One session per thread
2. **SessionFactory IS thread-safe**: Shared across threads
3. **Thread-local sessions**: Use ThreadLocal or request-scoped sessions
4. **Connection Pooling**: Proper pool size for concurrency
5. **Transaction Management**: Use `@Transactional` with proper propagation
6. **StatelessSession**: For bulk operations (no cache, thread-safe)

Best practice: Use Spring's `@Transactional` which manages sessions per thread automatically."

#### Q14: Explain Hibernate second-level cache strategies.

**Answer:**
"Cache concurrency strategies:
1. **READ_ONLY**: Read-only entities, no updates, fastest
2. **READ_WRITE**: Read-write, uses soft locks, good for read-heavy
3. **NONSTRICT_READ_WRITE**: No locking, eventual consistency, fastest writes
4. **TRANSACTIONAL**: Full ACID, requires transactional cache provider

Choose based on:
- Read/write ratio
- Consistency requirements
- Performance needs
- Cache provider capabilities"

#### Q15: How do you migrate from Hibernate 4 to Hibernate 5/6?

**Answer:**
"Migration considerations:
1. **API Changes**: Some deprecated methods removed
2. **Configuration**: New configuration properties
3. **Java 8+**: Hibernate 5+ requires Java 8+
4. **JPA 2.1/2.2**: Hibernate 5 supports newer JPA versions
5. **Cache Providers**: Some cache providers updated
6. **Dialects**: Updated dialect classes
7. **Testing**: Comprehensive testing required

Steps:
1. Update dependencies
2. Update configuration
3. Fix deprecated API usage
4. Update cache providers
5. Test thoroughly
6. Monitor performance"

---

## Summary

### Key Takeaways

1. **Hibernate Architecture**: SessionFactory → Session → Persistent Context → Database
2. **Entity States**: Transient → Persistent → Detached → Removed
3. **Caching**: First-level (session) and second-level (application-wide)
4. **Relationships**: One-to-One, One-to-Many, Many-to-Many with proper fetch strategies
5. **Query Languages**: HQL (Hibernate) and JPQL (JPA standard)
6. **Performance**: Avoid N+1, use batch processing, caching, projections
7. **Best Practices**: Proper session management, transactions, lazy loading

### When to Use Hibernate

**✅ Use Hibernate when:**
- Complex object-relational mapping
- Need database independence
- Want to reduce boilerplate code
- Need caching and lazy loading
- Working with Spring/JPA ecosystem

**❌ Don't use Hibernate when:**
- Simple CRUD operations
- Need fine-grained SQL control
- Performance is critical (consider JDBC)
- Working with NoSQL databases

---

**Happy Coding with Hibernate! 🎯**

